![433d6b08b630f051631c0188bbd5a3f9](https://github.com/user-attachments/assets/8d910da6-b354-41c4-a492-66dfa8f16176)# Orangpi-Desktop
Change XFCE to GNOME  

将香橙派官方提供的系统转换成熟悉的原生Ubuntu系统风格

在使用香橙派5 ultra 过程中，使用不习惯香橙派官方提供的XFCE桌面
Ubuntu 22.04及以上都有很好的系统适配，如项目 https://github.com/Joshua-Riek/ubuntu-rockchip

但是大多为rk3588s的芯片开发板或常用型号的开发板（如我用的香橙派5 ultra项目上就没有提供原生Ubuntu系统，但是实测刷写Joshua-Riek提供的香橙派5 Plus的系统可以运行，正常使用，但是如风扇接口等小地方可能有问题），同时当前大多适配的都是Ubuntu22.04及以上系统

下面这个方法目标是实现：使用Orangepi官方提供的系统时，无损的从XFCE转换成我们熟悉的GNOME桌面

已在香橙派5 Ultra上实现官方Ubuntu20.04系统转换，理论上可以适配大部分香橙派开发板

------------------------------------------------
一、已经安装了XFCE，现在「额外」装上GNOME  
（登录界面里保留XFCE，也能随时切回）

1. 更新软件源  
   sudo apt update

2. 安装GNOME核心会话+常用应用  
### 最小方案（约400 MB）  
   sudo apt install gnome-session gdm3 gnome-terminal  

### 完整方案（约1.3 GB，含LibreOffice、文本编辑、截图等）  
   sudo apt install ubuntu-desktop  

   安装过程中会弹出「选择显示管理器」蓝框，选gdm3（GNOME官方DM）即可；若打算继续用lightdm也可，但gdm3对GNOME锁屏/休眠更友好。

3. 装完重启  
   sudo reboot

4. 在登录界面  
   - 点击用户名 → 右下角齿轮 → 选「Ubuntu」或「Ubuntu on Wayland」 → 输入密码登录即进入GNOME。  
   - 想回XFCE，同一齿轮里选「Xfce Session」即可。

------------------------------------------------
二、只想「彻底」换成GNOME，把XFCE相关包全部删掉  
（节省空间，登录界面只剩GNOME）

1. 先按上面步骤1~3装好GNOME并重启一次，确认能正常进入GNOME。

2. 卸载XFCE  
   sudo apt remove --auto-remove --purge xfce4\* xfwm4\* xfdesktop4\* xfce4-session\* thunar\*  
   sudo apt autoremove

3. 如果之前用的是lightdm，可再执行  
   sudo apt remove --purge lightdm  
   sudo dpkg-reconfigure gdm3    # 确保gdm3为默认

4. 重启即可。
   
------------------------------------------------
三、问题及解决办法
运行 sudo apt install ubuntu-desktop 出现如下报错：

//

orangepi@orangepi5ultra:~$ sudo apt install ubuntu-desktop
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 ubuntu-desktop : Depends: ubuntu-desktop-minimal but it is not going to be installed
                  Depends: ubuntu-session but it is not going to be installed
E: Unable to correct problems, you have held broken packages.

//

### 1. 先更新索引并清理残留状态
sudo apt update
sudo apt --fix-broken install   # 自动把中断的包补完
sudo dpkg --configure -a        # 把半配置的包走完
sudo apt autoremove

### 2. 单独把被 “hold” 住的包放开
sudo apt-mark showhold          # 查看哪些包被人工锁定

如果有输出，例如 ubuntu-desktop-minimal

sudo apt-mark unhold ubuntu-desktop-minimal ubuntu-session

### 3. 尝试一次性补装缺失的依赖
sudo apt install ubuntu-desktop-minimal ubuntu-session
- 若这一步仍提示 **“but it is not going to be installed”**，说明依赖版本冲突，继续第 4 步。  
- 若直接成功，再执行 `sudo apt install ubuntu-desktop` 即可。

### 4. 用 aptitude 给出降级/替换方案（20.04 验证最有效）
sudo apt install aptitude
sudo aptitude install ubuntu-desktop
-aptitude 会首先给出一个 **保持现状的方案（reject）**，  
-输入 `n` 拒绝后，它会再给出 **降级/删除/替换方案（often 2~3 个）**，  
-通常第二个方案就能解决依赖冲突，**接受（y）** 即可自动完成安装。

### 5. 若 aptitude 也报 “no version available”
说明你的源里确实缺少某些过渡包，把 **focal-updates** 与 **focal-security** 源加回来再更新一次：
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo tee -a /etc/apt/sources.list <<'EOF'
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
EOF
sudo apt update
sudo aptitude install ubuntu-desktop   # 重复 4 的交互步骤

### 6. 装完验证
sudo reboot
在 gdm3 登录界面 → 齿轮 → 选 Ubuntu / Ubuntu on Wayland 即进入 GNOME
  
祝安装顺利！
![1ffe9196ee9def41a615bc0c1c6f534a](https://github.com/user-attachments/assets/7ef126f3-0e1c-4f74-9485-dd88989134e1)




