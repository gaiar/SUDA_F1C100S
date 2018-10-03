# SUDA_F1C100S 把玩记录
### 定制交叉编译工具链——使用crosstool-ng脚本

1. 安装必要的依赖软件

   `apt-get install gcc gperf bison flex texinfo help2man make libncurses5-dev python-dev`

2. 进入crosstool-ng根目录下，运行`./bootstrap`

3. 执行安装之前的配置

   ```bash
   ./configure --prefix=/some/place
   make && make install
   ```

4. 添加环境变量

   ```bash
   export PATH=$PATH:/some/place/ct-ng/bin
   source /some/place/ct-ng/share/bash-completion/completions/ct-ng
   ```

5. 查看示例，选择arm-unknown-linux-musleabi，然后微调配置

   ```bash
   ct-ng list-samples
   ct-ng arm-unknown-linux-musleabi
   ct-ng menuconfig
   ```

   ![Path and misc options](https://s1.ax1x.com/2018/09/15/iVV0ln.png)

   ![Target options](https://s1.ax1x.com/2018/09/15/iVVNFg.png)

   ![Toolchain options](https://s1.ax1x.com/2018/09/15/iVVByq.png)

   ![Operating System子配置菜单](https://s1.ax1x.com/2018/09/15/iVELMq.png)

6. 编译`ct-ng build`



## 实验板——荔枝派nano

![引脚映射图](http://odfef978i.bkt.clouddn.com/Pin%20Map.png)

## u-boot

### 目录结构

```bash
.
├── api             //封装一些平台无关的操作，如字符串打印，显示，网络，内存
├── arch            //以平台架构区分
│   ├──arm
│   │   └──cpu
│   │   │   └──arm926ejs
│   │   │   │   └──sunxi   //cpu相关的一些操作，如定时器读取
│   │   │   │   │   └──u-boot-spl.lds  //spl的放置方法
│   │   └──dts
│   │   │   └──suniv-f1c100s-licheepi-nano.dts   // f1c100s芯片的一些配置
│   │   │   └──suniv-f1c100s-licheepi-nano.dtb
│   │   │   └──suniv-f1c100s.dtsi
│   │   │   └──suniv.dtsi
│   │   └──lib      //一些库文件
│   │   └──mach-sunxi
│   │   │   └──board.c          //board_init_f
│   │   │   └──dram_sun4i.c     //ddr的操作，复位，时钟，延时，odt，etc.
│   │   │   └──dram_helpers.c   //ddr的设置及读写测试
├── board
│   ├──sunxi
│   │   └──board.c              //sunxi_board_init 入口
│   │   └──dram_suniv.c        //DRAM的一些默认参数
├── cmd             //Uboot命令行的一些命令
├── common          //含spl
├── configs         //menuconfig里的默认配置,比如各类驱动适配
│   ├── licheepi_nano_defconfig
│   ├── licheepi_nano_spiflash_defconfig
├── disk            //硬盘分区的驱动
├── doc
├── drivers         //外设驱动
├── dts
├── examples
├── fs              //多种文件系统
├── include
│   ├──configs
│   │   └──sunxi_common.h   //预配置的参数，如串口号等
│   │   └──suniv.h
├── lib             //加密压缩等算法
├── net             //nfs,tftp等网络协议
├── post
├── scripts
```

### 安装依赖软件包

`sudo apt-get install swig`

### 配置&编译

1. `make ARCH=arm CROSS_COMPILE=arm-suda-linux-musleabi- licheepi_nano_spiflash_defconfig` 启用默认配置

2. `make ARCH=arm CROSS_COMPILE=arm-suda-linux-musleabi- menuconfig` 微调配置

   ![配置液晶显示屏参数](https://s1.ax1x.com/2018/09/18/iZvB5t.png)

   ![指定设备树文件](https://s1.ax1x.com/2018/10/03/i3lqwq.png)

   ![配置终端提示符](https://s1.ax1x.com/2018/09/18/iZvy28.png)

3. `make ARCH=arm CROSS_COMPILE=arm-suda-linux-musleabi- `

### 添加u-boot开机画面

1. `sudo apt-get install netpbm`

2. `pngtopnm sunxi.png | ppmquant 31 | ppmtobmp -bpp 8 > sunxi.bmp`，这里的sunxi代表u-boot中`board`环境变量的值

3. 将生成的bmp文件放入tools/logos文件夹下

4. 在配置文件`vim include/configs/sunxi-common.h`中添加如下信息

   ```c
   #define CONFIG_VIDEO_LOGO
   #define CONFIG_VIDEO_BMP_LOGO
   #define CONFIG_HIDE_LOGO_VERSION
   ```

### 烧写到spi-flash中（使用sunxi-tools工具）

```bash
sudo sunxi-fel -p spiflash-write 0 u-boot-sunxi-with-spl.bin 
100% [================================================]  1008 kB,   95.3 kB/s 
```

### 运行输出

```bash
U-Boot SPL 2018.01suda-05679-g013ca457fd (Oct 03 2018 - 10:14:37)            
DRAM: 32 MiB                                                                 
Trying to boot from MMC1                                                     
Card did not respond to voltage select!                                      
mmc_init: -95, time 22                  
spl: mmc init failed with error: -95    
Trying to boot from sunxi SPI           
                                        
                                        
U-Boot 2018.01suda-05679-g013ca457fd (Oct 03 2018 - 10:14:37 +0800) Allwinner Ty

CPU:   Allwinner F Series (SUNIV)
Model: Lichee Pi Nano
DRAM:  32 MiB                                                                   
MMC:   SUNXI SD/MMC: 0                                                          
SF: Detected w25q128bv with page size 256 Bytes, erase size 4 KiB, total 16 MiB 
*** Warning - bad CRC, using default environment                                
                                                                                
Setting up a 480x272 lcd console (overscan 0x0)                                 
In:    serial@1c25000                                                           
Out:   serial@1c25000                                                           
Err:   serial@1c25000                                                           
Net:   No ethernet found.                                                       
starting USB...                                                                 
No controllers found                                                            
Hit any key to stop autoboot:  0                                                
Card did not respond to voltage select!                                         
mmc_init: -95, time 22                                                          
starting USB...                                                                 
No controllers found                                                            
USB is stopped. Please issue 'usb start' first.                                 
starting USB...                                                                 
No controllers found                                                            
No ethernet found.                                                              
missing environment variable: pxeuuid                                           
missing environment variable: bootfile                                          
Retrieving file: pxelinux.cfg/00000000                                          
No ethernet found.                                                              
missing environment variable: bootfile                                          
Retrieving file: pxelinux.cfg/0000000                                           
No ethernet found.                                                              
missing environment variable: bootfile                                          
Retrieving file: pxelinux.cfg/000000                                            
No ethernet found.                                                              
missing environment variable: bootfile                                          
Retrieving file: pxelinux.cfg/00000                                             
No ethernet found.                                                              
missing environment variable: bootfile                                          
Retrieving file: pxelinux.cfg/0000                                              
No ethernet found.                                                              
missing environment variable: bootfile                                          
Retrieving file: pxelinux.cfg/000                                               
No ethernet found.                                                              
missing environment variable: bootfile                                          
Retrieving file: pxelinux.cfg/00                                                
No ethernet found.                                                              
missing environment variable: bootfile                                          
Retrieving file: pxelinux.cfg/0                                                 
No ethernet found.                                                              
missing environment variable: bootfile                                          
Retrieving file: pxelinux.cfg/default-arm-sunxi                                 
No ethernet found.                                                              
missing environment variable: bootfile                                          
Retrieving file: pxelinux.cfg/default-arm                                       
No ethernet found.                                                              
missing environment variable: bootfile                                          
Retrieving file: pxelinux.cfg/default                                           
No ethernet found.                                                              
Config file not found                                                           
starting USB...                                                                 
No controllers found                                                            
No ethernet found.                                                              
No ethernet found.                                                              
suda#
```



## kernel

### 安装依赖软件包

`sudo apt install libssl-dev`

### 配置&编译

1. `wget http://nano.lichee.pro/_static/step_by_step/lichee_nano_linux.config` 下载荔枝派nano的默认内核配置文件，`cp lichee_nano_linux.config ./config` 启用默认配置

2. `make ARCH=arm CROSS_COMPILE=arm-suda-linux-musleabi- menuconfig` 微调配置

   ![设置本地版本号](https://s1.ax1x.com/2018/09/18/iZxeit.png)

   ![选择CPU型号](https://s1.ax1x.com/2018/09/18/iZxnRf.png)

   ![文件系统配置](https://s1.ax1x.com/2018/09/18/iZxYiq.png)

3. `make ARCH=arm CROSS_COMPILE=arm-suda-linux-musleabi- -j4` 编译



## rootfs

### 配置&编译

1. `make ARCH=arm CROSS_COMPILE=arm-suda-linux-musleabi- menuconfig`

   ![Target配置](https://s1.ax1x.com/2018/09/18/ieJusf.png)

   ![Toolchain配置](https://s1.ax1x.com/2018/09/18/ieJcS1.png)

   ![System配置](https://s1.ax1x.com/2018/09/18/ieJoYd.png)

   ![getty设置](https://s1.ax1x.com/2018/09/18/ieJHSI.png)

   ![软件包配置](https://s1.ax1x.com/2018/09/18/ieYoHU.png)

2. `make ARCH=arm CROSS_COMPILE=arm-suda-linux-musleabi-` 下载源码，编译，打包生成最终的根文件系统

   

   



## sunxi-tools安装

1. `sudo apt-get install zlib1g-dev libusb-1.0-0-dev`
2. `make && sudo make install`

### sunxi-tools命令使用

> 进入fel模式需要在上电前拉低spi-flash的`cs`引脚（一般是1号脚），上电后再松开，可以通过`sudo sunxi-fel ver`命令来确认有无成功进入fel模式

* 基本命令使用

1. 查看芯片信息

   `sudo sunxi-fel ver`

2. 列出所有芯片的信息

   `sudo sunxi-fel -l`

3. 加载并执行uboot的spl

   `sudo sunxi-fel spl file`

4. 加载并执行uboot

   `sudo sunxi-fel uboot file-with-spl`

5. 把文件内容写入内存指定地址并显示进度

   `sudo sunxi-fel -p write address file`

6. 运行指定地址的函数

   `sudo sunxi-fel exec 地址`

7. 查看spiflash的信息

   `sudo sunxi-fel spiflash-info`

8. 显示spiflash指定地址的数据并写入到文件

   `sudo sunxi-fel spiflash-read addr length file`

9. 写入指定文件的指定长度的内容到spiflash的指定地址

   `sudo sunxi-fel spiflash-write address file`



## 参考文献

[crosstool-NG官方文档](https://crosstool-ng.github.io/docs/)

[荔枝派nano官方文档](http://nano.lichee.pro/)

[sunxi官网](https://linux-sunxi.org/Main_Page)
