# PwmCapture_2Chanel
 stm32 hal 库 定时器双通道 捕获pwm 波

﻿使用标准库来配置的，可以看我之前的博客
[https://blog.csdn.net/yuleitao/article/details/103721812](https://blog.csdn.net/yuleitao/article/details/103721812)
这个项目是使用CUBEMX配置，hal库来写，实现的功能一样

几个点注意

- 使用定时器1，将通道1设置为主模式（直接模式），通道2设置为从模式（非直接模式）
- 通道1捕获上升沿，通道二捕获下降沿
- 节省CPU时间，提高捕获精度

## CUBEMX配置

时钟，系统这些就不说了，直接最关键的部分，开定时器TIM1
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200120200413201.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l1bGVpdGFv,size_16,color_FFFFFF,t_70)

-  trigger source 选择 TI1FP1
-  clock source 选择 	internal clock
-  channel1 选择 input capture direct mode
-  channel2 选择 input capture indirect mode
   下方的设置按照图上所设一样就行

最后打开工程，插入如下代码，开启定时器1通道1和通道2的捕获中断

```
  /* USER CODE BEGIN 2 */

	HAL_TIM_IC_Start_IT(&htim1,TIM_CHANNEL_1);
	HAL_TIM_IC_Start_IT(&htim1,TIM_CHANNEL_2);

  /* USER CODE END 2 */
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200120200903256.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l1bGVpdGFv,size_16,color_FFFFFF,t_70)

在tim.c里添加如下代码，捕获中断的逻辑

```
/* USER CODE BEGIN 0 */

uint32_t Cap_Data1 ;
uint32_t Cap_Data2 ;
float Fre_Cap = 0.0;
float zhouqi_Cap = 0.0;
float Pwm_Duty = 0.0;

void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim)
{
	if(htim->Channel == HAL_TIM_ACTIVE_CHANNEL_1)
		{
			Cap_Data1 = HAL_TIM_ReadCapturedValue(&htim1,TIM_CHANNEL_1);
			if(Cap_Data1 != 0 )
			{
				Fre_Cap = 16.8 * 1000000/Cap_Data1;
				zhouqi_Cap = 1 / Fre_Cap;
				Pwm_Duty = Cap_Data2 * 1.0 /Cap_Data1;
			}
		}
	if(htim->Channel == HAL_TIM_ACTIVE_CHANNEL_2)
		{
			Cap_Data2 = HAL_TIM_ReadCapturedValue(&htim1,TIM_CHANNEL_2);
		}
}

/* USER CODE END 0 */
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200120201201289.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l1bGVpdGFv,size_16,color_FFFFFF,t_70)

## 工程文件

上传到我的github 链接
[https://github.com/yltzdhbc/PwmCapture_2Chanel](https://github.com/yltzdhbc/PwmCapture_2Chanel)