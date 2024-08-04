---
title: C++开发

date: 2022-07-30

tags: C++

categories: 开发经验
---

# C++开发

## public框架:

#### 心跳程序-框架中的心跳类

##### _public.h

```
// 信号量。
class CSEM
{
private:
  union semun  // 用于信号量操作的共同体。
  {
    int val;
    struct semid_ds *buf;
    unsigned short  *arry;
  };

  int   m_semid;         // 信号量描述符。

  // 如果把sem_flg设置为SEM_UNDO，操作系统将跟踪进程对信号量的修改情况，
  // 在全部修改过信号量的进程（正常或异常）终止后，操作系统将把信号量恢
  // 复为初始值（就像撤消了全部进程对信号的操作）。
  // 如果信号量用于表示可用资源的数量（不变的），设置为SEM_UNDO更合适。
  // 如果信号量用于生产消费者模型，设置为0更合适。
  // 注意，网上查到的关于sem_flg的用法基本上是错的，一定要自己动手多测试。
  short m_sem_flg;
public:
  CSEM();
  // 如果信号量已存在，获取信号量；如果信号量不存在，则创建它并初始化为value。
  bool init(key_t key,unsigned short value=1,short sem_flg=SEM_UNDO); 
  bool P(short sem_op=-1); // 信号量的P操作。
  bool V(short sem_op=1);  // 信号量的V操作。
  int  value();            // 获取信号量的值，成功返回信号量的值，失败返回-1。
  bool destroy();          // 销毁信号量。
 ~CSEM();
};

// 进程心跳信息的结构体。
struct st_procinfo
{
  int    pid;         // 进程id。
  char   pname[51];   // 进程名称，可以为空。
  int    timeout;     // 超时时间，单位：秒。
  time_t atime;       // 最后一次心跳的时间，用整数表示。
};

#define MAXNUMP     1000    // 最大的进程数量。
#define SHMKEYP   0x5095    // 共享内存的key。
#define SEMKEYP   0x5095    // 信号量的key。

// 查看共享内存：  ipcs -m
// 删除共享内存：  ipcrm -m shmid
// 查看信号量：    ipcs -s
// 删除信号量：    ipcrm sem semid

// 进程心跳操作类。
class CPActive
{
private:
  CSEM m_sem;                 // 用于给共享内存加锁的信号量id。
  int  m_shmid;               // 共享内存的id。
  int  m_pos;                 // 当前进程在共享内存进程组中的位置。
  st_procinfo *m_shm;         // 指向共享内存的地址空间。

public:
  CPActive();  // 初始化成员变量。

  // 把当前进程的心跳信息加入共享内存进程组中。
  bool AddPInfo(const int timeout,const char *pname=0,CLogFile *logfile=0);

  // 更新共享内存进程组中当前进程的心跳时间。
  bool UptATime();

  ~CPActive();  // 从共享内存中删除当前进程的心跳记录。
};


```

##### _public.cpp

```
CSEM::CSEM()
{
  m_semid=-1;
  m_sem_flg=SEM_UNDO;
}

// 如果信号量已存在，获取信号量；如果信号量不存在，则创建它并初始化为value。
bool CSEM::init(key_t key,unsigned short value,short sem_flg)
{
  if (m_semid!=-1) return false;

  m_sem_flg=sem_flg;

  // 信号量的初始化不能直接用semget(key,1,0666|IPC_CREAT)，因为信号量创建后，初始值是0。

  // 信号量的初始化分三个步骤：
  // 1）获取信号量，如果成功，函数返回。
  // 2）如果失败，则创建信号量。
  // 3) 设置信号量的初始值。

  // 获取信号量。
  if ( (m_semid=semget(key,1,0666)) == -1)
  {
    // 如果信号量不存在，创建它。
    if (errno==2)
    {
      // 用IPC_EXCL标志确保只有一个进程创建并初始化信号量，其它进程只能获取。
      if ( (m_semid=semget(key,1,0666|IPC_CREAT|IPC_EXCL)) == -1)
      {
        if (errno!=EEXIST)
        {
          perror("init 1 semget()"); return false;
        }
        if ( (m_semid=semget(key,1,0666)) == -1)
        { perror("init 2 semget()"); return false; }
    
        return true;
      }

      // 信号量创建成功后，还需要把它初始化成value。
      union semun sem_union;
      sem_union.val = value;   // 设置信号量的初始值。
      if (semctl(m_semid,0,SETVAL,sem_union) <  0) { perror("init semctl()"); return false; }
    }
    else
    { perror("init 3 semget()"); return false; }
  }

  return true;
}

bool CSEM::P(short sem_op)
{
  if (m_semid==-1) return false;

  struct sembuf sem_b;
  sem_b.sem_num = 0;      // 信号量编号，0代表第一个信号量。
  sem_b.sem_op = sem_op;  // P操作的sem_op必须小于0。
  sem_b.sem_flg = m_sem_flg;   
  if (semop(m_semid,&sem_b,1) == -1) { perror("p semop()"); return false; }

  return true;
}

bool CSEM::V(short sem_op)
{
  if (m_semid==-1) return false;

  struct sembuf sem_b;
  sem_b.sem_num = 0;      // 信号量编号，0代表第一个信号量。
  sem_b.sem_op = sem_op;  // V操作的sem_op必须大于0。
  sem_b.sem_flg = m_sem_flg;
  if (semop(m_semid,&sem_b,1) == -1) { perror("V semop()"); return false; }

  return true;
}

// 获取信号量的值，成功返回信号量的值，失败返回-1。
int CSEM::value()
{
  return semctl(m_semid,0,GETVAL);
}

bool CSEM::destroy()
{
  if (m_semid==-1) return false;

  if (semctl(m_semid,0,IPC_RMID) == -1) { perror("destroy semctl()"); return false; }

  return true;
}

CSEM::~CSEM()
{
}

CPActive::CPActive()
{
  m_shmid=0;
  m_pos=-1;
  m_shm=0;
}

// 把当前进程的心跳信息加入共享内存进程组中。
bool CPActive::AddPInfo(const int timeout,const char *pname,CLogFile *logfile)
{
  if (m_pos!=-1) return true;

  if (m_sem.init(SEMKEYP) == false)  // 初始化信号量。
  {
    if (logfile!=0) logfile->Write("创建/获取信号量(%x)失败。\n",SEMKEYP); 
    else printf("创建/获取信号量(%x)失败。\n",SEMKEYP);

    return false;
  }

  // 创建/获取共享内存，键值为SHMKEYP，大小为MAXNUMP个st_procinfo结构体的大小。
  if ( (m_shmid = shmget((key_t)SHMKEYP, MAXNUMP*sizeof(struct st_procinfo), 0666|IPC_CREAT)) == -1)
  { 
    if (logfile!=0) logfile->Write("创建/获取共享内存(%x)失败。\n",SHMKEYP); 
    else printf("创建/获取共享内存(%x)失败。\n",SHMKEYP);

    return false; 
  }

  // 将共享内存连接到当前进程的地址空间。
  m_shm=(struct st_procinfo *)shmat(m_shmid, 0, 0);
  
  struct st_procinfo stprocinfo;    // 当前进程心跳信息的结构体。
  memset(&stprocinfo,0,sizeof(stprocinfo));

  stprocinfo.pid=getpid();            // 当前进程号。
  stprocinfo.timeout=timeout;         // 超时时间。
  stprocinfo.atime=time(0);           // 当前时间。
  STRNCPY(stprocinfo.pname,sizeof(stprocinfo.pname),pname,50); // 进程名。

  // 进程id是循环使用的，如果曾经有一个进程异常退出，没有清理自己的心跳信息，
  // 它的进程信息将残留在共享内存中，不巧的是，当前进程重用了上述进程的id，
  // 这样就会在共享内存中存在两个进程id相同的记录，守护进程检查到残留进程的
  // 心跳时，会向进程id发送退出信号，这个信号将误杀当前进程。

  // 如果共享内存中存在当前进程编号，一定是其它进程残留的数据，当前进程就重用该位置。
  for (int ii=0;ii<MAXNUMP;ii++)
  {
    if ( (m_shm+ii)->pid==stprocinfo.pid ) { m_pos=ii; break; }
  }

  m_sem.P();  // 给共享内存上锁。

  if (m_pos==-1)
  {
    // 如果m_pos==-1，共享内存的进程组中不存在当前进程编号，找一个空位置。
    for (int ii=0;ii<MAXNUMP;ii++)
      if ( (m_shm+ii)->pid==0 ) { m_pos=ii; break; }
  }

  if (m_pos==-1) 
  { 
    if (logfile!=0) logfile->Write("共享内存空间已用完。\n");
    else printf("共享内存空间已用完。\n");

    m_sem.V();  // 解锁。

    return false; 
  }

  // 把当前进程的心跳信息存入共享内存的进程组中。
  memcpy(m_shm+m_pos,&stprocinfo,sizeof(struct st_procinfo)); 

  m_sem.V();   // 解锁。

  return true;
}

// 更新共享内存进程组中当前进程的心跳时间。
bool CPActive::UptATime()
{
  if (m_pos==-1) return false;

  (m_shm+m_pos)->atime=time(0);

  return true;
}

CPActive::~CPActive()
{
  // 把当前进程从共享内存的进程组中移去。
  if (m_pos!=-1) memset(m_shm+m_pos,0,sizeof(struct st_procinfo));

  // 把共享内存从当前进程中分离。
  if (m_shm!=0) shmdt(m_shm);
}


```



#### 调度程序

procctl.cpp

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main(int argc,char *argv[])
{
  if (argc<3)
  {
    printf("Using:./procctl timetvl program argv ...\n");
    printf("Example:/project/tools1/bin/procctl 5 /usr/bin/tar zcvf /tmp/tmp.tgz /usr/include\n\n");

    printf("本程序是服务程序的调度程序，周期性启动服务程序或shell脚本。\n");
    printf("timetvl 运行周期，单位：秒。被调度的程序运行结束后，在timetvl秒后会被procctl重新启动。\n");
    printf("program 被调度的程序名，必须使用全路径。\n");
    printf("argvs   被调度的程序的参数。\n");
    printf("注意，本程序不会被kill杀死，但可以用kill -9强行杀死。\n\n\n");

    return -1;
  }

  // 关闭信号和IO，本程序不希望被打扰。
  for (int ii=0;ii<64;ii++)
  {
    signal(ii,SIG_IGN); close(ii);
  }

  // 生成子进程，父进程退出，让程序运行在后台，由系统1号进程托管。
  if (fork()!=0) exit(0);

  // 启用SIGCHLD信号，让父进程可以wait子进程退出的状态。
  signal(SIGCHLD,SIG_DFL);

  char *pargv[argc];
  for (int ii=2;ii<argc;ii++)
    pargv[ii-2]=argv[ii];

  pargv[argc-2]=NULL;

  while (true)
  {
    if (fork()==0)
    {
      execv(argv[2],pargv);
      exit(0);
    }
    else
    {
      int status;
      wait(&status);
      sleep(atoi(argv[1]));
    }
  }
}

```

##### 编译文件

makefile

```
all: procctl 

procctl:procctl.cpp
	g++ -o procctl procctl.cpp
	cp procctl ../bin/.

clean:
	rm -f procctl 
```



#### 守护进程程序

checkproc.cpp

```
#include "_public.h"

// 程序运行的日志。
CLogFile logfile;

int main(int argc,char *argv[])
{
  // 程序的帮助。
  if (argc != 2)
  {
    printf("\n");
    printf("Using:./checkproc logfilename\n");

    printf("Example:/project/tools1/bin/procctl 10 /project/tools1/bin/checkproc /tmp/log/checkproc.log\n\n");

    printf("本程序用于检查后台服务程序是否超时，如果已超时，就终止它。\n");
    printf("注意：\n");
    printf("  1）本程序由procctl启动，运行周期建议为10秒。\n");
    printf("  2）为了避免被普通用户误杀，本程序应该用root用户启动。\n");
    printf("  3）如果要停止本程序，只能用killall -9 终止。\n\n\n");

    return 0;
  }

  // 忽略全部的信号和IO，不希望程序被干扰。
  CloseIOAndSignal(true);

  // 打开日志文件。
  if (logfile.Open(argv[1],"a+")==false)
  { printf("logfile.Open(%s) failed.\n",argv[1]); return -1; }

  int shmid=0;

  // 创建/获取共享内存，键值为SHMKEYP，大小为MAXNUMP个st_procinfo结构体的大小。
  if ( (shmid = shmget((key_t)SHMKEYP, MAXNUMP*sizeof(struct st_procinfo), 0666|IPC_CREAT)) == -1)
  {
    logfile.Write("创建/获取共享内存(%x)失败。\n",SHMKEYP); return false;
  }

  // 将共享内存连接到当前进程的地址空间。
  struct st_procinfo *shm=(struct st_procinfo *)shmat(shmid, 0, 0);

  // 遍历共享内存中全部的记录。
  for (int ii=0;ii<MAXNUMP;ii++)
  {
    // 如果记录的pid==0，表示空记录，continue;
    if (shm[ii].pid==0) continue;

    // 如果记录的pid!=0，表示是服务程序的心跳记录。

    // 程序稳定运行后，以下两行代码可以注释掉。
    //logfile.Write("ii=%d,pid=%d,pname=%s,timeout=%d,atime=%d\n",\
    //               ii,shm[ii].pid,shm[ii].pname,shm[ii].timeout,shm[ii].atime);

    // 向进程发送信号0，判断它是否还存在，如果不存在，从共享内存中删除该记录，continue;
    int iret=kill(shm[ii].pid,0);
    if (iret==-1)
    {
      logfile.Write("进程pid=%d(%s)已经不存在。\n",(shm+ii)->pid,(shm+ii)->pname);
      memset(shm+ii,0,sizeof(struct st_procinfo)); // 从共享内存中删除该记录。
      continue;
    }

    time_t now=time(0);   // 取当前时间。

    // 如果进程未超时，continue;
    if (now-shm[ii].atime<shm[ii].timeout) continue;

    // 如果已超时。
    logfile.Write("进程pid=%d(%s)已经超时。\n",(shm+ii)->pid,(shm+ii)->pname);

    // 发送信号15，尝试正常终止进程。
    kill(shm[ii].pid,15);     

    // 每隔1秒判断一次进程是否存在，累计5秒，一般来说，5秒的时间足够让进程退出。
    for (int jj=0;jj<5;jj++)
    {
      sleep(1);
      iret=kill(shm[ii].pid,0);     // 向进程发送信号0，判断它是否还存在。
      if (iret==-1) break;     // 进程已退出。
    } 

    // 如果进程仍存在，就发送信号9，强制终止它。
    if (iret==-1)
      logfile.Write("进程pid=%d(%s)已经正常终止。\n",(shm+ii)->pid,(shm+ii)->pname);
    else
    {
      kill(shm[ii].pid,9);  // 如果进程仍存在，就发送信号9，强制终止它。
      logfile.Write("进程pid=%d(%s)已经强制终止。\n",(shm+ii)->pid,(shm+ii)->pname);
    }
    
    // 从共享内存中删除已超时进程的心跳记录。
    memset(shm+ii,0,sizeof(struct st_procinfo)); // 从共享内存中删除该记录。
  }

  // 把共享内存从当前进程中分离。
  shmdt(shm);

  return 0;
}

```

##### 编译文件

makefile

```
# 开发框架头文件路径。
PUBINCL = -I/project/public

# 开发框架cpp文件名，这里直接包含进来，没有采用链接库，是为了方便调试。
PUBCPP = /project/public/_public.cpp

# 编译参数。
CFLAGS = -g

all: checkproc 

checkproc:checkproc.cpp
	g++ $(CFLAGS) -o checkproc checkproc.cpp $(PUBINCL) $(PUBCPP) -lm -lc
	cp checkproc ../bin/.

clean:
	rm -f checkproc

```



#### 压缩文件程序

gzipfiles.cpp

```
#include "_public.h"

// 程序退出和信号2、15的处理函数。
void EXIT(int sig);

int main(int argc,char *argv[])
{
  // 程序的帮助。
  if (argc != 4)
  {
    printf("\n");
    printf("Using:/project/tools1/bin/gzipfiles pathname matchstr timeout\n\n");

    printf("Example:/project/tools1/bin/gzipfiles /log/idc \"*.log.20*\" 0.02\n");
    printf("        /project/tools1/bin/gzipfiles /tmp/idc/surfdata \"*.xml,*.json\" 0.01\n");
    printf("        /project/tools1/bin/procctl 300 /project/tools1/bin/gzipfiles /log/idc \"*.log.20*\" 0.02\n");
    printf("        /project/tools1/bin/procctl 300 /project/tools1/bin/gzipfiles /tmp/idc/surfdata \"*.xml,*.json\" 0.01\n\n");

    printf("这是一个工具程序，用于压缩历史的数据文件或日志文件。\n");
    printf("本程序把pathname目录及子目录中timeout天之前的匹配matchstr文件全部压缩，timeout可以是小数。\n");
    printf("本程序不写日志文件，也不会在控制台输出任何信息。\n");
    printf("本程序调用/usr/bin/gzip命令压缩文件。\n\n\n");

    return -1;
  }

  // 关闭全部的信号和输入输出。
  // 设置信号,在shell状态下可用 "kill + 进程号" 正常终止些进程。
  // 但请不要用 "kill -9 +进程号" 强行终止。
  CloseIOAndSignal(true); signal(SIGINT,EXIT);  signal(SIGTERM,EXIT);

  // 获取文件超时的时间点。
  char strTimeOut[21];
  LocalTime(strTimeOut,"yyyy-mm-dd hh24:mi:ss",0-(int)(atof(argv[3])*24*60*60));

  CDir Dir;
  // 打开目录，CDir.OpenDir()
  if (Dir.OpenDir(argv[1],argv[2],10000,true)==false)
  {
    printf("Dir.OpenDir(%s) failed.\n",argv[1]); return -1;
  }

  char strCmd[1024]; // 存放gzip压缩文件的命令。

  // 遍历目录中的文件名。
  while (true)
  {
    // 得到一个文件的信息，CDir.ReadDir()
    if (Dir.ReadDir()==false) break;
  
    // 与超时的时间点比较，如果更早，就需要压缩
    if ( (strcmp(Dir.m_ModifyTime,strTimeOut)<0) && (MatchStr(Dir.m_FileName,"*.gz")==false) )
    {
      // 压缩文件，调用操作系统的gzip命令。
      SNPRINTF(strCmd,sizeof(strCmd),1000,"/usr/bin/gzip -f %s 1>/dev/null 2>/dev/null",Dir.m_FullFileName);
      if (system(strCmd)==0) 
        printf("gzip %s ok.\n",Dir.m_FullFileName);
      else
        printf("gzip %s failed.\n",Dir.m_FullFileName);
    }
  }

  return 0;
}

void EXIT(int sig)
{
  printf("程序退出，sig=%d\n\n",sig);

  exit(0);
}

```

##### 编译文件

makefile

```
# 开发框架头文件路径。
PUBINCL = -I/project/public

# 开发框架cpp文件名，这里直接包含进来，没有采用链接库，是为了方便调试。
PUBCPP = /project/public/_public.cpp

# 编译参数。
CFLAGS = -g

all: gzipfiles

gzipfiles:gzipfiles.cpp
	g++ $(CFLAGS) -o gzipfiles gzipfiles.cpp $(PUBINCL) $(PUBCPP) -lm -lc
	cp gzipfiles ../bin/.

clean:
	rm -f gzipfiles

```



#### 清理文件程序

deletefiles.cpp

```
#include "_public.h"

// 程序退出和信号2、15的处理函数。
void EXIT(int sig);

int main(int argc,char *argv[])
{
  // 程序的帮助。
  if (argc != 4)
  {
    printf("\n");
    printf("Using:/project/tools1/bin/deletefiles pathname matchstr timeout\n\n");

    printf("Example:/project/tools1/bin/deletefiles /log/idc \"*.log.20*\" 0.02\n");
    printf("        /project/tools1/bin/deletefiles /tmp/idc/surfdata \"*.xml,*.json\" 0.01\n");
    printf("        /project/tools1/bin/procctl 300 /project/tools1/bin/deletefiles /log/idc \"*.log.20*\" 0.02\n");
    printf("        /project/tools1/bin/procctl 300 /project/tools1/bin/deletefiles /tmp/idc/surfdata \"*.xml,*.json\" 0.01\n\n");

    printf("这是一个工具程序，用于删除历史的数据文件或日志文件。\n");
    printf("本程序把pathname目录及子目录中timeout天之前的匹配matchstr文件全部删除，timeout可以是小数。\n");
    printf("本程序不写日志文件，也不会在控制台输出任何信息。\n\n\n");

    return -1;
  }

  // 关闭全部的信号和输入输出。
  // 设置信号,在shell状态下可用 "kill + 进程号" 正常终止些进程。
  // 但请不要用 "kill -9 +进程号" 强行终止。
  // CloseIOAndSignal(true); 
  signal(SIGINT,EXIT);  signal(SIGTERM,EXIT);

  // 获取文件超时的时间点。
  char strTimeOut[21];
  LocalTime(strTimeOut,"yyyy-mm-dd hh24:mi:ss",0-(int)(atof(argv[3])*24*60*60));

  CDir Dir;
  // 打开目录，CDir.OpenDir()
  if (Dir.OpenDir(argv[1],argv[2],10000,true)==false)
  {
    printf("Dir.OpenDir(%s) failed.\n",argv[1]); return -1;
  }

  // 遍历目录中的文件名。
  while (true)
  {
    // 得到一个文件的信息，CDir.ReadDir()
    if (Dir.ReadDir()==false) break;
printf("=%s=\n",Dir.m_FullFileName);  
    // 与超时的时间点比较，如果更早，就需要删除。
    if (strcmp(Dir.m_ModifyTime,strTimeOut)<0) 
    {
      if (REMOVE(Dir.m_FullFileName)==0) 
        printf("REMOVE %s ok.\n",Dir.m_FullFileName);
      else
        printf("REMOVE %s failed.\n",Dir.m_FullFileName);
    }
  }

  return 0;
}

void EXIT(int sig)
{
  printf("程序退出，sig=%d\n\n",sig);

  exit(0);
}

```



##### 编译文件

```
# 开发框架头文件路径。
PUBINCL = -I/project/public

# 开发框架cpp文件名，这里直接包含进来，没有采用链接库，是为了方便调试。
PUBCPP = /project/public/_public.cpp

# 编译参数。
CFLAGS = -g

all: deletefiles

deletefiles:deletefiles.cpp
	g++ $(CFLAGS) -o deletefiles deletefiles.cpp $(PUBINCL) $(PUBCPP) -lm -lc
	cp deletefiles ../bin/.

clean:
	rm -f deletefiles

```



#### 模拟生产测试数据文件

crtsurfdata.cpp

```
/*
 *  程序名：crtsurfdata.cpp  本程序用于生成全国气象站点观测的分钟数据。
*/

#include "_public.h"

CPActive PActive;   // 进程心跳。

// 全国气象站点参数结构体。
struct st_stcode
{
  char provname[31]; // 省
  char obtid[11];    // 站号
  char obtname[31];  // 站名
  double lat;        // 纬度
  double lon;        // 经度
  double height;     // 海拔高度
};

vector<struct st_stcode> vstcode; // 存放全国气象站点参数的容器。

// 把站点参数文件中加载到vstcode容器中。
bool LoadSTCode(const char *inifile);

// 全国气象站点分钟观测数据结构
struct st_surfdata
{
  char obtid[11];      // 站点代码。
  char ddatetime[21];  // 数据时间：格式yyyymmddhh24miss
  int  t;              // 气温：单位，0.1摄氏度。
  int  p;              // 气压：0.1百帕。
  int  u;              // 相对湿度，0-100之间的值。
  int  wd;             // 风向，0-360之间的值。
  int  wf;             // 风速：单位0.1m/s
  int  r;              // 降雨量：0.1mm。
  int  vis;            // 能见度：0.1米。
};

vector<struct st_surfdata> vsurfdata;  // 存放全国气象站点分钟观测数据的容器

char strddatetime[21]; // 观测数据的时间。

// 模拟生成全国气象站点分钟观测数据，存放在vsurfdata容器中。
void CrtSurfData();

CFile File;  // 文件操作对象。

// 把容器vsurfdata中的全国气象站点分钟观测数据写入文件。
bool CrtSurfFile(const char *outpath,const char *datafmt);

CLogFile logfile;    // 日志类。

void EXIT(int sig);  // 程序退出和信号2、15的处理函数。

int main(int argc,char *argv[])
{
  if ( (argc!=5) && (argc!=6) )
  {
    // 如果参数非法，给出帮助文档。
    printf("Using:./crtsurfdata inifile outpath logfile datafmt [datetime]\n");
    printf("Example:/project/idc1/bin/crtsurfdata /project/idc1/ini/stcode.ini /tmp/idc/surfdata /log/idc/crtsurfdata.log xml,json,csv\n");
    printf("        /project/idc1/bin/crtsurfdata /project/idc1/ini/stcode.ini /tmp/idc/surfdata /log/idc/crtsurfdata.log xml,json,csv 20210710123000\n");
    printf("        /project/tools1/bin/procctl 60 /project/idc1/bin/crtsurfdata /project/idc1/ini/stcode.ini /tmp/idc/surfdata /log/idc/crtsurfdata.log xml,json,csv\n\n\n");

    printf("inifile  全国气象站点参数文件名。\n");
    printf("outpath  全国气象站点数据文件存放的目录。\n");
    printf("logfile  本程序运行的日志文件名。\n");
    printf("datafmt  生成数据文件的格式，支持xml、json和csv三种格式，中间用逗号分隔。\n");
    printf("datetime 这是一个可选参数，表示生成指定时间的数据和文件。\n\n\n");

    return -1;
  }

  // 关闭全部的信号和输入输出。
  // 设置信号,在shell状态下可用 "kill + 进程号" 正常终止些进程。
  // 但请不要用 "kill -9 +进程号" 强行终止。
  CloseIOAndSignal(true); signal(SIGINT,EXIT);  signal(SIGTERM,EXIT);

  // 打开程序的日志文件。
  if (logfile.Open(argv[3],"a+",false)==false)
  {
    printf("logfile.Open(%s) failed.\n",argv[3]); return -1;
  }

  logfile.Write("crtsurfdata 开始运行。\n");

  PActive.AddPInfo(20,"crtsurfdata");

  // 把站点参数文件中加载到vstcode容器中。 
  if (LoadSTCode(argv[1])==false) return -1;

  // 获取当前时间，当作观测时间。
  memset(strddatetime,0,sizeof(strddatetime));
  if (argc==5)
    LocalTime(strddatetime,"yyyymmddhh24miss");
  else
    STRCPY(strddatetime,sizeof(strddatetime),argv[5]);

  // 模拟生成全国气象站点分钟观测数据，存放在vsurfdata容器中。
  CrtSurfData();

  // 把容器vsurfdata中的全国气象站点分钟观测数据写入文件。
  if (strstr(argv[4],"xml")!=0) CrtSurfFile(argv[2],"xml");
  if (strstr(argv[4],"json")!=0) CrtSurfFile(argv[2],"json");
  if (strstr(argv[4],"csv")!=0) CrtSurfFile(argv[2],"csv");

  logfile.Write("crtsurfdata 运行结束。\n");

  return 0;
}

// 把站点参数文件中加载到vstcode容器中。 
bool LoadSTCode(const char *inifile)
{
  // 打开站点参数文件。
  if (File.Open(inifile,"r")==false)
  {
    logfile.Write("File.Open(%s) failed.\n",inifile); return false;
  }

  char strBuffer[301];

  CCmdStr CmdStr;

  struct st_stcode stcode;

  while (true)
  {
    // 从站点参数文件中读取一行，如果已读取完，跳出循环。
    if (File.Fgets(strBuffer,300,true)==false) break;

    // 把读取到的一行拆分。
    CmdStr.SplitToCmd(strBuffer,",",true);

    if (CmdStr.CmdCount()!=6) continue;     // 扔掉无效的行。

    // 把站点参数的每个数据项保存到站点参数结构体中。
    memset(&stcode,0,sizeof(struct st_stcode));
    CmdStr.GetValue(0, stcode.provname,30); // 省
    CmdStr.GetValue(1, stcode.obtid,10);    // 站号
    CmdStr.GetValue(2, stcode.obtname,30);  // 站名
    CmdStr.GetValue(3,&stcode.lat);         // 纬度
    CmdStr.GetValue(4,&stcode.lon);         // 经度
    CmdStr.GetValue(5,&stcode.height);      // 海拔高度

    // 把站点参数结构体放入站点参数容器。
    vstcode.push_back(stcode);
  }

  /*
  for (int ii=0;ii<vstcode.size();ii++)
    logfile.Write("provname=%s,obtid=%s,obtname=%s,lat=%.2f,lon=%.2f,height=%.2f\n",\
                   vstcode[ii].provname,vstcode[ii].obtid,vstcode[ii].obtname,vstcode[ii].lat,\
                   vstcode[ii].lon,vstcode[ii].height);
  */

  return true;
}

// 模拟生成全国气象站点分钟观测数据，存放在vsurfdata容器中。
void CrtSurfData()
{
  // 播随机数种子。
  srand(time(0));

  struct st_surfdata stsurfdata;

  // 遍历气象站点参数的vstcode容器。
  for (int ii=0;ii<vstcode.size();ii++)
  {
    memset(&stsurfdata,0,sizeof(struct st_surfdata));

    // 用随机数填充分钟观测数据的结构体。
    strncpy(stsurfdata.obtid,vstcode[ii].obtid,10); // 站点代码。
    strncpy(stsurfdata.ddatetime,strddatetime,14);  // 数据时间：格式yyyymmddhh24miss
    stsurfdata.t=rand()%351;       // 气温：单位，0.1摄氏度
    stsurfdata.p=rand()%265+10000; // 气压：0.1百帕
    stsurfdata.u=rand()%100+1;     // 相对湿度，0-100之间的值。
    stsurfdata.wd=rand()%360;      // 风向，0-360之间的值。
    stsurfdata.wf=rand()%150;      // 风速：单位0.1m/s
    stsurfdata.r=rand()%16;        // 降雨量：0.1mm
    stsurfdata.vis=rand()%5001+100000;  // 能见度：0.1米

    // 把观测数据的结构体放入vsurfdata容器。
    vsurfdata.push_back(stsurfdata);
  }
}

// 把容器vsurfdata中的全国气象站点分钟观测数据写入文件。
bool CrtSurfFile(const char *outpath,const char *datafmt)
{
  // 拼接生成数据的文件名，例如：/tmp/idc/surfdata/SURF_ZH_20210629092200_2254.csv
  char strFileName[301];
  sprintf(strFileName,"%s/SURF_ZH_%s_%d.%s",outpath,strddatetime,getpid(),datafmt);

  // 打开文件。
  if (File.OpenForRename(strFileName,"w")==false)
  {
    logfile.Write("File.OpenForRename(%s) failed.\n",strFileName); return false;
  }

  if (strcmp(datafmt,"csv")==0) File.Fprintf("站点代码,数据时间,气温,气压,相对湿度,风向,风速,降雨量,能见度\n");
  if (strcmp(datafmt,"xml")==0) File.Fprintf("<data>\n");
  if (strcmp(datafmt,"json")==0) File.Fprintf("{\"data\":[\n");

  // 遍历存放观测数据的vsurfdata容器。
  for (int ii=0;ii<vsurfdata.size();ii++)
  {
    // 写入一条记录。
    if (strcmp(datafmt,"csv")==0)
      File.Fprintf("%s,%s,%.1f,%.1f,%d,%d,%.1f,%.1f,%.1f\n",\
         vsurfdata[ii].obtid,vsurfdata[ii].ddatetime,vsurfdata[ii].t/10.0,vsurfdata[ii].p/10.0,\
         vsurfdata[ii].u,vsurfdata[ii].wd,vsurfdata[ii].wf/10.0,vsurfdata[ii].r/10.0,vsurfdata[ii].vis/10.0);

    if (strcmp(datafmt,"xml")==0)
      File.Fprintf("<obtid>%s</obtid><ddatetime>%s</ddatetime><t>%.1f</t><p>%.1f</p>"\
                   "<u>%d</u><wd>%d</wd><wf>%.1f</wf><r>%.1f</r><vis>%.1f</vis><endl/>\n",\
         vsurfdata[ii].obtid,vsurfdata[ii].ddatetime,vsurfdata[ii].t/10.0,vsurfdata[ii].p/10.0,\
         vsurfdata[ii].u,vsurfdata[ii].wd,vsurfdata[ii].wf/10.0,vsurfdata[ii].r/10.0,vsurfdata[ii].vis/10.0);

    if (strcmp(datafmt,"json")==0)
    {
      File.Fprintf("{\"obtid\":\"%s\",\"ddatetime\":\"%s\",\"t\":\"%.1f\",\"p\":\"%.1f\","\
                   "\"u\":\"%d\",\"wd\":\"%d\",\"wf\":\"%.1f\",\"r\":\"%.1f\",\"vis\":\"%.1f\"}",\
         vsurfdata[ii].obtid,vsurfdata[ii].ddatetime,vsurfdata[ii].t/10.0,vsurfdata[ii].p/10.0,\
         vsurfdata[ii].u,vsurfdata[ii].wd,vsurfdata[ii].wf/10.0,vsurfdata[ii].r/10.0,vsurfdata[ii].vis/10.0);

      if (ii<vsurfdata.size()-1) File.Fprintf(",\n");
      else   File.Fprintf("\n");
    }
  }

  if (strcmp(datafmt,"xml")==0) File.Fprintf("</data>\n");
  if (strcmp(datafmt,"json")==0) File.Fprintf("]}\n");

  // 关闭文件。
  File.CloseAndRename();

  UTime(strFileName,strddatetime);  // 修改文件的时间属性。

  logfile.Write("生成数据文件%s成功，数据时间%s，记录数%d。\n",strFileName,strddatetime,vsurfdata.size());

  return true;
}

// 程序退出和信号2、15的处理函数。
void EXIT(int sig)  
{
  logfile.Write("程序退出，sig=%d\n\n",sig);

  exit(0);
}

```

##### 编译文件

makfile

```
# 开发框架头文件路径。
PUBINCL = -I/project/public

# 开发框架cpp文件名，这里直接包含进来，没有采用链接库，是为了方便调试。
PUBCPP = /project/public/_public.cpp

# 编译参数。
CFLAGS = -g 

all:crtsurfdata 

crtsurfdata:crtsurfdata.cpp
	g++ $(CFLAGS) -o crtsurfdata crtsurfdata.cpp $(PUBINCL) $(PUBCPP) -lm -lc
	cp crtsurfdata ../bin/.

clean:
	rm -f crtsurfdata 

```



#### ftp上传下载

##### 安装ftp

去看安装ftp说明



##### ftp客户端下载

ftpgetfiles.cpp

```
#include "_public.h"
#include "_ftp.h"

// 程序运行参数的结构体。
struct st_arg
{
  char host[31];           // 远程服务端的IP和端口。
  int  mode;               // 传输模式，1-被动模式，2-主动模式，缺省采用被动模式。
  char username[31];       // 远程服务端ftp的用户名。
  char password[31];       // 远程服务端ftp的密码。
  char remotepath[301];    // 远程服务端存放文件的目录。
  char localpath[301];     // 本地文件存放的目录。
  char matchname[101];     // 待下载文件匹配的规则。
  char listfilename[301];  // 下载前列出服务端文件名的文件。
  int  ptype;              // 下载后服务端文件的处理方式：1-什么也不做；2-删除；3-备份。
  char remotepathbak[301]; // 下载后服务端文件的备份目录。
  char okfilename[301];    // 已下载成功文件名清单。
  bool checkmtime;         // 是否需要检查服务端文件的时间，true-需要，false-不需要，缺省为false。
  int  timeout;            // 进程心跳的超时时间。
  char pname[51];          // 进程名，建议用"ftpgetfiles_后缀"的方式。
} starg;

// 文件信息的结构体。
struct st_fileinfo
{
  char filename[301];   // 文件名。
  char mtime[21];       // 文件时间。
};

vector<struct st_fileinfo> vlistfile1;    // 已下载成功文件名的容器，从okfilename中加载。
vector<struct st_fileinfo> vlistfile2;    // 下载前列出服务端文件名的容器，从nlist文件中加载。
vector<struct st_fileinfo> vlistfile3;    // 本次不需要下载的文件的容器。
vector<struct st_fileinfo> vlistfile4;    // 本次需要下载的文件的容器。

// 加载okfilename文件中的内容到容器vlistfile1中。
bool LoadOKFile();

// 比较vlistfile2和vlistfile1，得到vlistfile3和vlistfile4。
bool CompVector();

// 把容器vlistfile3中的内容写入okfilename文件，覆盖之前的旧okfilename文件。
bool WriteToOKFile();

// 如果ptype==1，把下载成功的文件记录追加到okfilename文件中。
bool AppendToOKFile(struct st_fileinfo *stfileinfo);

// 把ftp.nlist()方法获取到的list文件加载到容器vlistfile2中。
bool LoadListFile();

CLogFile logfile;

Cftp ftp;

// 程序退出和信号2、15的处理函数。
void EXIT(int sig);

void _help();

// 把xml解析到参数starg结构中。
bool _xmltoarg(char *strxmlbuffer);

// 下载文件功能的主函数。
bool _ftpgetfiles();

CPActive PActive;  // 进程心跳。

int main(int argc,char *argv[])
{
  if (argc!=3) { _help(); return -1; }

  // 关闭全部的信号和输入输出。
  // 设置信号,在shell状态下可用 "kill + 进程号" 正常终止些进程。
  // 但请不要用 "kill -9 +进程号" 强行终止。
  CloseIOAndSignal(); signal(SIGINT,EXIT); signal(SIGTERM,EXIT);

  // 打开日志文件。
  if (logfile.Open(argv[1],"a+")==false)
  {
    printf("打开日志文件失败（%s）。\n",argv[1]); return -1;
  }

  // 解析xml，得到程序运行的参数。
  if (_xmltoarg(argv[2])==false) return -1;

  PActive.AddPInfo(starg.timeout,starg.pname);  // 把进程的心跳信息写入共享内存。

  // 登录ftp服务端。
  if (ftp.login(starg.host,starg.username,starg.password,starg.mode)==false)
  {
    logfile.Write("ftp.login(%s,%s,%s) failed.\n",starg.host,starg.username,starg.password); return -1;
  }

  // logfile.Write("ftp.login ok.\n");  // 正式运行后，可以注释这行代码。

  _ftpgetfiles();

  ftp.logout();

  return 0;
}

// 下载文件功能的主函数。
bool _ftpgetfiles()
{
  // 进入ftp服务端存放文件的目录。
  if (ftp.chdir(starg.remotepath)==false)
  {
    logfile.Write("ftp.chdir(%s) failed.\n",starg.remotepath); return false;
  }

  // 调用ftp.nlist()方法列出服务端目录中的文件，结果存放到本地文件中。
  if (ftp.nlist(".",starg.listfilename)==false)
  {
    logfile.Write("ftp.nlist(%s) failed.\n",starg.remotepath); return false;
  }

  PActive.UptATime();   // 更新进程的心跳。

  // 把ftp.nlist()方法获取到的list文件加载到容器vlistfile2中。
  if (LoadListFile()==false)
  {
    logfile.Write("LoadListFile() failed.\n");  return false;
  }

  PActive.UptATime();   // 更新进程的心跳。

  if (starg.ptype==1)
  {
    // 加载okfilename文件中的内容到容器vlistfile1中。
    LoadOKFile();

    // 比较vlistfile2和vlistfile1，得到vlistfile3和vlistfile4。
    CompVector();

    // 把容器vlistfile3中的内容写入okfilename文件，覆盖之前的旧okfilename文件。
    WriteToOKFile();

    // 把vlistfile4中的内容复制到vlistfile2中。
    vlistfile2.clear(); vlistfile2.swap(vlistfile4);
  }

  PActive.UptATime();   // 更新进程的心跳。

  char strremotefilename[301],strlocalfilename[301];

  // 遍历容器vlistfile2。
  for (int ii=0;ii<vlistfile2.size();ii++)
  {
    SNPRINTF(strremotefilename,sizeof(strremotefilename),300,"%s/%s",starg.remotepath,vlistfile2[ii].filename);
    SNPRINTF(strlocalfilename,sizeof(strlocalfilename),300,"%s/%s",starg.localpath,vlistfile2[ii].filename);

    logfile.Write("get %s ...",strremotefilename);

    // 调用ftp.get()方法从服务端下载文件。
    if (ftp.get(strremotefilename,strlocalfilename)==false) 
    {
      logfile.WriteEx("failed.\n"); return false;
    }

    logfile.WriteEx("ok.\n");

    PActive.UptATime();   // 更新进程的心跳。
    
    // 如果ptype==1，把下载成功的文件记录追加到okfilename文件中。
    if (starg.ptype==1) AppendToOKFile(&vlistfile2[ii]);

    // 删除服务端的文件。
    if (starg.ptype==2) 
    {
      if (ftp.ftpdelete(strremotefilename)==false)
      {
        logfile.Write("ftp.ftpdelete(%s) failed.\n",strremotefilename); return false;
      }
    }

    // 把服务端的文件转存到备份目录。
    if (starg.ptype==3) 
    {
      char strremotefilenamebak[301];
      SNPRINTF(strremotefilenamebak,sizeof(strremotefilenamebak),300,"%s/%s",starg.remotepathbak,vlistfile2[ii].filename);
      if (ftp.ftprename(strremotefilename,strremotefilenamebak)==false)
      {
        logfile.Write("ftp.ftprename(%s,%s) failed.\n",strremotefilename,strremotefilenamebak); return false;
      }
    }
  }

  return true;
}

void EXIT(int sig)
{
  printf("程序退出，sig=%d\n\n",sig);

  exit(0);
}

void _help()
{
  printf("\n");
  printf("Using:/project/tools1/bin/ftpgetfiles logfilename xmlbuffer\n\n");

  printf("Sample:/project/tools1/bin/procctl 30 /project/tools1/bin/ftpgetfiles /log/idc/ftpgetfiles.log \"<host>127.0.0.1:21</host><mode>1</mode><username>lijialin</username><password>6VzFeKBnSoV.</password><localpath>/home/lijialin/ftpget</localpath><remotepath>/log/idc</remotepath><matchname>*</matchname><listfilename>/home/lijialin/ftpgetfiles.list</listfilename><ptype>1</ptype><remotepathbak>/tmp/idc/surfdatabak</remotepathbak><okfilename>/home/lijialin/ftpgetfiles.xml</okfilename><checkmtime>true</checkmtime><timeout>80</timeout><pname>ftplijialingetfiles</pname>\"\n\n\n");

  printf("本程序是通用的功能模块，用于把远程ftp服务端的文件下载到本地目录。\n");
  printf("logfilename是本程序运行的日志文件。\n");
  printf("xmlbuffer为文件下载的参数，如下：\n");
  printf("<host>127.0.0.1:21</host> 远程服务端的IP和端口。\n");
  printf("<mode>1</mode> 传输模式，1-被动模式，2-主动模式，缺省采用被动模式。\n");
  printf("<username>wucz</username> 远程服务端ftp的用户名。\n");
  printf("<password>wuczpwd</password> 远程服务端ftp的密码。\n");
  printf("<remotepath>/tmp/idc/surfdata</remotepath> 远程服务端存放文件的目录。\n");
  printf("<localpath>/idcdata/surfdata</localpath> 本地文件存放的目录。\n");
  printf("<matchname>SURF_ZH*.XML,SURF_ZH*.CSV</matchname> 待下载文件匹配的规则。"\
         "不匹配的文件不会被下载，本字段尽可能设置精确，不建议用*匹配全部的文件。\n");
  printf("<listfilename>/idcdata/ftplist/ftpgetfiles_surfdata.list</listfilename> 下载前列出服务端文件名的文件。\n");
  printf("<ptype>1</ptype> 文件下载成功后，远程服务端文件的处理方式：1-什么也不做；2-删除；3-备份，如果为3，还要指定备份的目录。\n");
  printf("<remotepathbak>/tmp/idc/surfdatabak</remotepathbak> 文件下载成功后，服务端文件的备份目录，此参数只有当ptype=3时才有效。\n");
  printf("<okfilename>/idcdata/ftplist/ftpgetfiles_surfdata.xml</okfilename> 已下载成功文件名清单，此参数只有当ptype=1时才有效。\n");
  printf("<checkmtime>true</checkmtime> 是否需要检查服务端文件的时间，true-需要，false-不需要，此参数只有当ptype=1时才有效，缺省为false。\n");
  printf("<timeout>80</timeout> 下载文件超时时间，单位：秒，视文件大小和网络带宽而定。\n");
  printf("<pname>ftpgetfiles_surfdata</pname> 进程名，尽可能采用易懂的、与其它进程不同的名称，方便故障排查。\n\n\n");
}

// 把xml解析到参数starg结构中。
bool _xmltoarg(char *strxmlbuffer)
{
  memset(&starg,0,sizeof(struct st_arg));

  GetXMLBuffer(strxmlbuffer,"host",starg.host,30);   // 远程服务端的IP和端口。
  if (strlen(starg.host)==0)
  { logfile.Write("host is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"mode",&starg.mode);   // 传输模式，1-被动模式，2-主动模式，缺省采用被动模式。
  if (starg.mode!=2)  starg.mode=1;

  GetXMLBuffer(strxmlbuffer,"username",starg.username,30);   // 远程服务端ftp的用户名。
  if (strlen(starg.username)==0)
  { logfile.Write("username is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"password",starg.password,30);   // 远程服务端ftp的密码。
  if (strlen(starg.password)==0)
  { logfile.Write("password is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"remotepath",starg.remotepath,300);   // 远程服务端存放文件的目录。
  if (strlen(starg.remotepath)==0)
  { logfile.Write("remotepath is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"localpath",starg.localpath,300);   // 本地文件存放的目录。
  if (strlen(starg.localpath)==0)
  { logfile.Write("localpath is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"matchname",starg.matchname,100);   // 待下载文件匹配的规则。
  if (strlen(starg.matchname)==0)
  { logfile.Write("matchname is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"listfilename",starg.listfilename,300);   // 下载前列出服务端文件名的文件。
  if (strlen(starg.listfilename)==0)
  { logfile.Write("listfilename is null.\n");  return false; }

  // 下载后服务端文件的处理方式：1-什么也不做；2-删除；3-备份。
  GetXMLBuffer(strxmlbuffer,"ptype",&starg.ptype);   
  if ( (starg.ptype!=1) && (starg.ptype!=2) && (starg.ptype!=3) )
  { logfile.Write("ptype is error.\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"remotepathbak",starg.remotepathbak,300); // 下载后服务端文件的备份目录。
  if ( (starg.ptype==3) && (strlen(starg.remotepathbak)==0) )
  { logfile.Write("remotepathbak is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"okfilename",starg.okfilename,300); // 已下载成功文件名清单。
  if ( (starg.ptype==1) && (strlen(starg.okfilename)==0) )
  { logfile.Write("okfilename is null.\n");  return false; }

  // 是否需要检查服务端文件的时间，true-需要，false-不需要，此参数只有当ptype=1时才有效，缺省为false。
  GetXMLBuffer(strxmlbuffer,"checkmtime",&starg.checkmtime);

  GetXMLBuffer(strxmlbuffer,"timeout",&starg.timeout);   // 进程心跳的超时时间。
  if (starg.timeout==0) { logfile.Write("timeout is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"pname",starg.pname,50);     // 进程名。
  if (strlen(starg.pname)==0) { logfile.Write("pname is null.\n");  return false; }

  return true;
}

// 把ftp.nlist()方法获取到的list文件加载到容器vlistfile2中。
bool LoadListFile()
{
  vlistfile2.clear();

  CFile  File;

  if (File.Open(starg.listfilename,"r")==false)
  {
    logfile.Write("File.Open(%s) 失败。\n",starg.listfilename); return false;
  }

  struct st_fileinfo stfileinfo;

  while (true)
  {
    memset(&stfileinfo,0,sizeof(struct st_fileinfo));
   
    if (File.Fgets(stfileinfo.filename,300,true)==false) break;

    if (MatchStr(stfileinfo.filename,starg.matchname)==false) continue;

    if ( (starg.ptype==1) && (starg.checkmtime==true) )
    {
      // 获取ftp服务端文件时间。
      if (ftp.mtime(stfileinfo.filename)==false)
      {
        logfile.Write("ftp.mtime(%s) failed.\n",stfileinfo.filename); return false;
      }
    
      strcpy(stfileinfo.mtime,ftp.m_mtime);
    }

    vlistfile2.push_back(stfileinfo);
  }

  /*
  for (int ii=0;ii<vlistfile2.size();ii++)
    logfile.Write("filename=%s=\n",vlistfile2[ii].filename);
  */

  return true;
}

// 加载okfilename文件中的内容到容器vlistfile1中。
bool LoadOKFile()
{
  vlistfile1.clear();

  CFile File;

  // 注意：如果程序是第一次下载，okfilename是不存在的，并不是错误，所以也返回true。
  if ( (File.Open(starg.okfilename,"r"))==false )  return true;

  char strbuffer[501];

  struct st_fileinfo stfileinfo;

  while (true)
  {
    memset(&stfileinfo,0,sizeof(struct st_fileinfo));

    if (File.Fgets(strbuffer,300,true)==false) break;

    GetXMLBuffer(strbuffer,"filename",stfileinfo.filename);
    GetXMLBuffer(strbuffer,"mtime",stfileinfo.mtime);

    vlistfile1.push_back(stfileinfo);
  }

  return true;
}

// 比较vlistfile2和vlistfile1，得到vlistfile3和vlistfile4。
bool CompVector()
{
  vlistfile3.clear(); vlistfile4.clear();

  int ii,jj;

  // 遍历vlistfile2。
  for (ii=0;ii<vlistfile2.size();ii++)
  {
    // 在vlistfile1中查找vlistfile2[ii]的记录。
    for (jj=0;jj<vlistfile1.size();jj++)
    {
      // 如果找到了，把记录放入vlistfile3。
      if ( (strcmp(vlistfile2[ii].filename,vlistfile1[jj].filename)==0) &&
           (strcmp(vlistfile2[ii].mtime,vlistfile1[jj].mtime)==0) )
      {
        vlistfile3.push_back(vlistfile2[ii]); break;
      }
    }

    // 如果没有找到，把记录放入vlistfile4。
    if (jj==vlistfile1.size()) vlistfile4.push_back(vlistfile2[ii]);
  }

  return true;
}

// 把容器vlistfile3中的内容写入okfilename文件，覆盖之前的旧okfilename文件。
bool WriteToOKFile()
{
  CFile File;    

  if (File.Open(starg.okfilename,"w")==false)
  {
    logfile.Write("File.Open(%s) failed.\n",starg.okfilename); return false;
  }

  for (int ii=0;ii<vlistfile3.size();ii++)
    File.Fprintf("<filename>%s</filename><mtime>%s</mtime>\n",vlistfile3[ii].filename,vlistfile3[ii].mtime);

  return true;
}

// 如果ptype==1，把下载成功的文件记录追加到okfilename文件中。
bool AppendToOKFile(struct st_fileinfo *stfileinfo)
{
  CFile File;

  if (File.Open(starg.okfilename,"a")==false)
  {
    logfile.Write("File.Open(%s) failed.\n",starg.okfilename); return false;
  }

  File.Fprintf("<filename>%s</filename><mtime>%s</mtime>\n",stfileinfo->filename,stfileinfo->mtime);

  return true;
}

```

##### ftp客户端上传

ftpputfiles.cpp

```
#include "_public.h"
#include "_ftp.h"

// 程序运行参数的结构体。
struct st_arg
{
  char host[31];           // 远程服务端的IP和端口。
  int  mode;               // 传输模式，1-被动模式，2-主动模式，缺省采用被动模式。
  char username[31];       // 远程服务端ftp的用户名。
  char password[31];       // 远程服务端ftp的密码。
  char remotepath[301];    // 远程服务端存放文件的目录。
  char localpath[301];     // 本地文件存放的目录。
  char matchname[101];     // 待上传文件匹配的规则。
  int  ptype;              // 上传后客户端文件的处理方式：1-什么也不做；2-删除；3-备份。
  char localpathbak[301];  // 上传后客户端文件的备份目录。
  char okfilename[301];    // 已上传成功文件名清单。
  int  timeout;            // 进程心跳的超时时间。
  char pname[51];          // 进程名，建议用"ftpputfiles_后缀"的方式。
} starg;

// 文件信息的结构体。
struct st_fileinfo
{
  char filename[301];   // 文件名。
  char mtime[21];       // 文件时间。
};

vector<struct st_fileinfo> vlistfile1;    // 已上传成功文件名的容器，从okfilename中加载。
vector<struct st_fileinfo> vlistfile2;    // 上传前列出客户端文件名的容器，从nlist文件中加载。
vector<struct st_fileinfo> vlistfile3;    // 本次不需要上传的文件的容器。
vector<struct st_fileinfo> vlistfile4;    // 本次需要上传的文件的容器。

// 加载okfilename文件中的内容到容器vlistfile1中。
bool LoadOKFile();

// 比较vlistfile2和vlistfile1，得到vlistfile3和vlistfile4。
bool CompVector();

// 把容器vlistfile3中的内容写入okfilename文件，覆盖之前的旧okfilename文件。
bool WriteToOKFile();

// 如果ptype==1，把上传成功的文件记录追加到okfilename文件中。
bool AppendToOKFile(struct st_fileinfo *stfileinfo);

// 把localpath目录下的文件加载到vlistfile2容器中。
bool LoadLocalFile();

CLogFile logfile;

Cftp ftp;

// 程序退出和信号2、15的处理函数。
void EXIT(int sig);

void _help();

// 把xml解析到参数starg结构中。
bool _xmltoarg(char *strxmlbuffer);

// 上传文件功能的主函数。
bool _ftpputfiles();

CPActive PActive;  // 进程心跳。

int main(int argc,char *argv[])
{
  if (argc!=3) { _help(); return -1; }

  // 关闭全部的信号和输入输出。
  // 设置信号,在shell状态下可用 "kill + 进程号" 正常终止些进程。
  // 但请不要用 "kill -9 +进程号" 强行终止。
  CloseIOAndSignal(); signal(SIGINT,EXIT); signal(SIGTERM,EXIT);

  // 打开日志文件。
  if (logfile.Open(argv[1],"a+")==false)
  {
    printf("打开日志文件失败（%s）。\n",argv[1]); return -1;
  }

  // 解析xml，得到程序运行的参数。
  if (_xmltoarg(argv[2])==false) return -1;

  PActive.AddPInfo(starg.timeout,starg.pname);  // 把进程的心跳信息写入共享内存。

  // 登录ftp服务端。
  if (ftp.login(starg.host,starg.username,starg.password,starg.mode)==false)
  {
    logfile.Write("ftp.login(%s,%s,%s) failed.\n",starg.host,starg.username,starg.password); return -1;
  }

  // logfile.Write("ftp.login ok.\n");  // 正式运行后，可以注释这行代码。

  _ftpputfiles();

  ftp.logout();

  return 0;
}

// 上传文件功能的主函数。
bool _ftpputfiles()
{
  // 把localpath目录下的文件加载到vlistfile2容器中。
  if (LoadLocalFile()==false)
  {
    logfile.Write("LoadLocalFile() failed.\n");  return false;
  }

  PActive.UptATime();   // 更新进程的心跳。

  if (starg.ptype==1)
  {
    // 加载okfilename文件中的内容到容器vlistfile1中。
    LoadOKFile();

    // 比较vlistfile2和vlistfile1，得到vlistfile3和vlistfile4。
    CompVector();

    // 把容器vlistfile3中的内容写入okfilename文件，覆盖之前的旧okfilename文件。
    WriteToOKFile();

    // 把vlistfile4中的内容复制到vlistfile2中。
    vlistfile2.clear(); vlistfile2.swap(vlistfile4);
  }

  PActive.UptATime();   // 更新进程的心跳。

  char strremotefilename[301],strlocalfilename[301];

  // 遍历容器vlistfile2。
  for (int ii=0;ii<vlistfile2.size();ii++)
  {
    SNPRINTF(strremotefilename,sizeof(strremotefilename),300,"%s/%s",starg.remotepath,vlistfile2[ii].filename);
    SNPRINTF(strlocalfilename,sizeof(strlocalfilename),300,"%s/%s",starg.localpath,vlistfile2[ii].filename);

    logfile.Write("put %s ...",strlocalfilename);

    // 调用ftp.put()方法把文件上传到服务端，第三个参数填true的目的是确保文件上传成功，对方不可抵赖。
    if (ftp.put(strlocalfilename,strremotefilename,true)==false) 
    {
      logfile.WriteEx("failed.\n"); return false;
    }

    logfile.WriteEx("ok.\n");

    PActive.UptATime();   // 更新进程的心跳。
    
    // 如果ptype==1，把上传成功的文件记录追加到okfilename文件中。
    if (starg.ptype==1) AppendToOKFile(&vlistfile2[ii]);

    // 删除文件。
    if (starg.ptype==2)
    {
      if (REMOVE(strlocalfilename)==false)
      {
        logfile.Write("REMOVE(%s) failed.\n",strlocalfilename); return false;
      }
    }

    // 转存到备份目录。
    if (starg.ptype==3)
    {
      char strlocalfilenamebak[301];
      SNPRINTF(strlocalfilenamebak,sizeof(strlocalfilenamebak),300,"%s/%s",starg.localpathbak,vlistfile2[ii].filename);
      if (RENAME(strlocalfilename,strlocalfilenamebak)==false)
      {
        logfile.Write("RENAME(%s,%s) failed.\n",strlocalfilename,strlocalfilenamebak); return false;
      }
    }
  }

  return true;
}

void EXIT(int sig)
{
  printf("程序退出，sig=%d\n\n",sig);

  exit(0);
}

void _help()
{
  printf("\n");
  printf("Using:/project/tools1/bin/ftpputfiles logfilename xmlbuffer\n\n");

  printf("Sample:/project/tools1/bin/procctl 30 /project/tools1/bin/ftpputfiles /log/idc/ftpputfiles.log \"<host>127.0.0.1:21</host><mode>1</mode><username>lijialin</username><password>6VzFeKBnSoV.</password><localpath>/home/lijialin/ftptext</localpath><remotepath>/tmp/ftptext</remotepath><matchname>*</matchname><ptype>1</ptype><localpathbak>/tmp/idc/surfdatabak</localpathbak><okfilename>/home/lijialin/textlist.xml</okfilename><timeout>80</timeout><pname>ftplijialinputfiletext</pname>\"\n\n\n");

  printf("本程序是通用的功能模块，用于把本地目录中的文件上传到远程的ftp服务器。\n");
  printf("logfilename是本程序运行的日志文件。\n");
  printf("xmlbuffer为文件上传的参数，如下：\n");
  printf("<host>127.0.0.1:21</host> 远程服务端的IP和端口。\n");
  printf("<mode>1</mode> 传输模式，1-被动模式，2-主动模式，缺省采用被动模式。\n");
  printf("<username>wucz</username> 远程服务端ftp的用户名。\n");
  printf("<password>wuczpwd</password> 远程服务端ftp的密码。\n");
  printf("<remotepath>/tmp/ftpputest</remotepath> 远程服务端存放文件的目录。\n");
  printf("<localpath>/tmp/idc/surfdata</localpath> 本地文件存放的目录。\n");
  printf("<matchname>SURF_ZH*.JSON</matchname> 待上传文件匹配的规则。"\
         "不匹配的文件不会被上传，本字段尽可能设置精确，不建议用*匹配全部的文件。\n");
  printf("<ptype>1</ptype> 文件上传成功后，本地文件的处理方式：1-什么也不做；2-删除；3-备份，如果为3，还要指定备份的目录。\n");
  printf("<localpathbak>/tmp/idc/surfdatabak</localpathbak> 文件上传成功后，本地文件的备份目录，此参数只有当ptype=3时才有效。\n");
  printf("<okfilename>/idcdata/ftplist/ftpputfiles_surfdata.xml</okfilename> 已上传成功文件名清单，此参数只有当ptype=1时才有效。\n");
  printf("<timeout>80</timeout> 上传文件超时时间，单位：秒，视文件大小和网络带宽而定。\n");
  printf("<pname>ftpputfiles_surfdata</pname> 进程名，尽可能采用易懂的、与其它进程不同的名称，方便故障排查。\n\n\n");
}

// 把xml解析到参数starg结构中。
bool _xmltoarg(char *strxmlbuffer)
{
  memset(&starg,0,sizeof(struct st_arg));

  GetXMLBuffer(strxmlbuffer,"host",starg.host,30);   // 远程服务端的IP和端口。
  if (strlen(starg.host)==0)
  { logfile.Write("host is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"mode",&starg.mode);   // 传输模式，1-被动模式，2-主动模式，缺省采用被动模式。
  if (starg.mode!=2)  starg.mode=1;

  GetXMLBuffer(strxmlbuffer,"username",starg.username,30);   // 远程服务端ftp的用户名。
  if (strlen(starg.username)==0)
  { logfile.Write("username is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"password",starg.password,30);   // 远程服务端ftp的密码。
  if (strlen(starg.password)==0)
  { logfile.Write("password is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"remotepath",starg.remotepath,300);   // 远程服务端存放文件的目录。
  if (strlen(starg.remotepath)==0)
  { logfile.Write("remotepath is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"localpath",starg.localpath,300);   // 本地文件存放的目录。
  if (strlen(starg.localpath)==0)
  { logfile.Write("localpath is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"matchname",starg.matchname,100);   // 待上传文件匹配的规则。
  if (strlen(starg.matchname)==0)
  { logfile.Write("matchname is null.\n");  return false; }

  // 上传后客户端文件的处理方式：1-什么也不做；2-删除；3-备份。
  GetXMLBuffer(strxmlbuffer,"ptype",&starg.ptype);   
  if ( (starg.ptype!=1) && (starg.ptype!=2) && (starg.ptype!=3) )
  { logfile.Write("ptype is error.\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"localpathbak",starg.localpathbak,300); // 上传后客户端文件的备份目录。
  if ( (starg.ptype==3) && (strlen(starg.localpathbak)==0) )
  { logfile.Write("localpathbak is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"okfilename",starg.okfilename,300); // 已上传成功文件名清单。
  if ( (starg.ptype==1) && (strlen(starg.okfilename)==0) )
  { logfile.Write("okfilename is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"timeout",&starg.timeout);   // 进程心跳的超时时间。
  if (starg.timeout==0) { logfile.Write("timeout is null.\n");  return false; }

  GetXMLBuffer(strxmlbuffer,"pname",starg.pname,50);     // 进程名。
  if (strlen(starg.pname)==0) { logfile.Write("pname is null.\n");  return false; }

  return true;
}

// 把localpath目录下的文件加载到vlistfile2容器中。
bool LoadLocalFile()
{
  vlistfile2.clear();

  CDir Dir;

  Dir.SetDateFMT("yyyymmddhh24miss");

  // 不包括子目录。
  // 注意，如果本地目录下的总文件数超过10000，增量上传文件功能将有问题。
  // 建议用deletefiles程序及时清理本地目录中的历史文件。
  if (Dir.OpenDir(starg.localpath,starg.matchname)==false)
  {
    logfile.Write("Dir.OpenDir(%s) 失败。\n",starg.localpath); return false;
  }

  struct st_fileinfo stfileinfo;

  while (true)
  {
    memset(&stfileinfo,0,sizeof(struct st_fileinfo));
   
    if (Dir.ReadDir()==false) break;

    strcpy(stfileinfo.filename,Dir.m_FileName);   // 文件名，不包括目录名。
    strcpy(stfileinfo.mtime,Dir.m_ModifyTime);    // 文件时间。

    vlistfile2.push_back(stfileinfo);
  }

  return true;
}

// 加载okfilename文件中的内容到容器vlistfile1中。
bool LoadOKFile()
{
  vlistfile1.clear();

  CFile File;

  // 注意：如果程序是第一次上传，okfilename是不存在的，并不是错误，所以也返回true。
  if ( (File.Open(starg.okfilename,"r"))==false )  return true;

  char strbuffer[501];

  struct st_fileinfo stfileinfo;

  while (true)
  {
    memset(&stfileinfo,0,sizeof(struct st_fileinfo));

    if (File.Fgets(strbuffer,300,true)==false) break;

    GetXMLBuffer(strbuffer,"filename",stfileinfo.filename);
    GetXMLBuffer(strbuffer,"mtime",stfileinfo.mtime);

    vlistfile1.push_back(stfileinfo);
  }

  return true;
}

// 比较vlistfile2和vlistfile1，得到vlistfile3和vlistfile4。
bool CompVector()
{
  vlistfile3.clear(); vlistfile4.clear();

  int ii,jj;

  // 遍历vlistfile2。
  for (ii=0;ii<vlistfile2.size();ii++)
  {
    // 在vlistfile1中查找vlistfile2[ii]的记录。
    for (jj=0;jj<vlistfile1.size();jj++)
    {
      // 如果找到了，把记录放入vlistfile3。
      if ( (strcmp(vlistfile2[ii].filename,vlistfile1[jj].filename)==0) &&
           (strcmp(vlistfile2[ii].mtime,vlistfile1[jj].mtime)==0) )
      {
        vlistfile3.push_back(vlistfile2[ii]); break;
      }
    }

    // 如果没有找到，把记录放入vlistfile4。
    if (jj==vlistfile1.size()) vlistfile4.push_back(vlistfile2[ii]);
  }

  return true;
}

// 把容器vlistfile3中的内容写入okfilename文件，覆盖之前的旧okfilename文件。
bool WriteToOKFile()
{
  CFile File;    

  if (File.Open(starg.okfilename,"w")==false)
  {
    logfile.Write("File.Open(%s) failed.\n",starg.okfilename); return false;
  }

  for (int ii=0;ii<vlistfile3.size();ii++)
    File.Fprintf("<filename>%s</filename><mtime>%s</mtime>\n",vlistfile3[ii].filename,vlistfile3[ii].mtime);

  return true;
}

// 如果ptype==1，把上传成功的文件记录追加到okfilename文件中。
bool AppendToOKFile(struct st_fileinfo *stfileinfo)
{
  CFile File;

  if (File.Open(starg.okfilename,"a")==false)
  {
    logfile.Write("File.Open(%s) failed.\n",starg.okfilename); return false;
  }

  File.Fprintf("<filename>%s</filename><mtime>%s</mtime>\n",stfileinfo->filename,stfileinfo->mtime);

  return true;
}

```

##### ftp编译

makefile

```
# 开发框架头文件路径。
PUBINCL = -I/project/public

# 开发框架cpp文件名，这里直接包含进来，没有采用链接库，是为了方便调试。
PUBCPP = /project/public/_public.cpp

# 编译参数。
CFLAGS = -g

all: ftpgetfiles ftpputfiles

ftpgetfiles:ftpgetfiles.cpp
	g++ $(CFLAGS) -o ftpgetfiles ftpgetfiles.cpp $(PUBINCL) $(PUBCPP) /project/public/libftp.a /project/public/_ftp.cpp -lm -lc
	cp ftpgetfiles ../bin/.

ftpputfiles:ftpputfiles.cpp
	g++ $(CFLAGS) -o ftpputfiles ftpputfiles.cpp $(PUBINCL) $(PUBCPP) /project/public/libftp.a /project/public/_ftp.cpp -lm -lc
	cp ftpputfiles ../bin/.

clean:
	rm -f ftpgetfiles ftpputfiles

```



#### TCP上传下载

##### tcp客户端上传

tcpputfiles.cpp

```
/*
 * 程序名：tcpputfiles.cpp，采用tcp协议，实现文件上传的客户端。
*/
#include "_public.h"

// 程序运行的参数结构体。
struct st_arg
{
  int  clienttype;          // 客户端类型，1-上传文件；2-下载文件。
  char ip[31];              // 服务端的IP地址。
  int  port;                // 服务端的端口。
  int  ptype;               // 文件上传成功后本地文件的处理方式：1-删除文件；2-移动到备份目录。
  char clientpath[301];     // 本地文件存放的根目录。
  char clientpathbak[301];  // 文件成功上传后，本地文件备份的根目录，当ptype==2时有效。
  bool andchild;            // 是否上传clientpath目录下各级子目录的文件，true-是；false-否。
  char matchname[301];      // 待上传文件名的匹配规则，如"*.TXT,*.XML"。
  char srvpath[301];        // 服务端文件存放的根目录。
  int  timetvl;             // 扫描本地目录文件的时间间隔，单位：秒。
  int  timeout;             // 进程心跳的超时时间。
  char pname[51];           // 进程名，建议用"tcpputfiles_后缀"的方式。
} starg;

CLogFile logfile;

// 程序退出和信号2、15的处理函数。
void EXIT(int sig);

void _help();

// 把xml解析到参数starg结构中。
bool _xmltoarg(char *strxmlbuffer);

CTcpClient TcpClient;

bool Login(const char *argv);    // 登录业务。

bool ActiveTest();    // 心跳。

char strrecvbuffer[1024];   // 发送报文的buffer。
char strsendbuffer[1024];   // 接收报文的buffer。

// 文件上传的主函数，执行一次文件上传的任务。
bool _tcpputfiles();
bool bcontinue=true;   // 如果调用_tcpputfiles发送了文件，bcontinue为true，初始化为true。

// 把文件的内容发送给对端。
bool SendFile(const int sockfd,const char *filename,const int filesize);

// 删除或者转存本地的文件。
bool AckMessage(const char *strrecvbuffer);

CPActive PActive;  // 进程心跳。

int main(int argc,char *argv[])
{
  if (argc!=3) { _help(); return -1; }

  // 关闭全部的信号和输入输出。
  // 设置信号,在shell状态下可用 "kill + 进程号" 正常终止些进程。
  // 但请不要用 "kill -9 +进程号" 强行终止。
  CloseIOAndSignal(); signal(SIGINT,EXIT); signal(SIGTERM,EXIT);

  // 打开日志文件。
  if (logfile.Open(argv[1],"a+")==false)
  {
    printf("打开日志文件失败（%s）。\n",argv[1]); return -1;
  }

  // 解析xml，得到程序运行的参数。
  if (_xmltoarg(argv[2])==false) return -1;

  PActive.AddPInfo(starg.timeout,starg.pname);  // 把进程的心跳信息写入共享内存。

  // 向服务端发起连接请求。
  if (TcpClient.ConnectToServer(starg.ip,starg.port)==false)
  {
    logfile.Write("TcpClient.ConnectToServer(%s,%d) failed.\n",starg.ip,starg.port); EXIT(-1);
  }

  // 登录业务。
  if (Login(argv[2])==false) { logfile.Write("Login() failed.\n"); EXIT(-1); }

  while (true)
  {
    // 调用文件上传的主函数，执行一次文件上传的任务。
    if (_tcpputfiles()==false) { logfile.Write("_tcpputfiles() failed.\n"); EXIT(-1); }

    if (bcontinue==false)
    {
      sleep(starg.timetvl);

      if (ActiveTest()==false) break;
    }

    PActive.UptATime();
  }
   
  EXIT(0);
}

// 心跳。 
bool ActiveTest()    
{
  memset(strsendbuffer,0,sizeof(strsendbuffer));
  memset(strrecvbuffer,0,sizeof(strrecvbuffer));
 
  SPRINTF(strsendbuffer,sizeof(strsendbuffer),"<activetest>ok</activetest>");
  // logfile.Write("发送：%s\n",strsendbuffer);
  if (TcpClient.Write(strsendbuffer)==false) return false; // 向服务端发送请求报文。

  if (TcpClient.Read(strrecvbuffer,20)==false) return false; // 接收服务端的回应报文。
  // logfile.Write("接收：%s\n",strrecvbuffer);

  return true;
}

// 登录业务。 
bool Login(const char *argv)    
{
  memset(strsendbuffer,0,sizeof(strsendbuffer));
  memset(strrecvbuffer,0,sizeof(strrecvbuffer));
 
  SPRINTF(strsendbuffer,sizeof(strsendbuffer),"%s<clienttype>1</clienttype>",argv);
  logfile.Write("发送：%s\n",strsendbuffer);
  if (TcpClient.Write(strsendbuffer)==false) return false; // 向服务端发送请求报文。

  if (TcpClient.Read(strrecvbuffer,20)==false) return false; // 接收服务端的回应报文。
  logfile.Write("接收：%s\n",strrecvbuffer);

  logfile.Write("登录(%s:%d)成功。\n",starg.ip,starg.port); 

  return true;
}

void EXIT(int sig)
{
  logfile.Write("程序退出，sig=%d\n\n",sig);

  exit(0);
}

void _help()
{
  printf("\n");
  printf("Using:/project/tools1/bin/tcpputfiles logfilename xmlbuffer\n\n");

  printf("Sample:/project/tools1/bin/procctl 20 /project/tools1/bin/tcpputfiles /log/idc/tcpputfiles_surfdata.log \"<ip>39.106.159.62</ip><port>5005</port><ptype>1</ptype><clientpath>/tmp/tcp/surfdata1</clientpath><andchild>true</andchild><matchname>*.XML,*.CSV,*.JSON</matchname><srvpath>/tmp/tcp/surfdata2</srvpath><timetvl>10</timetvl><timeout>50</timeout><pname>tcpputfiles_surfdata</pname>\"\n");
  printf("       /project/tools1/bin/procctl 20 /project/tools1/bin/tcpputfiles /log/idc/tcpputfiles_surfdata.log \"<ip>39.106.159.62</ip><port>5005</port><ptype>2</ptype><clientpath>/tmp/tcp/surfdata1</clientpath><clientpathbak>/tmp/tcp/surfdata1bak</clientpathbak><andchild>true</andchild><matchname>*.XML,*.CSV,*.JSON</matchname><srvpath>/tmp/tcp/surfdata2</srvpath><timetvl>10</timetvl><timeout>50</timeout><pname>tcpputfiles_surfdata</pname>\"\n\n\n");

  printf("本程序是数据中心的公共功能模块，采用tcp协议把文件上传给服务端。\n");
  printf("logfilename   本程序运行的日志文件。\n");
  printf("xmlbuffer     本程序运行的参数，如下：\n");
  printf("ip            服务端的IP地址。\n");
  printf("port          服务端的端口。\n");
  printf("ptype         文件上传成功后的处理方式：1-删除文件；2-移动到备份目录。\n");
  printf("clientpath    本地文件存放的根目录。\n");
  printf("clientpathbak 文件成功上传后，本地文件备份的根目录，当ptype==2时有效。\n");
  printf("andchild      是否上传clientpath目录下各级子目录的文件，true-是；false-否，缺省为false。\n");
  printf("matchname     待上传文件名的匹配规则，如\"*.TXT,*.XML\"\n");
  printf("srvpath       服务端文件存放的根目录。\n");
  printf("timetvl       扫描本地目录文件的时间间隔，单位：秒，取值在1-30之间。\n");
  printf("timeout       本程序的超时时间，单位：秒，视文件大小和网络带宽而定，建议设置50以上。\n");
  printf("pname         进程名，尽可能采用易懂的、与其它进程不同的名称，方便故障排查。\n\n");
}

// 把xml解析到参数starg结构
bool _xmltoarg(char *strxmlbuffer)
{
  memset(&starg,0,sizeof(struct st_arg));

  GetXMLBuffer(strxmlbuffer,"ip",starg.ip);
  if (strlen(starg.ip)==0) { logfile.Write("ip is null.\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"port",&starg.port);
  if ( starg.port==0) { logfile.Write("port is null.\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"ptype",&starg.ptype);
  if ((starg.ptype!=1)&&(starg.ptype!=2)) { logfile.Write("ptype not in (1,2).\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"clientpath",starg.clientpath);
  if (strlen(starg.clientpath)==0) { logfile.Write("clientpath is null.\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"clientpathbak",starg.clientpathbak);
  if ((starg.ptype==2)&&(strlen(starg.clientpathbak)==0)) { logfile.Write("clientpathbak is null.\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"andchild",&starg.andchild);

  GetXMLBuffer(strxmlbuffer,"matchname",starg.matchname);
  if (strlen(starg.matchname)==0) { logfile.Write("matchname is null.\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"srvpath",starg.srvpath);
  if (strlen(starg.srvpath)==0) { logfile.Write("srvpath is null.\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"timetvl",&starg.timetvl);
  if (starg.timetvl==0) { logfile.Write("timetvl is null.\n"); return false; }

  // 扫描本地目录文件的时间间隔，单位：秒。
  // starg.timetvl没有必要超过30秒。
  if (starg.timetvl>30) starg.timetvl=30;

  // 进程心跳的超时时间，一定要大于starg.timetvl，没有必要小于50秒。
  GetXMLBuffer(strxmlbuffer,"timeout",&starg.timeout);
  if (starg.timeout==0) { logfile.Write("timeout is null.\n"); return false; }
  if (starg.timeout<50) starg.timeout=50;

  GetXMLBuffer(strxmlbuffer,"pname",starg.pname,50);
  if (strlen(starg.pname)==0) { logfile.Write("pname is null.\n"); return false; }

  return true;
}

// 文件上传的主函数，执行一次文件上传的任务。
bool _tcpputfiles()
{
  CDir Dir;

  // 调用OpenDir()打开starg.clientpath目录。
  if (Dir.OpenDir(starg.clientpath,starg.matchname,10000,starg.andchild)==false)
  {
    logfile.Write("Dir.OpenDir(%s) 失败。\n",starg.clientpath); return false;
  }

  int delayed=0;        // 未收到对端确认报文的文件数量。
  int buflen=0;         // 用于存放strrecvbuffer的长度。

  bcontinue=false;

  while (true)
  {
    memset(strsendbuffer,0,sizeof(strsendbuffer));
    memset(strrecvbuffer,0,sizeof(strrecvbuffer));

    // 遍历目录中的每个文件，调用ReadDir()获取一个文件名。
    if (Dir.ReadDir()==false) break;

    bcontinue=true;

    // 把文件名、修改时间、文件大小组成报文，发送给对端。
    SNPRINTF(strsendbuffer,sizeof(strsendbuffer),1000,"<filename>%s</filename><mtime>%s</mtime><size>%d</size>",Dir.m_FullFileName,Dir.m_ModifyTime,Dir.m_FileSize);

    // logfile.Write("strsendbuffer=%s\n",strsendbuffer);
    if (TcpClient.Write(strsendbuffer)==false)
    {
      logfile.Write("TcpClient.Write() failed.\n"); return false;
    }

    // 把文件的内容发送给对端。
    logfile.Write("send %s(%d) ...",Dir.m_FullFileName,Dir.m_FileSize);
    if (SendFile(TcpClient.m_connfd,Dir.m_FullFileName,Dir.m_FileSize)==true)
    {
      logfile.WriteEx("ok.\n"); delayed++;
    }
    else
    {
      logfile.WriteEx("failed.\n"); TcpClient.Close(); return false;
    }

    PActive.UptATime();

    // 接收对端的确认报文。
    while (delayed>0)
    {
      memset(strrecvbuffer,0,sizeof(strrecvbuffer));
      if (TcpRead(TcpClient.m_connfd,strrecvbuffer,&buflen,-1)==false) break;
      // logfile.Write("strrecvbuffer=%s\n",strrecvbuffer);

      // 删除或者转存本地的文件。
      delayed--;
      AckMessage(strrecvbuffer);
    }
  }

  // 继续接收对端的确认报文。
  while (delayed>0)
  {
    memset(strrecvbuffer,0,sizeof(strrecvbuffer));
    if (TcpRead(TcpClient.m_connfd,strrecvbuffer,&buflen,10)==false) break;
    // logfile.Write("strrecvbuffer=%s\n",strrecvbuffer);

    // 删除或者转存本地的文件。
    delayed--;
    AckMessage(strrecvbuffer);
  }

  return true;
}


// 把文件的内容发送给对端。
bool SendFile(const int sockfd,const char *filename,const int filesize)
{
  int  onread=0;        // 每次调用fread时打算读取的字节数。 
  int  bytes=0;         // 调用一次fread从文件中读取的字节数。
  char buffer[1000];    // 存放读取数据的buffer。
  int  totalbytes=0;    // 从文件中已读取的字节总数。
  FILE *fp=NULL;

  // 以"rb"的模式打开文件。
  if ( (fp=fopen(filename,"rb"))==NULL )  return false;

  while (true)
  {
    memset(buffer,0,sizeof(buffer));

    // 计算本次应该读取的字节数，如果剩余的数据超过1000字节，就打算读1000字节。
    if (filesize-totalbytes>1000) onread=1000;
    else onread=filesize-totalbytes;

    // 从文件中读取数据。
    bytes=fread(buffer,1,onread,fp);

    // 把读取到的数据发送给对端。
    if (bytes>0)
    {
      if (Writen(sockfd,buffer,bytes)==false) { fclose(fp); return false; }
    }

    // 计算文件已读取的字节总数，如果文件已读完，跳出循环。
    totalbytes=totalbytes+bytes;

    if (totalbytes==filesize) break;
  }

  fclose(fp);

  return true;
}

// 删除或者转存本地的文件。
bool AckMessage(const char *strrecvbuffer)
{
  char filename[301];
  char result[11];

  memset(filename,0,sizeof(filename));
  memset(result,0,sizeof(result));

  GetXMLBuffer(strrecvbuffer,"filename",filename,300);
  GetXMLBuffer(strrecvbuffer,"result",result,10);

  // 如果服务端接收文件不成功，直接返回。
  if (strcmp(result,"ok")!=0) return true;

  // ptype==1，删除文件。
  if (starg.ptype==1)
  {
    if (REMOVE(filename)==false) { logfile.Write("REMOVE(%s) failed.\n",filename); return false; }
  }
  
  // ptype==2，移动到备份目录。
  if (starg.ptype==2)
  {
    // 生成转存后的备份目录文件名。
    char bakfilename[301];
    STRCPY(bakfilename,sizeof(bakfilename),filename);
    UpdateStr(bakfilename,starg.clientpath,starg.clientpathbak,false);
    if (RENAME(filename,bakfilename)==false) 
    { logfile.Write("RENAME(%s,%s) failed.\n",filename,bakfilename); return false; }
  }

  return true;
}
```



##### tcp客户端下载

tcpgetfiles.cpp

```
/*
 * 程序名：tcpgetfiles.cpp，采用tcp协议，实现文件下载的客户端。
*/
#include "_public.h"

// 程序运行的参数结构体。
struct st_arg
{
  int  clienttype;          // 客户端类型，1-上传文件；2-下载文件。
  char ip[31];              // 服务端的IP地址。
  int  port;                // 服务端的端口。
  int  ptype;               // 文件下载成功后服务端文件的处理方式：1-删除文件；2-移动到备份目录。
  char srvpath[301];        // 服务端文件存放的根目录。
  char srvpathbak[301];     // 文件成功下载后，服务端文件备份的根目录，当ptype==2时有效。
  bool andchild;            // 是否下载srvpath目录下各级子目录的文件，true-是；false-否。
  char matchname[301];      // 待下载文件名的匹配规则，如"*.TXT,*.XML"。
  char clientpath[301];     // 客户端文件存放的根目录。
  int  timetvl;             // 扫描服务端目录文件的时间间隔，单位：秒。
  int  timeout;             // 进程心跳的超时时间。
  char pname[51];           // 进程名，建议用"tcpgetfiles_后缀"的方式。
} starg;

CLogFile logfile;

// 程序退出和信号2、15的处理函数。
void EXIT(int sig);

void _help();

// 把xml解析到参数starg结构中。
bool _xmltoarg(char *strxmlbuffer);

CTcpClient TcpClient;

bool Login(const char *argv);    // 登录业务。

char strrecvbuffer[1024];   // 发送报文的buffer。
char strsendbuffer[1024];   // 接收报文的buffer。

// 文件下载的主函数。
void _tcpgetfiles();

// 接收文件的内容。
bool RecvFile(const int sockfd,const char *filename,const char *mtime,int filesize);

CPActive PActive;  // 进程心跳。

int main(int argc,char *argv[])
{
  if (argc!=3) { _help(); return -1; }

  // 关闭全部的信号和输入输出。
  // 设置信号,在shell状态下可用 "kill + 进程号" 正常终止些进程。
  // 但请不要用 "kill -9 +进程号" 强行终止。
  CloseIOAndSignal(); signal(SIGINT,EXIT); signal(SIGTERM,EXIT);

  // 打开日志文件。
  if (logfile.Open(argv[1],"a+")==false)
  {
    printf("打开日志文件失败（%s）。\n",argv[1]); return -1;
  }

  // 解析xml，得到程序运行的参数。
  if (_xmltoarg(argv[2])==false) return -1;

  PActive.AddPInfo(starg.timeout,starg.pname);  // 把进程的心跳信息写入共享内存。

  // 向服务端发起连接请求。
  if (TcpClient.ConnectToServer(starg.ip,starg.port)==false)
  {
    logfile.Write("TcpClient.ConnectToServer(%s,%d) failed.\n",starg.ip,starg.port); EXIT(-1);
  }

  // 登录业务。
  if (Login(argv[2])==false) { logfile.Write("Login() failed.\n"); EXIT(-1); }

  // 调用文件下载的主函数。
  _tcpgetfiles();

  EXIT(0);
}


// 登录业务。 
bool Login(const char *argv)    
{
  memset(strsendbuffer,0,sizeof(strsendbuffer));
  memset(strrecvbuffer,0,sizeof(strrecvbuffer));
 
  SPRINTF(strsendbuffer,sizeof(strsendbuffer),"%s<clienttype>2</clienttype>",argv);
  logfile.Write("发送：%s\n",strsendbuffer);
  if (TcpClient.Write(strsendbuffer)==false) return false; // 向服务端发送请求报文。

  if (TcpClient.Read(strrecvbuffer,20)==false) return false; // 接收服务端的回应报文。
  logfile.Write("接收：%s\n",strrecvbuffer);

  logfile.Write("登录(%s:%d)成功。\n",starg.ip,starg.port); 

  return true;
}

void EXIT(int sig)
{
  logfile.Write("程序退出，sig=%d\n\n",sig);

  exit(0);
}

void _help()
{
  printf("\n");
  printf("Using:/project/tools1/bin/tcpgetfiles logfilename xmlbuffer\n\n");

  printf("Sample:/project/tools1/bin/procctl 20 /project/tools1/bin/tcpgetfiles /log/idc/tcpgetfiles_surfdata.log \"<ip>192.168.174.132</ip><port>5005</port><ptype>1</ptype><srvpath>/tmp/tcp/surfdata2</srvpath><andchild>true</andchild><matchname>*.XML,*.CSV,*.JSON</matchname><clientpath>/tmp/tcp/surfdata3</clientpath><timetvl>10</timetvl><timeout>50</timeout><pname>tcpgetfiles_surfdata</pname>\"\n");
  printf("       /project/tools1/bin/procctl 20 /project/tools1/bin/tcpgetfiles /log/idc/tcpgetfiles_surfdata.log \"<ip>192.168.174.132</ip><port>5005</port><ptype>2</ptype><srvpath>/tmp/tcp/surfdata2</srvpath><srvpathbak>/tmp/tcp/surfdata2bak</srvpathbak><andchild>true</andchild><matchname>*.XML,*.CSV,*.JSON</matchname><clientpath>/tmp/tcp/surfdata3</clientpath><timetvl>10</timetvl><timeout>50</timeout><pname>tcpgetfiles_surfdata</pname>\"\n\n\n");

  printf("本程序是数据中心的公共功能模块，采用tcp协议从服务端下载文件。\n");
  printf("logfilename   本程序运行的日志文件。\n");
  printf("xmlbuffer     本程序运行的参数，如下：\n");
  printf("ip            服务端的IP地址。\n");
  printf("port          服务端的端口。\n");
  printf("ptype         文件下载成功后服务端文件的处理方式：1-删除文件；2-移动到备份目录。\n");
  printf("srvpath       服务端文件存放的根目录。\n");
  printf("srvpathbak    文件成功下载后，服务端文件备份的根目录，当ptype==2时有效。\n");
  printf("andchild      是否下载srvpath目录下各级子目录的文件，true-是；false-否，缺省为false。\n");
  printf("matchname     待下载文件名的匹配规则，如\"*.TXT,*.XML\"\n");
  printf("clientpath    客户端文件存放的根目录。\n");
  printf("timetvl       扫描服务目录文件的时间间隔，单位：秒，取值在1-30之间。\n");
  printf("timeout       本程序的超时时间，单位：秒，视文件大小和网络带宽而定，建议设置50以上。\n");
  printf("pname         进程名，尽可能采用易懂的、与其它进程不同的名称，方便故障排查。\n\n");
}

// 把xml解析到参数starg结构
bool _xmltoarg(char *strxmlbuffer)
{
  memset(&starg,0,sizeof(struct st_arg));

  GetXMLBuffer(strxmlbuffer,"ip",starg.ip);
  if (strlen(starg.ip)==0) { logfile.Write("ip is null.\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"port",&starg.port);
  if ( starg.port==0) { logfile.Write("port is null.\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"ptype",&starg.ptype);
  if ((starg.ptype!=1)&&(starg.ptype!=2)) { logfile.Write("ptype not in (1,2).\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"srvpath",starg.srvpath);
  if (strlen(starg.srvpath)==0) { logfile.Write("srvpath is null.\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"srvpathbak",starg.srvpathbak);
  if ((starg.ptype==2)&&(strlen(starg.srvpathbak)==0)) { logfile.Write("srvpathbak is null.\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"andchild",&starg.andchild);

  GetXMLBuffer(strxmlbuffer,"matchname",starg.matchname);
  if (strlen(starg.matchname)==0) { logfile.Write("matchname is null.\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"clientpath",starg.clientpath);
  if (strlen(starg.clientpath)==0) { logfile.Write("clientpath is null.\n"); return false; }

  GetXMLBuffer(strxmlbuffer,"timetvl",&starg.timetvl);
  if (starg.timetvl==0) { logfile.Write("timetvl is null.\n"); return false; }

  // 扫描服务端目录文件的时间间隔，单位：秒。
  // starg.timetvl没有必要超过30秒。
  if (starg.timetvl>30) starg.timetvl=30;

  // 进程心跳的超时时间，一定要大于starg.timetvl，没有必要小于50秒。
  GetXMLBuffer(strxmlbuffer,"timeout",&starg.timeout);
  if (starg.timeout==0) { logfile.Write("timeout is null.\n"); return false; }
  if (starg.timeout<50) starg.timeout=50;

  GetXMLBuffer(strxmlbuffer,"pname",starg.pname,50);
  if (strlen(starg.pname)==0) { logfile.Write("pname is null.\n"); return false; }

  return true;
}

// 文件下载的主函数。
void _tcpgetfiles()
{
  PActive.AddPInfo(starg.timeout,starg.pname);

  while (true)
  {
    memset(strsendbuffer,0,sizeof(strsendbuffer));
    memset(strrecvbuffer,0,sizeof(strrecvbuffer));

    PActive.UptATime();

    // 接收服务端的报文。
    // 第二个参数的取值必须大于starg.timetvl，小于starg.timeout。
    if (TcpClient.Read(strrecvbuffer,starg.timetvl+10)==false)
    {
      logfile.Write("TcpClient.Read() failed.\n"); return;
    }
    // logfile.Write("strrecvbuffer=%s\n",strrecvbuffer);

    // 处理心跳报文。
    if (strcmp(strrecvbuffer,"<activetest>ok</activetest>")==0)
    {
      strcpy(strsendbuffer,"ok");
      // logfile.Write("strsendbuffer=%s\n",strsendbuffer);
      if (TcpClient.Write(strsendbuffer)==false)
      {
        logfile.Write("TcpClient.Write() failed.\n"); return;
      }
    }

    // 处理下载文件的请求报文。
    if (strncmp(strrecvbuffer,"<filename>",10)==0)
    {
      // 解析下载文件请求报文的xml。
      char serverfilename[301];  memset(serverfilename,0,sizeof(serverfilename));
      char mtime[21];            memset(mtime,0,sizeof(mtime));
      int  filesize=0;
      GetXMLBuffer(strrecvbuffer,"filename",serverfilename,300);
      GetXMLBuffer(strrecvbuffer,"mtime",mtime,19);
      GetXMLBuffer(strrecvbuffer,"size",&filesize);

      // 客户端和服务端文件的目录是不一样的，以下代码生成客户端的文件名。
      // 把文件名中的srvpath替换成clientpath，要小心第三个参数
      char clientfilename[301];  memset(clientfilename,0,sizeof(clientfilename));
      strcpy(clientfilename,serverfilename);
      UpdateStr(clientfilename,starg.srvpath,starg.clientpath,false);

      // 接收文件的内容。
      logfile.Write("recv %s(%d) ...",clientfilename,filesize);
      if (RecvFile(TcpClient.m_connfd,clientfilename,mtime,filesize)==true)
      {
        logfile.WriteEx("ok.\n");
        SNPRINTF(strsendbuffer,sizeof(strsendbuffer),1000,"<filename>%s</filename><result>ok</result>",serverfilename);
      }
      else
      {
        logfile.WriteEx("failed.\n");
        SNPRINTF(strsendbuffer,sizeof(strsendbuffer),1000,"<filename>%s</filename><result>failed</result>",serverfilename);
      }

      // 把接收结果返回给对端。
      // logfile.Write("strsendbuffer=%s\n",strsendbuffer);
      if (TcpClient.Write(strsendbuffer)==false)
      {
        logfile.Write("TcpClient.Write() failed.\n"); return;
      }
    }
  }
}

// 接收文件的内容。
bool RecvFile(const int sockfd,const char *filename,const char *mtime,int filesize)
{
  // 生成临时文件名。
  char strfilenametmp[301];
  SNPRINTF(strfilenametmp,sizeof(strfilenametmp),300,"%s.tmp",filename);

  int  totalbytes=0;        // 已接收文件的总字节数。
  int  onread=0;            // 本次打算接收的字节数。
  char buffer[1000];        // 接收文件内容的缓冲区。
  FILE *fp=NULL;

  // 创建临时文件。
  if ( (fp=FOPEN(strfilenametmp,"wb"))==NULL ) return false;

  while (true)
  {
    memset(buffer,0,sizeof(buffer));

    // 计算本次应该接收的字节数。
    if (filesize-totalbytes>1000) onread=1000;
    else onread=filesize-totalbytes;

    // 接收文件内容。
    if (Readn(sockfd,buffer,onread)==false) { fclose(fp); return false; }

    // 把接收到的内容写入文件。
    fwrite(buffer,1,onread,fp);

    // 计算已接收文件的总字节数，如果文件接收完，跳出循环。
    totalbytes=totalbytes+onread;

    if (totalbytes==filesize) break;
  }

  // 关闭临时文件。
  fclose(fp);

  // 重置文件的时间。
  UTime(strfilenametmp,mtime);

  // 把临时文件RENAME为正式的文件。
  if (RENAME(strfilenametmp,filename)==false) return false;

  return true;
}
```





##### tcp服务端

fileserver.cpp

```
/*
 * 程序名：fileserver.cpp，文件传输的服务端。
*/
#include "_public.h"

// 程序运行的参数结构体。
struct st_arg
{
  int  clienttype;          // 客户端类型，1-上传文件；2-下载文件。
  char ip[31];              // 服务端的IP地址。
  int  port;                // 服务端的端口。
  int  ptype;               // 文件成功传输后的处理方式：1-删除文件；2-移动到备份目录。
  char clientpath[301];     // 客户端文件存放的根目录。
  bool andchild;            // 是否传输各级子目录的文件，true-是；false-否。
  char matchname[301];      // 待传输文件名的匹配规则，如"*.TXT,*.XML"。
  char srvpath[301];        // 服务端文件存放的根目录。
  char srvpathbak[301];     // 文件成功下载后，服务端文件备份的根目录，当ptype==2时有效。
  int  timetvl;             // 扫描目录文件的时间间隔，单位：秒。
  int  timeout;             // 进程心跳的超时时间。
  char pname[51];           // 进程名，建议用"tcpgetfiles_后缀"的方式。
} starg;

// 把xml解析到参数starg结构中。
bool _xmltoarg(char *strxmlbuffer);

CLogFile logfile;      // 服务程序的运行日志。
CTcpServer TcpServer;  // 创建服务端对象。

void FathEXIT(int sig);  // 父进程退出函数。
void ChldEXIT(int sig);  // 子进程退出函数。

bool ActiveTest();    // 心跳。

char strrecvbuffer[1024];   // 发送报文的buffer。
char strsendbuffer[1024];   // 接收报文的buffer。

// 文件下载的主函数，执行一次文件下载的任务。
bool _tcpputfiles();
bool bcontinue=true;   // 如果调用_tcpputfiles发送了文件，bcontinue为true，初始化为true。

// 把文件的内容发送给对端。
bool SendFile(const int sockfd,const char *filename,const int filesize);

// 删除或者转存本地的文件。
bool AckMessage(const char *strrecvbuffer);

// 登录业务处理函数。
bool ClientLogin();

// 上传文件的主函数。
void RecvFilesMain();

// 下载文件的主函数。
void SendFilesMain();

// 接收文件的内容。
bool RecvFile(const int sockfd,const char *filename,const char *mtime,int filesize);

CPActive PActive;  // 进程心跳。

int main(int argc,char *argv[])
{
  if (argc!=3)
  {
    printf("Using:./fileserver port logfile\n");
    printf("Example:./fileserver 5005 /log/idc/fileserver.log\n"); 
    printf("         /project/tools1/bin/procctl 10 /project/tools1/bin/fileserver 5005 /log/idc/fileserver.log\n\n\n"); 
    return -1;
  }

  // 关闭全部的信号和输入输出。
  // 设置信号,在shell状态下可用 "kill + 进程号" 正常终止些进程
  // 但请不要用 "kill -9 +进程号" 强行终止
  CloseIOAndSignal(); signal(SIGINT,FathEXIT); signal(SIGTERM,FathEXIT);

  if (logfile.Open(argv[2],"a+")==false) { printf("logfile.Open(%s) failed.\n",argv[2]); return -1; }

  // 服务端初始化。
  if (TcpServer.InitServer(atoi(argv[1]))==false)
  {
    logfile.Write("TcpServer.InitServer(%s) failed.\n",argv[1]); return -1;
  }

  while (true)
  {
    // 等待客户端的连接请求。
    if (TcpServer.Accept()==false)
    {
      logfile.Write("TcpServer.Accept() failed.\n"); FathEXIT(-1);
    }

    logfile.Write("客户端（%s）已连接。\n",TcpServer.GetIP());

    if (fork()>0) { TcpServer.CloseClient(); continue; }  // 父进程继续回到Accept()。
   
    // 子进程重新设置退出信号。
    signal(SIGINT,ChldEXIT); signal(SIGTERM,ChldEXIT);

    TcpServer.CloseListen();

    // 子进程与客户端进行通讯，处理业务。

    // 处理登录客户端的登录报文。
    if (ClientLogin()==false) ChldEXIT(-1);

    // 如果clienttype==1，调用上传文件的主函数。
    if (starg.clienttype==1) RecvFilesMain();

    // 如果clienttype==2，调用下载文件的主函数。
    if (starg.clienttype==2) SendFilesMain();

    ChldEXIT(0);
  }
}

// 父进程退出函数。
void FathEXIT(int sig)  
{
  // 以下代码是为了防止信号处理函数在执行的过程中被信号中断。
  signal(SIGINT,SIG_IGN); signal(SIGTERM,SIG_IGN);

  logfile.Write("父进程退出，sig=%d。\n",sig);

  TcpServer.CloseListen();    // 关闭监听的socket。

  kill(0,15);     // 通知全部的子进程退出。

  exit(0);
}

// 子进程退出函数。
void ChldEXIT(int sig)  
{
  // 以下代码是为了防止信号处理函数在执行的过程中被信号中断。
  signal(SIGINT,SIG_IGN); signal(SIGTERM,SIG_IGN);

  logfile.Write("子进程退出，sig=%d。\n",sig);

  TcpServer.CloseClient();    // 关闭客户端的socket。

  exit(0);
}

// 登录。
bool ClientLogin()
{
  memset(strrecvbuffer,0,sizeof(strrecvbuffer));
  memset(strsendbuffer,0,sizeof(strsendbuffer));

  if (TcpServer.Read(strrecvbuffer,20)==false)
  {
    logfile.Write("TcpServer.Read() failed.\n"); return false;
  }
  logfile.Write("strrecvbuffer=%s\n",strrecvbuffer);

  // 解析客户端登录报文。
  _xmltoarg(strrecvbuffer);

  if ( (starg.clienttype!=1) && (starg.clienttype!=2) )
    strcpy(strsendbuffer,"failed");
  else
    strcpy(strsendbuffer,"ok");

  if (TcpServer.Write(strsendbuffer)==false)
  {
    logfile.Write("TcpServer.Write() failed.\n"); return false;
  }

  logfile.Write("%s login %s.\n",TcpServer.GetIP(),strsendbuffer);
  
  return true;
}

// 把xml解析到参数starg结构中
bool _xmltoarg(char *strxmlbuffer)
{
  memset(&starg,0,sizeof(struct st_arg));

  // 不需要对参数做合法性判断，客户端已经判断过了。
  GetXMLBuffer(strxmlbuffer,"clienttype",&starg.clienttype);
  GetXMLBuffer(strxmlbuffer,"ptype",&starg.ptype);
  GetXMLBuffer(strxmlbuffer,"clientpath",starg.clientpath);
  GetXMLBuffer(strxmlbuffer,"andchild",&starg.andchild);
  GetXMLBuffer(strxmlbuffer,"matchname",starg.matchname);
  GetXMLBuffer(strxmlbuffer,"srvpath",starg.srvpath);
  GetXMLBuffer(strxmlbuffer,"srvpathbak",starg.srvpathbak);

  GetXMLBuffer(strxmlbuffer,"timetvl",&starg.timetvl);
  if (starg.timetvl>30) starg.timetvl=30;

  GetXMLBuffer(strxmlbuffer,"timeout",&starg.timeout);
  if (starg.timeout<50) starg.timeout=50;

  GetXMLBuffer(strxmlbuffer,"pname",starg.pname,50);
  strcat(starg.pname,"_srv");

  return true;
}

// 上传文件的主函数。
void RecvFilesMain()
{
  PActive.AddPInfo(starg.timeout,starg.pname);

  while (true)
  {
    memset(strsendbuffer,0,sizeof(strsendbuffer));
    memset(strrecvbuffer,0,sizeof(strrecvbuffer));

    PActive.UptATime();

    // 接收客户端的报文。
    // 第二个参数的取值必须大于starg.timetvl，小于starg.timeout。
    if (TcpServer.Read(strrecvbuffer,starg.timetvl+10)==false)
    {
      logfile.Write("TcpServer.Read() failed.\n"); return;
    }
    // logfile.Write("strrecvbuffer=%s\n",strrecvbuffer);

    // 处理心跳报文。
    if (strcmp(strrecvbuffer,"<activetest>ok</activetest>")==0)
    {
      strcpy(strsendbuffer,"ok");
      // logfile.Write("strsendbuffer=%s\n",strsendbuffer);
      if (TcpServer.Write(strsendbuffer)==false)
      {
        logfile.Write("TcpServer.Write() failed.\n"); return;
      }
    }

    // 处理上传文件的请求报文。
    if (strncmp(strrecvbuffer,"<filename>",10)==0)
    {
      // 解析上传文件请求报文的xml。
      char clientfilename[301];  memset(clientfilename,0,sizeof(clientfilename));
      char mtime[21];            memset(mtime,0,sizeof(mtime));
      int  filesize=0;
      GetXMLBuffer(strrecvbuffer,"filename",clientfilename,300);
      GetXMLBuffer(strrecvbuffer,"mtime",mtime,19);
      GetXMLBuffer(strrecvbuffer,"size",&filesize);

      // 客户端和服务端文件的目录是不一样的，以下代码生成服务端的文件名。
      // 把文件名中的clientpath替换成srvpath，要小心第三个参数
      char serverfilename[301];  memset(serverfilename,0,sizeof(serverfilename));
      strcpy(serverfilename,clientfilename);
      UpdateStr(serverfilename,starg.clientpath,starg.srvpath,false);

      // 接收文件的内容。
      logfile.Write("recv %s(%d) ...",serverfilename,filesize);
      if (RecvFile(TcpServer.m_connfd,serverfilename,mtime,filesize)==true)
      {
        logfile.WriteEx("ok.\n");
        SNPRINTF(strsendbuffer,sizeof(strsendbuffer),1000,"<filename>%s</filename><result>ok</result>",clientfilename);
      }
      else
      {
        logfile.WriteEx("failed.\n");
        SNPRINTF(strsendbuffer,sizeof(strsendbuffer),1000,"<filename>%s</filename><result>failed</result>",clientfilename);
      }

      // 把接收结果返回给对端。
      // logfile.Write("strsendbuffer=%s\n",strsendbuffer);
      if (TcpServer.Write(strsendbuffer)==false)
      {
        logfile.Write("TcpServer.Write() failed.\n"); return;
      }
    }
  }
}

// 接收文件的内容。
bool RecvFile(const int sockfd,const char *filename,const char *mtime,int filesize)
{
  // 生成临时文件名。
  char strfilenametmp[301];
  SNPRINTF(strfilenametmp,sizeof(strfilenametmp),300,"%s.tmp",filename);

  int  totalbytes=0;        // 已接收文件的总字节数。
  int  onread=0;            // 本次打算接收的字节数。
  char buffer[1000];        // 接收文件内容的缓冲区。
  FILE *fp=NULL;

  // 创建临时文件。
  if ( (fp=FOPEN(strfilenametmp,"wb"))==NULL ) return false;

  while (true)
  {
    memset(buffer,0,sizeof(buffer));

    // 计算本次应该接收的字节数。
    if (filesize-totalbytes>1000) onread=1000;
    else onread=filesize-totalbytes;

    // 接收文件内容。
    if (Readn(sockfd,buffer,onread)==false) { fclose(fp); return false; }

    // 把接收到的内容写入文件。
    fwrite(buffer,1,onread,fp);

    // 计算已接收文件的总字节数，如果文件接收完，跳出循环。
    totalbytes=totalbytes+onread;

    if (totalbytes==filesize) break;
  }

  // 关闭临时文件。
  fclose(fp);

  // 重置文件的时间。
  UTime(strfilenametmp,mtime);

  // 把临时文件RENAME为正式的文件。
  if (RENAME(strfilenametmp,filename)==false) return false;

  return true;
}

// 下载文件的主函数。
void SendFilesMain()
{
  PActive.AddPInfo(starg.timeout,starg.pname);

  while (true)
  {
    // 调用文件下载的主函数，执行一次文件下载的任务。
    if (_tcpputfiles()==false) { logfile.Write("_tcpputfiles() failed.\n"); return; }

    if (bcontinue==false)
    {
      sleep(starg.timetvl);

      if (ActiveTest()==false) break;
    }

    PActive.UptATime();
  }
}

// 心跳。
bool ActiveTest()
{
  memset(strsendbuffer,0,sizeof(strsendbuffer));
  memset(strrecvbuffer,0,sizeof(strrecvbuffer));

  SPRINTF(strsendbuffer,sizeof(strsendbuffer),"<activetest>ok</activetest>");
  // logfile.Write("发送：%s\n",strsendbuffer);
  if (TcpServer.Write(strsendbuffer)==false) return false; // 向服务端发送请求报文。

  if (TcpServer.Read(strrecvbuffer,20)==false) return false; // 接收服务端的回应报文。
  // logfile.Write("接收：%s\n",strrecvbuffer);

  return true;
}

// 文件下载的主函数，执行一次文件下载的任务。
bool _tcpputfiles()
{
  CDir Dir;

  // 调用OpenDir()打开starg.srvpath目录。
  if (Dir.OpenDir(starg.srvpath,starg.matchname,10000,starg.andchild)==false)
  {
    logfile.Write("Dir.OpenDir(%s) 失败。\n",starg.srvpath); return false;
  }

  int delayed=0;        // 未收到对端确认报文的文件数量。
  int buflen=0;         // 用于存放strrecvbuffer的长度。

  bcontinue=false;

  while (true)
  {
    memset(strsendbuffer,0,sizeof(strsendbuffer));
    memset(strrecvbuffer,0,sizeof(strrecvbuffer));

    // 遍历目录中的每个文件，调用ReadDir()获取一个文件名。
    if (Dir.ReadDir()==false) break;

    bcontinue=true;

    // 把文件名、修改时间、文件大小组成报文，发送给对端。
    SNPRINTF(strsendbuffer,sizeof(strsendbuffer),1000,"<filename>%s</filename><mtime>%s</mtime><size>%d</size>",Dir.m_FullFileName,Dir.m_ModifyTime,Dir.m_FileSize);

    // logfile.Write("strsendbuffer=%s\n",strsendbuffer);
    if (TcpServer.Write(strsendbuffer)==false)
    {
      logfile.Write("TcpServer.Write() failed.\n"); return false;
    }

    // 把文件的内容发送给对端。
    logfile.Write("send %s(%d) ...",Dir.m_FullFileName,Dir.m_FileSize);
    if (SendFile(TcpServer.m_connfd,Dir.m_FullFileName,Dir.m_FileSize)==true)
    {
      logfile.WriteEx("ok.\n"); delayed++;
    }
    else
    {
      logfile.WriteEx("failed.\n"); TcpServer.CloseClient(); return false;
    }

    PActive.UptATime();

    // 接收对端的确认报文。
    while (delayed>0)
    {
      memset(strrecvbuffer,0,sizeof(strrecvbuffer));
      if (TcpRead(TcpServer.m_connfd,strrecvbuffer,&buflen,-1)==false) break;
      // logfile.Write("strrecvbuffer=%s\n",strrecvbuffer);

      // 删除或者转存本地的文件。
      delayed--;
      AckMessage(strrecvbuffer);
    }
  }

  // 继续接收对端的确认报文。
  while (delayed>0)
  {
    memset(strrecvbuffer,0,sizeof(strrecvbuffer));
    if (TcpRead(TcpServer.m_connfd,strrecvbuffer,&buflen,10)==false) break;
    // logfile.Write("strrecvbuffer=%s\n",strrecvbuffer);

    // 删除或者转存本地的文件。
    delayed--;
    AckMessage(strrecvbuffer);
  }

  return true;
}

// 把文件的内容发送给对端。
bool SendFile(const int sockfd,const char *filename,const int filesize)
{
  int  onread=0;        // 每次调用fread时打算读取的字节数。
  int  bytes=0;         // 调用一次fread从文件中读取的字节数。
  char buffer[1000];    // 存放读取数据的buffer。
  int  totalbytes=0;    // 从文件中已读取的字节总数。
  FILE *fp=NULL;

  // 以"rb"的模式打开文件。
  if ( (fp=fopen(filename,"rb"))==NULL )  return false;

  while (true)
  {
    memset(buffer,0,sizeof(buffer));

    // 计算本次应该读取的字节数，如果剩余的数据超过1000字节，就打算读1000字节。
    if (filesize-totalbytes>1000) onread=1000;
    else onread=filesize-totalbytes;

    // 从文件中读取数据。
    bytes=fread(buffer,1,onread,fp);

    // 把读取到的数据发送给对端。
    if (bytes>0)
    {
      if (Writen(sockfd,buffer,bytes)==false) { fclose(fp); return false; }
    }

    // 计算文件已读取的字节总数，如果文件已读完，跳出循环。
    totalbytes=totalbytes+bytes;

    if (totalbytes==filesize) break;
  }

  fclose(fp);

  return true;
}

// 删除或者转存本地的文件。
bool AckMessage(const char *strrecvbuffer)
{
  char filename[301];
  char result[11];

  memset(filename,0,sizeof(filename));
  memset(result,0,sizeof(result));

  GetXMLBuffer(strrecvbuffer,"filename",filename,300);
  GetXMLBuffer(strrecvbuffer,"result",result,10);

  // 如果服务端接收文件不成功，直接返回。
  if (strcmp(result,"ok")!=0) return true;

  // ptype==1，删除文件。
  if (starg.ptype==1)
  {
    if (REMOVE(filename)==false) { logfile.Write("REMOVE(%s) failed.\n",filename); return false; }
  }

  // ptype==2，移动到备份目录。
  if (starg.ptype==2)
  {
    // 生成转存后的备份目录文件名。
    char bakfilename[301];
    STRCPY(bakfilename,sizeof(bakfilename),filename);
    UpdateStr(bakfilename,starg.srvpath,starg.srvpathbak,false);
    if (RENAME(filename,bakfilename)==false)
    { logfile.Write("RENAME(%s,%s) failed.\n",filename,bakfilename); return false; }
  }

  return true;
}
```





##### tcp编译

makefile

```
# 开发框架头文件路径。
PUBINCL = -I/project/public

# 开发框架cpp文件名，这里直接包含进来，没有采用链接库，是为了方便调试。
PUBCPP = /project/public/_public.cpp

# 编译参数。
CFLAGS = -g

all:tcpputfiles fileserver tcpgetfiles 

tcpputfiles:tcpputfiles.cpp
	g++ $(CFLAGS) -o tcpputfiles tcpputfiles.cpp $(PUBINCL) $(PUBCPP) -lm -lc
	cp tcpputfiles ../bin/.

fileserver:fileserver.cpp
	g++ $(CFLAGS) -o fileserver fileserver.cpp $(PUBINCL) $(PUBCPP) -lm -lc
	cp fileserver ../bin/.

tcpgetfiles:tcpgetfiles.cpp
	g++ $(CFLAGS) -o tcpgetfiles tcpgetfiles.cpp $(PUBINCL) $(PUBCPP) -lm -lc
	cp tcpgetfiles ../bin/.

clean:
	rm -f tcpputfiles fileserver tcpgetfiles 

```

#### mysql数据库开发

##### 查看修改数据库字符集

```
mysql -u root -p

SHOW VARIABLES LIKE 'character%';

quit

vi /etc/my.cnf

[mysqld]
character-set-server=utf8
 
[client]
default-character-set=utf8
```

###### A. 查看修改数据库的字符集方式

```
-- database_name 为数据库名称
SHOW CREATE DATABASE database_name;

-- database_name 为数据库名称
-- utf8为目标字符编码
ALTER DATABSE database_name DEFAULT CHARACTER SET utf8;
```

###### B. 查看修改表的字符集方式

```
-- table_name为表的名称
SHOW CREATE TABLE table_name;

-- table_name为表的名称
-- utf8为目标字符编码
ALTER TABLE table_name DEFAULT CHARACTER SET utf8;
```

###### C. 查看字段的字符集方式

```
-- column_name为字段名称
SHOW FULL COLUMNS FROM column_name;

-- table_name为表的名称
-- column_name为字段名称
-- varchar(20)为字段的类型
-- utf8为目标字符集
ALTER TABLE table_name CHANGE column_name column_name VARCHAR(20) CHARACTER SET utf8;
```

###### D. 同时修改表和表中所有字符类型的字段字符集方式

```
-- 例子：alter table user2 convert to character set utf8 collate utf8_general_ci;
ALTER TABLE tbl_name CONVERT TO CHARACTER SET character_name [COLLATE ...]
```

##### 创建表

createtable.cpp

```
/*
 *  程序名：createtable.cpp，此程序演示开发框架操作MySQL数据库（创建表）。
*/

#include "_mysql.h"       // 开发框架操作MySQL的头文件。

int main(int argc,char *argv[])
{
  connection conn;   // 数据库连接类。

  // 登录数据库，返回值：0-成功；其它是失败，存放了MySQL的错误代码。
  // 失败代码在conn.m_cda.rc中，失败描述在conn.m_cda.message中。
  if (conn.connecttodb("127.0.0.1,root,phcQdNaiZ.g2,mysql,3306","utf8")!=0)
  {
    printf("connect database failed.\n%s\n",conn.m_cda.message); return -1;
  }
  
  sqlstatement stmt(&conn);  // 操作SQL语句的对象。

  // 准备创建表的SQL语句。
  // 超女表girls，超女编号id，超女姓名name，体重weight，报名时间btime，超女说明memo，超女图片pic。
  stmt.prepare("create table girls(id      bigint(10),\
                   name    varchar(30),\
                   weight  decimal(8,2),\
                   btime   datetime,\
                   memo    longtext,\
                   pic     longblob,\
                   primary key (id))");
  /*
  1、int prepare(const char *fmt,...)，SQL语句可以多行书写。
  2、SQL语句最后的分号可有可无，建议不要写（兼容性考虑）。
  3、SQL语句中不能有说明文字。
  4、可以不用判断stmt.prepare()的返回值，stmt.execute()时再判断。
  */

  // 执行SQL语句，一定要判断返回值，0-成功，其它-失败。
  // 失败代码在stmt.m_cda.rc中，失败描述在stmt.m_cda.message中。
  if (stmt.execute()!=0)
  {
    printf("stmt.execute() failed.\n%s\n%s\n",stmt.m_sql,stmt.m_cda.message); return -1;
  }

  printf("create table girls ok.\n");

  return 0;
}

/*
-- 超女基本信息表。
create table girls(id      bigint(10),    -- 超女编号。
                   name    varchar(30),   -- 超女姓名。
                   weight  decimal(8,2),  -- 超女体重。
                   btime   datetime,      -- 报名时间。
                   memo    longtext,      -- 备注。
                   pic     longblob,      -- 照片。
                   primary key (id));
*/



```



##### 插入数据

inserttable.cpp

```
/*
 *  程序名：inserttable.cpp，此程序演示开发框架操作MySQL数据库（向表中插入5条记录）。
*/
#include <unistd.h>
#include "_mysql.h"       // 开发框架操作MySQL的头文件。

int main(int argc,char *argv[])
{
  connection conn;   // 数据库连接类。

  // 登录数据库，返回值：0-成功；其它是失败，存放了MySQL的错误代码。
  // 失败代码在conn.m_cda.rc中，失败描述在conn.m_cda.message中。
  if (conn.connecttodb("127.0.0.1,root,phcQdNaiZ.g2,mysql,3306","utf8")!=0)
  {
    printf("connect database failed.\n%s\n",conn.m_cda.message); return -1;
  }

  // 定义用于超女信息的结构，与表中的字段对应。
  struct st_girls
  {
    long   id;        // 超女编号
    char   name[31];  // 超女姓名
    double weight;    // 超女体重
    char   btime[20]; // 报名时间
  } stgirls;

  sqlstatement stmt(&conn);  // 操作SQL语句的对象。

  // 准备插入表的SQL语句。
  stmt.prepare("\
    insert into girls(id,name,weight,btime) values(:1+1,:2,:3+45.35,str_to_date(:4,'%%Y-%%m-%%d %%H:%%i:%%s'))");
    //insert into girls(id,name,weight,btime) values(?+1,?,?+45.35,to_date(?,'yyyy-mm-dd hh24:mi:ss'))");
  /*
    注意事项：
    1、参数的序号从1开始，连续、递增，参数也可以用问号表示，但是，问号的兼容性不好，不建议；
    2、SQL语句中的右值才能作为参数，表名、字段名、关键字、函数名等都不能作为参数；
    3、参数可以参与运算或用于函数的参数；
    4、如果SQL语句的主体没有改变，只需要prepare()一次就可以了；
    5、SQL语句中的每个参数，必须调用bindin()绑定变量的地址；
    6、如果SQL语句的主体已改变，prepare()后，需重新用bindin()绑定变量；
    7、prepare()方法有返回值，一般不检查，如果SQL语句有问题，调用execute()方法时能发现；
    8、bindin()方法的返回值固定为0，不用判断返回值；
    9、prepare()和bindin()之后，每调用一次execute()，就执行一次SQL语句，SQL语句的数据来自被绑定变量的值。
  */
  stmt.bindin(1,&stgirls.id);
  stmt.bindin(2, stgirls.name,30);
  stmt.bindin(3,&stgirls.weight);
  stmt.bindin(4, stgirls.btime,19);

  // 模拟超女数据，向表中插入5条测试数据。
  for (int ii=0;ii<5;ii++)
  {
    memset(&stgirls,0,sizeof(struct st_girls));         // 结构体变量初始化。

    // 为结构体变量的成员赋值。
    stgirls.id=ii;                                     // 超女编号。
    sprintf(stgirls.name,"西施%05dgirl",ii+1);         // 超女姓名。
    stgirls.weight=ii;                                 // 超女体重。
    sprintf(stgirls.btime,"2021-08-25 10:33:%02d",ii); // 报名时间。 

    // 执行SQL语句，一定要判断返回值，0-成功，其它-失败。
    // 失败代码在stmt.m_cda.rc中，失败描述在stmt.m_cda.message中。
    if (stmt.execute()!=0)
    {
      printf("stmt.execute() failed.\n%s\n%s\n",stmt.m_sql,stmt.m_cda.message); return -1;
    }

    printf("成功插入了%ld条记录。\n",stmt.m_cda.rpc); // stmt.m_cda.rpc是本次执行SQL影响的记录数。
  }

  printf("insert table girls ok.\n");

  conn.commit();   // 提交数据库事务。

  return 0;
}


```

##### 修改数据

updatetable.cpp

```
/*
 *  程序名：updatetable.cpp，此程序演示开发框架操作MySQL数据库（修改表中的记录）。
*/

#include "_mysql.h"       // 开发框架操作MySQL的头文件。

int main(int argc,char *argv[])
{
  connection conn;   // 数据库连接类。

  // 登录数据库，返回值：0-成功；其它是失败，存放了MySQL的错误代码。
  // 失败代码在conn.m_cda.rc中，失败描述在conn.m_cda.message中。
  if (conn.connecttodb("127.0.0.1,root,phcQdNaiZ.g2,mysql,3306","utf8")!=0)
  {
    printf("connect database failed.\n%s\n",conn.m_cda.message); return -1;
  }

  // 定义用于超女信息的结构，与表中的字段对应。
  struct st_girls
  {
    long   id;        // 超女编号
    char   name[31];  // 超女姓名
    double weight;    // 超女体重
    char   btime[20]; // 报名时间
  } stgirls;

  sqlstatement stmt(&conn);  // 操作SQL语句的对象。

  // 准备修改表的SQL语句。
  stmt.prepare("\
    update girls set name=:1,weight=:2,btime=str_to_date(:3,'%%Y-%%m-%%d %%H:%%i:%%s') where id=:4");
  /*
    注意事项：
    1、参数的序号从1开始，连续、递增，参数也可以用问号表示，但是，问号的兼容性不好，不建议；
    2、SQL语句中的右值才能作为参数，表名、字段名、关键字、函数名等都不能作为参数；
    3、参数可以参与运算或用于函数的参数；
    4、如果SQL语句的主体没有改变，只需要prepare()一次就可以了；
    5、SQL语句中的每个参数，必须调用bindin()绑定变量的地址；
    6、如果SQL语句的主体已改变，prepare()后，需重新用bindin()绑定变量；
    7、prepare()方法有返回值，一般不检查，如果SQL语句有问题，调用execute()方法时能发现；
    8、bindin()方法的返回值固定为0，不用判断返回值；
    9、prepare()和bindin()之后，每调用一次execute()，就执行一次SQL语句，SQL语句的数据来自被绑定变量的值。
  */
  stmt.bindin(1, stgirls.name,30);
  stmt.bindin(2,&stgirls.weight);
  stmt.bindin(3, stgirls.btime,19);
  stmt.bindin(4,&stgirls.id);

  // 模拟超女数据，修改超女信息表中的全部记录。
  for (int ii=0;ii<5;ii++)
  {
    memset(&stgirls,0,sizeof(struct st_girls));         // 结构体变量初始化。

    // 为结构体变量的成员赋值。
    stgirls.id=ii+1;                                   // 超女编号。
    sprintf(stgirls.name,"貂蝉%05dgirl",ii+1);         // 超女姓名。
    stgirls.weight=ii+48.39;                           // 超女体重。
    sprintf(stgirls.btime,"2021-10-02 11:25:%02d",ii); // 报名时间。 

    // 执行SQL语句，一定要判断返回值，0-成功，其它-失败。
    // 失败代码在stmt.m_cda.rc中，失败描述在stmt.m_cda.message中。
    if (stmt.execute()!=0)
    {
      printf("stmt.execute() failed.\n%s\n%s\n",stmt.m_sql,stmt.m_cda.message); return -1;
    }

    printf("成功修改了%ld条记录。\n",stmt.m_cda.rpc); // stmt.m_cda.rpc是本次执行SQL影响的记录数。
  }

  printf("update table girls ok.\n");

  conn.commit();   // 提交数据库事务。

  return 0;
}

```



##### 查询数据

selecttable.cpp

```
/*
 *  程序名：selecttable.cpp，此程序演示开发框架操作MySQL数据库（查询表中的记录）。
*/
#include <unistd.h>
#include "_mysql.h"       // 开发框架操作MySQL的头文件。

int main(int argc,char *argv[])
{
  connection conn;   // 数据库连接类。

  // 登录数据库，返回值：0-成功；其它是失败，存放了MySQL的错误代码。
  // 失败代码在conn.m_cda.rc中，失败描述在conn.m_cda.message中。
  if (conn.connecttodb("127.0.0.1,root,phcQdNaiZ.g2,mysql,3306","utf8")!=0)
  {
    printf("connect database failed.\n%s\n",conn.m_cda.message); return -1;
  }

  // 定义用于超女信息的结构，与表中的字段对应。
  struct st_girls
  {
    long   id;        // 超女编号
    char   name[31];  // 超女姓名
    double weight;    // 超女体重
    char   btime[20]; // 报名时间
  } stgirls;

  sqlstatement stmt(&conn);  // 操作SQL语句的对象。

  int iminid,imaxid;  // 查询条件最小和最大的id。

  // 准备查询表的SQL语句。
  stmt.prepare("\
    select id,name,weight,date_format(btime,'%%Y-%%m-%%d %%H:%%i:%%s') from girls where id>=:1 and id<=:2");
  /*
    注意事项：
    1、如果SQL语句的主体没有改变，只需要prepare()一次就可以了；
    2、结果集中的字段，调用bindout()绑定变量的地址；
    3、bindout()方法的返回值固定为0，不用判断返回值；
    4、如果SQL语句的主体已改变，prepare()后，需重新用bindout()绑定变量；
    5、调用execute()方法执行SQL语句，然后再循环调用next()方法获取结果集中的记录；
    6、每调用一次next()方法，从结果集中获取一条记录，字段内容保存在已绑定的变量中。
  */
  // 为SQL语句绑定输入变量的地址，bindin方法不需要判断返回值。
  stmt.bindin(1,&iminid);
  stmt.bindin(2,&imaxid);
  // 为SQL语句绑定输出变量的地址，bindout方法不需要判断返回值。
  stmt.bindout(1,&stgirls.id);
  stmt.bindout(2, stgirls.name,30);
  stmt.bindout(3,&stgirls.weight);
  stmt.bindout(4, stgirls.btime,19);

  iminid=1;    // 指定待查询记录的最小id的值。
  imaxid=3;    // 指定待查询记录的最大id的值。

  // 执行SQL语句，一定要判断返回值，0-成功，其它-失败。
  // 失败代码在stmt.m_cda.rc中，失败描述在stmt.m_cda.message中。
  if (stmt.execute() != 0)
  {
    printf("stmt.execute() failed.\n%s\n%s\n",stmt.m_sql,stmt.m_cda.message); return -1;
  }

  // 本程序执行的是查询语句，执行stmt.execute()后，将会在数据库的缓冲区中产生一个结果集。
  while (true)
  {
    memset(&stgirls,0,sizeof(struct st_girls));         // 结构体变量初始化。

    // 从结果集中获取一条记录，一定要判断返回值，0-成功，1403-无记录，其它-失败。
    // 在实际开发中，除了0和1403，其它的情况极少出现。
    if (stmt.next()!=0) break;

    // 把获取到的记录的值打印出来。
    printf("id=%ld,name=%s,weight=%.02f,btime=%s\n",stgirls.id,stgirls.name,stgirls.weight,stgirls.btime);
  }

  // 请注意，stmt.m_cda.rpc变量非常重要，它保存了SQL被执行后影响的记录数。
  printf("本次查询了girls表%ld条记录。\n",stmt.m_cda.rpc);

  return 0;
}


```



##### 删除数据

deletetable.cpp

```
/*
 *  程序名：deletetable.cpp，此程序演示开发框架操作MySQL数据库（删除表中的记录）。
*/

#include "_mysql.h"       // 开发框架操作MySQL的头文件。

int main(int argc,char *argv[])
{
  connection conn;   // 数据库连接类。

  // 登录数据库，返回值：0-成功；其它是失败，存放了MySQL的错误代码。
  // 失败代码在conn.m_cda.rc中，失败描述在conn.m_cda.message中。
  if (conn.connecttodb("127.0.0.1,root,phcQdNaiZ.g2,mysql,3306","utf8")!=0)
  {
    printf("connect database failed.\n%s\n",conn.m_cda.message); return -1;
  }

  sqlstatement stmt(&conn);  // 操作SQL语句的对象。

  int iminid,imaxid;  // 删除条件最小和最大的id。

  // 准备删除表的SQL语句。
  stmt.prepare("delete from girls where id>=:1 and id<=:2");
  // 为SQL语句绑定输入变量的地址，bindin方法不需要判断返回值。
  stmt.bindin(1,&iminid);
  stmt.bindin(2,&imaxid);

  iminid=1;    // 指定待删除记录的最小id的值。
  imaxid=3;    // 指定待删除记录的最大id的值。

  // 执行SQL语句，一定要判断返回值，0-成功，其它-失败。
  // 失败代码在stmt.m_cda.rc中，失败描述在stmt.m_cda.message中。
  if (stmt.execute() != 0)
  {
    printf("stmt.execute() failed.\n%s\n%s\n",stmt.m_sql,stmt.m_cda.message); return -1;
  }

  // 请注意，stmt.m_cda.rpc变量非常重要，它保存了SQL被执行后影响的记录数。
  printf("本次删除了girls表%ld条记录。\n",stmt.m_cda.rpc);

  conn.commit();   // 提交数据库事务。

  return 0;
}


```

##### 二进制大对象存放

filetoblob.cpp

```
/*
 *  程序名：filetoblob.cpp，此程序演示开发框架操作MySQL数据库（把图片文件存入BLOB字段）。
*/

#include "_mysql.h"       // 开发框架操作MySQL的头文件。

int main(int argc,char *argv[])
{
  connection conn;   // 数据库连接类。

  // 登录数据库，返回值：0-成功；其它是失败，存放了MySQL的错误代码。
  // 失败代码在conn.m_cda.rc中，失败描述在conn.m_cda.message中。
  if (conn.connecttodb("127.0.0.1,root,phcQdNaiZ.g2,mysql,3306","utf8")!=0)
  {
    printf("connect database failed.\n%s\n",conn.m_cda.message); return -1;
  }

  // 定义用于超女信息的结构，与表中的字段对应。
  struct st_girls
  {
    long   id;             // 超女编号
    char   pic[100000];    // 超女图片的内容。
    unsigned long picsize; // 图片内容占用的字节数。
  } stgirls;

  sqlstatement stmt(&conn);  // 操作SQL语句的对象。

  // 准备修改表的SQL语句。
  stmt.prepare("update girls set pic=:1 where id=:2");
  stmt.bindinlob(1, stgirls.pic,&stgirls.picsize);
  stmt.bindin(2,&stgirls.id);

  // 修改超女信息表中id为1、2的记录。
  for (int ii=1;ii<3;ii++)
  {
    memset(&stgirls,0,sizeof(struct st_girls));         // 结构体变量初始化。

    // 为结构体变量的成员赋值。
    stgirls.id=ii;                                   // 超女编号。
    // 把图片的内容加载到stgirls.pic中。
    if (ii==1) stgirls.picsize=filetobuf("1.jpg",stgirls.pic);
    if (ii==2) stgirls.picsize=filetobuf("2.jpg",stgirls.pic);

    // 执行SQL语句，一定要判断返回值，0-成功，其它-失败。
    // 失败代码在stmt.m_cda.rc中，失败描述在stmt.m_cda.message中。
    if (stmt.execute()!=0)
    {
      printf("stmt.execute() failed.\n%s\n%s\n",stmt.m_sql,stmt.m_cda.message); return -1;
    }

    printf("成功修改了%ld条记录。\n",stmt.m_cda.rpc); // stmt.m_cda.rpc是本次执行SQL影响的记录数。
  }

  printf("update table girls ok.\n");

  conn.commit();   // 提交数据库事务。

  return 0;
}


```

##### 二进制大对象抽取

blobtofile.cpp

```
/*
 *  程序名：blobtofile.cpp，此程序演示开发框架操作MySQL数据库（提取BLOB字段内容到图片文件中）。
*/

#include "_mysql.h"       // 开发框架操作MySQL的头文件。

int main(int argc,char *argv[])
{
  connection conn;   // 数据库连接类。

  // 登录数据库，返回值：0-成功；其它是失败，存放了MySQL的错误代码。
  // 失败代码在conn.m_cda.rc中，失败描述在conn.m_cda.message中。
  if (conn.connecttodb("127.0.0.1,root,phcQdNaiZ.g2,mysql,3306","utf8")!=0)
  {
    printf("connect database failed.\n%s\n",conn.m_cda.message); return -1;
  }

  // 定义用于超女信息的结构，与表中的字段对应。
  struct st_girls
  {
    long   id;             // 超女编号
    char   pic[100000];    // 超女图片的内容。
    unsigned long picsize; // 图片内容占用的字节数。
  } stgirls;

  sqlstatement stmt(&conn);  // 操作SQL语句的对象。

  // 准备查询表的SQL语句。
  stmt.prepare("select id,pic from girls where id in (1,2)");
  stmt.bindout(1,&stgirls.id);
  stmt.bindoutlob(2, stgirls.pic,100000,&stgirls.picsize);

  // 执行SQL语句，一定要判断返回值，0-成功，其它-失败。
  // 失败代码在stmt.m_cda.rc中，失败描述在stmt.m_cda.message中。
  if (stmt.execute()!=0)
  {
    printf("stmt.execute() failed.\n%s\n%s\n",stmt.m_sql,stmt.m_cda.message); return -1;
  }

  // 本程序执行的是查询语句，执行stmt.execute()后，将会在数据库的缓冲区中产生一个结果集。
  while (true)
  {
    memset(&stgirls,0,sizeof(stgirls)); // 先把结构体变量初始化。

    // 从结果集中获取一条记录，一定要判断返回值，0-成功，1403-无记录，其它-失败。
    // 在实际开发中，除了0和1403，其它的情况极少出现。
    if (stmt.next()!=0) break;

    // 生成文件名。
    char filename[101]; memset(filename,0,sizeof(filename));
    sprintf(filename,"%d_out.jpg",stgirls.id);
    
    // 把内容写入文件。
    buftofile(filename,stgirls.pic,stgirls.picsize);
  }

  // 请注意，stmt.m_cda.rpc变量非常重要，它保存了SQL被执行后影响的记录数。
  printf("本次查询了girls表%ld条记录。\n",stmt.m_cda.rpc);

  return 0;
}


```



##### 编译文件

makefile

```
# mysql头文件存放的目录。locate mysql.h
MYSQLINCL = -I/usr/include/mysql

# mysql库文件存放的目录。locate libmysqlclient.a
MYSQLLIB = -L/usr/lib64/mysql

#_mysql.h _mysql.cpp在当前文件夹

# 需要链接的mysql库。
MYSQLLIBS = -lmysqlclient

CFLAGS=-g -Wno-write-strings

all:createtable  inserttable updatetable selecttable deletetable filetoblob blobtofile book1 book2

createtable:createtable.cpp _mysql.h _mysql.cpp
	g++ $(CFLAGS) -o createtable createtable.cpp $(MYSQLINCL) $(MYSQLLIB) $(MYSQLLIBS) _mysql.cpp -lm -lc

inserttable:inserttable.cpp _mysql.h _mysql.cpp
	g++ $(CFLAGS) -o inserttable inserttable.cpp $(MYSQLINCL) $(MYSQLLIB) $(MYSQLLIBS) _mysql.cpp -lm -lc

updatetable:updatetable.cpp _mysql.h _mysql.cpp
	g++ $(CFLAGS) -o updatetable updatetable.cpp $(MYSQLINCL) $(MYSQLLIB) $(MYSQLLIBS) _mysql.cpp -lm -lc

selecttable:selecttable.cpp _mysql.h _mysql.cpp
	g++ $(CFLAGS) -o selecttable selecttable.cpp $(MYSQLINCL) $(MYSQLLIB) $(MYSQLLIBS) _mysql.cpp -lm -lc

deletetable:deletetable.cpp _mysql.h _mysql.cpp
	g++ $(CFLAGS) -o deletetable deletetable.cpp $(MYSQLINCL) $(MYSQLLIB) $(MYSQLLIBS) _mysql.cpp -lm -lc

filetoblob:filetoblob.cpp _mysql.h _mysql.cpp
	g++ $(CFLAGS) -o filetoblob filetoblob.cpp $(MYSQLINCL) $(MYSQLLIB) $(MYSQLLIBS) _mysql.cpp -lm -lc

blobtofile:blobtofile.cpp _mysql.h _mysql.cpp
	g++ $(CFLAGS) -o blobtofile blobtofile.cpp $(MYSQLINCL) $(MYSQLLIB) $(MYSQLLIBS) _mysql.cpp -lm -lc

book1:book1.cpp _mysql.h _mysql.cpp
	g++ $(CFLAGS) -o book1 book1.cpp $(MYSQLINCL) $(MYSQLLIB) $(MYSQLLIBS) _mysql.cpp -lm -lc

book2:book2.cpp _mysql.h _mysql.cpp
	g++ $(CFLAGS) -o book2 book2.cpp $(MYSQLINCL) $(MYSQLLIB) $(MYSQLLIBS) _mysql.cpp -lm -lc

clean:
	rm -rf createtable  inserttable updatetable selecttable deletetable filetoblob blobtofile book1 book2

```



