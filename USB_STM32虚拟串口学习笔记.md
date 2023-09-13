# USB_VPC使用

**!!!使用CubeMx生成USB相关内容有可能会重新覆盖工程文件!!!**

如果有此方面的需求, 记得做好备份, 在生成后重新配置工程

## VPC

Virtual Port Com 虚拟串口

鉴于 RM 激烈的对抗场景, 寻常的使用 USB_TTL 的硬件串口方案肯定是行不通的, 在询问学长往届是如何处理 视觉与电控的信息收发后, 得到了通过虚拟串口发送的答案, 之后侯文辉先生又发了我一份他过往的详细虚拟串口配置经历, 但在实际使用的过程发现讲解仍然不够详细, 于是在其基础增加一点内容

以下是pansy先生撰写的usb-cdc使用说明

[‍⁡‬‍⁡‌⁣⁢﻿﻿⁢﻿⁢⁣⁤⁤‬⁣⁣⁤‬‍‍⁤⁢‍⁡⁢⁤⁢⁣⁤‍⁡‬‍‍﻿⁤‍⁢usb-cdc从实践到理论 - 飞书云文档 (feishu.cn)](https://bnmp7m19gb.feishu.cn/docx/doxcn6ULDgUxGFnjRaMpcqKviJY)

以下是补充内容: 

CDC_Receive_FS 是一个中断回调函数, 且不能直接调用/重定义该函数, 如果需要进行USB_FS的接收数据处理, 需要将具体的数据处理函数放入该回调函数中, 并且需要以中断函数的 Buf 和 Len 两个变量入参, 下面的pathfinder_data_handle 函数是一个使用例子.

```c
static int8_t CDC_Receive_FS(uint8_t* Buf, uint32_t *Len)
{
  /* USER CODE BEGIN 6 */
  USBD_CDC_SetRxBuffer(&hUsbDeviceFS, &Buf[0]);
  USBD_CDC_ReceivePacket(&hUsbDeviceFS);
	pathfinder_data_handle(Buf, Len);
  
  return (USBD_OK);
  /* USER CODE END 6 */
}
```

最开始在看资料的时候以为并非中断回调, 直接调用后不断报错, 在翻了很多篇文章后才找到一篇文章 这是回调函数, 在询问chatgpt后确定这是中断函数的结论.

(浪费了一个多小时, 引以为戒, 在遇到一个陌生的函数时, 如果他的函数说明不清晰, 就速速去问GPT, 别浪费时间) 



在一篇文章中提到:

**在使用上面代码的时候存在一个问题：每次下载完程序后都需要重新拔插一次USB才可以识别串口，这是由于芯片在下载完程序后没有重新枚举所导致的。需要在设备上电后对USB进行重新枚举即可，使用方法为将USB DP（PA12）引脚拉低一段时间后即可。**  

省流: 为了方便调试时不用多次插拔 USB 可以在USB初始化函数执行前先拉低 USB_DP 电平一段时间后再拉高, 

```c
/*USB 重新枚举函数*/
void USB_Reset(void)
{
  	GPIO_InitTypeDef GPIO_InitStruct = {0};
	__HAL_RCC_GPIOA_CLK_ENABLE();
	GPIO_InitStruct.Pin = GPIO_PIN_12;
	GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
	GPIO_InitStruct.Pull = GPIO_NOPULL;
	GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
	HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
	HAL_GPIO_WritePin(GPIOA,GPIO_PIN_12,GPIO_PIN_RESET);
	HAL_Delay(100);
	HAL_GPIO_WritePin(GPIOA,GPIO_PIN_12,GPIO_PIN_SET);
}
```

对应位置如下 : 

```c
 /* Configure the system clock */
  SystemClock_Config();

  /* USER CODE BEGIN SysInit */
  USB_Reset();
  /* USER CODE END SysInit */

  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  MX_USART1_UART_Init();
  MX_USB_DEVICE_Init();
  MX_USART2_UART_Init();
```

测试后发现该方法可行

除Pansy先生在他的原博客中提到的引用文章外, 我也在这里放置几篇, 用于参考

[STM32 USB 系列之 虚拟串口(VPC)_stm32 usb虚拟串口_非典型技术宅的博客-CSDN博客](https://blog.csdn.net/mirco_mcu/article/details/106081950)
