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

   ![配置终端提示符](https://s1.ax1x.com/2018/09/18/iZvy28.png)

3. `make ARCH=arm CROSS_COMPILE=arm-suda-linux-musleabi- `



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



## sunxi-tools安装

1. `sudo apt-get install zlib1g-dev libusb-1.0-0-dev`
2. `make && sudo make install`



## 参考文献

[crosstool-NG官方文档](https://crosstool-ng.github.io/docs/)

[荔枝派nano官方文档](http://nano.lichee.pro/)