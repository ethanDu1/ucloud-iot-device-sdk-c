## 简介

本文主要介绍如何将C-SDK移植到无运行操作系统无联网通讯能力的STM32上，并通过STM32的串口发送AT指令控制移远EC20模组连接云平台，本文仅介绍演示移植MQTT传输功能。<br>
由于运行在裸机上，不能使用C-SDK自带的编译系统，需要抽取C-SDK的部分文件到用户使用的编译系统上，本文中使用IAR进行演示编译。

![连接云图.png](/../images/连接云图.png)

## 准备工作

* 安装STM32CubeMX软件，我们将使用该软件配置生成一个STM32的基础工程，并在其基础上加入C-SDk的代码，移植MQTT传输的功能。

* 准备STM32开发板，本文使用的是STM32F03RB开发板。

* 下载ucloud-iot-device-sdk-c代码。

```
  git init
  git clone https://github.com/ucloud/ucloud-iot-device-sdk-c.git
```

## 使用STM32CubeMX工具生成移植基础工程
STM32Cube使用参考：[STM32Cube使用参考](/stm32cube_user_guide.md)

因为要通过串口控制通信模组,配置新增一个串口，打开串口中断模式。

![stm32f1新增模组控制串口.png](/../images/stm32f1新增模组控制串口.png)

完成其他配置后选择生成EWARM V8工程

## 抽取C-SDK中的代码到生成的STM32工程

![加入工程1.png](/../images/加入工程1.png)

![加入工程2.png](/../images/加入工程2.png)

### API部分文件

* at：AT命令处理

* certs: MQTT连接时的认证证书

* mqtt: MQTT协议连接云平台收发数据的接口实现和声明

* sdk-impl: 接口的头文件声明

* utils: 公共库

![stm32f1抽取代码.png](/../images/stm32f1抽取代码.png)

sdk-impl文件夹只需要抽取使用到的API的头文件

![sdk-impl部分.png](/../images/sdk-impl部分.png)

### AT模组实现部分，只选择当前使用的通信模组

![stm32f1根据使用模组选择文件夹.png](/../images/stm32f1根据使用模组选择文件夹.png)

### HAL层相关的接口

使用了AT指令进行通信，需要抽取AT指令HAL层的接口文件

![stm32f1抽取代码2.png](/../images/stm32f1抽取代码2.png)

本案例中没有运行操作系统。因此需要抽取nos下的文件

![stm32f1抽取代码3.png](/../images/stm32f1抽取代码3.png)

### 将抽取的文件加入STM32CubeMX生成的工程

![stm32f1将代码加入工程.png](/../images/stm32f1将代码加入工程.png)

配置编译宏和添加头文件

![stm32f1增加编译宏和头文件.png](/../images/stm32f1增加编译宏和头文件.png)

### 修改HAL层接口
根据使用得硬件平台逐一适配HAL层接口

AT命令相关     | 说明
-------------- | --------------------------
HAL_AT_Init    | 对通信模组做一些初始化配置
HAL_AT_Read    | 读取AT设备的执行结果
HAL_AT_Write   | 发送AT命令给模组

定时器相关             | 说明
---------------------- | -----------------------------------------
HAL_Timer_Expired      | 判断定时器时间是否已经过期
HAL_Timer_Countdown_ms | 根据timeout时间开启定时器计时, 单位: ms
HAL_Timer_Countdown    | 根据timeout时间开启定时器计时, 单位: s
HAL_Timer_Remain_ms    | 检查给定定时器剩余时间
HAL_Timer_Init         | 初始化定时器结构体

系统相关               | 说明
---------------------- | ------------------------------------------------------------------------------
HAL_MutexCreate        | 创建互斥量
HAL_MutexDestroy       | 销毁互斥量
HAL_MutexLock          | 阻塞式加锁
HAL_MutexUnlock        | 释放互斥量
HAL_Malloc             | 申请内存块
HAL_Free               | 释放内存块
HAL_Printf             | 打印函数，向标准输出格式化打印一个字符串
HAL_Snprintf           | 打印函数, 向内存缓冲区格式化打印一个字符串
HAL_Vsnprintf          | 打印函数, 格式化输出字符串到指定buffer中
HAL_SleepMs            | 休眠，单位毫秒
HAL_GetProductSN       | 获取产品序列号。从设备持久化存储（例如FLASH）中读取产品序列号
HAL_GetProductSecret   | 获取产品密钥（动态注册）。从设备持久化存储（例如FLASH）中读取产品密钥
HAL_GetDeviceSN        | 获取设备序列号。从设备持久化存储（例如FLASH）中读取设备序列号
HAL_GetDeviceSecret    | 获取设备密钥。从设备持久化存储（例如FLASH）中读取设备密钥
HAL_SetProductSN       | 设置产品序列号。将产品序列号烧写到设备持久化存储（例如FLASH）中，以备后续使用
HAL_SetProductSecret   | 设置产品密钥。将产品密钥烧写到设备持久化存储（例如FLASH）中，以备后续使用
HAL_SetDeviceSN        | 设置设备序列号。将设备序列号烧写到设备持久化存储（例如FLASH）中，以备后续使用
HAL_SetDeviceSecret    | 设置设备密钥。将设备密钥烧写到设备持久化存储（例如FLASH）中，以备后续使用


### 修改硬件平台的接口实现
修改stm32f1中的串口中断处理接口，目的是在中断处理中直接把通信模组发给MCU的数据存到循环队列中方便处理。<br>
示例中修改了stm32f1xx_hal_uart.c文件中的串口接收中断处理函数

```
#include "stm32f1xx_hal.h"
#include "at_ringbuff.h"

.........

extern sRingbuff g_ring_buff; 
/**
* @brief Receives an amount of data in non blocking mode
* @param huart Pointer to a UART_HandleTypeDef structure that contains
* the configuration information for the specified UART module.
* @retval HAL status
*/
static HAL_StatusTypeDef UART_Receive_IT(UART_HandleTypeDef *huart)
{
  uint16_t *tmp;
  uint8_t ch;

  /* Check that a Rx process is ongoing */
  if (huart->RxState == HAL_UART_STATE_BUSY_RX)
  {
    if (huart->Init.WordLength == UART_WORDLENGTH_9B)
    {
      tmp = (uint16_t *) huart->pRxBuffPtr;
      if (huart->Init.Parity == UART_PARITY_NONE)
      {
        *tmp = (uint16_t)(huart->Instance->DR & (uint16_t)0x01FF);
        huart->pRxBuffPtr += 2U;
      }
    else
    {
      *tmp = (uint16_t)(huart->Instance->DR & (uint16_t)0x00FF);
      huart->pRxBuffPtr += 1U;
    }
  }
  else
  {
    if (huart->Init.Parity == UART_PARITY_NONE)
    {
      ch = (uint8_t) READ_REG(huart->Instance->DR)&0xFF;
      ring_buff_push_data(&g_ring_buff, &ch, 1);
    }
    else
    {
      *huart->pRxBuffPtr++ = (uint8_t)(huart->Instance->DR & (uint8_t)0x007F);
    }
  }
#if 0
  if (--huart->RxXferCount == 0U)
  {
    /* Disable the UART Data Register not empty Interrupt */
    __HAL_UART_DISABLE_IT(huart, UART_IT_RXNE);

    /* Disable the UART Parity Error Interrupt */
    __HAL_UART_DISABLE_IT(huart, UART_IT_PE);

    /* Disable the UART Error Interrupt: (Frame error, noise error, overrun error) */
    __HAL_UART_DISABLE_IT(huart, UART_IT_ERR);

    /* Rx process is completed, restore huart->RxState to Ready */
    huart->RxState = HAL_UART_STATE_READY;

    #if (USE_HAL_UART_REGISTER_CALLBACKS == 1)
    /*Call registered Rx complete callback*/
    huart->RxCpltCallback(huart);
    #else
    /*Call legacy weak Rx complete callback*/
    HAL_UART_RxCpltCallback(huart);
    #endif /* USE_HAL_UART_REGISTER_CALLBACKS */

    return HAL_OK;
  }
#endif 
  return HAL_OK;
  }
  else
  {
    return HAL_BUSY;
  }
}
```

根据C-SDK中的ucloud-iot-device-sdk-c\samples\mqtt\mqtt_sample.c修改main.c函数，编译运行。

![stm32f1编译运行.png](/../images/stm32f1编译运行.png)

