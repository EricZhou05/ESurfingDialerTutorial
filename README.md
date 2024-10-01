# 广东电信天翼校园网自动认证教程
---

![GitHub License](https://img.shields.io/github/license/EricZhou05/ESurfingDialerTutorial)

借助第三方 [广东电信天翼校园（ZSM验证）登入认证客户端](https://github.com/Rsplwe/ESurfingDialer)，实现自动认证联网的效果。配合软路由 Docker 部署可实现多设备共享。脚本作者教程比较简略难懂，自己一步步摸索过来的，提供给后面的学弟学妹，希望能让大家少走一些弯路。

## 运行环境

- **Java 21 及以上**
- **x86_64** 或 **ARMv8** 架构
- **glibc** (仅限 Linux)
- **内存 ≥ 200M**

## 使用教程
---

### Windows 部署

#### 1. 安装 Java 21 并配置环境

> [Java官网下载 JDK21 版本详细教程（下载、安装、环境变量配置）](https://blog.csdn.net/Du_XiaoNan/article/details/137373260)

#### 2. 运行程序

1. 下载发行版并重命名为 `client.jar`。

   > [ESurfingDialer 发行版](https://github.com/Rsplwe/ESurfingDialer/releases)

2. 新建记事本，填入以下代码，保存后重命名为“校园网自动登陆.bat”。

   > **注意**：将 `<用户名/手机号>` 和 `<密码>` 替换为你自己的！

   ```bash
   @echo off
   cd D:\Log in
   java -jar client.jar -u <用户名/手机号> -p <密码>
   pause
   ```

3. 在 D 盘新建 Log in 文件夹，并将 `client.jar` 和 `校园网自动登陆.bat` 放入该文件夹中。

4. 关闭所有杀毒软件或将脚本设置为白名单，双击 `校园网自动登陆.bat` 运行。

5. （进阶）通过计划任务设置开机自启动。

   > [Windows 任务计划程序（task scheduler）介绍](https://blog.csdn.net/glenshappy/article/details/128567122)

---

### OpenWrt 部署（支持多设备共享）

#### 1. 购置软路由

**注意：**
- 仅支持 **x86_64** 或 **ARMv8** 架构的 CPU（因Docker 环境要求）
- 内存 **≥ 200M**（因Docker 环境要求）
- 双网口（用于多设备共享）

**推荐：**
我使用的是 J1800 小主机，在二手市场上可以低价购买（百元）。在购买时要询问卖家是否为双千兆网口，以便发挥宽带的全部性能。最好配有 12V DC 供电接口，方便插座供电。

#### 2. 刷入 OpenWrt 系统

推荐使用 **iStoreOS**，这是一款适合新手的 OpenWrt 软路由系统，**自带 Docker 环境**，UI 美观。  
下载适配设备的镜像文件：[iStoreOS 下载](https://fw.koolcenter.com/iStoreOS)

刷入教程详见此视频的 4:10 - 7:40 分钟：[刷入教程视频](https://www.bilibili.com/video/BV13p4y1u78R)

> **提示**：网口地址修改部分较为复杂难懂，建议多查看视频和资料，谨慎操作。

#### 3. 部署 Docker 镜像

进入 OpenWrt 系统后，即可部署 Docker 镜像并创建容器。

##### 方法一：使用现成的 Docker 镜像

1. 下载我打包好的镜像文件（版本：ESurfingDialer-1.7.0-all）：[dialer.tar 下载链接](https://www.jjekjfjke.com/dialer.tar)

2. 在OpenWrt系统主页点击“文件管理”，进入 `/tmp` 目录，上传下载好的 `dialer.tar`。

3. 返回主页，点击“服务” -> “终端”，输入账号 `root` 和密码，连接到终端。

4. 在终端中粘贴以下命令，加载 Docker 镜像：

   ```bash
   docker load -i /tmp/dialer.tar
   ```

5. 镜像创建完成后，粘贴以下命令运行容器：

   ```bash
   docker run -itd -e DIALER_USER=<用户名/手机号> -e DIALER_PASSWORD=<密码> --name dialer-client --network host --restart=always dialer
   ```

   > **注意**：将 `<用户名/手机号>` 和 `<密码>` 替换为你的信息。

6. 输入以下命令查看容器日志，检查运行状态：

   ```bash
   docker logs -f dialer-client
   ```

   如果输出以下信息，则表示部署成功：

   ```
   INFO [com.rsplwe.esurfing.Client] (Client:82) - The login has been authorized.
   ```

##### 方法二：自行打包 Docker 镜像

1. 在电脑上搭建 Docker 环境，运行 Docker Desktop 应用。

2. 参考此视频的 4:30 - 6:15 分钟：[打包镜像视频](https://www.bilibili.com/video/BV1ai421S7zj)

3. 下载原脚本并重命名为 `client.jar`：[原脚本下载链接](https://github.com/Rsplwe/ESurfingDialer/releases)

4. 创建 Dockerfile 文件，输入以下内容：

   ```dockerfile
   FROM openjdk:21
   WORKDIR /app
   COPY run.sh /app
   COPY client.jar /app
   RUN chmod +x /app/run.sh
   CMD ["./run.sh"]
   ```
   > **注意**：Windows 和 Linux 的换行符不同，可能会导致脚本错误，详细请查看 [换行符问题解决方案](https://blog.csdn.net/hyj_king/article/details/120301359)。

5. 创建 `run.sh` 文件，输入以下内容：

   ```bash
   #!/bin/sh
   java -jar client.jar -u ${DIALER_USER} -p ${DIALER_PASSWORD}
   ```
   > **注意**：Windows 和 Linux 的换行符不同，可能会导致脚本错误，详细请查看 [换行符问题解决方案](https://blog.csdn.net/hyj_king/article/details/120301359)。


6. 将 `Dockerfile`、`run.sh` 和 `client.jar` 放入同一个没有中文路径的文件夹中。

7. 在文件夹内按住 Shift 键并右键选择“在此处打开 PowerShell 窗口”，粘贴以下命令构建 Docker 容器：

   ```bash
   docker build -t dialer .
   ```

8. 构建完成后，粘贴以下命令导出镜像：

   ```bash
   docker save -o dialer.tar dialer
   ```

9. 上传 `dialer.tar` 文件至软路由 `/tmp` 目录。

10. 在软路由终端中粘贴以下命令加载镜像：

    ```bash
    docker load -i /tmp/dialer.tar
    ```

11. 最后粘贴以下命令运行容器：

    ```bash
    docker run -itd -e DIALER_USER=<用户名/手机号> -e DIALER_PASSWORD=<密码> --name dialer-client --network host --restart=always dialer
    ```

    > **注意**：将 `<用户名/手机号>` 和 `<密码>` 替换为你的信息。

12. 使用以下命令查看容器日志，检查运行状态：

    ```bash
    docker logs -f dialer-client
    ```

    如果输出以下信息，则表示部署成功：

    ```
    INFO [com.rsplwe.esurfing.Client] (Client:82) - The login has been authorized.
    ```
#### 4. 部署Wifi
购买一个千兆普通路由器，将wan口连接软路由的lan口，然后将上网方式改为DHCP，就可以开Wifi多设备使用啦！
