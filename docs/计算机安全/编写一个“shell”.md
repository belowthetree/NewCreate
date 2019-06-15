# shell是什么
shell是linux、unix系统中的一个命令行交互工具，可以方便得通过shell在命令行中执行各种命令。这次通过配置系统自带的bash这个shell的路径来启动我们自己的shell。

## 配置路径
首先打开~/.bashrc这个文件，然后在最底下加入`export PATH="direction:$PATH"`，其中的direction是你的程序的路径。

## 读取命令
```
char comm[100];
cout<<"-> shell ";
while(cin>>comm){
  ......
  char **send = (char**)malloc(sizeof(char**)*20);
  char *tmp;
  int i = 0;
   for(tmp = strtok(comm, " ");tmp != NULL;tmp = strtok(NULL, " ")){
         send[i++] = tmp;
         f(i==20){
               perror("too many argument!");
               exit(EXIT_FAILURE);
         }
   }
```

命令的读取需要用char*类型，方便分割转移。


## 命令的执行
```
void exec_cmd(char **arg_vec)
{
    int   status;
    pid_t pid;

    pid = fork();
    if (pid == -1) {
        perror("fork");
        exit(EXIT_FAILURE);
    } else if (pid == 0) { // child process, run the command
        // execvp() returns only if an error occurs
        execvp(arg_vec[0], arg_vec);
        perror("execvp");
        exit(EXIT_FAILURE);
    } else {               // parent process, wait for children to exit
        while (wait(&status) != pid)
            ;
    }
}
```

调用系统带的exec系函数执行读取到的指令。
