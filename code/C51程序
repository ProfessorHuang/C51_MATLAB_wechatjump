#include<reg52.h>
sbit PWM =P2^0;
sbit led1=P1^0;	//方便观察用
sbit led2=P1^1;

unsigned char Count=0; //中断次数（若T0为0.1ms中断一次，200次为20ms）
unsigned char Hcount=15;//高电平次数，即占空比（若T0为0.1ms中断一次，15次为1.5ms）
unsigned char Delay=0;//角度保持时间，(若T0为0.1ms中断一次,1次20ms)
void All_Init();
void main()
{
	
   All_Init();
   while(1)
   {
   	if(Hcount==19)
	{
		led1=1;
		led2=0;
	}
	else
	{ 
		led1=0;
		led2=1;
	}
				
   		
   }
}

void All_Init()
{
 //串口初始化
  PCON&=0x7f;
  TMOD=0x21;
  SCON=0x50;
  TH1=0xfd;
  TL1=0xfd;
  EA=1;
  ES=1;
  ET1=0;
  TR1=1;
//定时器T0初始化
  TH0=0xff;
  TL0=0xa4;
  ET0=1;
  TR0=1;
}

/*T0的中断服务 */

void Timer0_Int() interrupt 1
{

  TR0=0;
  TH0=0xff;
  TL0=0xa4;

  if(Count<=Hcount)
  {
    PWM=1;
  }
  else
  {
   PWM=0;
  }

  Count++;

  if(Count>=200)
  {
    if(Delay<=0)
     {
       Hcount=15;
     }
	 else
     {
	 	Delay--;
	 }
     Count=0; 
	 
  }

  TR0=1;
}

/*串口中断服务*/

void Ser_Int() interrupt 4
{
   if(RI==0)
  {
   return;
  }
   ES=0;
   RI=0;

   Delay=SBUF;
   Hcount=19;   //19控制旋转角度为72°
   Count=0;
   ES=1;
}
