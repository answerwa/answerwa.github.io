title: linux_pipe
date: 2015-01-28 21:26:55
tags: [linux,pipe]
---
## 函数介绍
### 定义
create pipe，创建管道，包含在头文件 unistd.h 中，定义如下：
```c
#include <unistd.h>
int pipe(int pipefd[2]); 
```
### 参数
创建管道后，将文件描述词由参数pipefd数组返回。  
pipefd[0]为管道的读取端；  
pipefd[1]为管道的写入端。 
### 返回值
成功创建后，返回0，错误则返回-1，并将错误原因写入errno中。
## 测试代码  
```c
#include <sys/wait.h>
#include <assert.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int
main(int argc, char *argv[])
{
    int pipefd[2];
    pid_t cpid;
    char buf;

    assert(argc == 2);

    if (pipe(pipefd) == -1) {
        perror("pipe");
        exit(EXIT_FAILURE);
    }

    cpid = fork();
    if (cpid == -1) {
        perror("fork");
        exit(EXIT_FAILURE);
    }

    if (cpid == 0) {    /* Child reads from pipe */
        close(pipefd[1]);          /* Close unused write end */

        while (read(pipefd[0], &buf, 1) > 0)
            write(STDOUT_FILENO, &buf, 1);

        write(STDOUT_FILENO, "\n", 1);
        close(pipefd[0]);
        _exit(EXIT_SUCCESS);

    } else {            /* Parent writes argv[1] to pipe */
        close(pipefd[0]);          /* Close unused read end */
        write(pipefd[1], argv[1], strlen(argv[1]));
        close(pipefd[1]);          /* Reader will see EOF */
        wait(NULL);                /* Wait for child */
        exit(EXIT_SUCCESS);
    }
}
```
补充说明一下：由于wait函数本身是有缺陷的，因此换成**waitpid(cpid, NULL, 0)**比较合适(此代码为manpage中例)。