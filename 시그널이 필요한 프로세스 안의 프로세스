#include<fcntl.h>
#include<unistd.h>
#include<stdio.h>
#include<string.h>
#include<dirent.h>
#include<ftw.h>
#include<signal.h>
#include<stdlib.h>
#include<sys/types.h>
#include<sys/wait.h>
#include<sys/stat.h>
#include<sys/mman.h>
#include<sys/ipc.h>
#include<sys/msg.h>
#include<sys/sem.h>

int main(void){
        char in[50], *res[20]={0};
        int i, n, status, status2;
        pid_t pid, b_pid;
        static struct sigaction act;

        //SIGINT register
        act.sa_handler = SIG_IGN;
        sigaction(SIGINT, &act, NULL);

        b_pid = fork();

        // move backupm 
        if(b_pid == 0){
                execl("backup", "backup", ".", (char*) 0);
                exit(0);
        }


        while (1){
                printf("> ");
                gets(in);
                if (in[0]=='\0')
                        continue;
                i=0;
                res[i]=strtok(in, " ");
                while (res[i]){
                        res[++i]=strtok(NULL, " ");
                }

                if (strcmp(res[0], "exit")==0){
		kill(b_pid, SIGUSR1);
                        waitpid(b_pid, &status, 0);
                        exit(0);
                }


                else if (strcmp(res[0], "cd_m")==0){
                        chdir(res[1]);
                }

                else{
                        if ((pid=fork())==0){
                                act.sa_handler = SIG_DFL;
                                sigaction(SIGINT, &act, NULL);
                                execvp(res[0], res);
                                exit(0);
                        }
                        else{
                                waitpid(pid, &status, 0);
                                if(WIFSIGNALED(status)){
                                        printf("SIGNALED exit :  %d\n", status);
                                }
                                else if(WIFEXITED(status)){
                                        printf("normal exit : %d\n", status);
                                }
                        }
                }

        }

        return 0;

}



char dname[100];

int backup(const char*name, const struct stat *status, int type){
        char temp[100];
        char buf[512];
        int len1, len2, i, fd1, fd2;

        if(type == FTW_F){
                strcpy(temp, dname);
                strcat(temp, "/");
                len1 = strlen(dname);
                strcat(temp, name+len1-4);
                len2 = strlen(temp);

                for(i=len1+1; i<len2; i++){
                        if(temp[i] =='/')
                                temp[i] = '_';
                }

                fd1 = open(name, O_RDONLY);
                fd2 = open(temp, O_CREAT | O_TRUNC | O_WRONLY, 0600);

                int n=0;
                while(n = read(fd1, buf, 512)){
                        write(fd2, buf, n);
                }

        }

        return 0;

}

void catchAlrm(int signo){
        alarm(10);
}

void  main(int argc, char** argv)
{
        static struct sigaction act;
        sigset_t mask;

        //SIGALRM 등록
        act.sa_handler = catchAlrm;
        sigaction(SIGALRM, &act, NULL);

        //SIGINT 등록
        act.sa_handler = SIG_IGN;
        sigaction(SIGINT, &act, NULL);

        //SIGUSR1 등록
        act.sa_handler = SIG_DFL;
        sigaction(SIGUSR1, &act, NULL);

        sigemptyset(&mask);
        sigaddset(&mask, SIGUSR1);

        strcpy(dname, argv[1]);
        strcat(dname, "/TEMP");
        mkdir(dname, 0300);


        alarm(10);
        while(1){
                sigprocmask(SIG_SETMASK, &mask, NULL);
                printf("*************BACK-UP STARTS****************\n");
                sleep(5);
                ftw(".", backup, 1);
                printf("*************BACK-UP ENDS*****************\n");
                sigprocmask(SIG_UNBLOCK, &mask, NULL);

                //busy_waiting 방지
                pause();
        }
        exit(1);
}
