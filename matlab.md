## 信号与系统实验

```
>> xn=[1,1,1,1,1,1,0,0,0,0,0];
>> hn=[0,1,2,3,4,5];
>> y2=filter(hn,1,xn);
>> ny=length(y2);
>> stem(0:ny-1,y2);
```
