# ubuntu装机流程

## 1. 存储空间分配

安装系统的Installation type步骤中选择Something else。

只需要创建四个分区：

* / Primary Ext4 
* /home Primary Ext4
* swap -> 一般设置为运行内存的1-1.5倍
* EFI -> 200M



## 2. 检查基础环境

```cs
sudo apt-get install build-essential
```



## 3. 安装中文输入法

安装google pinyin输入法

1. **安装汉语语言包**

   ```cs
   sudo apt-get install language-pack-zh-hans
   ```

2. **安装google pinyin输入法和fcitx**

   ```cs
   sudo apt-get install fcitx-googlepinyin
   ```

3. **配置fcitx**

   ```cs
   im-config
   ```

​		点击流程：OK -> YES -> chose fcitx -> OK -> OK -> reboot system

4. **配置google pinyin**

   ```cs
   fcitx-config-gtk3
   ```

   在Input Method中添加Google Pinyin

通过快捷键ctrl + space 或者左shift切换中英文



## 4. 安装nvidia显卡驱动、CUDA、CUDNN

### 4.1 禁用ubuntu原有驱动nouveau

1. **禁用ubuntu原有驱动nouveau**

   ```cs
   sudo gedit /etc/modprobe.d/blacklist.conf
   ```

   最后一行添加：

   > blacklist nouveau
   >
   > options nouveau modeset=0

2. **更新并重启**

   ```cs
   sudo update-initramfs -u
   reboot
   ```

3. **检查是否禁用成功**

   ```cs
   lsmod | grep nouveau
   ```

   没有任何输出代表禁用成功。

### 4.2 安装nvidia驱动

1. **查看显卡型号**

   ```cs
   lspci | grep -i nvidia
   ```

2. **nvidia官网根据显卡型号下载驱动文件**

   https://www.nvidia.com/Download/Find.aspx?lang=en-us#

3. **进入tty模式，关闭图形化界面**

   ```cs
   sudo telinit 3
   ```

   登录依次输入用户名和密码

4. **禁用X-window服务**

   ```cs
   sudo service gdm3 stop
   ```

5. **cd到下载文件所在路径开始安装**

   ```cs
   sudo chmod 777 NVIDIA-Linux-x86_64-525.53.run
   sudo ./NVIDIA-Linux-x86_64-525.53.run –no-opengl-files -no-x-check
   ```

   -no-opengl-files：只安装驱动文件，不安装OpenGL文件。

   -no-x-check：安装驱动时关闭X服务，不设置可能导致安装失败。

   安装过程会出现的选项：

   > * Install Nvidia's 32-bit compatibility libraries?
   >   选择 "No"
   > * Would you like to run the nvidia-xconfig utility to automatically update your X configuration file so that the NVIDIA X driver dill be used dhen you restart X? Any pre-existing X configuration file will be backed up.
   >   选择 "Yes"

6. 返回图形化界面

   ```cs
   sudo service gdm3 start
   ```

7. 验证是否安装成功

   ```cs
   nvidia-smi
   ```

   输出显卡信息则安装成功。

### 4.3 安装CUDA

建议不要在本地安装CUDA，而是在Anaconda环境里根据要求安装cudatoolkit即可：
   ```cs
   conda install -c nvidia cudatoolkit=11.3
   conda install nvidia/label/cuda-11.3.0::cuda-nvcc  # 用于编译cuda工程
   ```

本地安装步骤如下：

1. 查看显卡驱动支持的最大CUDA版本

   ```cs
   nvidia-smi
   ```

   右上角的CUDA Version。

2. 官网查看对应版本的安装流程

   https://developer.nvidia.com/cuda-toolkit-archive

   选择runfile安装形式，**在安装过程中把Driver选项去掉**。

3. 根据提示在bashrc添加环境变量

   ```cs
   export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-XX.X/lib64
   export PATH=/usr/local/cuda-XX.X/bin:$PATH
   ```

4. 测试安装是否成功

   ```cs
   nvcc -V
   ```



### 4.4 安装CUDNN

1. 官网查看对应cuda版本的cudnn版本并下载deb文件

   https://developer.nvidia.com/rdp/cudnn-download

2. 根据官方文档安装cudann

   https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html#install-linux

3. 测试是否安装成功

   ```cs
   sudo apt-get install libfreeimage3 libfreeimage-dev
   cp -r /usr/src/cudnn_samples_v8/ $HOME
   cd  $HOME/cudnn_samples_v8/mnistCUDNN
   make clean && make
   ./mnistCUDNN
   # 测试通过
   Test passed!
   ```
   
   

## 5. 安装ROS

### 5.1 添加清华源

1. 进入tsinghua镜像站，并搜索ubuntu和选择正确版本

   https://mirrors.tuna.tsinghua.edu.cn

2. 文本框中会生成需要的 **sources.list** 文件内容，把内容写入文件

   ```cs
   sudo apt update
   sudo gedit /etc/apt/sources.list
   ```

### 5.2 安装ROS

根据官网wiki安装即可：wiki.ros.org/noetic/Installation/Ubuntu

注意ros版本和ubuntu对应

由于后续要安装anaconda，此时版本存在两个版本的python3，一个是ubuntu自带的，一个是anaconda的。ROS只能使用ubuntu自带的python3，因此在创建工作空间的时候需要指定python：

```cs
catkin_make -DPYTHON_EXECUTABLE=/usr/bin/python3
```

再在bashrc上写入：

```cs
source ~/catkin_ws/devel/setup.bash
```



## 6. 安装anaconda3

 1. 官网安装anaconda

    https://www.anaconda.com/download

 2. 安装anaconda

    ```cs
    bash Anaconda3-XXXX.sh
    ```

    **安装完成后，会收到是否自动加入环境变量的提示信息，输入no**

 3. 手动添加激活conda环境指令

    ```cs
    sudo gedit ~/.bashrc
    ```

    在文件最后添加

    > export PATH="/home/your_name/anaconda3/bin:$PATH"
    >
    > alias setconda='. ~/anaconda3/bin/activate'



# 7. 固定内核

防止ubuntu自动更新内核

1. 查看正在使用的内核

   ```cs
   uname -a
   ```

2. 禁止内核更新(4.4.0-21 is the version of kernel you are used)

   ```cs
   sudo apt-mark hold linux-image-4.4.0-21-generic
   ```

3. NOTE: 重启内核更新

   ```cs
   sudo apt-mark unhold linux-image-4.4.0-21-generic
   ```

   
