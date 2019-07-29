# lora网关设计

## 资料阅读

### 锐米资料

1. 锐米开源资料

* **锐米开源五：LoRaWAN网关** -->  树莓派及sx1301相关教程

* **Win32DiskImager.exe**   --> 树莓派镜像烧录软件

* **源代码** --> 此代码为 [lora_net](https://github.com/Lora-net/lora_gateway)开源代码

* **其余**  --> 硬件原理图、说明书等

2. semtech资料

* semtech 相关的一些开源代码，参考说明书，主要涉及到以下模块源代码  [picoGW_mcu](https://github.com/Lora-net/picoGW_mcu)、[picoGW_hal](https://github.com/Lora-net/picoGW_hal) 、[picoGW_packet_forwarder](https://github.com/Lora-net/picoGW_packet_forwarder)、[dfu-util](https://git.code.sf.net/p/dfu-util/dfu-util)

3. 补充说明semtech 官方各个库的作用

* packet_forwarder   包转发
* lora_gateway 驱动 基于sx1301 concentrator + sx1257/sx1255 射频收发器
* picoGW_hal  驱动USB通信 与 基于sx1308 concentrator + sx1257/sx1255 射频收发器 
* picoGW_mcu  驱动 Picocell Gateway concentrator board. 
* picoGW_packet_forwarder 运行在 Picocell Gateway上的包转发器

* LoRaMac-node 终端节点
* fuota  固件空中升级

## 树莓派镜像烧录和配置

1. [历史镜像](http://domoticx.com/raspberry-pi-sd-image-raspbian-linux-os/)

2. 下载2017-6-25版本即可

3. Win32 Disk Imager 烧录好，在boot的根目录下新建一个ssh文件。

4. 更换国内镜像源

```bash
# 编辑 `/etc/apt/sources.list` 文件，删除原文件所有内容，用以下内容取代：
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ jessie main non-free contrib
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ jessie main non-free contrib

# 编辑 `/etc/apt/sources.list.d/raspi.list` 文件，删除原文件所有内容，用以下内容取代：
deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ jessie main ui

# 编辑完后更新
sudo apt update
```

5. 安装一些常用的软件如git、vim等

```bash
apt install git vim
```

## semtech教程验证

1. 安装最新的semtech软件

```bash
mkdir lora-net && cd lora-net

git clone https://github.com/Lora-net/picoGW_packet_forwarder.git

git clone https://github.com/Lora-net/picoGW_mcu.git

git clone https://github.com/Lora-net/picoGW_hal.git
```

2. 安装设备固件升级工具(dfu-util tool)


```bash
sudo apt-get install autoconf

git clone https://git.code.sf.net/p/dfu-util/dfu-util

cd dfu-util

./autogen.sh

sudo apt-get install libusb-1.0-0-dev

./configue

make

sudo make install

....待补充

```


## 锐米验证
取自教程<<锐米开源五: LoRaWAN网关>>

1. 锐米官网提供的树莓派镜像为 树莓派官方镜像，未做修改，因为直接采用上面的镜像，因此配置过程不再介绍

2. 驱动spi过程参考教程，启动成功

3. 直接使用默认用户pi，不另新建用户rime

4. 下载并编译源程序

![下载源程序](https://i.loli.net/2019/07/29/5d3e6ff1ac14850882.png)

```bash
cd home/pi/lora-net/

git clone https://github.com/Lora-net/lora_gateway.git

git clone https://github.com/Lora-net/packet_forwarder

```

编译网关
![编译网关](https://i.loli.net/2019/07/29/5d3e709c6d5da73016.png)

编译包转发模块
![编译包转发模块](https://i.loli.net/2019/07/29/5d3e70d1690d490904.png)

5. 运行程序


```bash
# 进入目录下运行
# 复位SX1301
sudo /home/rime/lora_gateway/reset_lgw.sh start 22


# 进入packet_forwarder/lora_pkt_fwd 目录下运行
cd ~/lora-net/packet_forwarder/lora_pkt_fwd/
# 启动 gateway 进程
sudo ./lora_pkt_fwd
```

##  未连接concentrator 的情况下运行的日志

网上解释说未连接concentrator，可能是没有连接sx1301造成的
[https://www.thethingsnetwork.org/forum/t/error-main-failed-to-start-the-concentrator/5856/38]

```bash
*** Beacon Packet Forwarder for Lora Gateway ***
Version: 4.0.1
*** Lora concentrator HAL library version info ***
Version: 5.0.1;
***
INFO: Little endian host
INFO: found global configuration file global_conf.json, parsing it
INFO: global_conf.json does contain a JSON object named SX1301_conf, parsing SX1301 parameters
INFO: lorawan_public 1, clksrc 1
INFO: LBT is disabled
INFO: antenna_gain 0 dBi
INFO: Configuring TX LUT with 16 indexes
INFO: radio 0 enabled (type SX1257), center frequency 867500000, RSSI offset -166.000000, tx enabled 1, tx_notch_freq 129000
INFO: radio 1 enabled (type SX1257), center frequency 868500000, RSSI offset -166.000000, tx enabled 0, tx_notch_freq 0
INFO: Lora multi-SF channel 0>  radio 1, IF -400000 Hz, 125 kHz bw, SF 7 to 12
INFO: Lora multi-SF channel 1>  radio 1, IF -200000 Hz, 125 kHz bw, SF 7 to 12
INFO: Lora multi-SF channel 2>  radio 1, IF 0 Hz, 125 kHz bw, SF 7 to 12
INFO: Lora multi-SF channel 3>  radio 0, IF -400000 Hz, 125 kHz bw, SF 7 to 12
INFO: Lora multi-SF channel 4>  radio 0, IF -200000 Hz, 125 kHz bw, SF 7 to 12
INFO: Lora multi-SF channel 5>  radio 0, IF 0 Hz, 125 kHz bw, SF 7 to 12
INFO: Lora multi-SF channel 6>  radio 0, IF 200000 Hz, 125 kHz bw, SF 7 to 12
INFO: Lora multi-SF channel 7>  radio 0, IF 400000 Hz, 125 kHz bw, SF 7 to 12
INFO: Lora std channel> radio 1, IF -200000 Hz, 250000 Hz bw, SF 7
INFO: FSK channel> radio 1, IF 300000 Hz, 125000 Hz bw, 50000 bps datarate
INFO: global_conf.json does contain a JSON object named gateway_conf, parsing gateway parameters
INFO: gateway MAC address is configured to AA555A0000000000
INFO: server hostname or IP address is configured to "localhost"
INFO: upstream port is configured to "1680"
INFO: downstream port is configured to "1680"
INFO: downstream keep-alive interval is configured to 10 seconds
INFO: statistics display interval is configured to 30 seconds
INFO: upstream PUSH_DATA time-out is configured to 100 ms
INFO: packets received with a valid CRC will be forwarded
INFO: packets received with a CRC error will NOT be forwarded
INFO: packets received with no CRC will NOT be forwarded
INFO: found local configuration file local_conf.json, parsing it
INFO: redefined parameters will overwrite global parameters
INFO: local_conf.json does not contain a JSON object named SX1301_conf
INFO: local_conf.json does contain a JSON object named gateway_conf, parsing gateway parameters
INFO: gateway MAC address is configured to B827EBFFFE643963
INFO: server hostname or IP address is configured to "49.4.71.166"
INFO: upstream port is configured to "1700"
INFO: downstream port is configured to "1700"
INFO: packets received with a valid CRC will be forwarded
INFO: packets received with a CRC error will NOT be forwarded
INFO: packets received with no CRC will NOT be forwarded
ERROR: [main] failed to start the concentrator

```


## 方案对比

|厂商|方案类型|concentrator|网关|lora server| 参考开源框架|
|-----|-----|-----|----|----|---|
|锐米|室外网关|sx1301|树莓派|lora-server|1.lora_gateway+packet_fowarder 2.开源loraserver|
|semtech|室内网关|sx1308|树莓派|？|picoGW_hal+picoGW_packet_forwarder+picoGW_mcu|

1. 网关主机均选择树莓派2、3，建议选择树莓派提高开发效率
2. concentrator的选择决定了树莓派上运行哪一种程序,锐米教程使用的是sx1301，和semtech提供的教程基本一致：[sx1301+树莓派](https://github.com/Lora-net/packet_forwarder/wiki/Use-with-Raspberry-Pi) ， semtech 使用sx1308(室内网关芯片)
3. lora服务器锐米使用的是开源的方案，semtech暂时没体现lora server

其他方案对比
* [低成本的lora网关](https://github.com/CongducPham/LowCostLoRaGw) SX1272/76 +  Raspberry + Arduino 

* [lora concentrator板开源](https://github.com/will127534/LoRa-concentrator) 开源的sx1301+sx125x lora 网关/集中板

## 网关设计

待补充

