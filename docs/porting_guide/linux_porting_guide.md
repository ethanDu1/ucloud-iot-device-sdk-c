## 应用场景
当用户需要在Linux操作系统下使用C-SDK时参考本文

## 使用方法

### 下载C-SDK源码
```
git init
git clone https://github.com/ucloud/ucloud-iot-device-sdk-c.git
```
或在windows主机下载完成后使用winscp等软件拷贝到Linux主机

### 修改源码示例中的认证三要素
将认证三要素修改为用户在云平台上自己创建的
```
#define UIOT_MY_PRODUCT_SN            "PRODUCT_SN"

#define UIOT_MY_DEVICE_SN             "DEVICE_SN"

#define UIOT_MY_DEVICE_SECRET         "DEVICE_SECRET"
```

### 编译代码
```
cd ucloud-iot-device-sdk-c/
make
```
编译成功后有静态库文件的构成图
```
o BINARY FOOTPRINT CONSIST:                                                                                                         
----                                                                                                                                
                                                                                                                                    
     10.02%                      dm_mqtt.o | libiot_sdk.a  24456 / 244168                                                           
     8.55%         shadow_client_manager.o | libiot_sdk.a  20872 / 244168                                                           
     7.94%                    ota_client.o | libiot_sdk.a  19392 / 244168                                                           
     7.43%            mqtt_client_common.o | libiot_sdk.a  18136 / 244168                                                           
     6.01%                   mqtt_client.o | libiot_sdk.a  14664 / 244168                                                           
     4.98%                   utils_httpc.o | libiot_sdk.a  12160 / 244168                                                           
     4.20%              http_upload_file.o | libiot_sdk.a  10256 / 244168                                                           
     3.97%                   http_client.o | libiot_sdk.a   9696 / 244168                                                           
     3.61%                 shadow_client.o | libiot_sdk.a   8816 / 244168                                                           
     3.60%             mqtt_client_yield.o | libiot_sdk.a   8800 / 244168                                                           
     3.00%           mqtt_client_publish.o | libiot_sdk.a   7320 / 244168                                                           
     2.94%                       ota_lib.o | libiot_sdk.a   7184 / 244168                                                           
     2.58%                    json_token.o | libiot_sdk.a   6288 / 244168                                                           
     2.54%            shadow_client_json.o | libiot_sdk.a   6208 / 244168                                                           
     2.42%                  string_utils.o | libiot_sdk.a   5912 / 244168                                                           
     2.29%           mqtt_client_connect.o | libiot_sdk.a   5600 / 244168                                                           
     2.27%                     utils_md5.o | libiot_sdk.a   5544 / 244168                                                           
     2.20%         mqtt_client_subscribe.o | libiot_sdk.a   5376 / 244168                                                           
     2.17%          shadow_client_common.o | libiot_sdk.a   5296 / 244168                                                           
     2.17%                      ota_mqtt.o | libiot_sdk.a   5296 / 244168                                                           
     2.06%                    utils_sha2.o | libiot_sdk.a   5032 / 244168                                                           
     2.05%       mqtt_client_unsubscribe.o | libiot_sdk.a   5016 / 244168                                                           
     1.79%                   json_parser.o | libiot_sdk.a   4368 / 244168                                                           
     1.76%                            ca.o | libiot_sdk.a   4304 / 244168                                                           
     1.70%                     utils_net.o | libiot_sdk.a   4152 / 244168                                                           
     1.52%                    utils_list.o | libiot_sdk.a   3704 / 244168                                                           
     1.47%                     ota_fetch.o | libiot_sdk.a   3600 / 244168                                                           
     1.34%                     dm_client.o | libiot_sdk.a   3280 / 244168                                                           
     0.83%                   utils_timer.o | libiot_sdk.a   2016 / 244168                                                           
     0.58%               mqtt_client_net.o | libiot_sdk.a   1424 / 244168                                                           
    -----------------------------------------------------------------                                                               
                                                                                                                                    
     100.00%  [ libiot_sdk.a ]     244168 Bytes                                                                                     
                                                                                                                                    
     39.27%              HAL_TLS_mbedtls.o | libiot_platform.a  10376 / 26424                                                       
     27.40%                 HAL_OS_linux.o | libiot_platform.a   7240 / 26424                                                       
     23.77%                HAL_TCP_linux.o | libiot_platform.a   6280 / 26424                                                       
     9.57%               HAL_Timer_linux.o | libiot_platform.a   2528 / 26424                                                       
    -----------------------------------------------------------------                                                               
                                                                                                                                    
     100.00%  [ libiot_platform.a ]      26424 Bytes                                                                                
========================================================================= 
```

### 编译输出
编译输出的文件在/output/release下

文件夹        | 说明
------------- | -------------------------------
bin           | 各个功能的运行示例，可以直接运行
include       | API接口声明头文件
lib           | 生成的静态库文件
unittest      | 各个功能的单元测试文件

### 运行示例
```
cd ucloud-iot-device-sdk-c/output/release/bin
./mqtt_sample
```


### 配置修改
可以通过修改配置文件make.setting来删减C-SDK的功能，修改文件中的FEATURE_开头变量的值，当值为y时为打开，为n时关闭

编译开关                       | 说明
------------------------------ | ---------------------------------------------------------
FEATURE_MQTT_COMM_ENABLED      | 是否打开MQTT连接云平台功能
FEATURE_DEVICE_SHADOW_ENABLED  | 是否打开设备影子功能，依赖MQTT功能
FEATURE_OTA_ENABLED            | 是否打开OTA升级功能，依赖MQTT功能
FEATURE_DEVICE_MODEL_ENABLED   | 是否打开物模型功能，依赖MQTT功能
FEATURE_HTTP_CLIENT_ENABLED    | 是否打开HTTP连接云平台功能，依赖MQTT功能
FEATURE_AUTH_MODE_DYNAMIC      | 是否打开动态认证功能，依赖MQTT功能
FEATURE_SUPPORT_TLS            | 是否打开tls功能，依赖MQTT或者HTTP功能
FEATURE_SUPPORT_AT_CMD         | 是否打开AT指令控制功能，依赖MQTT功能，Linux下一般不需要开启
FEATURE_SDK_TESTS_ENABLED      | 是否打开单元测试功能



