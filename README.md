# SerialPortCodeOnLinux

## 用来测试linux的串口编程

### 串口参数的配置主要包括
- 波特率
- 数据位
- 停止位
- 流控协议

linux中的串口设备文件放于/de/目录下，串口一，串口二分别为"/dev/ttyS0","/dev/ttyS1".在linux下操作串口与操作文件相同.

### 串口详细配置
包括
- 波特率
- 数据位
- 校验位
- 停止位等

### 串口设置
由下面的结构体实现：
```
struct termios {
    tcflag_t c_iflag; /* 输入参数 */
    tcflag_t c_oflag; /* 输出参数 */
    tcflag_t c_cflag; /* 控制参数*/
    tcflag_t c_ispeed; /* 输入波特率 */
    tcflag_t c_ospeed; /* 输出波特率 */
    cc_t c_line; /* 线控制 */
    cc_t c_cc[NCCS]; /* 控制字符*/
}; 
```

该结构体中**c_cflag**最为重要，可设置波特率、数据位、校验位、停止位。

在设置波特率时需要在数字前加上'B'，如B9600，B15200.使用其需通过“与”“或”操作方式：

### 串口控制函数

 tcgetattr 取属性(termios结构)    
 tcsetattr 设置属性(termios结构)    
 cfgetispeed 得到输入速度   
 cfgetospeed 得到输出速度       
 cfsetispeed 设置输入速度   
 cfsetospeed 设置输出速度   
 tcdrain 等待所有输出都被传输  
 tcflow 起传输或接收    
 tcflush 刷清未决输入和/或输出      
 tcsendbreak 送BREAK字符    
 tcgetpgrp 得到前台进程组ID     
 tcsetpgrp 设置前台进程组ID 


### 串口配置流程

1. 保存原先串口配置，用`tcgetattr(fd,&oldtio)`函数
```
    struct termios newtio,oldtio;
    tcgetattr(fd,&oldtio);
```
2. 激活选项有CLOCAL和CREAD，用于本地连接和接收使用
```
    newtio.c_cflag | = CLOCAL | CREAD;
```
3. 设置波特率，使用函数cfsetispeed、cfsetospeed
```
    cfsetispeed(&newtio,B115200);
    cfsetospeed(&newtio,B115200);
```
4. 设置数据位，需使用掩码设置
```
    newtio.c_cflag &= ~CSIZE;
    newtio.c_cflag |= CS8;
```
5. 设置奇偶校验位，使用`c_cflag`和`c_iflag`   
##### 设置奇校验
```    
    newtio.c_cflag |= PARENB;
    newtio.c_cflag |= PARODD;
    newtio.c_iflag |= (INPCK | ISTRIP);
```                             
        
##### 设置偶校验

 
```
    newtio.c_iflag |= (INPCK｜ISTRIP);
    newtio.c_cflag |= PARENB;
    newtio.c_cflag |= ~PARODD;
 ```
6. 设置停止位，通过激活`c_cflag`中的`CSTOPB`实现。若停止位为`1`，则清除`CSTOPB`，若停止位为`2`，则激活`CSTOPB`
```
    newtio.c_cflag &= ~CSTOPB;
```
7. 设置最少字符和等待时间，对于接收字符和等待时间没有特别的要求时，可设为0：
```
    newtio.c_cc[VTIME] = 0;
    newtio.c_cc[VMIN]  = 0;
```
8. 处理要写入的引用对象 

    `tcflush函数`    

刷清(抛弃)输入缓存(终端驱动程序已接收到，但用户程序尚未读)   
输出缓存(用户程序已经写，但尚未发送).
```
    int tcflush(int filedes,int quene)
```
quene数应当是下列三个常数之一:    
    *TCIFLUSH  刷清输入队列     
    *TCOFLUSH  刷清输出队列     
    *TCIOFLUSH 刷清输入、输出队列   
  例如：`tcflush(fd,TCIFLUSH);`

9. 激活配置     
在完成配置后，需要激活配置使其生效。使用`tcsetattr()`函数：
```
int tcsetattr(int filedes,int opt,const struct termios *termptr);
```
opt使我们可以指定在什么时候新的终端属性才起作用，       
  *TCSANOW:更改立即发生     
  *TCSADRAIN:发送了所有输出后更改才发生。若更改输出参数则应使用此选项       
  *TCSAFLUSH:发送了所有输出后更改才发生。更进一步，在更改发生时未读的所有输入数据都被删除(刷清).        

例如: `tcsetattr(fd,TCSANOW,&newtio)`;

### 串口使用详解

#### 打开串口
```
 fd = open("/dev/ttyS0",O_RDWR | O_NOCTTY | O_NDELAY);

```
参数        
`O_NOCTTY`：通知linux系统，这个程序不会成为这个端口的控制终端.       
`O_NDELAY`：通知linux系统不关心DCD信号线所处的状态(端口的另一端是否激活或者停止).
#### 恢复串口的状态为阻塞状态，用于等待串口数据的读入，用fcntl函数:
```
 fcntl(fd,F_SETFL,0);  //F_SETFL：设置文件flag为0，即默认，即阻塞状态
```
#### 接着测试打开的文件描述符是否应用一个终端设备，以进一步确认串口是否正确打开.
```
isatty(STDIN_FILENO);
```

#### 读写串口
 串口的读写与普通文件一样，使用read，write函数      
`read(fd,buff,8);`      
`write(fd,buff,8);`