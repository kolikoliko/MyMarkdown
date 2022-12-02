# STM32CUBEMX 使用笔记

## 基本操作

---

> 1.  打开 `rcc` 的高速时钟
> 2.  配置时钟树为最大的`72mhz`
> 3.  `project`上选择开发的`mdk`编辑器
> 4.  `code generator`上勾选第一个
> 5.  最好打开 degug

---

## 库函数代码使用

### PWM

> **频率**=定时器时钟/(pre 预分频+1）/(count period 计数值+1)

> **占空比**=Pulse 对比值/(count period 计数值)%

        HAL_TIM_PWM_Start(&htim2,TIM_CHANNEL_1);//定时器通道使能
        __HAL_TIM_SET_COMPARE(&htim2,TIM_CHANNEL_1,z);//修改占空比，使用z值来修改

### ADC 轮询采集

### OLED 显示

> 参考网站：https://blog.csdn.net/qq_39542860/article/details/105907958

注意：

- 包含头文件`oled.h`
- 初始化`OLED.Init()`
- `OLED_Clear(); `清屏操作
- `OLED_Display_On()；`操作打开显示屏！！！很重要(之前漏加了导致一直没有上电显示)，后面则不需要类似 _reflesh_ 的操作
- C6T6 的板子 Cubemx 有一个很傻逼的 bug，在 i2c 那里要修改一行代码` GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;`
- **最好不要用 oled 来 debug（OLED 的代码过长，如果在串口使用时，消息会丢失）**

### HC-05 蓝牙模块

> 参考资料：https://blog.csdn.net/qq_38410730/article/details/80368485 > https://blog.csdn.net/lzzzzzzm/article/details/114916509

默认波特率：9600
AT 模式下波特率：38400

### USART 串口收发

> 参考资料：https://www.bilibili.com/video/BV1q4411d7RX?p=9&vd_source=0eab86b58f0ee8a55b25e7648743b65a

- 阻塞式收发（必须要等收发完成后才执行下一步）

        uint8_t temp[] = "test";
        HAL_UART_Transmit(&huart2,temp,5,50);

- `printf` 重定向

                  int fputc(int ch, FILE *f){
                  uint8_t temp[1] = {ch};
                  HAL_UART_Transmit(&huart|, temp ,1, 2);
                  return ch;
                  }

  ！！！使用重定向必须勾选 `MicroLIB` (搞了我好久)

在串口收发的是时候不要轻易加操作，可能会触发中断导致接收异常（比如在接收里面加串口的发送）

### IO 口操作

> `Input` 操作：

        HAL_GPIO_ReadPin(GPIOA,GPIO_PIN_13)

> `Output` 操作

        HAL_GPIO_WritePin(GPIOA,GPIO_PIN_8,GPIO_PIN_RESET)
        HAL_GPIO_WritePin(GPIOA,GPIO_PIN_8,GPIO_PIN_SET)

### 矩阵键盘

- 可参考 22 年的省赛作品

扫描检测，一边使用推挽输出，一边使用 Input，上拉，按键按下时，下降沿触发中断，结合宏定义，节省代码时间，建议搭配定时器中断使用。

                A1;B1;C1;			//输出IO口置高，然后在进行检测
            uint8_t KeyNum=0;

                A0;					//A端IO口拉低检测
                if(a==0){KeyNum=1;HAL_Delay(20);while(a==0)HAL_Delay(20);}
                //下降沿触发，20ms消抖
                if(b==0){KeyNum=4;HAL_Delay(20);while(b==0)HAL_Delay(20);}
                if(c==0){KeyNum=7;HAL_Delay(20);while(c==0)HAL_Delay(20);}
                if(d==0){KeyNum=10;HAL_Delay(20);while(d==0)HAL_Delay(20);}
                A1;

### 定时器的中断

> cubemx 层

- 选择定时器
- 选择 `Internal clock` 配置预分频值和计数值（具体可参考 pwm 输出）
- 打开定时器中断，配置优先级。

> keil 层

- `HAL_TIM_Base_Start_IT(&htim2);` 打开定时器中断（**写在 tim 初始化函数的下面**）
- 写定时器中断回调函数

        void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
        {
                static unsigned char ledState = 0;
                if (htim == (&htim2))
                {

                }
        }

### 网页控制 stm32

https://www.jianshu.com/p/46cb78f88d51
