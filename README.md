# SUDA_F1C100S
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







## 参考文献

[crosstool-NG官方文档](https://crosstool-ng.github.io/docs/)