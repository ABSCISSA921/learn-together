## 1.构建与初始化
这些指令用于从零配置环境，或者在修改了底层配置文件（Dockerfile 或 docker-compose.yml）后，让变更生效。必须在包含配置文件的工程目录（~/common-ros-noetic-docker）下执行

编译构建镜像：

Bash

`docker compose build`

(仅在使用场景：当修改了 Dockerfile，例如需要通过 apt 安装新的系统级依赖包时，才需要执行此步骤重新打包镜像。)

按图纸创建并运行容器：

Bash

`docker compose up -d`

(使用场景：初次部署项目，或者修改了 docker-compose.yml（如增删挂载目录、修改端口映射、添加独显穿透配置）后，执行此命令。Docker 会自动拉取镜像、分配网络和显卡，并在后台（-d）启动容器。)

## 2.日常启动（常规开机）
当的容器已经通过 up -d 创建好，且配置没有变化，只是因为电脑重启或主动执行了 stop 而处于 Exited（已关机）状态时，使用此命令。可以在宿主机的任何目录下执行。

启动现有容器：

Bash

`docker start ros1_noetic_dev (alisa dsr)`

(执行速度极快，因为它不需要重新解析配置，只是唤醒现有的进程。这是你每天来到实验室后敲的第一个命令。)

## 3.进入容器内部
容器运行在后台后，需要“走进去”才能执行 ROS 指令或写代码。

方法 1：命令行进入（开通终端）

Bashsudo chown -R $USER:$USER .

`docker exec -it ros1_noetic_dev bash (alisa der)`

(说明：-it 参数表示开启一个交互式的终端。你可以开多个宿主机终端，在每个里面都执行这条命令，从而在容器里获得多个独立的操作窗口，用于分别启动 roscore、catkin build 或 roslaunch。)

方法 2：IDE 可视化进入（日常主力开发模式）
操作： 宿主机打开 VS Code -> 点击左下角绿色/蓝色 >< 图标 -> 选择 "Attach to Running Container..." -> 选择 ros1_noetic_dev。

## 4.退出容器终端（人离开，容器继续运行）
当你通过 docker exec -it ros1_noetic_dev bash 进入容器后，如果你只是想回到宿主机（Ubuntu）的终端，而不影响容器里正在跑的 ROS 节点或后台程序，你可以使用以下两种方法：

指令法： 直接在容器的命令行输入：

Bash

`exit`

快捷键法（最常用）： 在容器终端里直接按下键盘的：

Plaintext

Ctrl + D

💡 说明： 你的容器在 docker-compose.yml 里配置了 command: tail -f /dev/null，这意味着它是一个长驻后台的服务。你退出 bash 终端，仅仅是关闭了你和容器通讯的“一扇门”，容器本身依然在后台活得好好的。

## 5.停止容器进程（给容器关机）
如果今天的工作结束了，想把容器关掉以释放宿主机的内存和 CPU，你必须在宿主机的终端执行停止指令。

方法 1：使用 Docker 原生命令（在任何目录都能执行）

Bash

`docker stop ros1_noetic_dev`

(执行后会卡顿 1~2 秒，这是 Docker 在给容器发送优雅关机的信号，随后输出容器名即代表停止成功。)

方法 2：使用 Compose 命令（必须在工程目录下执行）

Bash

`docker compose stop`

## 6.彻底销毁容器（拆除系统）
如果你修改了 docker-compose.yml，或者容器环境被你玩坏了想要重置，你需要销毁它。别担心，你的代码和 VS Code 插件都在挂载卷里，不会丢失。

必须在宿主机的工程目录（~/common-ros-noetic-docker）下执行：

Bash

`docker compose down`

(这条指令不仅会停止容器，还会把它从硬盘上删掉。下次你需要用 docker compose up -d 重新建一个。)

## 7.辅助查询指令（时刻掌握容器状态）
在宿主机终端执行，用来查看容器到底处于什么状态：

查看当前正在运行的容器：

Bash

`docker ps`

查看所有容器（包括已经关机停止的）：

Bash

`docker ps -a`

(可以通过这一条指令的 STATUS 字段，清楚地看到 ROS 容器是 Up（运行中）还是 Exited（已关机）)

## 给docker独显驱动

### 添加NVIDIA Container Toolkit仓库
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

### 安装NVIDIA Container Toolkit
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit

### 配置Docker使用NVIDIA运行时
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

## 队内docker配置仓库

https://github.com/HydrogenZp/common-ros-noetic-docker 预装常用调车工具以及依赖，构建后可顺利编译队内代码的Dockerfile和docker-compose.yml文件 https://github.com/users/HydrogenZp/packages/container/package/common-ros-noetic-docker 对应的镜像，每当那个仓库有commit都会跑一次action把镜像提交到这个github package

## 其他
### 释放文件所有权
bash
`sudo chown -R $USER:$USER .`