#include<sys/types.h>
#include<sys/ipc.h>
#include<sys/shm.h>
#include<sys/sem.h>
#include<sys/wait.h>
#include<unistd.h>
#include<stdio.h>
#include<fcntl.h>
#include<errno.h>
#include<stdlib.h>
#include<string.h>

#define R 4
#define B 5   
#define L 512

struct databuf1{
        char data[L];
        int id;
        int remainCount;
};

struct databuf2{
        struct databuf1 buf[B];
        int in;
        int joinId[R];
        int memberCount;
};

union semun{
        int val;
        struct semid_ds *buf;
        ushort *array;
};

void sem_wait(int semid, int semidx){
        struct sembuf p_buf;

        p_buf.sem_num=semidx;
        p_buf.sem_op=-1;
        p_buf.sem_flg=0;
        semop(semid, &p_buf, 1);
}

void sem_signal(int semid, int semidx){
        struct sembuf p_buf;
        p_buf.sem_num=semidx;
        p_buf.sem_op=1;
        p_buf.sem_flg=0;
        semop(semid, &p_buf, 1);
}

void reader(int N, int rindex, struct databuf2 *buf, int semid){
        int i, n;

        for (i=rindex; ; i=(i+1)%B){
                // 실행예시 2 & 3 테스트 용
                sem_wait(semid, N);
                if (N==1) sleep(5);
                if (N==2) sleep(3);
                if(buf->buf[i].id == N+1){
                        if(strcmp(buf->buf[i].data, "talk_quit") == 0){
                                (buf->buf[i].remainCount)--;

                                if(buf->buf[i].remainCount == 0){
                                        sem_signal(semid, 5);
                                }
                                exit(0);
                        }
                }
                // 경우의 수 : 보낸 프로세스가 살아 있는 경우 or 죽어있는 경우
                // 경우의 수 : 읽고 있는 프로세스가 살아 있는 경우 or 죽어있는 경우
                // 경우의 수는 4가지이나 하나를 제외하고는 동일한 액션을 취함

                n = (buf->buf[i].id) -1 ;
                //보낸 프로세스가 살아 있는 경우
                if(buf->joinId[n] == 1){

                        //읽고 있는 프로세스가 살아 있는 경우
                        if(buf->joinId[N] == 1){
                                if((buf->buf[i].id) != (N+1)){
                                        printf("%d : %s\n", buf->buf[i].id, buf->buf[i].data);
                                }
                        }
                        //읽고 있는 프로세스가 죽은 경우
                        (buf->buf[i].remainCount)--;

                }
                //보낸 프로세스가 죽은 경우
                if(buf->joinId[n] == 0){
                        (buf->buf[i].remainCount)--;
                }                                                                                     
	//버퍼에 적혀진 데이터를 모든 프로세스들이 다 읽은 경우
                if(buf->buf[i].remainCount == 0){
                        sem_signal(semid, 5);
                }
        }

        exit(0);
}

void writer(int N, struct databuf2 *buf, int semid){
        char temp[L];
        int i, n, flag=0;

        for (;;){


                scanf("%s", temp);
                sem_wait(semid, 5);
                sem_wait(semid, 4);
                //종료를 명령 받은 경우
                if(strcmp(temp, "talk_quit") == 0){
                        (buf->memberCount)--;
                        n = buf->in;
                        buf->buf[n].id = N+1;
                        strncpy(buf->buf[n].data, temp, 512);
                        buf->in = (n+1) % B;
                        for(i=0; i<4; i++){
                                if(buf->joinId[i] == 1){
                                                sem_signal(semid, i);
                                                buf->buf[n].remainCount++;
                                }
                        }
                        buf->joinId[N] = flag;
                        sem_signal(semid, 4);
                        return;
                }

                // 문자열 보내기
                n = (buf->in);
                buf->buf[n].id = (N+1);
                strncpy(buf->buf[n].data, temp, 512);
                buf->in = (n+1) % B;
                //critical section end
                sem_signal(semid, 4);
                for(i=0; i<4; i++){
                        if(buf->joinId[i] == 1){
 sem_signal(semid, i);
                                        buf->buf[n].remainCount++;
                        }
                }
        }
}

int main(int argc, char** argv){
        int id, semid, shmid, i, j, N, rindex;
        // init 선언
        key_t key1, key2;
        union semun arg;
        struct databuf2 *buf;
        ushort  init[7] = {0,0,0,0,1,5,1};

        // key1, key2 설정
        key1 = ftok("main.c", 1);
        key2 = ftok("main.c", 2);
        shmid=shmget(key1, sizeof(struct databuf2), 0600|IPC_CREAT);
        buf=(struct databuf2 *)shmat(shmid,0,0);

        if ((semid=semget(key2, 6, 0600|IPC_CREAT|IPC_EXCL))>0){
                arg.array=init;
                semctl(semid, 0, SETALL, arg);
                buf->in = 0;
                buf->memberCount =0;
                for(i=0; i<R; i++)
                        buf->joinId[i] = 0;
        }
        else{
                semid=semget(key2, 6, 0);
        }

        id=atoi(argv[1]);
        printf("id=%d\n", id);

        // 시작전 할일
        N = id - 1;
        rindex = (buf->in);
        (buf->memberCount)++;
        buf->joinId[N] = 1;

        if (fork()==0){
	         reader(N, rindex, buf, semid);
        }       
        
        writer(N, buf, semid);
        wait(0);
        if (buf->memberCount == 0){
                semctl(semid, IPC_RMID, 0);
                shmctl(shmid, IPC_RMID, 0);
        }       
        exit(0);
}       
                                      
                                                         
