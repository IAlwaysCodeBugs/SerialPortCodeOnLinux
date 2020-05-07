# SerialPortCodeOnLinux

## 本用来测试linux的串口编程

串口参数的配置主要包括:波特率、数据位、停止位、流控协议。

linux中的串口设备文件放于/de/目录下，串口一，串口二分别为"/dev/ttyS0","/dev/ttyS1".在linux下操作串口与操作文件相同.
.串口详细配置
 包括：波特率、数据位、校验位、停止位等。串口设置由下面的结构体实现：
      struct termios
      {
 tcflag_t  c_iflag;  //input flags
 tcflag_t  c_oflag;  //output flags
 tcflag_t  c_cflag;  //control flags
 tcflag_t  c_lflag;  //local flags
 cc_t      c_cc[NCCS]; //control characters
      }; 
        该结构体中c_cflag最为重要，可设置波特率、数据位、校验位、停止位。在设置波特率时需要在数字前加上'B'，

如B9600，B15200.使用其需通过“与”“或”操作方式：