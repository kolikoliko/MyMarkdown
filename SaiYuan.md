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

```c
//************************************************************
//  Copyright (c) 
//	文件名称	: SCDriver_ESP8266.C
//	作者		: Andy
//	模块功能	: ESP8266 WIFI模块应用驱动程序
//  最后更正日期: 2019/1/20
// 	版本		: V0.1 
//  说明        ：该库使用AT指令实现对ESP8266 WIFI模块的控制。
//*************************************************************	
#include"..\H\SCDriver_ESP8266.h"
#include "SC_Init.h"
#include "stdio.h"
#define	SCD_ESP8266_UART0_TIMER_SELECT  UART0_CLOCK_TIMER1	 //使用UART0的时候才需要选择，使用UART1时忽略此值

#define IO_NULL  0xFF00
#define  SCD_IO_PORT(IO)    (uint8_t)(IO>>8)
#define  SCD_IO_PIN(IO)	    (uint8_t)(IO)

#define SCD_ESP8266_UART0		0x00
#define SCD_ESP8266_UART1		0x01
#define	SCD_ESP8266_MODE_STA 	'1'
#define	SCD_ESP8266_MODE_AP	  	'2'
#define	SCD_ESP8266_MODE_APSTA	'3'
#define SCD_ESP8266_OPEN	     0
#define SCD_ESP8266_WPA_PSK		 2
#define SCD_ESP8266_WPA2_PSK     3
#define SCD_ESP8266_WPA_WPA2_PSK 4
//extern unsigned char xdata SCD_ESP8266_BUFF[SCD_ESP8266_BUFF_LENGTH];//ESP8266接收缓存
unsigned char xdata *SCD_ESP8266_BuffPoint;
unsigned int SCD_ESP8266_BuffLength;
unsigned char SCD_ESP8266_BUFF_Number=0; //当前接受编号
unsigned char overFlag=0,mqttFlag,MQTT_BUFF_Number=0;
extern unsigned char MQTT_BUFF[128];

bit SCD_ESP8266_UartReceFlag = 0;
bit SCD_ESP8266_UartSendFlag = 0;
//{
//函数名  #SCD_NT_PinMode#
//函数功能#对应的数码管驱动脚输出高#
//输入参数#
//			uint16_t IO_Pxx, 			  选择需要配置的IO口
//			GPIO_Mode_TypeDef IO_Pxx_Mode 选择IO口的工作模式
//		  #
//输出参数#void#
//}
void SCD_ESP8266_PinMode(uint16_t IO_Pxx, GPIO_Mode_TypeDef IO_Pxx_Mode)
{
	if(IO_Pxx != IO_NULL)
	{
		GPIO_Init(SCD_IO_PORT(IO_Pxx), SCD_IO_PIN(IO_Pxx), IO_Pxx_Mode);	
	}	
}
//{
//函数名  #SCD_ESP8266_Delay#
//函数功能#延时函数#
//输入参数#uint16_t n  延时时间长度#
//输出参数#void#
//}
void SCD_ESP8266_Delay(uint16_t n)
{
	uint16_t  i,j;
	for(i=0;i<n;i++)
		for(j=0;j<2000;j++);
}
//{
//函数名  #SCD_ESP8266_Init#
//函数功能#该函数用于初始化ESP8266所需使用的GPIO口状态#
//输入参数#void#
//输出参数#void#
//}
void SCD_ESP8266_Init(unsigned char xdata *buffpoint,unsigned int bufflength)
{
	SCD_ESP8266_PinMode(SCD_ESP8266_TXD_INIT, GPIO_MODE_IN_PU);
	SCD_ESP8266_PinMode(SCD_ESP8266_RXD_INIT, GPIO_MODE_IN_PU);
	USCI0_UART_Init(SCD_ESP8266_FSYS, SCD_ESP8266_BAUD, USCI0_UART_Mode_10B, USCI0_UART_RX_ENABLE);
	//波特率115200，SCD_ESP8266_UART0_TIMER_SELECT做时钟源，允许接收
	USCI0_ITConfig(ENABLE, LOW);	//使能uart0中断
	SCD_ESP8266_BuffPoint = buffpoint;
	SCD_ESP8266_BuffLength= bufflength;

}
//{
//函数名  #SCD_ESP8266_Uart_Send_Byte#
//函数功能#通过UART发送一个BYTE#
//输入参数#unsigned char value#
//输出参数#void#
//}
void SCD_ESP8266_Uart_Send_Byte(unsigned char value)
{
	US0CON3 = value;
	
	while(!SCD_ESP8266_UartSendFlag);
	SCD_ESP8266_UartSendFlag = 0;
}
//{
//函数名  #SCD_ESP8266_Uart_Send_String#
//函数功能#该函数用于通过串口向ESP8266发字符串#
//输入参数#unsigned char *string 需要发送的字符串#
//输出参数#void#
//}
void SCD_ESP8266_Uart_Send_String(unsigned char *string)
{
	while(*string)
	{
		SCD_ESP8266_Uart_Send_Byte(*string++);
	}
}

//{
//函数名  #SCD_ESP8266_Uart_Send_String#
//函数功能#该函数用于通过串口向ESP8266发字符串#
//输入参数#unsigned char *string 需要发送的字符串#
//输出参数#void#
//}
void SCD_ESP8266_Uart_Send_String2(unsigned char *string,unsigned char lenth)
{
	unsigned char i;
	for(i=0;i<lenth;i++)
	{
		SCD_ESP8266_Uart_Send_Byte(string[i]);		
	}
}

//{
//函数名  #SCD_ESP8266_MODE#
//函数功能#ESP8266工作模式选择#
//输入参数#unsigned char ModeSelect 模式选择‘1’ ‘2’ ‘3’#
//输出参数#void#
//}
void SCD_ESP8266_MODE(unsigned char ModeSelect)
{
	SCD_ESP8266_Uart_Send_String("AT+CWMODE=");
	SCD_ESP8266_Uart_Send_Byte(ModeSelect); //需要填入ASII码值
	SCD_ESP8266_Uart_Send_Byte('\r');
	SCD_ESP8266_Uart_Send_Byte('\n');
}
//{
//函数名  #SCD_ESP8266_CWLAP#
//函数功能#扫描当前可用AP(WIFI)#
//输入参数#void#
//输出参数#void#
//}
void SCD_ESP8266_CWLAP()
{
	SCD_ESP8266_Uart_Send_String("AT+CWLAP\r\n");
}
//{
//函数名  #SCD_ESP8266_CWQAP#
//函数功能#断开与AP的连接#
//输入参数#void#
//输出参数#void#
//}
void SCD_ESP8266_CWQAP()
{
	SCD_ESP8266_Uart_Send_String("AT+CWQAP\r\n");	
}

//{
//函数名  #SCD_ESP8266_CWSAP_READ#
//函数功能#查看ESP8266的AP设置，通过串口获得结果#
//输入参数#void#
//输出参数#void#
//}
void SCD_ESP8266_CWSAP_READ()
{
	SCD_ESP8266_Uart_Send_String("AT+CWSAP?\r\n");
}
//{
//函数名  #SCD_ESP8266_CWSAP#
//函数功能#设置AP参数#
//输入参数#
//			unsigned char *ssid	 WIFI名
//			unsigned char *pwd	 WIFI密码
//			unsigned char chl		 通道号
//			unsigned char ecn		 加密方式$0-OPEN，1-WEP，2-WPA_PSK，3-WPA2_PSK，4-WPA_WPA2_PSK$
//		  #
//输出参数#void#
//}
void SCD_ESP8266_CWSAP(unsigned char *ssid,unsigned char *pwd,unsigned char chl,unsigned char ecn)
{
	SCD_ESP8266_Uart_Send_String("AT+CWSAP=\"");
	SCD_ESP8266_Uart_Send_String(ssid);
	SCD_ESP8266_Uart_Send_Byte('\"');
	SCD_ESP8266_Uart_Send_Byte(',');
	SCD_ESP8266_Uart_Send_Byte('\"');
	SCD_ESP8266_Uart_Send_String(pwd);
	SCD_ESP8266_Uart_Send_Byte('\"');
	SCD_ESP8266_Uart_Send_Byte(',');
	SCD_ESP8266_Uart_Send_Byte(chl+0x30);
	SCD_ESP8266_Uart_Send_Byte(',');
	SCD_ESP8266_Uart_Send_Byte(ecn+0x30);
	SCD_ESP8266_Uart_Send_Byte('\r');
	SCD_ESP8266_Uart_Send_Byte('\n');
}
//{
//函数名  #SCD_ESP8266_CWJAP#
//函数功能#连接WIFI网络#
//输入参数#
//			unsigned char *ssid	 WIFI名
//			unsigned char *pwd	 WIFI密码
//		  #
//输出参数#void#
//}
void SCD_ESP8266_CWJAP(unsigned char *ssid,unsigned char *pwd) 
{
	SCD_ESP8266_Uart_Send_String("AT+CWJAP=\"");
	SCD_ESP8266_Uart_Send_String(ssid);
	SCD_ESP8266_Uart_Send_Byte('\"');
	SCD_ESP8266_Uart_Send_Byte(',');
	SCD_ESP8266_Uart_Send_Byte('\"');
	SCD_ESP8266_Uart_Send_String(pwd);
	SCD_ESP8266_Uart_Send_Byte('\"');
	SCD_ESP8266_Uart_Send_Byte('\r');
	SCD_ESP8266_Uart_Send_Byte('\n');
}
//{
//函数名  #SCD_ESP8266_CIFSR#
//函数功能#查阅本地IP地址#
//输入参数#void#
//输出参数#void#
//}
void SCD_ESP8266_CIFSR()
{
	SCD_ESP8266_Uart_Send_String("AT+CIFSR\r\n");	
}
//{
//函数名  #SCD_ESP8266_AP_ServerSet#
//函数功能#AP模式下开启服务器#
//输入参数#
//			unsigned char server   服务器开关，1表示开，0表示关
//			unsigned char *port_t	 服务器端口号
//		  #
//输出参数#void#
//}
void SCD_ESP8266_AP_ServerSet(unsigned char server,unsigned char *port_t)	  //建立或关闭server
{
	SCD_ESP8266_Uart_Send_String("AT+CIPSERVER=");
	SCD_ESP8266_Uart_Send_Byte(server+0X30);
	SCD_ESP8266_Uart_Send_Byte(',');
	SCD_ESP8266_Uart_Send_String(port_t);
	SCD_ESP8266_Uart_Send_Byte('\r');
	SCD_ESP8266_Uart_Send_Byte('\n');
}
//{
//函数名  #SCD_ESP8266_CIPMODE#
//函数功能#选择传输模式#
//输入参数#unsigned char select  0：普通模式；1：透传模式#
//输出参数#void#
//}
void SCD_ESP8266_CIPMODE(unsigned char select)
{
	SCD_ESP8266_Uart_Send_String("AT+CIPMODE=");
	SCD_ESP8266_Uart_Send_Byte(select+0X30);
	SCD_ESP8266_Uart_Send_Byte('\r');
	SCD_ESP8266_Uart_Send_Byte('\n');
}
//{
//函数名  #SCD_ESP8266_RST#
//函数功能#复位#
//输入参数#void#
//输出参数#void#
//}
void SCD_ESP8266_RST()
{
	SCD_ESP8266_Uart_Send_String("AT+RST\r\n");	
}
//{
//函数名  #SCD_ESP8266_TEST#
//函数功能#查阅当前版本号，结果通过串口获得#
//输入参数#void#
//输出参数#void#
//}
void SCD_ESP8266_VIEW()
{
	SCD_ESP8266_Uart_Send_String("AT+GMR\n");	//查阅当前版本
}
//{
//函数名  #SCD_ESP8266_AP_Init#
//函数功能#AP模式下开启服务器#
//输入参数#
//			unsigned char *ssid	 WIFI名
//			unsigned char *pwd	 WIFI密码
//			unsigned char chl		 通道号
//			unsigned char ecn		 加密方式$0-OPEN，1-WEP，2-WPA_PSK，3-WPA2_PSK，4-WPA_WPA2_PSK$
//		  #
//输出参数#void#
//}
void SCD_ESP8266_AP_Init(unsigned char *ssid,unsigned char *pwd,unsigned char chl,unsigned char ecn) //将ESP8266设置为AP模式进行工作
{
	SCD_ESP8266_MODE(SCD_ESP8266_MODE_AP);	  //选择工作模式为AP模式
	SCD_ESP8266_Delay(1000);
	SCD_ESP8266_RST();
	SCD_ESP8266_Delay(3000);//复位
	SCD_ESP8266_CWSAP(ssid,pwd,chl,ecn);
	SCD_ESP8266_Delay(1000);
	SCD_ESP8266_Uart_Send_String("AT+CIPMUX=1\r\n");//1启动多连接,0启动单连接
	SCD_ESP8266_Delay(1000);
}
//{
//函数名  #SCD_ESP8266_STA_LinkServer#
//函数功能#AP模式下开启服务器#
//输入参数#
//			unsigned char *IP_t    服务器IP地址
//			unsigned char *port_t	 服务器端口号
//		  #
//输出参数#void#
//}
void SCD_ESP8266_STA_LinkServer(unsigned char *IP_t,unsigned char *port_t)//连接服务器
{
	SCD_ESP8266_Uart_Send_String("AT+CIPSTART=\"TCP\"");
	SCD_ESP8266_Uart_Send_Byte(',');
	SCD_ESP8266_Uart_Send_Byte('\"');
	SCD_ESP8266_Uart_Send_String(IP_t);
	SCD_ESP8266_Uart_Send_Byte('\"');
	SCD_ESP8266_Uart_Send_Byte(',');
	SCD_ESP8266_Uart_Send_String(port_t);
	SCD_ESP8266_Uart_Send_Byte('\r');
	SCD_ESP8266_Uart_Send_Byte('\n');
	SCD_ESP8266_Delay(1000);
}
//{
//函数名  #SCD_ESP8266_STA_Init#
//函数功能#将ESP8266设置为STA模式进行工作#
//输入参数#
//			unsigned char *ssid	 WIFI名
//			unsigned char *pwd	 WIFI密码
//		  #
//输出参数#void#
//}
void SCD_ESP8266_STA_Init(unsigned char *ssid,unsigned char *pwd) //将ESP8266设置为STA模式进行工作
{
	SCD_ESP8266_MODE(SCD_ESP8266_MODE_STA);	  //选择工作模式为STA模式
	SCD_ESP8266_Delay(1000);
	SCD_ESP8266_RST();
	SCD_ESP8266_Delay(3000);//复位
	SCD_ESP8266_CWJAP(ssid, pwd);
	SCD_ESP8266_Delay(1000);
}
//{
//函数名  #SCD_ESP8266_AP_SendData#
//函数功能#在AP模式下发送数据#
//输入参数#
//			unsigned char Addr		发送地址编号
//			unsigned char DataLength	发送数据长度
//			unsigned char*Data		发送数据
//		  #
//输出参数#void#
//}
void SCD_ESP8266_AP_SendData(unsigned char Addr,unsigned char DataLength,unsigned char*Data)
{
	SCD_ESP8266_Uart_Send_String("AT+CIPSEND=");
	SCD_ESP8266_Uart_Send_Byte(Addr+0X30);
	SCD_ESP8266_Uart_Send_Byte(',');
	SCD_ESP8266_Uart_Send_Byte(DataLength+0X30);
	SCD_ESP8266_Uart_Send_Byte('\r');
	SCD_ESP8266_Uart_Send_Byte('\n');
	SCD_ESP8266_Delay(10);
	SCD_ESP8266_Uart_Send_String(Data);
}
//{
//函数名  #SCD_ESP8266_STA_SendData#
//函数功能#在STA模式下发送数据#
//输入参数#
//			unsigned char DataLength	发送数据长度
//			unsigned char*Data		发送数据
//		  #
//输出参数#void#
//}
void SCD_ESP8266_STA_SendData(unsigned char DataLength,unsigned char*Data)
{
	SCD_ESP8266_Uart_Send_String("AT+CIPSEND=");
	SCD_ESP8266_Uart_Send_Byte(DataLength+0X30);
	SCD_ESP8266_Uart_Send_Byte('\r');
	SCD_ESP8266_Uart_Send_Byte('\n');
	SCD_ESP8266_Delay(10);
	SCD_ESP8266_Uart_Send_String(Data);
}
//{
//函数名  #SCD_ESP8266_STA_PassThrough#
//函数功能#STA模式下的透传开关#
//输入参数#
//			FunctionalState Start	透传开关$FunctionalState$
//		  #
//输出参数#void#
//}
void SCD_ESP8266_STA_PassThrough(FunctionalState Start)
{
	if(Start)
	{
	   SCD_ESP8266_Uart_Send_String("AT+CIPMODE=1\r\n"); //使能透传模式
	   SCD_ESP8266_Delay(1000);
	   SCD_ESP8266_Uart_Send_String("AT+CIPSEND\r\n");	 //进入发送数据模式
	   SCD_ESP8266_Delay(1000);
	}
	else
	{
	   SCD_ESP8266_Uart_Send_String("+++");
	}
	
}
//{
//函数名  #SCD_ESP8266_BUFF_Clear#
//函数功能#清空接收缓存#
//输入参数#void#
//输出参数#void#
//}
void SCD_ESP8266_BUFF_Clear()
{
	unsigned char i;
	for(i=0;i<SCD_ESP8266_BuffLength;i++)
	{
	   SCD_ESP8266_BuffPoint[i] = 0;
	}
	SCD_ESP8266_BUFF_Number=0;
}
//{
//函数名  #MQTT_BUFF_Clear#
//函数功能#清空接收缓存#
//输入参数#void#
//输出参数#void#
//}
void MQTT_Clear()
{
	unsigned char i;
	for(i=0;i<128;i++)
	{
	   MQTT_BUFF[i] = 0;
	}
	MQTT_BUFF_Number=0;
}
//{
//函数名  #SCD_ESP8266_BUFF_Update#
//函数功能#接收缓存更新#
//输入参数#void#
//输出参数#void#
//}
void SCD_ESP8266_BUFF_Update()
{
    SCD_ESP8266_BUFF_Number++;
	if(SCD_ESP8266_BUFF_Number>=SCD_ESP8266_BuffLength)
	{
		SCD_ESP8266_BUFF_Number	= 0;
		
		SCD_ESP8266_BUFF_Clear();//清空接收缓冲区
	}
}
//{
//函数名  #SCD_ESP8266_InterruptWork#
//函数功能#该函数必须放于对应的UART中断函数内，用于清中断和接收数据#
//输入参数#unsigned char *string 需要发送的字符串#
//输出参数#void#
//}
void SCD_ESP8266_InterruptWork()
{
	if(USCI0_GetFlagStatus(USCI0_UART_FLAG_RI))
	{
		USCI0_ClearFlag(USCI0_UART_FLAG_RI);
		SCD_ESP8266_UartReceFlag = 1;
		SCD_ESP8266_BuffPoint[SCD_ESP8266_BUFF_Number] = USCI0_UART_ReceiveData8();			
		
		if(SCD_ESP8266_BuffPoint[SCD_ESP8266_BUFF_Number]=='{') mqttFlag=1;
		else if(SCD_ESP8266_BuffPoint[SCD_ESP8266_BUFF_Number]=='}')
		{
			overFlag=1;
			mqttFlag=0;
			MQTT_BUFF_Number=0;
		}
		
		if(mqttFlag==1)MQTT_BUFF[MQTT_BUFF_Number++]= USCI0_UART_ReceiveData8();
		
		SCD_ESP8266_BUFF_Update();
	}
	if(USCI0_GetFlagStatus(USCI0_UART_FLAG_TI))
	{
		USCI0_ClearFlag(USCI0_UART_FLAG_TI);
		SCD_ESP8266_UartSendFlag = 1;
	}
}

//==========================================================
//	函数名称：	ESP8266_SendData
//
//	函数功能：	发送数据
//
//	入口参数：	data：数据
//				len：长度
//
//	返回参数：	无
//
//	说明：		
//==========================================================
void ESP8266_SendData(unsigned char *_data, unsigned short len)
{

	char cmdBuf[32];
	
	SCD_ESP8266_BUFF_Clear();							//清空接收缓存
	sprintf(cmdBuf, "AT+CIPSEND=%d\r\n", len);		//发送命令
	SCD_ESP8266_Uart_Send_String(cmdBuf);

	SCD_ESP8266_Delay(1000);
	SCD_ESP8266_Delay(1000);
	SCD_ESP8266_Delay(1000);
	SCD_ESP8266_Delay(100);
	SCD_ESP8266_Uart_Send_String2(_data,len);
	//Usart_SendString(USART2, data, len);		//发送设备连接请求数据
}
					 

```

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
#include "TK_Key.h"
#include "oled.h"
#include "light.h"
#include "delay.h"
/**************************************Generated by EasyCodeCube*************************************/

/*************************************.Generated by EasyCodeCube.************************************/
/*****************************************************************************************************
* Function Name: main
* Description  : This function implements main function.
* Arguments    : None
* Return Value : None
******************************************************************************************************/
#include "stdio.h"

char putchar(char c)
{
	UART0_SendData8(c);
	while(!Uart0SendFlag);
	Uart0SendFlag = 0;
	return c;
}

unsigned char j;
extern unsigned char mqttFlag,keyWrong,revFlag;
void main(void)
{
    /*<Generated by EasyCodeCube begin>*/
    /*<UserCodeStart>*//*<SinOne-Tag><186>*/    
    SC_Init();
    PWM_IndependentModeConfigEX(PWM00,0, PWM_OUTPUTSTATE_ENABLE);
    PWM_CmdEX(PWM2_Type,DISABLE);
    /*<UserCodeEnd>*//*<SinOne-Tag><186>*/
    /*<UserCodeStart>*//*<SinOne-Tag><9>*/
    OLED_Init();
    TouchKeyInit();
    Light_AllLowLight();
	MQTT_Init();
	
	OLED_Clear();
    for(j=0;j<4;j++)
    {
        OLED_ShowChar((j+1)*24,18,'*',24);
    }
    OLED_Refresh();
    
	
    KeyReadRom();
    /*<UserCodeEnd>*//*<SinOne-Tag><9>*/
    /*<UserCodeStart>*//*<SinOne-Tag><10>*/
   
    while(1)
    {
        if(keyWrong<4)
		{
			TK_Key();
		}
		if(revFlag==1)
		{
			OneNet_RevSub();
			revFlag=0;
		}
    }
}

```

```C
//************************************************************
//  Copyright (c)   
//	FileName	  : SC_it.c
//	Function	  : Interrupt Service Routine
//  Instructions  :  
//  Date          : 2022/03/03
// 	Version		  : V1.0002
//*************************************************************
/********************Includes************************************************************************/
#include "SC_it.h"
#include "..\Drivers\SCDriver_list.h"
#include "HeadFiles\SC_itExtern.h"
/**************************************Generated by EasyCodeCube*************************************/

/*************************************.Generated by EasyCodeCube.************************************/
void INT0Interrupt()		interrupt 0				
{
    TCON &= 0XFD;//Clear interrupt flag bit
    /*INT0_it write here begin*/
    /*INT0_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*INT0Interrupt Flag Clear begin*/
    /*INT0Interrupt Flag Clear end*/
}
void Timer0Interrupt()		interrupt 1			   
{
    /*TIM0_it write here begin*/
    TIM0_Mode1SetReloadCounter(38869);
    /*TIM0_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*Timer0Interrupt Flag Clear begin*/
    /*Timer0Interrupt Flag Clear end*/		
}
void INT1Interrupt()		interrupt 2		
{
    TCON &= 0XF7;//Clear interrupt flag bit
    /*INT1_it write here begin*/
    /*INT1_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*INT1Interrupt Flag Clear begin*/
    /*INT1Interrupt Flag Clear end*/					
}
void Timer1Interrupt()		interrupt 3		
{
    /*TIM1_it write here begin*/
    /*TIM1_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*Timer1Interrupt Flag Clear begin*/
    /*Timer1Interrupt Flag Clear end*/		
}
#if defined (SC92F742x) || defined (SC92F7490)
void SSI0Interrupt()		interrupt 4		
{
    /*SSI0_it write here begin*/
    /*SSI0_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*SSI0Interrupt Flag Clear begin*/
    /*SSI0Interrupt Flag Clear end*/		
}
#else
void UART0Interrupt()		interrupt 4		
{
    /*UART0_it write here begin*/
    /*UART0_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*UART0Interrupt Flag Clear begin*/
	Uart0SendFlag = 1;
    UART0_ClearFlag(UART0_FLAG_TI);
    UART0_ClearFlag(UART0_FLAG_RI);
    /*UART0Interrupt Flag Clear end*/		
}
#endif
void Timer2Interrupt()		interrupt 5		
{
    /*TIM2_it write here begin*/
    /*TIM2_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*Timer2Interrupt Flag Clear begin*/
    PWM_ClearFlagEX(PWM2_Type);
    /*Timer2Interrupt Flag Clear end*/		
}
void ADCInterrupt()			interrupt 6		
{
    /*ADC_it write here begin*/
    /*ADC_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*ADCInterrupt Flag Clear begin*/
    /*ADCInterrupt Flag Clear end*/		
}
#if defined (SC92F854x) || defined (SC92F754x) ||defined  (SC92F844xB) || defined (SC92F744xB)||defined  (SC92F84Ax_2) || defined (SC92F74Ax_2)|| defined (SC92F846xB) \
|| defined (SC92F746xB) || defined (SC92F836xB) || defined (SC92F736xB) || defined (SC92F8003)||defined  (SC92F84Ax) || defined (SC92F74Ax) || defined  (SC92F83Ax) \
|| defined (SC92F73Ax) || defined (SC92F7003) || defined (SC92F740x) || defined (SC92FWxx) || defined (SC93F743x) || defined (SC93F833x) || defined (SC93F843x)\
|| defined (SC92F848x) || defined (SC92F748x)|| defined (SC92F859x) || defined (SC92F759x)
void SSIInterrupt()			interrupt 7		
{
    /*SSI_it write here begin*/
    /*SSI_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*SSIInterrupt Flag Clear begin*/
    /*SSIInterrupt Flag Clear end*/		
}
#elif defined (SC92F742x) || defined (SC92F7490)
void SSI1Interrupt()		interrupt 7		
{
    /*SSI1_it write here begin*/
    /*SSI1_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*SSI1Interrupt Flag Clear begin*/
    /*SSI1Interrupt Flag Clear end*/		
}
#else 
void USCI0Interrupt()			interrupt 7		
{
    /*USCI0_it write here begin*/
    /*USCI0_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*USCI0Interrupt Flag Clear begin*/
    SCD_ESP8266_InterruptWork();
	if(USCI0_GetFlagStatus(USCI0_UART_FLAG_RI))
	{
		USCI0_ClearFlag(USCI0_UART_FLAG_RI);
	}
	if(USCI0_GetFlagStatus(USCI0_UART_FLAG_TI))
	{
		USCI0_ClearFlag(USCI0_UART_FLAG_TI);
	}
    /*USCI0Interrupt Flag Clear end*/		
}
#endif
void PWMInterrupt()			interrupt 8
{
    /*PWM_it write here begin*/
    /*PWM_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*PWMInterrupt Flag Clear begin*/
    /*PWMInterrupt Flag Clear end*/		
}
#if !defined (TK_USE_BTM)
void BTMInterrupt()			interrupt 9
{
    /*BTM_it write here begin*/
    /*BTM_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*BTMInterrupt Flag Clear begin*/
    /*BTMInterrupt Flag Clear end*/		
}
#endif
void INT2Interrupt()		interrupt 10
{	
    /*INT2_it write here begin*/
    /*INT2_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*INT2Interrupt Flag Clear begin*/
    /*INT2Interrupt Flag Clear end*/		
}
void ACMPInterrupt()		interrupt 12
{
    /*ACMP_it write here begin*/
    /*ACMP_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*ACMPInterrupt Flag Clear begin*/
    /*ACMPInterrupt Flag Clear end*/		
}
void Timer3Interrupt()		interrupt 13
{
    /*Timer3_it write here begin*/
    /*Timer3_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*Timer3Interrupt Flag Clear begin*/
    /*Timer3Interrupt Flag Clear end*/		
}
void Timer4Interrupt()		interrupt 14
{
    /*Timer4_it write here begin*/
    /*Timer4_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*Timer4Interrupt Flag Clear begin*/
    /*Timer4Interrupt Flag Clear end*/		
}
void USCI1Interrupt()		interrupt 15		
{	
    /*USCI1_it write here begin*/
    /*USCI1_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*USCI1Interrupt Flag Clear begin*/
    /*USCI1Interrupt Flag Clear end*/		
}
void USCI2Interrupt()		interrupt 16		
{	
    /*USCI2_it write here begin*/
    /*USCI2_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*USCI2Interrupt Flag Clear begin*/
    /*USCI2Interrupt Flag Clear end*/		
}
void USCI3Interrupt()		interrupt 17		
{
    /*USCI3_it write here begin*/
    /*USCI3_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*USCI3Interrupt Flag Clear begin*/
    /*USCI3Interrupt Flag Clear end*/		
}
void USCI4Interrupt()		interrupt 18		
{
    /*USCI4_it write here begin*/
    /*USCI4_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*USCI4Interrupt Flag Clear begin*/
    /*USCI4Interrupt Flag Clear end*/		
}
void USCI5Interrupt()		interrupt 19		
{
    /*USCI5_it write here begin*/	
    /*USCI5_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*USCI5Interrupt Flag Clear begin*/
    /*USCI5Interrupt Flag Clear end*/		
}
void LPDInterrupt()		interrupt 22		
{
    /*LPD_it write here begin*/
    /*LPD_it write here*/
    /*<Generated by EasyCodeCube begin>*/
    /*<Generated by EasyCodeCube end>*/
    /*LPDInterrupt Flag Clear begin*/
    /*LPDInterrupt Flag Clear end*/		
}

```

```C
#include "SC_Init.h"
#include "SC_it.h"
#include "SCDriver_list.h"
#include "SysFunVarDefine.h"

#include "oled.h"
#include "TK_Key.h"
#include "delay.h"
#include "Light.h"
#include "Key.h"

#define   Key1_ROM    64522   //记录密码在ROM中的地址
#define   Key2_ROM    64622
#define   Key3_ROM    64722

unsigned char i;
unsigned char exCount;			//抵消按键检测次数
unsigned char firstCount;		//计算第一个与第二个按键之间间隔时间

unsigned char keyMode;			//输入模式    0:未选择  1：四位按键密码  2：六位手势识别密码
unsigned char keyCAllow=0;		//密码可修改模式
unsigned char keyGet;			//获取键值缓存
uint8_t keyIn[6];				//输入的密码
unsigned char keyPos=0;			//密码坐标

uint8_t key1[4]={1,2,3,4};			//四位按键密码
uint8_t key2[6]={1,2,3,4,5,6};		//六位手势识别密码
uint8_t key3[4]={1,1,1,4};			//四位管理员密码
uint8_t key4[4];					//游客访问临时密码
uint8_t keyTemp[6];					//修改密码时的缓存密码

unsigned char key4_allow=0;		//游客访问允许
unsigned char keyWrong=0;			//错误识别次数

unsigned char keyCheck;

unsigned char LowPowerMode;

/**
  * @brief  在main中做循环，检测密码，检测模式等等所有操作
  * @param  无
  * @retval 无
  */
void TK_Key()
{
	TK_Touch();
}

/**
  * @brief  魔盒生成的tk触控检测代码
  * @param  无
  * @retval 无
  */
void TK_Touch()
{
    if(LowPowerMode)
    {
		keyPos=0;
		
		OLED_Clear();
		for(i=0;i<4;i++)
		{
			OLED_ShowChar((i+1)*24,18,'*',24);
		}
		OLED_Refresh();

        Light_Over();
		LowPowerMode=0;
    }
	

        if(SOCAPI_TouchKeyStatus & 0x40)
        {
            SOCAPI_TouchKeyStatus &= 0xbf;
            TouchKeyRestart();
        }
        if(SOCAPI_TouchKeyStatus & 0x80)
        {
            SOCAPI_TouchKeyStatus &= 0x7f;
            exKeyValueFlag = TouchKeyScan();
			
            if(exKeyValueFlag == 0)
            {		
                TK_NoKeyCount++;	//计数
				firstCount++;
                if(TK_NoKeyCount  > 4300)
                {
                    TK_NoKeyCount = 0;
                    LowPowerMode=1;
                }
            }
            else
            {
				exCount++;			//抵消检测
                TK_NoKeyCount = 0;
            }
			MyCode();
            TouchKeyRestart();
        }
}


/**
  * @brief  逻辑处理
  * @param  
  * @retval 
  */
void Mycode(){
	if(exCount==2){	//获取按键值
		exCount=0;	//清除exCount计数
		
		
		if(keyPos==0)
		{
			Buzzer(); 	//蜂鸣器短响
			keyGet=Key_Transit(exKeyValueFlag);
			if( keyGet==10 && keyCAllow==1 ){
				//进入修改密码环节		
				OLED_Clear();
				OLED_ShowString(18,12,"change",24);
				OLED_Refresh();
				Light_AllHighLight();
				delay_ms(1000);
				
				OLED_Clear();
				for(i=0;i<4;i++)
				{
					OLED_ShowChar((i+1)*24,18,'*',24);
				}
				OLED_Refresh();
				
				keyCAllow=2;
			}else if(keyGet==12){
				if(keyCAllow==1)
				{
					keyCAllow=0;
				}
				return;				
			}else{
				if(keyCAllow==1)
				{
					keyCAllow=0;
				}
				
				keyIn[keyPos++]=keyGet;	
				keyMode=1;				
			}		
		}
		else if(keyPos==1)
		{
			keyIn[keyPos++]=Key_Transit(exKeyValueFlag);
			Buzzer(); 	//蜂鸣器短响
			if(firstCount<100 && (keyIn[0]!=keyIn[1]))
			{
				keyMode=2;
				OLED_Clear();
				OLED_ShowString(24,18,"slide",24);
				OLED_Refresh();
			}
		}
		else
		{
			switch (keyMode)
            {
            	case 1:
					KeyIn1();
            		break;
            	case 2:
					KeyIn2();
            		break;
				case 3:
            		break;
            	default:
            		break;
            }
		}
		firstCount = 0;		//清计数
		ShowIn();
	}

}

/**
  * @brief 四位按键输入模式输入 
  * @param  
  * @retval 
  */
void KeyIn1()
{
	Buzzer(); 	//蜂鸣器短响
	if(Key_Transit(exKeyValueFlag)==12)//确认键后检查
	{
		KeyCheck(1);
	}
	else if(Key_Transit(exKeyValueFlag)==10)
	{
		keyPos=0;
		
		for(i=0;i<4;i++)
		{
			OLED_ShowChar((i+1)*24,18,'*',24);
		}
		OLED_Refresh();
	}
	else if(keyPos<=3)
	{
		keyIn[keyPos++]=Key_Transit(exKeyValueFlag);
	}
	Light_AllLowLight();
}

/**
  * @brief 六位按键输入模式
  * @param  
  * @retval 
  */
void KeyIn2()
{
	if(keyPos<6)
	{
		if(Key_Transit(exKeyValueFlag)!=keyIn[keyPos-1])
		{
			Buzzer(); 	//蜂鸣器短响
			keyIn[keyPos++]=Key_Transit(exKeyValueFlag);
		}

		if(keyPos==6)
		{
			KeyCheck(2);
		}
	}
	
}

/**
  * @brief  密码检查()
  * @param  
  * @retval 
  */
void KeyCheck(unsigned char mode)
{
	keyMode=0;

	if(mode==1 && keyCAllow==0)
	{
		//先进行管理员密码检测
		keyCheck=1;
		for(i=0;i<4;i++)
		{
			if(keyIn[i]!=key3[i])
			{
				keyCheck=0;
				break;
			}
		}
		
		//在进行普通密码密码检测
		if(keyCheck==0)
		{
			keyCheck=1;
			for(i=0;i<4;i++)
			{
				if(keyIn[i]!=key1[i])
				{
					keyCheck=0;
					break;
				}
			}
		}
		else//管理员密码通过
		{
			keyCAllow=1;//开启允许修改密码
			ShowResult(1);
			goto flag;
		}
		
		if(keyCheck==0 && key4_allow==1)
		{
			keyCheck=1;
			for(i=0;i<4;i++)
			{
				if(keyIn[i]!=key4[i])
				{
					keyCheck=0;
					break;
				}
			}
			key4_allow=0;
		}

		
		if(keyCheck)
		{
			ShowResult(1);
		}
		else
		{
			ShowResult(0);
		}
		flag:;
		
	}
	else if (mode==2 && keyCAllow==0)
	{
		//进行手势密码检测
		keyCheck=1;
		for(i=0;i<6;i++)
		{
			if(keyIn[i]!=key2[i])
			{
				keyCheck=0;
				break;
			}
		}
		if(keyCheck)
		{
			ShowResult(1);
		}
		else
		{
			ShowResult(0);
		}
	}
	else if (mode==1 && keyCAllow==2)
	{
		for(i=0;i<4;i++) keyTemp[i]=keyIn[i];
		keyCAllow=3;
		OLED_Clear();
		OLED_ShowString(24,12,"again",24);
		OLED_Refresh();
		delay_ms(1000);
		
		OLED_Clear();
		for(i=0;i<4;i++)
		{
			OLED_ShowChar((i+1)*24,18,'*',24);
		}
		OLED_Refresh();
		
		keyPos=0;
	}
	else if (mode==2 && keyCAllow==2)
	{
		for(i=0;i<6;i++) keyTemp[i]=keyIn[i];
		keyCAllow=3;
		OLED_Clear();
		OLED_ShowString(24,12,"again",24);
		OLED_Refresh();
		delay_ms(1000);
		
		OLED_Clear();
		for(i=0;i<4;i++)
		{
			OLED_ShowChar((i+1)*24,18,'*',24);
		}
		OLED_Refresh();
		
		keyPos=0;
	}else if (mode==1 && keyCAllow==3)
	{
		keyCheck=1;
		
		for(i=0;i<4;i++) //判断再次输入的密码是否正确
		{
			if(keyTemp[i]!=keyIn[i]) keyCheck=0;
		}
		
		if(keyCheck){	//两次密码确认正确
			OLED_Clear();
			OLED_ShowString(24,12,"right",24);
			OLED_Refresh();
			delay_ms(1000);
			
			for(i=0;i<4;i++) key1[i]=keyIn[i];	//密码重写
			KeyWriteRom(1);						//密码存进rom
			
			OLED_ShowString(12,12,"overwrite",24);//显示操作
			OLED_Refresh();
			delay_ms(1000);
			
			OLED_Clear();//清屏
			for(i=0;i<4;i++)
			{
				OLED_ShowChar((i+1)*24,18,'*',24);
			}
			OLED_Refresh();
			OLED_Refresh();
			
		}else{			//两次密码确认错误
			OLED_Clear();
			OLED_ShowString(24,12,"wrong",24);
			OLED_Refresh();
			delay_ms(1000);			
		}
		
		keyCAllow=0;
		keyPos=0;
	}
	else if (mode==2 && keyCAllow==3)
	{
		keyCheck=1;
		
		for(i=0;i<6;i++) //判断再次输入的密码是否正确
		{
			if(keyTemp[i]!=keyIn[i]) keyCheck=0;
		}
		
		if(keyCheck){	//两次密码确认正确
			OLED_Clear();
			OLED_ShowString(24,12,"right",24);
			OLED_Refresh();
			delay_ms(1000);
			
			for(i=0;i<4;i++) key1[i]=keyIn[i];	//密码重写
			KeyWriteRom(2);						//密码存进rom、
			
			OLED_ShowString(12,12,"overwrite",24);//OLED显示
			OLED_Refresh();
			delay_ms(1000);
			
			OLED_Clear();
			for(i=0;i<4;i++)
			{
				OLED_ShowChar((i+1)*24,12,'*',24);
			}
			OLED_Refresh();
		}else{			//两次密码确认错误
			OLED_Clear();
			OLED_ShowString(24,12,"wrong",24);
			OLED_Refresh();
			delay_ms(1000);	

			OLED_Clear();
			for(i=0;i<4;i++)
			{
				OLED_ShowChar((i+1)*24,18,'*',24);
			}
			OLED_Refresh();
		}
		
		keyCAllow=0;
		keyPos=0;
	}
	
	
}

/**
  * @brief  做出密码输入的反馈
  * @param  
  * @retval 
  */
void ShowIn()
{
	if(keyMode==1)
	{
		for(i=0;i<keyPos;i++)
		{
			OLED_ShowNum((i+1)*24,18,keyIn[i],1,24);
			OLED_Refresh();
		}
		Light_AllLowLight();
	}
}


/**
  * @brief  做出密码判断的反馈
  * @param  
  * @retval 
  */
void ShowResult(unsigned char flag)
{
	if(flag)//正确
	{
		OLED_Clear();
		OLED_ShowString(24,18,"welcome",24);
		OLED_Refresh();
		Buzzer();
		delay_ms(100);
		Buzzer();
		Light_AllHighLight();
		

		delay_ms(200);
		LowPowerMode=1;
		keyPos=0;
	}
	else
	{
		OLED_Clear();
		OLED_ShowString(24,18,"wrong",24);
		OLED_Refresh();
		
		Buzzer();Light_AllHighLight();//错误提示响4次，闪3次
		delay_ms(100);
		Buzzer();Light_AllLowLight();
		delay_ms(100);
		Buzzer();Light_AllHighLight();
		delay_ms(100);
		Buzzer();Light_AllLowLight();
		delay_ms(100);
		Light_AllHighLight();
		delay_ms(100);
		Light_AllLowLight();
		
		keyPos=0;
		
		OLED_Clear();
		for(i=0;i<4;i++)
		{
			OLED_ShowChar((i+1)*24,18,'*',24);
		}
		OLED_Refresh();

        Light_Over();
		
		keyWrong++;
		if(keyWrong>=4)
		{
			OLED_Clear();
			OLED_ShowString(24,18,"LOCK",24);
			OLED_Refresh();
		}
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
        for(i=0;i<4;i++) IAP_ProgramByte(Key1_ROM+i,key1[i],IAP_MEMTYPE_ROM,0xf0);
    }
    else if(type==2)//写密码2
    {
        for(i=0;i<6;i++) IAP_ProgramByte(Key2_ROM+i,key2[i],IAP_MEMTYPE_ROM,0xf0);
    }
    else if(type==3)//写密码3
    {
        for(i=0;i<4;i++) IAP_ProgramByte(Key3_ROM+i,key3[i],IAP_MEMTYPE_ROM,0xf0);
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
/**
  * @brief  读取rom里面的密码，注意开启烧录设置里的IAP ALL
  * @param  
  * @retval 
  */
void KeyReadRom()
{
    for(i=0;i<4;i++) key1[i]=IAP_ReadByte(Key1_ROM+i, IAP_MEMTYPE_ROM);
    for(i=0;i<6;i++) key2[i]=IAP_ReadByte(Key2_ROM+i, IAP_MEMTYPE_ROM);
    for(i=0;i<4;i++) key3[i]=IAP_ReadByte(Key3_ROM+i, IAP_MEMTYPE_ROM);
}


```

