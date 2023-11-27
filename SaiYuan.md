# Saiyuan代码

```c
infrare_send_on;
				if(infrare_not_receive){
                    needfood=0; 
                }
                else{
                    needfood=1;
                }
                //infrare_send_off;
                //缺粮
                if(needfood==1){
                    LED2_on;	
                    song_nanting();
                }
                if(needfood==0){
                    LED2_off;	
                }
                //缺水
                if(needwater==1){
                    LED3_on;	         
                    song_nanting();
                }
                if(needwater==0){
                    LED3_off;	
                }
                //网络状态
                if(internet==1){//连上了
                    LED1_on;
                }
                if(internet==0){//连不上
                    LED1_off;
                }
                //该吃饭了
                if(eat_time==1){      
                    song_guyong();
                }
                //-----------------会有延时? 因为中间如果要播音乐就会来不及控制它关？ 
                //到时间自动加粮--电磁铁
                if(addfood==1){
                    magnet_on;//电磁铁开
                }
                if(addfood==0){
                    magnet_off;
                } 
                //到时间自动加水--水泵
                if(addwater==1){

                }
                if(addwater==0){
                    PWM_IndependentModeConfig(PWM30,0);//占空比长度0
                }
                //初始页面：宠物自动喂食机
                if(oled==0){
                    oled0();
                }
                //童锁 (oled1
                if (SELECT && oled==0)
                {
                    Delay(20);
                    while (SELECT);
                    Delay(20);
                    oled=1;
                    OLED_Clear();
                }
                //1.选择功能：setting or power
                if(oled==1){   
                    oled1();
                }
                //1.1 选择了设定时间 oled2
                if (SETTING&& oled==1)
                {
                    Delay(20);
                    while (SETTING);
                    Delay(20);
                    oled=2;
                    OLED_Clear();
                }
                //1.1.1 选择设定喂食or喝水
                if(oled==2){
                    oled2();
                }
                //1.1.1 确定喂食 oled3
                if (CONFIRM&& oled==2)
                {
                    Delay(20);
                    while (CONFIRM);
                    Delay(20);
                    oled=3;
                    OLED_Clear();
                }
                //1.1.1.1 请设置喂食时间(时）
                if(oled==3){
                    oled3(hour,min);
                }
                //1.1.1.1.1 (+1)
                if (SETTING&& oled==3)//setting+1
                {
                    Delay(20);
                    while (SETTING);
                    Delay(20);
                    oled=3;
                    hour++;
                }
                //1.1.1.1.2 (-1)
                if (POWER&& oled==3)//power-1
                {
                    Delay(20);
                    while (POWER);
                    Delay(20);
                    oled=3;
                    if(hour==0){
                        hour=0;
                    }
                    else{
                        hour--;
                    }
                }
                //设置好小时以后 设置分钟 现在按select（当作confirm）选时间
                if (CONFIRM&& oled==3)
                {
                    Delay(20);
                    while (CONFIRM);
                    Delay(20);
                    oled=4;
                    OLED_Clear();
                }
                //1.1.1.1 请设置喂食时间(分）
                if(oled==4){
                    oled4(hour,min);
                }
                //1.1.1.1.1 (+1)
                if (SETTING&& oled==4)//setting+1
                {
                    Delay(20);
                    while (SETTING);
                    Delay(20);
                    oled=4;
                    min++;
                }
                //1.1.1.1.2 (-1)
                if (POWER&& oled==4)//power-1
                {
                    Delay(20);
                    while (POWER);
                    Delay(20);
                    oled=4;
                    if(min==0){
                        min=0;
                    }
                    else{
                        min--;
                    }
                }
                //设置好时间以后 设置次数 oled5
                if (CONFIRM&& oled==4)
                {
                    Delay(20);
                    while (CONFIRM);
                    Delay(20);
                    oled=5;
                    OLED_Clear();
                }
                //1.1.1.2 设置次数
                if(oled==5){
                    oled5(time);
                }
                //1.1.1.2.1 (+1)
                if (SETTING&& oled==5)//setting+1
                {
                    Delay(20);
                    while (SETTING);
                    Delay(20);
                    oled=5;
                    time++;
                }
                //1.1.1.2.2 (-1)
                if (POWER&& oled==5)//power-1
                {
                    Delay(20);
                    while (POWER);
                    Delay(20);
                    oled=5;
                    if(time==0){
                        time=0;
                    }
                    else{
                        time--;
                    }
                }
                //全部设置好了 返回oled0
                if (CONFIRM&& oled==5)
                {
                    Delay(20);
                    while (CONFIRM);
                    Delay(20);
                    oled_mid1(hour,min,time);
                    //这里要加上改变那个水泵or喂食！！！！！
                    oled=0;
                    OLED_Clear();
                }
                //1.1.2 确定喂水 oled6  
                if (SELECT&& oled==2)
                {
                    Delay(20);
                    while (SELECT);
                    Delay(20);
                    oled=6;
                    OLED_Clear();
                }
                if(oled==6){
                    oled6();
                }
                //回去喂食
                if (SELECT&& oled==6)
                {
                    Delay(20);
                    while (SELECT);
                    Delay(20);
                    oled=2;
                    OLED_Clear();
                }
                //显示”确定喂水“ oled7  
                if (CONFIRM&& oled==6)
                {
                    Delay(20);
                    while (CONFIRM);
                    Delay(20);
                    oled=7;
                    OLED_Clear();
                }
                //1.1.1.1 请设置喂食时间
                if(oled==7){
                    oled7(hour2,min2);
                }
                //1.1.1.1.1 (+1)
                if (SETTING&& oled==7)//setting+1
                {
                    Delay(20);
                    while (SETTING);
                    Delay(20);
                    oled=7;
                    hour2++;
                }
                //1.1.1.1.2 (-1)
                if (POWER&& oled==7)//power-1
                {
                    Delay(20);
                    while (POWER);
                    Delay(20);
                    oled=7;
                    if(hour2==0){
                        hour2=0;
                    }
                    else{
                        hour2--;
                    }
                }
                //设置好h以后 设置分钟（oled8
                if (CONFIRM&& oled==7)
                {
                    Delay(20);
                    while (CONFIRM);
                    Delay(20);
                    oled=8;
                    OLED_Clear();
                }
                if(oled==8){
                    oled8(hour2,min2);
                }
                //1.1.1.1.1 (+1)
                if (SETTING&& oled==8)//setting+1
                {
                    Delay(20);
                    while (SETTING);
                    Delay(20);
                    oled=8;
                    min2++;
                }
                //1.1.1.1.2 (-1)
                if (	POWER&& oled==8)//power-1
                {
                    Delay(20);
                    while (POWER);
                    Delay(20);
                    oled=8;
                    if(min2==0){
                        min2=0;
                    }
                    else{
                        min2--;
                    }
                }
                if (CONFIRM&& oled==8)
                {
                    Delay(20);
                    while (CONFIRM);
                    Delay(20);
                    oled=9;
                    OLED_Clear();
                }
                //1.1.1.2 设置次数
                if(oled==9){
                    oled9(time2);
                }
                //1.1.1.2.1 (+1)
                if (SETTING && oled==9)//setting+1
                {
                    Delay(20);
                    while (SETTING);
                    Delay(20);
                    oled=9;
                    time2++;
                }
                //1.1.1.2.2 (-1)
                if (POWER&& oled==9)//power-1
                {
                    Delay(20);
                    while (POWER);
                    Delay(20);
                    oled=9;
                    if(time2==0){
                        time2=0;
                    }
                    else{
                        time2--;
                    }
                }
                //全部设置好了 返回oled0
                if (CONFIRM && oled==9)
                {
                    Delay(20);
                    while (CONFIRM);
                    Delay(20);
                    oled_mid2(hour2,min2,time2);
                    //这里要加上改变那个水泵or喂食！！！！！
                    oled=0;
                    OLED_Clear();
                }		 
                //1.2 选择了设定power oled8
                if (POWER&& oled==1)
                {
                    Delay(20);
                    while (POWER);
                    Delay(20);
                    oled=0;
                    OLED_Clear();
                }
```



