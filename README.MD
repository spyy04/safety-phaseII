# lora网关设计

## UPDATE 2019.8.2 尝试了三种方案，均成功验证
以上方案均采用 node: 锐米终端，concentrator：rak2245，网关：树莓派3b+，lora服务器：自建开源lora服务器。仅仅是**固件版本**的不同

### 方案一 rak提供的树莓派3b+固件

1. 先下载固件 [RAK2245_based_on_Raspbian_OS_V2.9.1_20190717](https://downloads.rakwireless.com/cn/LoRa/RAK2245-Pi-HAT/C%E5%9B%BA%E4%BB%B6/RAK2245_based_on_Raspbian_OS_V2.9.1_20190717.zip)
![](https://i.loli.net/2019/08/03/NovBUwn9YDtmMOa.png)

2. 使用 eTcher 烧录该镜像到SD卡中，具体过程参考 [etcher烧录镜像文档](http://docs.rakwireless.com/en/LoRa/RAK2245-Pi-HAT/Tool/How_to_write_LoRa_Gatew ay_Image_to_Micro_SD.pdf)
![](https://i.loli.net/2019/08/03/VPTX7JrRC8npUxO.png)

3. 等待烧录完成，退出软件。拔掉sdcard，接上树莓派。按照教程[LoRaWAN树莓派网关使用指南](https://downloads.rakwireless.com/cn/LoRa/LoRaWAN%E7%BD%91%E5%85%B3%E5%85%A5%E9%97%A8%E5%BC%80%E5%8F%91%E5%A5%97%E4%BB%B6/%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97/LoRaWAN%E6%A0%91%E8%8E%93%E6%B4%BE%E7%BD%91%E5%85%B3%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97.pdf)设置好参数，然后设置接入 lora server，填写自建的地址，保存即可。

4. 此过程比较简单，根据提示设置即可。

### 方案二 lora server 开源项目提供的 lora gateway os 固件

此固件和上面的rak固件很像，不过设置方面没有rak那么好用，需要做一些额外的设置即可

1. 下载最新 gateway os full 固件[lora-gateway-os-full](https://artifacts.loraserver.io/downloads/lora-gateway-os/raspberrypi/raspberrypi3/3.0.0test1/lora-gateway-os-full-raspberrypi3--20190714105631.sdimg.gz)
![](https://i.loli.net/2019/08/03/RB6wEfuyCDYKvZW.png)

- 虽然镜像是给树莓派3准备的，但是亲测树莓派3b+ 也是可以正常运行的
- 镜像分为 lora-gateway-os-base 和 lora-gateway-os-full 版本，考虑到后者在功能上完全兼容前者，并且内置了lora 服务器和 lora 应用服务器，更加方便调试及减少了搭建服务器的成本。因此选用lora-gateway-os-full 版本镜像

2. 使用 eTcher 烧录该镜像到SD卡中，同上，比较简单。

3. sd卡插入树莓派，开机启动.参考教程[配置教程](https://www.loraserver.io/lora-gateway-os/use/getting-started/)。
![](https://www.loraserver.io/lora-gateway-os/img/gateway-config.png)

4. 默认是开启ssh的，且用户名和密码均为 admin、admin。由于该固件为国外开发的，所以找不到 CN-470频段，不过也没有关系，我们可以手动去修该 global_conf.json文件来实现 支持CN-470频段。

用以下内容替换原有的global_confi.json文件，注意需要设置最后面的 "server_address" 值为 自建的lora服务器或者是 127.0.0.1 本地内置lora服务器。

```json
{
	"SX1301_conf": {
		"lorawan_public": true,
		"clksrc": 1,
		"clksrc_desc": "radio_1 provides clock to concentrator for most devices except MultiTech. For MultiTech set to 0.",
		"antenna_gain": 0,
		"antenna_gain_desc": "antenna gain, in dBi",
		"radio_0": {
			"enable": true,
			"type": "SX1255",
			"freq": 486600000,
			"rssi_offset": -176.0,
			"tx_enable": true,
			"tx_freq_min": 470000000,
			"tx_freq_max": 510000000
		},
		"radio_1": {
			"enable": true,
			"type": "SX1255",
			"freq": 487400000,
			"rssi_offset": -176.0,
			"tx_enable": false
		},
		"chan_multiSF_0": {
			"desc": "Lora MAC, 125kHz, all SF, 487.1 MHz",
			"enable": true,
			"radio": 1,
			"if": -300000
		},
		"chan_multiSF_1": {
			"desc": "Lora MAC, 125kHz, all SF, 487.3 MHz",
			"enable": true,
			"radio": 1,
			"if": -100000
		},
		"chan_multiSF_2": {
			"desc": "Lora MAC, 125kHz, all SF, 487.5 MHz",
			"enable": true,
			"radio": 1,
			"if": 100000
		},
		"chan_multiSF_3": {
			"desc": "Lora MAC, 125kHz, all SF, 487.7 MHz",
			"enable": true,
			"radio": 1,
			"if": 300000
		},
		"chan_multiSF_4": {
			"desc": "Lora MAC, 125kHz, all SF, 486.3 MHz",
			"enable": true,
			"radio": 0,
			"if": -300000
		},
		"chan_multiSF_5": {
			"desc": "Lora MAC, 125kHz, all SF, 486.5 MHz",
			"enable": true,
			"radio": 0,
			"if": -100000
		},
		"chan_multiSF_6": {
			"desc": "Lora MAC, 125kHz, all SF, 486.7 MHz",
			"enable": true,
			"radio": 0,
			"if": 100000
		},
		"chan_multiSF_7": {
			"desc": "Lora MAC, 125kHz, all SF, 486.9 MHz",
			"enable": true,
			"radio": 0,
			"if": 300000
		},
		"chan_Lora_std": {
			"desc": "disabled",
			"enable": false
		},
		"chan_FSK": {
			"desc": "disabled",
			"enable": false
		},
		"tx_lut_0": {
			"desc": "TX gain table, index 0",
			"pa_gain": 0,
			"mix_gain": 8,
			"rf_power": -6,
			"dig_gain": 0
		},
		"tx_lut_1": {
			"desc": "TX gain table, index 1",
			"pa_gain": 0,
			"mix_gain": 10,
			"rf_power": -3,
			"dig_gain": 0
		},
		"tx_lut_2": {
			"desc": "TX gain table, index 2",
			"pa_gain": 0,
			"mix_gain": 12,
			"rf_power": 0,
			"dig_gain": 0
		},
		"tx_lut_3": {
			"desc": "TX gain table, index 3",
			"pa_gain": 1,
			"mix_gain": 8,
			"rf_power": 3,
			"dig_gain": 0
		},
		"tx_lut_4": {
			"desc": "TX gain table, index 4",
			"pa_gain": 1,
			"mix_gain": 10,
			"rf_power": 6,
			"dig_gain": 0
		},
		"tx_lut_5": {
			"desc": "TX gain table, index 5",
			"pa_gain": 1,
			"mix_gain": 12,
			"rf_power": 10,
			"dig_gain": 0
		},
		"tx_lut_6": {
			"desc": "TX gain table, index 6",
			"pa_gain": 1,
			"mix_gain": 13,
			"rf_power": 11,
			"dig_gain": 0
		},
		"tx_lut_7": {
			"desc": "TX gain table, index 7",
			"pa_gain": 2,
			"mix_gain": 9,
			"rf_power": 12,
			"dig_gain": 0
		},
		"tx_lut_8": {
			"desc": "TX gain table, index 8",
			"pa_gain": 1,
			"mix_gain": 15,
			"rf_power": 13,
			"dig_gain": 0
		},
		"tx_lut_9": {
			"desc": "TX gain table, index 9",
			"pa_gain": 2,
			"mix_gain": 10,
			"rf_power": 14,
			"dig_gain": 0
		},
		"tx_lut_10": {
			"desc": "TX gain table, index 10",
			"pa_gain": 2,
			"mix_gain": 11,
			"rf_power": 16,
			"dig_gain": 0
		},
		"tx_lut_11": {
			"desc": "TX gain table, index 11",
			"pa_gain": 3,
			"mix_gain": 9,
			"rf_power": 20,
			"dig_gain": 0
		},
		"tx_lut_12": {
			"desc": "TX gain table, index 12",
			"pa_gain": 3,
			"mix_gain": 10,
			"rf_power": 23,
			"dig_gain": 0
		},
		"tx_lut_13": {
			"desc": "TX gain table, index 13",
			"pa_gain": 3,
			"mix_gain": 11,
			"rf_power": 25,
			"dig_gain": 0
		},
		"tx_lut_14": {
			"desc": "TX gain table, index 14",
			"pa_gain": 3,
			"mix_gain": 12,
			"rf_power": 26,
			"dig_gain": 0
		},
		"tx_lut_15": {
			"desc": "TX gain table, index 15",
			"pa_gain": 3,
			"mix_gain": 14,
			"rf_power": 27,
			"dig_gain": 0
		}
	},
	"gateway_conf": {
		"gateway_ID": "0000000000000000",
		/* change with default server address/ports, or overwrite in local_conf.json */
		"server_address": "自建服务器地址",
		"serv_port_up": 1700,
		"serv_port_down": 1700,
		/* adjust the following parameters for your network */
		"keepalive_interval": 10,
		"stat_interval": 30,
		"push_timeout_ms": 100,
		/* forward only valid packets */
		"forward_crc_valid": true,
		"forward_crc_error": false,
		"forward_crc_disabled": false,
		/* gps enable */
		"gps": true,
		"gps_tty_path": "/dev/ttyAMA0",
		"fake_gps": false,
		"ref_latitude": 10,
		"ref_longitude": 20,
		"ref_altitude": -1,
		"autoquit_threshold": 6
	}
}
```
5. 在 gateway-config 中重启各个模块即可。



### 方案三  树莓派3b+官方固件 + semtech 软件

1. 下载一个树莓派3b+的官方固件，我使用的是 [Raspbian 2019-07-10 3GB Buster LITE (rpi3+4.img.zip)](https://www.raspberrypi.org/downloads/raspbian/)

2. 不需要解压，烧录同样可以使用etcher去烧录，比较简单

3. 烧录完成后，接着在boot分区的根目录下新建一个 ssh空白文件，该文件没有后缀名！！！。因为官方默认禁用ssh连接，新建该空白ssh文件可以实现启动ssh。

4. sd卡插入树莓派，开机启动，默认用户名和密码分别为：pi、raspberry。

- ssh进入终端，输入 `sudo raspi-config` 命令进入配置页面

- ![](https://i.loli.net/2019/08/03/QewhHLn4Itb28lB.png)

- ![](https://i.loli.net/2019/08/03/2cyCFRBZ4WTpv1x.png)

- 依次在 `Interfacing Options` 选项中开启 SPI、I2C、SSH，在 `Advanced Options` 选项中 开启 `Expanded Filesystem...`。重启之后再ssh重新连接。
![](https://i.loli.net/2019/08/03/LjrKcIozinH9SwG.png)
![](https://i.loli.net/2019/08/03/rVSljTJoI4q5yPv.png)

5. 下载并编译软件，使用修改后的semtech软件。

```bash
# home/pi目录下
mkdir lora-net && cd lora-net
# 下载修改后的源代码
git clone ssh://git@49.4.68.29:2222/fangkai/lora-net.git

# 编译lora-gateway
# 进入网关目录 
cd lora-gateway
# 编译
make clean all

# 进入packet_forwarder
cd ..
cd packet_forwarder/
# 编译packet_forwarder
make clean all

# 接着进入lora_pkt_fwd/目录
# 修改里面的local_conf.json中的 网关eui，设置成任意或者设置成网卡的mac地址也可以，这个随意
# 运行使用sudo bash start sh
sudo bash start.sh

```
![](https://i.loli.net/2019/08/03/a5EVMADkTNf9neQ.png)
![](https://i.loli.net/2019/08/03/uOiXS2vL4QhYTpr.png)

运行成功！


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

## 未连接concentrator 的情况下运行的日志

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

1、室内网关方案

* sx1308+MCU+树莓派3+开源lora server

2、室外网关方案

* sx1301+树莓派3+开源lora server



## 参考链接

(https://github.com/RAKWireless/RAK2245-RAK831-LoRaGateway-RPi-Raspbian-OS)
(https://github.com/RAKWireless/rak_common_for_gateway)
