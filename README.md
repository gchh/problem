# problem
遇到的一些问题  
1......  
在使用STVD COSMIC\CXSTM8  
ulong temp32;  
uint temp16;  
temp32=temp16×10;  
编译通过，但是烧录到单片机执行，temp32的值一直不对，随机变动；  
将temp32=temp16×10；拆分成temp32=temp16；temp32×=10；编译烧录执行，一切正确。  
---这个问题可能跟中断有关系，在计算过程中如果发生中断去执行中断服务程序，计算结果就可能错误。  

    TIM4_IER&=0xfe;//关闭中断，如果在ULONG计算过程中出现中断，计算结果可能不对
    ...计算过程
    TIM4_IER|=0x01;//打开中断  
在计算开始前关闭中断，计算结束再打开中断，计算结果就没有在出现错误的情况。

2......  
unsigned char bw;  
bw=bw+2;  
bw=bw>>1;  
  
上面两句在STVP+COSMIC 中编译都会出现”truncating assignment“ 警告  
要怎样改才不会出现 ”truncating assignment“ 警告  
也就是  使用这类语句都会   如  变量=变量+常数    变量=变量-常数   变量=变量\*常数    变量=变量/常数   
都会出现 “truncating assignment”警告   
|---------------------------------------------------------------------------------------|  
终于搞明白，多谢楼下的几位淫兄 ，散分结贴了；  
上面在使用8位变量操作前，都应该强制声明变量为8 位才不会出现警告，看了些资料得知是因STVD编译器默认是16位变量。或许这是C语言的标准  
  
上面改为下面后语句编译全通过，由于我是刚接触STM8，有点菜了，希望大家多指点  
unsigned char bw;  
bw=（unsigned char ) ( bw+2）;  
bw=( unsigned char ) ( bw>>1 );  
  
就连函数参数是8位的变量，都应这样增加类型强制  
write24c02 ( 0x00 ,  ( unsigned char  )  ( bw+1 )  );  
  
格式为：( unsigned char )（表达式）;如是16位的变量则不需要这样声明，   

3......  
STM8S为外部中断事件专门分配了五个中断向量：  
Port A 口的5个引脚：PA[6:2]  
Port B 口的8个引脚：PB[7:0]  
Port C 口的8个引脚：PC[7:0]  
Port D 口的7个引脚：PD[6:0]  
Port E 口的8个引脚：PE[7:0]  
PD7 是最高优先级的中断源 (TLI)。  
PA1不能作为外部中断使用。  

 #使用#if defined()组成复杂的预编译控制指令  
1. 综合运用#if、#defined()、#elif、#else和#endif来组成复杂的编译控制；  

    `#if defined(COMBO_ENABLE) && !defined(PDU_ENABLE)`  
    `/* COMBO configuration */`  
    `#elif defined(PDU_ENABLE) && !defined(COMBO_ENABLE)`  
    `/* PDU configuration */`  
    `#else`  
    `/* COMBO configuration is used */`  
    `#endif`   
2. #ifdef只能判断单一的宏是否定义，而#if defined()可以组成复杂的判别条件；  
对于单一的宏AAA来说，#ifdef AAA和#if defined(AAA)是完全相同的。  
而要组成复杂的判别条件，用#if defined()就灵活方便了，比如：#if defined(AAA) && (BBB >= 10)  
如果改用#ifdef则没法表示条件BBB>=10了。  

 #keil编译时出现下面的警告  
*** WARNING L16: UNCALLED SEGMENT, IGNORED FOR OVERLAY PROCESS  
    SEGMENT: ?PR?REMOTE_TRANSMIT?RC  
意思是名称为RC/rc/Rc/rC的C文件中的函数REMOTE_TRANSMIT（或小写）没有使用。

##
HIT-Xu 2019/3/18 15:58:37  
我今天学了一招新技能。大家有没有遭遇过VCC没有供电，但串口连了别的模块，别的模块先上电了，导致单片机主板通过串口信号线的上拉给板子反向供上了电。  
然而由于上拉电阻的限流，供电功率不足，板子并不能正常工作，如果有SD卡的话，甚至会把SD卡的数据损坏。  
HIT-Xu 2019/3/18 16:02:46  
解决思路很简单，给板子的VDD 3V3接个220R下拉电阻，当串口或I2C总线上被反向供电时，VDD 3V3的电压能保持在0.4V以下，就不会触发SD卡工作了。  
MR。LIAO 2019/3/18 16:04:12  
接下拉会不会通信异常  
HIT-Xu 2019/3/18 16:05:12  
不会通信异常的，VDD 3V3正常供上电以后，就是3.3V了  

## 将整形转换为字符串

    char str[5];
    void ConvertInt2Str(uint pAddr)
    {
      uchar i,t;
      char hex[] = "0123456789ABCDEF";

      for (i = 0; i < 2; ++i) //2代表pAddr为2个字节的整数
      {
        t=pAddr>>((1-i)*8);
        str[2*i] = hex[t >> 4];
        str[2*i+1] = hex[t & 0x0F];    
      }
      str[2*i] = '\0';
    }

 #STVD调试不了  
 提示Failed to launch child process:<gdb.exe>  >error=2-"the system cannot find the file specified".
 
 ##解决方法  
 1.在环境变量->Path中添加CXSTM8的路径，如C:\Program Files\COSMIC\CXSTM8，然后确定。  
 设置好后，还是发现只用使用管理员身份运行STVD才能调试。  
 2.找到stvdebug.exe，属性->运行兼容性疑难解答，设置为以管理员运行该程序。  
 此后，每次打开该软件，都会提示是否允许该程序修改系统。  
 
