#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <time.h>
#include <dirent.h>
#include <string.h>
#include <signal.h>
#include <sys/wait.h>
#include <ftw.h>
#include <sys/mman.h>
#include <sys/msg.h>
#include <sys/sem.h>
#include <sys/ipc.h>

int pip[5][2], pid[3];
int value, cnt, backuppoint, index;

void doChild(int signo){

        write(pip[1][1], &value, sizeof(int));
        scanf("%d", &num);
        backuppoint=cnt+num;

        write(pip[2][1], &cnt, sizeof(int));
        write(pip[2][1], &backuppoint, sizeof(int));

        //부모프로세스가 이전값 까지의 자식프로세스를 생성하기 전까지 전에있던 자식프로세스는 죽기전에 대기를 한다.     
        read(pip[3][0], &signal, sizeof(int));

        exit(0);
}


void doParent(int signo){
        read(pip[2][0], &cnt, sizeof(int));
        read(pip[2][0], &backuppoint, sizeof(int));

        //unix는 PCB까지 자식프로세스가 복사하므로, OS가 관리하는 내용[가득찬 파이프]이 자식에게  복사된다.
        pid[1] = fork();

        if(pid[1] != 0){
                write(pip[3][1], &signal, sizeof(int));
                wait(0);
        }
 //자식 프로세스는 다시 생겨났으므로, 시그널 등록을 다시 해줘야함.
        if(pid[1] == 0){
                act.sa_handler = doChild;
                sigaction(SIGINT, &act, NULL);
        }
}

void r_init(){
        for(i=0; i<3; i++){
                pipe(pip[i]);
        }

        //부모프로세스 
        pid[0] = getpid();
        pid[1] = fork();

        //부모 프로세스가 하는 일
        if(pid[1] != 0){
                act.sa_handler = doParent;
                sigaction(SIGINT, &act, NULL);

                //부모 프로세스는 대기
                //이때 pip[1]은 시그널을 받는 파이프
                read(pip[1][0], &value, sizeof(int));
        }
        //자식 프로세스가 해야하는 일
        else{
                act.sa_handler = doChild;
                sigaction(SIGINT, &act, NULL);
        }
}

void r_scanf(char *argv, int* in){
        //백업 모드일 경우
        //새로운 자식프로세스와 부모프로세스는 각자 백업포인트까지 연산할 것 임
.
        //main에 있는 index가 백업포인트보다 작은 경우
        //이때, 자식은 부모시그널에 등록된 함수에서 생성되었으므로, 부모와 자식
은 index가 동일하다.

        if(backuppoint>index){
                //부모일 경우
                if(pid[1] == 0)
                        read(pip[0][0], in, sizeof(int));

                //자식인 경우
                else
                        read(pip[4][0], in, sizeof(int));

                index++;


                if(index == backuppoint){

                        //부모일 경우, 해당 지점까지 연산이 완료되었으므로 다시
 재울 것임
                        if(pid[1] != 0)
                                read(pip[1][0], &signal, sizeof(int));          

                        //자식일 경우 부모전용파이프와 자식전용파이프에 쓰인 쓰
레기값을 청소
                        else{
                                for(k=backuppoint;k<cnt;k++){
                                        read(pip[0][0],&trash, sizeof(int));
                                        read(pip[4][0],&trash, sizeof(int));
                                }
                                //자식프로세스는 백업포인트까지 연산한 후 다시 외부입력을 받아야 하므로 cnt를 해당 지점으로 재지정함.
                                cnt = backuppoint;
                        }

                }
        }
        //일반모드는 자식인 경우에만 해당 됨.
        //이때 부모/자식용 읽기파이프를 각각 만들어서 백업시 활용 할 것임.
        else{
                scanf("%d", in);
                write(pip[0][1], in. sizeof(int));
                write(pip[4][1], in, sizeof(int));
                cnt++;
        }

}

void r_printf(char* argv, int* in){
        //일반모드일 경우
        if(backuppoint==index){{
                //자식이 출력을 한다. 부모는 백업으로만 사용한다.
                if(pid[1] == 0)
                        printf(argv, in);
        }

        //백업모드일 경우 출력을 하지 않겠습니다 . . .
}

void r_close(){

        if(pid[1] == 0){
                //자고 있는 부모 프로세스는 깨운다.
                //정상 종료 이므로 부모는 출력하지 않는다.
                write(pip[1][1], &signal, sizeof(int));
                exit(0);
        }

}
