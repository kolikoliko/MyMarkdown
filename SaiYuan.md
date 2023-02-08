# Saiyuan代码

```c
#include "SC_Init.h"
#include "SC_it.h"
#include "SCDriver_list.h"
#include "SysFunVarDefine.h"

#include "TK_Key.h"
#include "delay.h"
#include "Light.h"
#include "Key.h"

#define   Key1_ROM    64522   //记录密码在ROM中的地址
#define   Key2_ROM    64622
#define   Key3_ROM    64722

uint8_t i=0;
unsigned char KeyValue=0;//用来获取数值
unsigned char Key_Flag=0;//用来判断标志位
unsigned char Key_Throw=0; //丢弃标志位
unsigned char Count;//用来统计次数
int Key_TimeCount;//计时判断模式
unsigned char Key_Mode=0;//输入模式
bool Key_Check;
unsigned char Key_Time;
unsigned char Key_Pos=0;//用来记录密码位置
uint8_t Key_In[9]={20,20,20,20,20,20,20,20,20};//用来储存密码
void Key_Get_First(void);
//模式1
uint8_t Key_Mode1[4]={2,3,2,4};//用来储存密码
void Key_Get_1(void);
//模式2
unsigned char Key_Temp;
uint8_t Key_Mode2[6]={1,2,3,4,5,6};//用来储存密码
void Key_Get_2(void);
//模式3
unsigned char Key_CheckCount;//用来检测密码正确性
unsigned char Key_Again;//两次输入密码即可
bool Admin_Mode;//是否进入管理员模式
uint8_t Key_Mode3[4]={1,1,1,4};//用来储存密码
uint8_t Key_Mode3_Temp[4]={1,1,1,4};//用来储存设置密码
void Key_Get_3(void);
//其他
void Key_Insert(void);//获取按键存入密码
void Key_Che(void);//密码检查
void Buzzer(void);//蜂鸣器短响
void WriteHigh(void);
void TK_Touch(void);
void KeyWriteRom(unsigned char type);//写入密码进ROM
void KeyReadRom(void);//从ROM读密码

/**
  * @brief  在main中做循环，检测密码，检测模式等等所有操作
  * @param  无
  * @retval 无
  */
void TK_Key()
{
	TK_Touch();
        
        switch (Key_Mode)
        {
            case 0:
                Key_Get_First();
				delay_ms(5);
                break;
            case 1:
                Key_Get_1();
                break;
            case 2:
                Key_Get_2();
                break;
            case 3:
                Key_Get_3();
                break;
            default:
                break;
        }
        Key_Che();
}

/**
  * @brief  魔盒生成的tk触控检测代码
  * @param  无
  * @retval 无
  */
void TK_Touch()
{
    if(GetLowPowerScanFlag())
    {
        Key_Throw=3;
        Key_Pos=0;
        LowPower_Touchkey_Scan();
        Light_Over();
    }
    else
    {
        if(SOCAPI_TouchKeyStatus & 0x40)
        {
            SOCAPI_TouchKeyStatus &= 0xbf;
            TouchKeyRestart();
        }
        if(SOCAPI_TouchKeyStatus & 0x80)
        {
            SOCAPI_TouchKeyStatus &= 0x7f;
            exKeyValueFlag = TouchKeyScan();
            if(exKeyValueFlag!=0)
            {
                KeyValue=Key_Transit(exKeyValueFlag);
                Key_Flag=1;
            }
            if(exKeyValueFlag == 0)
            {		
                Key_Flag=0;
                if(Key_Mode!=1)
                {
                    Key_TimeCount++;
                }
                TK_NoKeyCount++;
                if(TK_NoKeyCount  > 4300)
                {
                    TK_NoKeyCount = 0;
                    TouchKey_IntoLowPowerMode();
                }
                if(Key_Mode == 1||Key_Mode == 3)
                {
                    Light_AllLowLight();
                }
            }
            else
            {
                TK_NoKeyCount = 0;
            }
            UserCode();
            TouchKeyRestart();
        }
    }
}

/**
  * @brief  检测第一位密码值并且分析间隔时间进入不同的模式
  * @param  无
  * @retval 无
  */
void Key_Get_First()
{
    if(Key_Pos==0)
    {
        Key_TimeCount=0;
		//Light_AllHighLight();
    }
    if(Key_Flag)
    {
        Count++;
        if(Count==3)
        {
            if(Key_Throw)
            {
                Key_Throw--;
                Light_AllHighLight();
                delay_ms(300);//别删！
                Light_AllLowLight();
            }
            else if(Key_Pos==0 && Key_Throw==0)
            {
                Light_Over();
                KeyValue=Key_Transit(exKeyValueFlag);
                Light_LightOne(KeyValue);
                Key_Insert();
                Key_Temp=KeyValue;
                Light_AllLowLight();
            }	
            else if(Key_Pos==1 && Key_Throw==0)
            {
                if(Key_TimeCount < 100 && Key_Temp!=KeyValue)
                {
                    Light_Over();
                    Light_LightOne(Key_In[0]);
                    Key_Mode=2;
                    Key_TimeCount=40;
                }
                else if(Key_TimeCount > 100)
                {
                    Light_Over();
                    Key_Mode=1;
                    Key_TimeCount=40;
                }
            }
            Key_Flag=0;
            Count=0;
        }
        Key_Flag=0;
    }
}

/**
  * @brief  获取按键密码值
  * @param  
  * @retval 
  */
void Key_Get_1()
{
    if(Key_Flag)
    {
        Count++;
        if(Count==3)
        {
            KeyValue=Key_Transit(exKeyValueFlag);
			Light_LightOne(KeyValue);
            Key_Insert();
			
            Key_Flag=0;
            Count=0;
        }
        Key_Flag=0;
    }
}

/**
  * @brief  获取手势密码值
  * @param  
  * @retval 
  */
void Key_Get_2()
{
    if(Key_Flag)
    {
        Count++;
        if(Count==3)
        {
            KeyValue=Key_Transit(exKeyValueFlag);
            if(Key_Temp != KeyValue)
            {
                Light_LightOne(KeyValue);
                Key_Insert();
            }
            Key_Flag=0;
            Count=0;
            Key_Temp=KeyValue;
        }
        Key_Flag=0;
    }
    Key_TimeCount=0;
}

/**
  * @brief  获取修改的密码
  * @param  
  * @retval 
  */
void Key_Get_3()
{
    if(Key_Flag)
    {
        Count++;
        if(Count==3)
        {
            Light_Over();
            KeyValue=Key_Transit(exKeyValueFlag);
            Light_LightOne(KeyValue);
            if(KeyValue==12)
            {
                Admin_Mode=0;
                Key_Again=0;
                Key_Mode=0;
                Key_Pos=0;
                Buzzer();
                delay_ms(1000);
                Buzzer();
            }
            if(Admin_Mode)
            {
                Key_Insert();
            }
            else if(KeyValue==10 && Admin_Mode==0)
            {
                Admin_Mode=1;
                Light_AllHighLight();
                Buzzer();
                delay_ms(800);
                Light_AllLowLight();
            }
            Key_Flag=0;
            Count=0;
        }
        Key_Flag=0;
    }
}
void Key_Insert()
{
    if(Key_Pos<9 && (Admin_Mode==0 || Admin_Mode==2))
    {
        Key_In[Key_Pos]=KeyValue;
        Key_Pos++;
        Buzzer();
        delay_ms(100);
    }
    if(Key_Pos<9 && Admin_Mode==1)
    {
        Key_Mode3_Temp[Key_Pos]=KeyValue;
        Key_Pos++;
        Buzzer();
        delay_ms(100);
    }
}

/**
  * @brief  密码检查
  * @param  
  * @retval 
  */
void Key_Che()
{
    if(Key_Mode==1)//密码模式以及管理员模式检查
    {
        if(Key_Pos>3)
        {
            Key_Check=1;
            for(i=0;i<4;i++)
            {
                if(Key_In[i]!=Key_Mode1[i])
                {
                    Key_Check=0;
                    break;
                }
            }
            for(i=0;i<4;i++)
            {
                if(Key_In[i]==Key_Mode3[i])
                {
                    Key_CheckCount++;
                }
            }
            if(Key_CheckCount==4)
            {
                Key_Check=1;
                Key_Mode=3;
                Key_CheckCount=0;
                Light_AllHighLight();
            }
            else Key_CheckCount=0;
            if(Key_Check)		//密码正确
            {
                Buzzer();
                if(Key_Mode!=3) Key_Mode=0;
                Key_Time=0;
            }
            else				//密码错误
            {
                Light_Over();
                Buzzer();
                delay_ms(100);
                Light_AllHighLight();
                delay_ms(100);
                Light_Over();
                Buzzer();
                delay_ms(100);
                Light_AllHighLight();
                delay_ms(100);
                Light_Over();
                Buzzer();
                delay_ms(100);
                Light_AllHighLight();
                delay_ms(100);
                Light_Over();
                Buzzer();
                Key_Mode=0;
                Key_Time++;//次数增加
            }
            for(i=0;i<4;i++)
            {
                Key_In[i]=20;
            }
            Light_AllLowLight();
            Key_Pos=0;
        }
    }
    else if(Key_Mode==2)//手势识别检查
    {
        if(Key_Pos>5)
        {
            Key_Check=1;
            for(i=0;i<6;i++)
            {
                if(Key_In[i]!=Key_Mode2[i])
                {
                    Key_Check=0;
                    break;
                }
            }
            if(Key_Check)		//密码正确
            {
                Buzzer();
                Key_Time=0;
            }
            else				//密码错误
            {
                Light_Over();
                Buzzer();
                delay_ms(100);
                Light_AllHighLight();
                delay_ms(100);
                Light_Over();
                Buzzer();
                delay_ms(100);
                Light_AllHighLight();
                delay_ms(100);
                Light_Over();
                Buzzer();
                delay_ms(100);
                Light_AllHighLight();
                delay_ms(100);
                Light_Over();
                Buzzer();
                Key_Time++;//次数增加
            }
            for(i=0;i<6;i++)
            {
                Key_In[i]=20;
            }
            Light_AllLowLight();
            Key_Mode=0;
            Key_Pos=0;
        }
    }
    else if(Key_Mode==3)//管理员重新设置密码
    {
        if(Key_Pos>3 && Admin_Mode==1)
        {
            Key_Pos=0;
            Admin_Mode++;
            Light_AllHighLight();
            Buzzer();
            delay_ms(1000);
            Light_AllLowLight();
        }
        if(Key_Pos>3 && Admin_Mode==2)
        {
            Key_Pos=0;
            Key_Mode=0;
            for(i=0;i<4;i++)
            {
                if(Key_Mode3_Temp[i]==Key_In[i])
                {
                    Key_CheckCount++;
                }
            }
            if(Key_CheckCount==4)
            {
                for(i=0;i<4;i++)
                {
                    Key_Mode1[i]=Key_Mode3_Temp[i];	
                }
                KeyWriteRom(4);//擦除
                KeyWriteRom(1);//写入
                Light_AllHighLight();
                Buzzer();
                delay_ms(1000);
                Buzzer();
                delay_ms(1000);
                Key_CheckCount=0;
                Key_Pos=0;
                Admin_Mode=0;
                Key_Mode=0;
                Buzzer();
                delay_ms(1000);
                Light_AllLowLight();
                Key_CheckCount=0;
                Key_Pos=0;
                Admin_Mode=0;
                Key_Mode=0;
            }
            else
            {
                Light_Over();
                Key_CheckCount=0;
                Key_Pos=0;
                Admin_Mode=0;
                Key_Mode=0;
                Buzzer();
                delay_ms(1000);
                Buzzer();
                delay_ms(1000);
                Buzzer();
                delay_ms(1000);
                Light_AllLowLight();
            }
        }
    }
    if(Key_Time==5)//检测是否输入错误次数大于五次
    {
        Light_Over();
        for(i=0;i<10;i++)
        {
            delay_ms(1000);
        }
        Key_Time=0;
    }
}

/**
  * @brief  蜂鸣器短响一声
  * @param  无
  * @retval 无
  */
void Buzzer()
{
    PWM_CmdEX(PWM2_Type,ENABLE);
    delay_ms(50);
    PWM_CmdEX(PWM2_Type,DISABLE);
}

void WriteHigh()
{
    GPIO_WriteHigh(GPIO3, GPIO_PIN_7);
    GPIO_WriteHigh(GPIO3, GPIO_PIN_6);
    GPIO_WriteHigh(GPIO3, GPIO_PIN_5);
    GPIO_WriteHigh(GPIO3, GPIO_PIN_4);
}

/**
  * @brief  写rom的密码
  * @param  传入值type分别对应下面的注释作用
  * @retval 无
  */
void KeyWriteRom(unsigned char type)
{
    if(type==1)//写密码1
    {
        for(i=0;i<4;i++) IAP_ProgramByte(Key1_ROM+i,Key_Mode1[i],IAP_MEMTYPE_ROM,0xf0);
    }
    else if(type==2)//写密码2
    {
        for(i=0;i<6;i++) IAP_ProgramByte(Key2_ROM+i,Key_Mode2[i],IAP_MEMTYPE_ROM,0xf0);
    }
    else if(type==3)//写密码3
    {
        for(i=0;i<4;i++) IAP_ProgramByte(Key3_ROM+i,Key_Mode3[i],IAP_MEMTYPE_ROM,0xf0);
    }
    else if(type==4)//擦除Key1
    {
        for(i=0;i<4;i++) IAP_SectorErase(IAP_MEMTYPE_ROM,Key1_ROM+i,0xf0);
    }
    else if(type==5)//擦除Key2
    {
        for(i=0;i<6;i++) IAP_SectorErase(IAP_MEMTYPE_ROM,Key2_ROM+i,0xf0);
    }
    else if(type==6)//擦除Key3
    {
        for(i=0;i<4;i++) IAP_SectorErase(IAP_MEMTYPE_ROM,Key3_ROM+i,0xf0);
    }
    else if(type==7)//重新设置
    {
        KeyWriteRom(4);
        KeyWriteRom(5);
        KeyWriteRom(6);
        KeyWriteRom(1);
        KeyWriteRom(2);
        KeyWriteRom(3);
    }
}
void KeyReadRom()
{
    for(i=0;i<4;i++) Key_Mode1[i]=IAP_ReadByte(Key1_ROM+i, IAP_MEMTYPE_ROM);
    for(i=0;i<6;i++) Key_Mode2[i]=IAP_ReadByte(Key2_ROM+i, IAP_MEMTYPE_ROM);
    for(i=0;i<4;i++) Key_Mode3[i]=IAP_ReadByte(Key3_ROM+i, IAP_MEMTYPE_ROM);
}



```



## main

```c
//************************************************************
//  Copyright (c)  
//	FileName	  : main.c
//	Function	  : Main Function
//  Instructions  : Contains the MCU initialization function and its H file
//************************************************************
/********************Includes************************************************************************/
#include "SC_Init.h"	// MCU initialization header file, including all firmware library header files
#include "SC_it.h"
#include "..\Drivers\SCDriver_list.h"
#include "HeadFiles\SysFunVarDefine.h"
#include "delay.h"
#include "Light.h"
#include "Key.h"
/**************************************Generated by EasyCodeCube*************************************/
int i=0;
unsigned char KeyValue=0;//用来获取数值
unsigned char Key_Flag=0; //用来判断标志位
unsigned char Key_Throw; //丢弃标志位
unsigned char Count;//用来统计次数

int Key_TimeCount;//计时判断模式
unsigned char Key_Mode=0;//输入模式

bool Key_Check;
unsigned char Key_Time;
unsigned char Key_Pos;//用来记录密码位置

unsigned char Key_In[9]={20,20,20,20,20,20,20,20,20};//用来储存密码
void Key_Get_First(void);

//模式1
unsigned char Key_Mode1[4]={1,2,3,4};//用来储存密码
void Key_Get_1(void);
//模式2
unsigned char Key_Temp;
unsigned char Key_Mode2[6]={1,2,3,6,5,4};//用来储存密码
void Key_Get_2(void);
//模式3
unsigned char Key_CheckCount;//用来检测密码正确性
unsigned char Key_Again;//两次输入密码即可
bool Admin_Mode;//是否进入管理员模式
unsigned char Key_Mode3[4]={1,1,1,4};//用来储存密码
unsigned char Key_Mode3_Temp[4]={1,1,1,4};//用来储存设置密码
void Key_Get_3(void);

void Key_Insert(void);
void Key_Che(void);

void Buzzer(void);

void WriteHigh(void);

void TK_Touch(void);

/*************************************.Generated by EasyCodeCube.************************************/
/*****************************************************************************************************
* Function Name: main
* Description  : This function implements main function.
* Arguments    : None
* Return Value : None
******************************************************************************************************/
void main(void)
{
    /*<Generated by EasyCodeCube begin>*/
    /*<UserCodeStart>*//*<SinOne-Tag><186>*/    
    SC_Init();
    
    PWM_IndependentModeConfigEX(PWM00,0, PWM_OUTPUTSTATE_ENABLE);
	PWM_CmdEX(PWM2_Type,DISABLE);
    /*<UserCodeEnd>*//*<SinOne-Tag><186>*/
    /*<UserCodeStart>*//*<SinOne-Tag><9>*/
    TouchKeyInit();
	Light_AllLowLight();
	
    /*<UserCodeEnd>*//*<SinOne-Tag><9>*/
    /*<UserCodeStart>*//*<SinOne-Tag><10>*/
    while(1)
    {
		TK_Touch();
		
		if(Key_Mode==2 && Key_TimeCount > 500)
		{
			Key_Pos=6;	
		}
		
		switch (Key_Mode)
        {
        	case 0:
				Key_Get_First();
        		break;
        	case 1:
				Key_Get_1();
        		break;
			case 2:
				Key_Get_2();
        		break;
			case 3:
				Key_Get_3();
        		break;
        	default:
        		break;
        }
		
		

		Key_Che();
		
    }
    /*<UserCodeEnd>*//*<SinOne-Tag><10>*/
    /*<Generated by EasyCodeCube end>*/
}

void TK_Touch()
{
        if(GetLowPowerScanFlag())
        {
			Key_Throw=3;
			Key_Pos=0;
            LowPower_Touchkey_Scan();
            Light_Over();
        }
        else
        {
            if(SOCAPI_TouchKeyStatus & 0x40)
            {
                SOCAPI_TouchKeyStatus &= 0xbf;
                TouchKeyRestart();
            }
            if(SOCAPI_TouchKeyStatus & 0x80)
            {
                SOCAPI_TouchKeyStatus &= 0x7f;
                exKeyValueFlag = TouchKeyScan();
				if(exKeyValueFlag!=0)
				{
					KeyValue=Key_Transit(exKeyValueFlag);
					Key_Flag=1;
				}
				
                if(exKeyValueFlag == 0)
                {
					Key_Flag=0;
					if(Key_Mode!=1)
					{
						Key_TimeCount++;
					}
                    TK_NoKeyCount++;
                    if(TK_NoKeyCount  > 4300)
                    {
                        TK_NoKeyCount = 0;
                        TouchKey_IntoLowPowerMode();
                    }
					if(Key_Mode == 1||Key_Mode == 3)
					{
						Light_AllLowLight();
					}
					
                }
                else
                {
                    TK_NoKeyCount = 0;
                }
                UserCode();


                TouchKeyRestart();
            }
        }
}

void Key_Get_First()
{
	if(Key_Pos==0)
	{
		Key_TimeCount=0;
	}
	
	if(Key_Flag)
	{
		Count++;
		if(Count==3)
		{
			if(Key_Throw)
			{
				Key_Throw--;
				Light_AllHighLight();
				delay_ms(300);//别删！
				Light_AllLowLight();
			}
			else if(Key_Pos==0 && Key_Throw==0)
			{
				Light_Over();
				KeyValue=Key_Transit(exKeyValueFlag);
				Light_LightOne(KeyValue);
				Key_Insert();
				Key_Temp=KeyValue;
				
				Light_AllLowLight();
			}	
			else if(Key_Pos==1 && Key_Throw==0)
			{
				if(Key_TimeCount < 100 && Key_Temp!=KeyValue)
				{
					Light_Over();
					Light_LightOne(Key_In[0]);
					Key_Mode=2;
					Key_TimeCount=40;
				}
				else if(Key_TimeCount > 100)
				{
					Light_Over();
					Key_Mode=1;
					Key_TimeCount=40;
				}
			}
			Key_Flag=0;
			Count=0;
		}
		Key_Flag=0;
	}
}

void Key_Get_1()
{
	if(Key_Flag)
	{
		Count++;
		if(Count==3)
		{
			Light_Over();
			KeyValue=Key_Transit(exKeyValueFlag);
			Light_LightOne(KeyValue);
			Key_Insert();
			Key_Flag=0;
			Count=0;
		}
		Key_Flag=0;
	}
	
}

void Key_Get_2()
{
	if(Key_Flag)
	{
		Count++;
		if(Count==3)
		{
			
			KeyValue=Key_Transit(exKeyValueFlag);
			if(Key_Temp != KeyValue)
			{
				Light_LightOne(KeyValue);
				Key_Insert();
			}
			Key_Flag=0;
			Count=0;
			Key_Temp=KeyValue;
		}
		Key_Flag=0;
	}
	Key_TimeCount=0;
}

void Key_Get_3()
{
	if(Key_Flag)
	{
		Count++;
		if(Count==3)
		{
			Light_Over();
			KeyValue=Key_Transit(exKeyValueFlag);
			Light_LightOne(KeyValue);
			
			if(KeyValue==12)
			{
				Admin_Mode=0;
				Key_Again=0;
				Key_Mode=0;
				Key_Pos=0;
				Buzzer();
				delay_ms(1000);
				Buzzer();
			}
			
			if(Admin_Mode)
			{
				Key_Insert();
			}
			else if(KeyValue==10 && Admin_Mode==0)
			{
				Admin_Mode=1;
				Light_AllHighLight();
				Buzzer();
				delay_ms(800);
				Light_AllLowLight();
			}
			Key_Flag=0;
			Count=0;
		}
		Key_Flag=0;
	}
}

void Key_Insert()
{
	if(Key_Pos<9 && (Admin_Mode==0 || Admin_Mode==2))
	{
		Key_In[Key_Pos]=KeyValue;
		Key_Pos++;
		Buzzer();
		delay_ms(100);
	}
	if(Key_Pos<9 && Admin_Mode==1)
	{
		Key_Mode3_Temp[Key_Pos]=KeyValue;
		Key_Pos++;
		Buzzer();
		delay_ms(100);
	}
}

void Key_Che()
{
	if(Key_Mode==1)//密码模式以及管理员模式检查
	{
		if(Key_Pos>3)
		{
			Key_Check=1;
			for(i=0;i<4;i++)
			{
				if(Key_In[i]!=Key_Mode1[i])
				{
					Key_Check=0;
					break;
				}
			}
			for(i=0;i<4;i++)
			{
				if(Key_In[i]==Key_Mode3[i])
				{
					Key_CheckCount++;
				}
			}
			
			if(Key_CheckCount==4)
			{
				Key_Check=1;
				Key_Mode=3;
				Key_CheckCount=0;
				Light_AllHighLight();
			}
			else Key_CheckCount=0;
			
			if(Key_Check)		//密码正确
			{
				Buzzer();
				if(Key_Mode!=3) Key_Mode=0;
				Key_Time=0;
			}
			else				//密码错误
			{
				Light_Over();
				Buzzer();
				delay_ms(100);
				Light_AllHighLight();
				delay_ms(100);
				Light_Over();
				Buzzer();
				delay_ms(100);
				Light_AllHighLight();
				delay_ms(100);
				Light_Over();
				Buzzer();
				delay_ms(100);
				Light_AllHighLight();
				delay_ms(100);
				Light_Over();
				Buzzer();
				Key_Mode=0;
				Key_Time++;//次数增加
			}
			for(i=0;i<4;i++)
			{
				Key_In[i]=20;
			}
			Light_AllLowLight();
			Key_Pos=0;
		}
	}
	

	else if(Key_Mode==2)//手势识别检查
	{
		if(Key_Pos>5)
		{
			Key_Check=1;
			
			for(i=0;i<6;i++)
			{
				if(Key_In[i]!=Key_Mode2[i])
				{
					Key_Check=0;
					break;
				}
			}
				
			if(Key_Check)		//密码正确
			{
				Buzzer();
				Key_Time=0;
			}
				
			else				//密码错误
			{
				Light_Over();
				Buzzer();
				delay_ms(100);
				Light_AllHighLight();
				delay_ms(100);
				Light_Over();
				Buzzer();
				delay_ms(100);
				Light_AllHighLight();
				delay_ms(100);
				Light_Over();
				Buzzer();
				delay_ms(100);
				Light_AllHighLight();
				delay_ms(100);
				Light_Over();
				Buzzer();
					
				Key_Time++;//次数增加
			}
			for(i=0;i<6;i++)
			{
				Key_In[i]=20;
			}
			Light_AllLowLight();
			Key_Mode=0;
			Key_Pos=0;
		}
		

	}
	
	
	
	else if(Key_Mode==3)//管理员重新设置密码
	{
		if(Key_Pos>3 && Admin_Mode==1)
		{
			Key_Pos=0;
			Admin_Mode++;
			Light_AllHighLight();
			Buzzer();
			delay_ms(1000);
			Light_AllLowLight();
		}
		if(Key_Pos>3 && Admin_Mode==2)
		{
			Key_Pos=0;
			Key_Mode=0;
			for(i=0;i<4;i++)
			{
				if(Key_Mode3_Temp[i]==Key_In[i])
				{
					Key_CheckCount++;
				}
			}
			if(Key_CheckCount==4)
			{
				for(i=0;i<4;i++)
				{
					Key_Mode1[i]=Key_Mode3_Temp[i];
				}
				Light_AllHighLight();
				Buzzer();
				delay_ms(1000);
				Buzzer();
				delay_ms(1000);
				Key_CheckCount=0;
				Key_Pos=0;
				Admin_Mode=0;
				Key_Mode=0;
				Buzzer();
				delay_ms(1000);
				Light_AllLowLight();
			}
			else
			{
				Key_CheckCount=0;
				Key_Pos=0;
				Admin_Mode=0;
				Key_Mode=0;
				Buzzer();
				delay_ms(1000);
				Buzzer();
				delay_ms(1000);
				Buzzer();
				delay_ms(1000);
				
			}
		}
	}
	
	if(Key_Time==5)//检测是否输入错误次数大于五次
	{
		for(i=0;i<10;i++)
		{
			delay_ms(1000);
		}
		Key_Time=0;
	}
}

void Buzzer()
{
	PWM_CmdEX(PWM2_Type,ENABLE);
	delay_ms(50);
	PWM_CmdEX(PWM2_Type,DISABLE);
}

void WriteHigh()
{
	GPIO_WriteHigh(GPIO3, GPIO_PIN_7);
    GPIO_WriteHigh(GPIO3, GPIO_PIN_6);
    GPIO_WriteHigh(GPIO3, GPIO_PIN_5);
    GPIO_WriteHigh(GPIO3, GPIO_PIN_4);
}

```

