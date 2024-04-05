# 协议分析

## USB解题

### 鼠标：

#### 1.直接使用现成的脚本 UsbMiceDataHacker.py 

#### 2.

1. 将pcap文件中的usbdata数据导出

   tshark -r usb.pcap -T fields -e usb.capdata > usbdata.txt
   tshark -r usb.pcap -T fields -e usb.capdata | sed '/^\s*$/d' > usbdata.txt #提取并去除空行

2. 使用脚本将文件加入冒号

   **需要注意的是鼠标流量长度为8，键盘流量长度为16**

3. 使用脚本测试信息隐藏位置，导出xy坐标信息

   [D:\CTF\MISC\USB\Mouse脚本]: 

   

4. 使用pnuplot将坐标转化成图像

   gnuplot
   gnuplot>plot "xy.txt"

   

###  键盘：

#### 1.直接使用现成的脚本UsbKeyboardDataHacker

#### 2.

1.  将pcap文件中的ubsdata数据导出

   tshark -r usb.pcap -T fields -e usb.capdata | sed '/^\s*$/d' > usbdata.txt #提取并去除空行

2. 使用脚本将文件加入冒号

   **需要注意的是鼠标流量长度为8，键盘流量长度为16**

3. 使用脚本还原对应数据信息

   [D:\CTF\MISC\USB\Keyboard脚本]: 



## TLS解题

1.导出密钥文件：TCP协议是文件传输协议，一般导出TCP—data

2.找到对应端口，导入秘钥，让wireshark自动选择相应的TLS流解密：

> 编辑-首选项-RSA keys list edit ‘+’ -选择端口，解密的协议（http），秘钥位置，-过滤http contains "INS"-找到数据包中http解析的flag

## IEEE 802.11解题

1.利用kali找到加密方式，口令

aircrack-ng -w password.txt ctf.pcap

-w 后面是准备好的字典

2.wireshark中导入口令自动解密

keytype:wep、wep-pwd、wpa-psk、tk

key 密码加口令

3.在http中找到flag

# 日志分析

## 一.web日志及分析方法

### 1.NCSA日志

#### (1).SQL注入

时间盲注每次都会利用!=判断当前ascii是否正确，将每个"!="后的ASCII码转换成字符拼接起来 (sqllog.py)

#### (2).日志内容较大

导入wps利用wps的一些统计、排序功能找到可疑流量

## 二.系统设备日志

# 电子取证

## 1.文件恢复

（1）PNG文件格式数据块：IHDR、gAMA、pHsy等

根据数据块判断文件类型，修复文件头

(2)Vim文件修复：利用strings命令或者010editor打开源文件，可以看到flag

## 2.磁盘恢复

## 3.内存取证

Volatility命令改为 vol.py -f .......

1.先查看profile信息 vol.py -f  [文件名] imageinfo

2.查看文件列表，寻找可疑文件导出 vol.py -f [文件名] filescan > file.txt

vol.py -f [文件名] --porfile=[主机信息] dumpfiles -Q 0X0000000001609628 -n --dump-dir

> -Q 表示使用物理偏移量进行转储（不太懂）
>
> -n表示以文件名保存 
>
> --dump-dir=为目标存储位置



提取进程号位PID的进程：vol.py -f 文件 --profile=xxx memdump -p [PID] -D [dump出的文件保存的位置] 

然后使用binwalk、strings等对进程做进一步分析

# 压缩文件

## 1.压缩包密码爆破

1.暴力破解

2.掩码攻击

知道部分破解全部

3.字典攻击

## 2.CRC碰撞破解压缩包

CRC用来检验数据传输后可能出现的错误，利用工具倒推出明文内容，只限制明文内容较短（1-6）

python2 crc32.py reverse 0x+CRC码

crc是十六进制，所以要加0X

## 3.已知明文攻击

ARCHPR导入已知明文压缩包和加密压缩包，很多题目需要使用多个工具（WinRAR，360压缩，7z，好压）





flag.vmdk可用7z解压，lunix下解压命令：7z  x flag.vmdk -o./



# 无线路由器备份文件

routerpassview可通过备份文件查看用户名和密码，备份文件后缀为bin，一般flag为用户名或密码
