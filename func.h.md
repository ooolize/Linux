 ```
 1 #include<stdio.h>
  2 #include<stdlib.h>
  3 #include<string.h>
  4 #include<sys/stat.h>
  5 #include<unistd.h>
  6 #include<dirent.h>
  7 #include<time.h>
  8 #include<pwd.h>
  9 #include<sys/types.h>
 10 #include<grp.h>
 11 #include<fcntl.h>
 12 #include<sys/mman.h>
 13 #include<sys/select.h>
 14 #include<sys/time.h>
 15 #include<strings.h>
 16 #include<sys/wait.h>
 17 #include<sys/ipc.h>
 18 #include<sys/shm.h>                                                                                                                                                 
 19 
 20 #define ARGS_CHECK(argc,val){if(argc!=val){printf("error args\n");return -1;}}
 21 #define ERROR_CHECK(ret,retval,message){if(ret==retval){perror(message);return -1;}}
~                                                                                           
```
