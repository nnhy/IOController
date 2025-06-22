# IO开关控制器

## 接口介绍
1. DC-005插座：DC7-24V电源输入插座；  
2. VCC，GND： DC7-24V 5.08mm电源输入端子；  
3. DC3.3-30V光耦信号输入：  
* IN1： 通道1正极  
* IN2： 通道2正极  
* IN3： 通道3正极  
* IN4： 通道4正极  
* GND_IN：公共端负极  
  
4. A+，B-：RS485通讯接口，A+，B-分别接外部控制端的A+，B-；  

5. 继电开关信号输出：  
+ NC，：常闭端，继电器吸合前与COM短接，吸合后悬空；  
+ COM：公共端；  
+ NO：常开端，继电器吸合前悬空，吸合后与COM短接。  

6. GND，RXD，TXD：TTL电平UART通讯接口，GND，RXD，TXD分别接外部控制端的GND，TXD，RXD；  

## Modbus RTU指令简介
Modbus设备通过接收来自外部控制端（如：上位机/MCU）的Modbus RTU指令来执行相关操作，一帧指令一般由设备地址、功能码、寄存器地址、寄存器数据、校验码组成，帧长度和功能码有关。一般每帧数据的首字节为设备地址，可设置范围为1-255，默认255（即0xFF），最后2字节为CRC校验码。  
假设设备地址为255，则常用的Modbus RTU指令如下：  
#### 1. 打开1号继电器（手动模式）
发送：`FF 05 00 00 FF 00 99 E4`  
返回：`FF 05 00 00 FF 00 99 E4`  
备注：
1. 发送帧的第3--4个字节代表继电器地址，继电器1--继电器8的地址分别为0x0000,0x0001,0x0002,0x0003,0x0004,0x0005,0x0006,0x0007  
2. 发送帧的第5--6个字节代表数据，0xFF00代表打开继电器，0x0000代表关闭继电器  
#### 2，关闭1号继电器（手动模式）
发送：`FF 05 00 00 00 00 D8 14`  
返回：`FF 05 00 00 00 00 D8 14`  
#### 3，打开2号继电器（手动模式）
发送：`FF 05 00 01 FF 00 C8 24`    
返回：`FF 05 00 01 FF 00 C8 24`  
#### 4，关闭2号继电器（手动模式）
发送：`FF 05 00 01 00 00 89 D4`  
返回：`FF 05 00 01 00 00 89 D4`  
#### 5，打开所有继电器
发送：`FF 0F 00 00 00 08 01 FF 30 1D`  
返回：`FF 0F 00 00 00 08 41 D3`  
#### 6，关闭所有继电器
发送：`FF 0F 00 00 00 08 01 00 70 5D`  
返回：`FF 0F 00 00 00 08 41 D3`  

#### 7，设置设备地址为1
发送：`00 10 00 00 00 01 02 00 01 6A 00`  
返回：`00 10 00 00 00 01 02 00 01 6A 00`  
备注：发送帧的第9个字节0x01为写入的设备地址 
#### 8，设置设备地址为255
发送：`00 10 00 00 00 01 02 00 FF EB 80`  
返回：`00 10 00 00 00 01 02 00 FF EB 80`  
备注：发送帧的第9个字节0xFF为写入的设备地址
#### 9，读取设备地址 
发送：`00 03 00 00 00 01 85 DB`  
返回：`00 03 02 00 FF C5 C4`  
备注：返回帧的第5个字节0xFF为读取到的设备地址

#### 10，读取继电器状态 
发送：`FF 01 00 00 00 08 28 12`  
返回：`FF 01 01 01 A1 A0`  
备注：返回帧的第4个字节0x01的Bit0--Bit7分别代表继电器1--继电器8，0为关闭，1为打开
#### 11，读取光耦输入状态 
发送：`FF 02 00 00 00 08 6C 12`  
返回：`FF 02 01 01 51 A0`  
备注：返回帧的第4个字节0x01的IN1--IN8分别代表光耦1--光耦8输入信号，0代表低电平，1代表高电平

#### 12，设置波特率为4800 
发送：`FF 10 03 E9 00 01 02 00 02 4A 0C`  
返回：`FF 10 03 E9 00 01 C5 A7`  
备注：发送帧的第9个字节为波特率设置值，0x02, 0x03, x04分别代表4800, 9600, 19200

#### 13，设置波特率为9600
发送：`FF 10 03 E9 00 01 02 00 03 8B CC`  
返回：`FF 10 03 E9 00 01 C5 A7` 

#### 14，设置波特率为19200
发送：`FF 10 03 E9 00 01 02 00 04 CA 0E`  
返回：`FF 10 03 E9 00 01 C5 A7` 
#### 15，读取波特率
发送：`FF 03 03 E8 00 01 11 A4`  
返回：`FF 03 02 00 04 90 53`  
备注：返回帧的第5个字节代表读取到的波特率，0x02, 0x03, x04分别代表4800, 9600, 19200

#### 16，打开1号继电器（闪闭模式2S）
发送：`FF 10 00 03 00 02 04 00 04 00 14 C5 9F`  
返回：`FF 10 00 03 00 02 A4 16`  
备注：
1. 发送帧的第3--4个字节代表继电器地址，继电器1--继电器8的地址分别为0x0003,0x0008,0x000D,0x0012,0x0017,0x001C,0x0021,0x0026
2. 发送帧的第10--11个字节代表延时设置值，延时基数为0.1S，故延时时间为0x0014*0.1=20*0.1S=2S，继电器打开2S后自动关闭

#### 17，关闭1号继电器（闪断模式3S）
发送：`FF 10 00 03 00 02 04 00 02 00 1E A5 99`  
返回：`FF 10 00 03 00 02 A4 16`  
备注：
1. 发送帧的第3--4个字节代表继电器地址，继电器1--继电器8的地址分别为0x0003,0x0008,0x000D,0x0012,0x0017,0x001C,0x0021,0x0026  
2. 发送帧的第10--11个字节代表延时设置值，延时基数为0.1S，故延时时间为0x001E*0.1=30*0.1S=3S，继电器关闭3S后自动打开  


## 简单使用说明
Modbus 继电器模块可经由RS485/TTL UART接口接收来自上位机/MCU的Modbus RTU指令来执行相关操作。下面以使用上位机软件通过RS485接口来打开继电器1和2（手动模式）为例，假设设备地址为255，波特率为9600，则使用步骤如下：
1. DC-005插座/5.08mm端子的VCC，GND接电源；
2. A+，B-分别USB转RS485模块输出端的A+和B-；
3. 打开上位机软件“ModbusRTU 配置工具”，选择正确的端口号，波特率选择9600，地址设为1，点击“连接”；
4. 再点击“打开1号”“打开2号”即可打开继电器1和2，同时继电器指示灯点亮。


---
## 新生命项目矩阵
各项目默认支持net9.0/netstandard2.1/netstandard2.0/net4.62/net4.5，旧版（2024.0801）支持net4.0/net2.0  

|                               项目                               | 年份  | 说明                                                                                        |
| :--------------------------------------------------------------: | :---: | ------------------------------------------------------------------------------------------- |
|                             基础组件                             |       | 支撑其它中间件以及产品项目                                                                  |
|          [NewLife.Core](https://github.com/NewLifeX/X)           | 2002  | 核心库，日志、配置、缓存、网络、序列化、APM性能追踪                                         |
|    [NewLife.XCode](https://github.com/NewLifeX/NewLife.XCode)    | 2005  | 大数据中间件，单表百亿级，MySql/SQLite/SqlServer/Oracle/PostgreSql/达梦，自动分表，读写分离 |
|      [NewLife.Net](https://github.com/NewLifeX/NewLife.Net)      | 2005  | 网络库，单机千万级吞吐率（2266万tps），单机百万级连接（400万Tcp长连接）                     |
| [NewLife.Remoting](https://github.com/NewLifeX/NewLife.Remoting) | 2011  | 协议通信库，提供CS应用通信框架，支持Http/RPC通信框架，高吞吐，物联网设备低开销易接入        |
|     [NewLife.Cube](https://github.com/NewLifeX/NewLife.Cube)     | 2010  | 魔方快速开发平台，集成了用户权限、SSO登录、OAuth服务端等，单表100亿级项目验证               |
|    [NewLife.Agent](https://github.com/NewLifeX/NewLife.Agent)    | 2008  | 服务管理组件，把应用安装成为操作系统守护进程，Windows服务、Linux的Systemd                   |
|     [NewLife.Zero](https://github.com/NewLifeX/NewLife.Zero)     | 2020  | Zero零代脚手架，基于NewLife组件生态的项目模板NewLife.Templates，Web、WebApi、Service        |
|                              中间件                              |       | 对接知名中间件平台                                                                          |
|    [NewLife.Redis](https://github.com/NewLifeX/NewLife.Redis)    | 2017  | Redis客户端，微秒级延迟，百万级吞吐，丰富的消息队列，百亿级数据量项目验证                   |
| [NewLife.RocketMQ](https://github.com/NewLifeX/NewLife.RocketMQ) | 2018  | RocketMQ纯托管客户端，支持Apache RocketMQ和阿里云消息队列，十亿级项目验                     |
|     [NewLife.MQTT](https://github.com/NewLifeX/NewLife.MQTT)     | 2019  | 物联网消息协议，MqttClient/MqttServer，客户端支持阿里云物联网                               |
|      [NewLife.IoT](https://github.com/NewLifeX/NewLife.IoT)      | 2022  | IoT标准库，定义物联网领域的各种通信协议标准规范                                             |
|   [NewLife.Modbus](https://github.com/NewLifeX/NewLife.Modbus)   | 2022  | ModbusTcp/ModbusRTU/ModbusASCII，基于IoT标准库实现，支持ZeroIoT平台和IoTEdge网关            |
|  [NewLife.Siemens](https://github.com/NewLifeX/NewLife.Siemens)  | 2022  | 西门子PLC协议，基于IoT标准库实现，支持IoT平台和IoTEdge                                      |
|      [NewLife.Map](https://github.com/NewLifeX/NewLife.Map)      | 2022  | 地图组件库，封装百度地图、高德地图、腾讯地图、天地图                                        |
|    [NewLife.Audio](https://github.com/NewLifeX/NewLife.Audio)    | 2023  | 音频编解码库，PCM/ADPCMA/G711A/G722U/WAV/AAC                                                |
|                             产品平台                             |       | 产品平台级，编译部署即用，个性化自定义                                                      |
|         [Stardust](https://github.com/NewLifeX/Stardust)         | 2018  | 星尘，分布式服务平台，节点管理、APM监控中心、配置中心、注册中心、发布中心                   |
|           [AntJob](https://github.com/NewLifeX/AntJob)           | 2019  | 蚂蚁调度，分布式大数据计算平台（实时/离线），蚂蚁搬家分片思想，万亿级数据量项目验证         |
|      [NewLife.ERP](https://github.com/NewLifeX/NewLife.ERP)      | 2021  | 企业ERP，产品管理、客户管理、销售管理、供应商管理                                           |
|         [CrazyCoder](https://github.com/NewLifeX/XCoder)         | 2006  | 码神工具，众多开发者工具，网络、串口、加解密、正则表达式、Modbus、MQTT                      |
|           [EasyIO](https://github.com/NewLifeX/EasyIO)           | 2023  | 简易文件存储，支持分布式系统中文件集中存储。                                                |
|           [XProxy](https://github.com/NewLifeX/XProxy)           | 2005  | 产品级反向代理，NAT代理、Http代理                                                           |
|        [HttpMeter](https://github.com/NewLifeX/HttpMeter)        | 2022  | Http压力测试工具                                                                            |
|         [GitCandy](https://github.com/NewLifeX/GitCandy)         | 2015  | Git源代码管理系统                                                                           |
|          [SmartOS](https://github.com/NewLifeX/SmartOS)          | 2014  | 嵌入式操作系统，完全独立自主，支持ARM Cortex-M芯片架构                                      |
|          [SmartA2](https://github.com/NewLifeX/SmartA2)          | 2019  | 嵌入式工业计算机，物联网边缘网关，高性能.NET8主机，应用于工业、农业、交通、医疗             |
|                          FIoT物联网平台                          | 2020  | 物联网整体解决方案，建筑、环保、农业，软硬件及大数据分析一体化，单机十万级点位项目验证      |
|                        UWB高精度室内定位                         | 2020  | 厘米级（10~20cm）高精度室内定位，软硬件一体化，与其它系统联动，大型展厅项目验证             |



## 新生命开发团队
![XCode](https://newlifex.com/logo.png)  

新生命团队（NewLife）成立于2002年，是新时代物联网行业解决方案提供者，致力于提供软硬件应用方案咨询、系统架构规划与开发服务。  
团队主导的80多个开源项目已被广泛应用于各行业，Nuget累计下载量高达400余万次。  
团队开发的大数据中间件NewLife.XCode、蚂蚁调度计算平台AntJob、星尘分布式平台Stardust、缓存队列组件NewLife.Redis以及物联网平台FIoT，均成功应用于电力、高校、互联网、电信、交通、物流、工控、医疗、文博等行业，为客户提供了大量先进、可靠、安全、高质量、易扩展的产品和系统集成服务。  

我们将不断通过服务的持续改进，成为客户长期信赖的合作伙伴，通过不断的创新和发展，成为国内优秀的IoT服务供应商。  

`新生命团队始于2002年，部分开源项目具有20年以上漫长历史，源码库保留有2010年以来所有修改记录`  
网站：https://newlifex.com  
开源：https://github.com/newlifex  
QQ群：1600800/1600838  
微信公众号：  
![智能大石头](https://newlifex.com/stone.jpg)  
