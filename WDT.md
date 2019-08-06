# 树莓派3b+ 设置看门狗

## 加载驱动

树莓派默认是不启用 bcm2835_wdt（ 以前版本为bcm2708_wdog，至于是多久以前，这个我也不清楚。）模块的，所以我们要用命令启用这个模块。登入命令行，执行这条命令：

```bash
sudo modprobe bcm2835_wdt
```

/etc/modules 进行编辑，添加 bcm2835_wdt

```bash
echo "bcm2835_wdt" | sudo tee -a /etc/modules
```

## 安装看门狗

硬件看门狗需要和软件通信来确定系统的状态。在 Raspbian 下这个软件是 watchdog，可以直接 apt-get 安装：

```bash
sudo apt-get install watchdog
```

然后由于显而易见的原因，要把它设置为开机启动：

```bash
systemctl enable watchdog
```

## 配置看门狗

然后编辑配置文件 /etc/watchdog.conf，作出如下修改：

1. 取消 #max-load-1 = 24 的注释（删除开头的 # 号），代表当系统 1 分钟内的负载高于 24（已经非常非常高了），就重启系统
2. 取消 #watchdog-device = /dev/watchdog 的注释，设置看门狗的路径
3. 增加一行 watchdog-timeout = 15，代表 15 秒内系统无响应就重启系统，在树莓派 3B 上这个值最高为15。注意不要设置的太小，否则可能造成系统反复重启。


```conf
#ping			= 172.31.14.1
#ping			= 172.26.1.255
#interface		= eth0
#file			= /var/log/messages
#change			= 1407

# Uncomment to enable test. Setting one of these values to '0' disables it.
# These values will hopefully never reboot your machine during normal use
# (if your machine is really hung, the loadavg will go much higher than 25)
max-load-1		= 24
#max-load-5		= 18
#max-load-15		= 12

# Note that this is the number of pages!
# To get the real size, check how large the pagesize is on your machine.
#min-memory		= 1
#allocatable-memory	= 1

#repair-binary		= /usr/sbin/repair
#repair-timeout		= 60
#test-binary		=
#test-timeout		= 60

# The retry-timeout and repair limit are used to handle errors in a more robust
# manner. Errors must persist for longer than retry-timeout to action a repair
# or reboot, and if repair-maximum attempts are made without the test passing a
# reboot is initiated anyway.
#retry-timeout		= 60
#repair-maximum		= 1

watchdog-device	= /dev/watchdog
watchdog-timeout = 15

# Defaults compiled into the binary
#temperature-sensor	=
#max-temperature	= 90

# Defaults compiled into the binary
#admin			= root
#interval		= 1
#logtick                = 1
#log-dir		= /var/log/watchdog

# This greatly decreases the chance that watchdog won't be scheduled before
# your machine is really loaded
realtime		= yes
priority		= 1

# Check if rsyslogd is still running by enabling the following line
#pidfile		= /var/run/rsyslogd.pid
```

保存修改，重启看门狗服务：

```bash
service watchdog restart
```

## 测试是否成功

可以通过 kill 掉看门狗服务来模拟系统死机的情况：

```bash
pkill -9 watchdog
pkill -9 wd_keepalive
```
过 15 秒后树莓派就会自动重启。