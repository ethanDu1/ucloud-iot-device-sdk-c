## 简介

C-SDK可以方便地移植到各种硬件平台上，本文主要介绍如何将C-SDK快速移植到运行Freertos系统的STM32上，通过网口连接到云端传输MQTT数据，本文仅演示移植MQTT传输的功能。<br>
由于运行在Freertos上，不能使用C-SDK自带的编译系统，需要抽取C-SDK的部分文件到用户使用的编译系统上，本文中使用IAR进行演示编译。

## 准备工作
* 安装STM32CubeMX软件，我们将使用该软件配置生成一个移植了Freertos和LWIP的基础工程，并在其基础上移植C-SDk的代码，加入MQTT传输的功能。
* 准备STM32开发板，本文示例使用的是STM32F767ZI，要求该开发板必须具有一个以太网口。
* 下载ucloud-iot-device-sdk-c代码。
```
  git clone https://github.com/ucloud/ucloud-iot-device-sdk-c.git
```

### 使用STM32CubeMX工具配置生成基础工程
STM32Cube使用参考：[STM32Cube使用参考](/stm32cube_user_guide.md)

需要连接网络，打开以太网接口

![Connect设置网络.png](/../images/Connect设置网络.png)

勾选freertos操作系统，选择CMSIS_V1版本

![操作系统freertos.png](/../images/操作系统freertos.png)

以太网口连接网络需要使用LWIP库

![cmd.png](/../images/使能LWIP.png)

选择必要的网络协议，比如DNS等解析域名

![cmd.png](/../images/协议选择.png)

填写项目名称等信息，点击生成代码，选择生成EWARM V8工程。

![cmd.png](/../images/填写项目信息.png)

生成的代码可以直接使用IAR打开，到这里一个运行Freertos带有网口的基础工程就准备完成了。

## 抽取C-SDK中的代码到生成的STM32功能

### API部分文件

* certs: MQTT连接时的认证证书

* mqtt: MQTT协议连接云平台收发数据的接口实现和声明

* sdk-impl: 接口的头文件声明

* utils: 公共库

![MQTT抽取代码.png](/../images/MQTT抽取代码.png)

sdk-impl文件夹只需要提取涉及到的API接口的声明头文件

![sdk-impl部分.png](/../images/sdk-impl部分.png)

运行freertos操作系统，需要抽取适配freertos操作系统的HAL接口

![MQTT抽取代码操作系统部分.png](/../images/MQTT抽取代码操作系统部分.png)

把抽取出的文件加入到前面生成的IAR工程中，相关HAL接口已提供Freertos实现，需要适配修改版本差异

网络协议相关(不使用TLS) | 说明
----------------------- | -----------------------------------------
HAL_TCP_Connect         | 建立TCP连接
HAL_TCP_Disconnect      | 断开TCP连接
HAL_TCP_Write           | 向指定的TCP连接写入数据
HAL_TCP_Read            | 从指定的TCP连接读取数据

网络协议相关(使用TLS) | 说明
----------------------- | -----------------------------------------
HAL_TLS_Connect         | 建立TLS连接
HAL_TLS_Disconnect      | 断开TLS连接
HAL_TLS_Write           | 向指定的TLS连接写入数据
HAL_TLS_Read            | 从指定的TLS连接读取数据

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

适配修改完成后编译运行即可，输出如下图

![添加文件.png](/../images/添加文件.png)






