### TCP协议
TCP是面向连接的协议，需要三次握手建立连接后，才能进行数据传输。
#### 服务器端
> socket bind listen accept send recv close
```c
#include<func.h>

int main(int argc,char* argv[]){
  int socketFd=socket(AF_INET,SOCK_STREAM,0);
  struct sockaddr addr;
  bzero(&addr,sizeof(addr));
  addr.sin_port=htons(atoi(argv[2]));
  addr.sin_family=AF_INET;
  addr.sin_addr.s_addr=inet_addr(argv[1]);
  
  int ret=bind(socketFd,(struct sockaddr*)&addr,sizeof(addr));
  listen(socketFd,10);
  struct sockaddr client;
  bzero(&client,sizeof(client));
  
  socklen_t clientlen=sizeof(client);
  int newfd=accept(socketFd,(struct socketaddr*)&client,&clientlen);
  fd_set rdst;
  char buf[1024]={0};
  while(1){
    FD_ZERO(&rdst);
    FD_SET(STDIN_FILENO,&rdst);
    FD_SET(&newfd,&rdst);
    ret=select(newfd+1,&fdst,NULL,NULL,NULL);
    if(ret>0){  
      if(FD_ISSET(STDIN_FILENO,&rdst)){
        bzero(&buf,sizeof(buf));
        ret=read(STDIN_FILENO,buf,sizeof(buf)-1);
        if(ret==0){
          printf("byebye\n");
          break;
        }
        send(newfd,buf,strlen(buf)-1,0);
      }
      else if(FD_ISSET(newfd,&rdst)){
        bzero(&buf,sizeof(buf));
        ret=recv(newfd,buf,sizeof(buf),0);
        if(ret==0){
          printf("byebye\n");
          break;
        }
        puts("%s\n",buf);
      }
    }
  }
}
```

这样建立的会话，客户端终止后不能再次连接，服务器也会break。实际上客户端退出后，服务器应该继续工作。

我们通过使用一个新的文件描述符集合needmonitor来监测每次哪个文件描述符(STDIN_FILENO,newfd,socketFd)打开了.
在每次accept后更新监听集合，在break前撤销监听newfd

```c
#include<func.h>

int main(int argc,char* argv[]){
  int socketFd=socket(AF_INET,SOCK_STREAM,0);
  struct sockaddr addr;
  bzero(&addr,sizeof(addr));
  addr.sin_port=htons(atoi(argv[2]));
  addr.sin_family=AF_INET;
  addr.sin_addr.s_addr=inet_addr(argv[1]);
  
  int ret=bind(socket,(struct sockaddr*)&addr,sizeof(addr));
  listen(socket,10);
  struct sockaddr client;
  bzero(&client,sizeof(client));
  
  int newfd;
  fd_set rdst,needMonitor;
  FD_ZERO(&needMonitor);
  FD_SET(STDIN_FILENO,&needMointor);
  FD_SET(socketFd,&needMonitor);
  char buf[1024]={0};
  while(1){
    memcpy(&rdst,&needMonitor);
    ret=select(newfd+1,&fdst,NULL,NULL,NULL);
    if(ret>0){
      if(FD_ISSET(socketFd,&rdst)){
        newfd=accept(socketFd,(struct sockaddr*)&client,sizeof(client));
        FD_SET(newfd,&needMonitor);
      }
      if(FD_ISSET(STDIN_FILENO,&rdst)){
        bzero(&buf,sizeof(buf));
        ret=read(STDIN_FILENO,buf,sizeof(buf)-1);
        if(ret==0){
          printf("byebye\n");
          break;
        }
        send(newfd,buf,strlen(buf)-1,0);
      }
      else if(FD_ISSET(newfd,&rdst)){
        bzero(&buf,sizeof(buf));
        ret=recv(newfd,buf,sizeof(buf),0);
        if(ret==0){
          printf("byebye\n");
          FD_CLR(newfd,&needMonitor);
          close(newfd);
        }
        puts("%s\n",buf);
      }
    }
```
#### 客户端
> socket connect send recv

客户端不需要bind listen accept，它只是向服务器请求(connect)链接。并且只需要一个套接字传递数据
```
#include<func.h>

int main(int argc,char* argv[]){
  int socketFd=socket(AF_INET,SOCK_STREAM,0);
  struct sockaddr addr;
  bzero(&addr,sizeof(addr));
  addr.sin_port=htons(atoi(argv[2]));
  addr.sin_family=AF_INET;
  addr.sin_addr.s_addr=inet_addr(argv[1]);
  
  int ret=connect(socketFd,(struct sockaddr*)&addr,sizeof(addr));
  fd_set rdst;
  while(1){
    FD_ZERO(&rdst);
    FD_SET(STDIN_FILENO,&rdst);
    FD_SET(socketFd,&rdst);
    ret=select(newfd+1,&fdst,NULL,NULL,NULL);
    if(ret>0){
      if(FD_ISSET(STDIN_FILENO,&rdst)){
        bzero(&buf,sizeof(buf));
        ret=read(STDIN_FILENO,buf,sizeof(buf)-1);
        if(ret==0){
          printf("byebye\n");
          break;
        }
        send(newfd,buf,strlen(buf)-1,0);
      }
      else if(FD_ISSET(newfd,&rdst)){
        bzero(&buf,sizeof(buf));
        ret=recv(newfd,buf,sizeof(buf),0);
        if(ret==0){
          printf("byebye\n");
          break;
        }
        puts("%s\n",buf);
    }
  }
```

### UDP
#### 服务器端
> socket bind recvfrom sendto 
udp是无连接的，不需要listen accept.只要bind端口就打开了。通过recvfrom,sendto的参数来指定信息的流向
```c
#include<func.h>
  
int main(int argc,char* argv[]){
      int socketFd=socket(AF_INET,SOCK_DGRAM,0);
      struct sockaddr_in addr;
      bzero(&addr,sizeof(addr));
      addr.sin_port=htons(atoi(argv[2]));
      addr.sin_family=AF_INET;
      addr.sin_addr.s_addr=inet_addr(argv[1]);
  
      int ret=bind(socketFd,(struct sockaddr*)&addr,sizeof(addr));
      struct sockaddr client;
      bzero(&client,sizeof(client));
      char buf[1024]={0};
  
      socklen_t clientlen=sizeof(client);
      recvfrom(socketFd,buf,sizeof(buf),0,(struct sockaddr*)&client,&clientlen);
  
      fd_set rdst;
      while(1){
          FD_ZERO(&rdst);
          FD_SET(STDIN_FILENO,&rdst);
          FD_SET(socketFd,&rdst);
          ret=select(socketFd+1,&rdst,NULL,NULL,NULL);
          if(ret>0){  
              if(FD_ISSET(STDIN_FILENO,&rdst)){
                  bzero(&buf,sizeof(buf));
                  ret=read(STDIN_FILENO,buf,sizeof(buf)-1);
                  if(ret==0){                                                     
                      printf("byebye\n");
                      break;
  
                  }
                  sendto(socketFd,buf,strlen(buf)-1,0,(struct sockaddr*)&client,si
  
              }
              else if(FD_ISSET(socketFd,&rdst)){
                  bzero(&buf,sizeof(buf));
                  ret=recvfrom(socketFd,buf,sizeof(buf),0,(struct sockaddr*)&clien
                  if(ret==0){
                      printf("byebye\n");
                      break;
  
                  }
                  printf("%s\n",buf);
  
              }
  
          }
  
      }
      close(socketFd);
  }         
```
#### 客户端
> socket sendto recvfrom
```c
#include <func.h>

int main(int argc,char*argv[])
{
    ARGS_CHECK(argc,3);
    int socketFd=socket(AF_INET,SOCK_DGRAM,0);
    struct sockaddr_in addr;
    bzero(&addr,sizeof(addr));
    addr.sin_family=AF_INET;
    addr.sin_port=htons(atoi(argv[2]));
    addr.sin_addr.s_addr=inet_addr(argv[1]);

    char buf[1024]={0};
    sendto(socketFd,"ni shi zhu",10,0,(struct sockaddr*)&addr,sizeof(addr));
    
    fd_set rdst;
    int ret;
    socklen_t fromlen=sizeof(struct sockaddr);                                    
    while(1){
        FD_ZERO(&rdst);
        FD_SET(STDIN_FILENO,&rdst);
        FD_SET(socketFd,&rdst);
        ret=select(socketFd+1,&rdst,NULL,NULL,NULL);
        if(ret>0){
            if(FD_ISSET(STDIN_FILENO,&rdst)){
                bzero(&buf,sizeof(buf));
                read(STDIN_FILENO,buf,sizeof(buf));
                sendto(socketFd,buf,strlen(buf)-1,0,(struct sockaddr*)&addr,sizeof
            }   
            else if(FD_ISSET(socketFd,&rdst)){
                bzero(&buf,sizeof(buf));
                recvfrom(socketFd,buf,sizeof(buf),0,(struct sockaddr*)&addr,&froml
                printf("%s\n",buf);
            } 

    }
    close(socketFd);
    return 0;
}
                                                                      
```
