# Digital-Twin-MVP-STM32-UE5 
A minimum-viable digital-twin stack that couples real-time STM32 sensor data with an Unreal Engine 5 (UE5) virtual model and closes the loop via UE5-driven closed-loop device control.

# 本项目遵循以下流程进行：🚀
  - 1.实现stm32温湿度传感器数据通过串口打印出来。
  - 2.实现通过串口助手传回控制数据点亮led。💡
  - 3.实现32的触点开关点按后改变led状态并发送对应状态到串口。led状态通过32内一个变量led_enable控制，32的触点开关被按下后改变变量led_enable的值，串口返回状态也依据这个led_enable变量，如此保证模型始终与现实中状态一致。
  - 4.实现ue5的一个简单模型。安装UE5，创建空项目。
  - 5.实现将串口的数据与32的数据联通，并能在ue5上显示温湿度🌡传感器的数据。分两步：1通过UE5的print方法直接打印。2通过在UE5内设置文本模块实时更新（仅验证框架）
  - 6.实现通过ue5的一个模型开关🕹的点按，反向控制32上的led亮灭。（仅验证框架）
  - 后续规划🗓：在实现以上功能的前提下，实现现实3个编码器控制stm32中的R🔴 G🟢 B🔵三个值然后据此调整led颜色，ue5孪生的led也随之改变颜色（R G B的值依然存在stm32中，状态更新同led_enable，以stm32的变量值为准）。在ue5中调整R🔴 G🟢 B🔵值现实中的led也随之改变颜色（通过串口或网络将控制状态返回给stm32）。
>本项目基于B站up莽小石的物联网数字孪生视频的指导与启发👨‍💻[莽小石](https://space.bilibili.com/232207878/upload/video "莽小石的B站空间")

## B站莽小石物联网数字孪生方案调研：
  - 物联网设备  ➡️ 串口服务器 > （modbus协议 物理接口用rs485（系工控领域开放协议） 转换为mqtt协议给rocky系统）
     ➡️  rocky系统 > （一个linux系统） ➡️  mqtt服务器 > （在工控机上）（物联网设备数据转发到UE5虚幻引擎）+node-red（运行在工控机 linux系统上的一个插件，可以图形化配置前后端）+mysql（用于实现注册登陆）
## 物联网设备到UE5数字孪生的系统调研：
  - 物联网设备 > （温湿度传感器） ➡️  串口服务器 > （一个串口服务器/边缘网关连接多个物联网设备）（tcp协议作为server） ➡️  UE5设置tcp插件作为client > （串口调试助手同时监听？）获取的数据在ue5 print（json 解析）出来  ➡️  显示在UE5的组件上。

# 基于以上内容，设计数字孪生最小可行系统的架构为：

- DigitalTwin/  
  - Firmware/           💡硬件层or现实层； stm32代码在这里
    - Core/
    - DeviceLink/       💡用于扩展传感器or别的孪生设备
    - TransportLink/    💡用于扩展通信手段。 便于从串口/蓝牙/wifi等切换。目前为uart
    - CMakeLists.txt
  - UE5/                💡模型层or孪生层； UE5相关文件在这里，仅作流程验证使用，因此简化。实现功能：
    - 1.基于串口（com）数据或网络数据（tcp mqtt）进行通信
    - 2.在UE5内实现print温湿度数据 
    - 3.在UE5内实现按钮，通过通信反向控制stm32的led
