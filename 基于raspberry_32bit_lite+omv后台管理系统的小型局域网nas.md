# 基于raspberry_32bit_lite+omv组建小型局域网nas

编辑于：2023.4.23 

##### 本文档来自于一个小白，刷了7遍系统，一下午+一晚上实践结果

##### 忘记用户密码，刷了第八遍系统    *更新于2023.4.23*

手敲代码量很少，适合新手

参考教程：@@@微信公众号/CSDN/B站：树莓派爱好者基地

### 前期软件，硬件准备

Raspberry pi imager：树莓派官方镜像烧录工具

xshell 7:远程ssh连接工具

读卡器

内存卡

树莓派

一块移动硬盘或者大容量U盘



### 系统镜像的选择及安装

打开“树莓派镜像烧录器”，选择Raspberry Pi OS(other)-Raspberry Pi OS lite(32bit）*一定要选32bit,不然有你好果子吃*

存储卡选择自己的sd卡，点击右下角设置（齿轮图标），配置无头树莓派初始设置

<img src="C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230423000734221.png" alt="image-20230423000734221" style="zoom:67%;" />

必选项目：

开启ssh

设置用户和密码（set username and password） 

配置WIFI开启

​	热点名填写所要连接的wifi名，密码填写wifi密码，wifi国家填写CN

语言设置

​	时区填写：Asia/ChongqingA 就行

一切设置结束后，即可开始烧录镜像

烧录系统完成后，即可把内存卡从读卡器拔出，插入树莓派



### 远程ssh连接

本文档基于笔记本自身移动热点建立，家用路由器同理

开启windows设置-网络和Internet-移动热点，*需要将网络频段设定为2.4Ghz频段，不然树莓派可能无法连接*

移动热点所配置的名称和密码，即刚刚烧录镜像时的wifi名称和密码

##### 忘记配置ssh及wifi设置时的解决方案：

1.将树莓派的内存卡插入读卡器连接电脑，在根目录boots中创建一个ssh文件（没有后缀）

2.在根目录boots中创建一个名为wpa_supplicant.conf的文档，打开并进行如下初始化设置

```
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
ssid="Wifi-A"
psk="xxxxxx"
key_mgmt=WPA-PSK
priority=1
}
```

ssid:wifi名称

psk:wifi密码

priority：优先级

打开ssh远程连接工具xshell-"新建会话"

<img src="C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230423002150270.png" alt="image-20230423002150270" style="zoom: 50%;" />

名称:随便设

协议:SSH

主机:设定为树莓派的ip

##### 树莓派ip查询方式：

1.采用本教程方法的话，会在“移动热点”页面直接显示ip

2.在cmd终端中输入`arp -a`

​	查询内部非255结尾，d开头的mac地址，有可能就是树莓派

3.拿Nmap扫

### 固定ip地址及更换清华源

##### 固定ip地址

在ssh软件中输入	`sudo nano /etc/dhcpcd.conf`

<img src="C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230423230503531.png" alt="image-20230423230503531" style="zoom:67%;" />

static ip_address='当前你树莓派所连接的ip'/‘端口号’

下面的routers,和domain_name_severs中的ip改为192.168.1.1即可，其余不动

然后重启树莓派	sudo reboot

##### 换清华源

在ssh软件中输入	`sudo nano /etc/apt/sources.list`	把原来的注释掉，输入以下内容：

```properties
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ bullseye main non-free contrib rpi
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ bullseye main non-free contrib rpi
```

继续

`sudo nano /etc/apt/sources.list.d/raspi.list`把原来的注释掉，输入以下内容：

```properties
deb https://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ bullseye main
```

记得一定要更新一下

```
sudo apt-get update  
sudo apt-get upgrade
```

最新树莓派清华源地址查询：[raspbian | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirror.tuna.tsinghua.edu.cn/help/raspbian/)



### 安装omv管理系统

```
wget  https://cdn.jsdelivr.net/gh/OpenMediaVault-Plugin-Developers/installScript@master/install
chmod +x install
sudo ./install -n
```

好像第一个cdn静态服务器网站好像挂了，可以更换为：

```
wget https://raw.githubusercontent.com/OpenMediaVault-Plugin-Developers/installScript/master/install
```

*如果你之前装了系统推荐的完整镜像，恭喜你，这步进行不下去了，会去重刷系统吧！记住一定是“32bit lite”版本镜像哟*

### omv管理系统后台配置

在本机浏览器输入树莓派IP地址就可以进入NAS后台管理系统

`用户名默认为admin，密码为openmediavault`

首先在系统设置-工作台里面设置一下登出时间60分钟，之前的太短了

<img src="C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230423232126065.png" alt="image-20230423232126065" style="zoom: 50%;" />

然后把硬盘插在树莓派上

<img src="C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230423232324087.png" alt="image-20230423232324087" style="zoom:50%;" />

在“存储器-磁盘”中搜索刚插上的硬盘/U盘，然后进行“擦除"，完成后关闭

![image-20230423232448502](C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230423232448502.png)

然后在文件系统中-建立并挂在文件系统-EXT4,并选择刚刚格式化的磁盘，保存，应用更改

![image-20230423232608442](C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230423232608442.png)

再点击”共享文件夹”中“创建”

<img src="C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230423232709213.png" alt="image-20230423232709213" style="zoom:50%;" />

输入要共享的文件夹，文件系统选择刚刚创建的文件系统，然后保存，应用更改

![image-20230423232847235](C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230423232847235.png)

接着在“SMB”服务中，勾选第一个工作组，然后保存，应用更改

![image-20230423233026291](C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230423233026291.png)

在共享中，选择刚刚的文件夹，并启动，然后保存，应用更改

![image-20230423233127551](C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230423233127551.png)

最后，在用户中编辑用户密码，然后保存，应用更改

### 本机nas服务器映射

在“网络”里即可发现树莓派服务器

<img src="C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230423233950443.png" alt="image-20230423233950443" style="zoom:50%;" />

输入刚刚omv后台配置的用户即密码就可以进入

网络磁盘映射

打开windows文件资源管理器，右侧三个点-映射网络驱动器，选择树莓派nas共享文件夹即可完成

<img src="C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230423234343632.png" alt="image-20230423234343632" style="zoom:40%;" />

##### 当密码更改后无法进入局域网nas服务器解决办法：

打开“控制面板-凭据管理器-windows凭据”

<img src="C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230423234157100.png" alt="image-20230423234157100" style="zoom:50%;" />

删除树莓派服务器的凭据，或者更改凭据密码

如果点开“···”没有映射网络驱动器选项，解决办法：

打开文件夹选项-查看

<img src="C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230423234550814.png" alt="image-20230423234550814" style="zoom:45%;" />

勾选“使用共享相当（推荐）”，即可出现映射网络驱动器



## 给NAS安一个小屏幕

启动树莓派的I2C功能

```
sudo apt-get install -y python3-smbus
sudo apt-get install -y i2c-tools
sudo raspi-config
```

选择“Interface Options”进入，选择“i2c”打开，确认后按esc退出

重启树莓派 sudo reboot

### 安装Adafruit-SSD1306库

Adafruit-SSD1306库是基于Python的OLED库，可以用于１２８＊６４，１２８＊３２像素SSD1306芯片控制的屏幕

适用于由ssd1306驱动控制的oled显示屏

```
sudo apt-get remove  python3-pip
sudo apt-get install  python3-pip
sudo python3 -m pip install --upgrade pip setuptools wheel
```

最后一条指令有时可能会报error,重新输入再次下载安装就行

安装PIL库,有一些图片处理的程序会用到这个。

`sudo apt-get install python3-pil`

使用pip安装Adafruit-SSD1306库

`sudo pip install Adafruit-SSD1306`

再下载一份包含代码示例的库后面用

```
cd  ~
sudo apt install git
git clone https://github.com/adafruit/Adafruit_Python_SSD1306.git
cd ~/Adafruit_Python_SSD1306/examples/
```

对于屏幕的接线，一定不要接错，树莓派引脚(所有树莓派4B针引脚都是这样排列，不需要因为不同版本而改动)如下图所示：

<img src="C:\Users\darkb\AppData\Roaming\Typora\typora-user-images\image-20230424000217417.png" alt="image-20230424000217417" style="zoom:60%;" />

根据屏幕 PCB 上引脚的功能标注接到树莓派上对应的 GPIO 上即可。

```
屏幕 GND 接树莓派 GND
屏幕 VCC 接树莓派 3V3/5V
屏幕 SDA 接树莓派 SDA
屏幕 SCL 接树莓派 SCL
```

*注意一定不要接反 VCC 和 GND，否则会烧坏屏幕!!!*

接上之后通过命令检测是否识别到i2c设备

`sudo i2cdetect -y 1`

修改一下程序

```
cd ~
sudo cp ~/Adafruit_Python_SSD1306/examples/stats.py ~/
sudo nano stats.py
```

把文件里面的内容全替换成下面的内容

```
import time
import Adafruit_GPIO.SPI as SPI
import Adafruit_SSD1306
from PIL import Image
from PIL import ImageDraw
from PIL import ImageFont
import subprocess
# Raspberry Pi pin configuration:
RST = None     # on the PiOLED this pin isnt used
# Note the following are only used with SPI:
DC = 23
SPI_PORT = 0
SPI_DEVICE = 0
# Beaglebone Black pin configuration:
# RST = 'P9_12'
# Note the following are only used with SPI:
# DC = 'P9_15'
# SPI_PORT = 1
# SPI_DEVICE = 0
# 128x32 display with hardware I2C:
#disp = Adafruit_SSD1306.SSD1306_128_32(rst=RST)
# 128x64 display with hardware I2C:
disp = Adafruit_SSD1306.SSD1306_128_64(rst=RST)
# Note you can change the I2C address by passing an i2c_address parameter like:
disp = Adafruit_SSD1306.SSD1306_128_64(rst=RST, i2c_address=0x3C)
# Alternatively you can specify an explicit I2C bus number, for example
# with the 128x32 display you would use:
# disp = Adafruit_SSD1306.SSD1306_128_32(rst=RST, i2c_bus=2)


# 128x32 display with hardware SPI:
# disp = Adafruit_SSD1306.SSD1306_128_32(rst=RST, dc=DC, spi=SPI.SpiDev(SPI_PORT, SPI_DEVICE, max_speed_hz=8000000))
# 128x64 display with hardware SPI:
# disp = Adafruit_SSD1306.SSD1306_128_64(rst=RST, dc=DC, spi=SPI.SpiDev(SPI_PORT, SPI_DEVICE, max_speed_hz=8000000))


# Alternatively you can specify a software SPI implementation by providing
# digital GPIO pin numbers for all the required display pins.  For example
# on a Raspberry Pi with the 128x32 display you might use:
# disp = Adafruit_SSD1306.SSD1306_128_32(rst=RST, dc=DC, sclk=18, din=25, cs=22)


# Initialize library.
disp.begin()


# Clear display.
disp.clear()
disp.display()


# Create blank image for drawing.
# Make sure to create image with mode '1' for 1-bit color.
width = disp.width
height = disp.height
image = Image.new('1', (width, height))


# Get drawing object to draw on image.
draw = ImageDraw.Draw(image)


# Draw a black filled box to clear the image.
draw.rectangle((0,0,width,height), outline=0, fill=0)
# Draw some shapes.
# First define some constants to allow easy resizing of shapes.
padding = -2
top = padding
bottom = height-padding
# Move left to right keeping track of the current x position for drawing shapes.
x = 0
# Load default font.
font = ImageFont.load_default()

# Alternatively load a TTF font.  Make sure the .ttf font file is in the same directory as the python script!
# Some other nice fonts to try: http://www.dafont.com/bitmap.php
# font = ImageFont.truetype('Minecraftia.ttf', 8)
def get_cpu_temp():
        tempfile=open("/sys/class/thermal/thermal_zone0/temp")
        cpu_temp=tempfile.read()
        tempfile.close()
        return float(cpu_temp)/1000
while True:


    # Draw a black filled box to clear the image.
    draw.rectangle((0,0,width,height), outline=0, fill=0)


    # Shell scripts for system monitoring from here : https://unix.stackexchange.com/questions/119126/command-to-display-memory-usage-disk-usage-and-cpu-load
    cmd = "hostname -I | cut -d' ' -f1"
    IP = subprocess.check_output(cmd, shell = True ).decode("utf-8")
    cmd = "top -bn1 | grep load | awk '{printf \"CPU Load: %.2f\", $(NF-2)}'"
    CPU = subprocess.check_output(cmd, shell = True ).decode("utf-8")
    cmd = "free -m | awk 'NR==2{printf \"Mem: %s/%sMB %.2f%% \", $3,$2,$3*100/$2 }'"
    MemUsage = subprocess.check_output(cmd, shell = True ).decode("utf-8")
    cmd = "df -h | awk '$NF==\"/\"{printf \"Disk: %d/%dGB %s\", $3,$2,$5}'"
    Disk = subprocess.check_output(cmd, shell = True ).decode("utf-8")


    # Write two lines of text.


    draw.text((x, top),       "IP: " + str(IP),  font=font, fill=255)
    draw.text((x, top+8),     str(CPU), font=font, fill=255)
    draw.text((x, top+16),    str(MemUsage),  font=font, fill=255)
    draw.text((x, top+25),    str(Disk),  font=font, fill=255)
    draw.text((x, top+35),    "Temp: "+str(get_cpu_temp()), font=font, fill=255)


    # Display image.
    disp.image(image)
    disp.display()
    time.sleep(.1)
```

为了让stats.py能够开机自动运行,我们可以做下面的配置，这样我们就可以不用通过工具或路由器去查找树莓派的IP地址等信息！！！

修改/etc/rc.local文件	`sudo nano /etc/rc.local`

在exit 0前面增加一行：

`sudo python /home/pi/stats.py &`

## 最后查看效果

`sudo python stats.py`





