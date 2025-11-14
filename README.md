# 🚀 GPU服务器快速入门指南

欢迎加入团队！为了帮助你快速、高效、和谐地使用我们强大的GPU服务器，请仔细阅读本指南。

本文档的目标是：
- 让你在5分钟内完成首次登录和环境配置。
- 掌握日常开发所需的核心命令。
- 理解服务器使用的基本规则，避免资源冲突。

---

## 目录
1. [第一步：账户与登录](#-第一步账户与登录)
2. [💻 环境管理：使用Conda](#-环境管理使用conda)
3. [💡 日常核心命令](#-日常核心命令)
    - [查看GPU状态](#查看gpu状态)
    - [运行你的代码](#运行你的代码)
    - [后台长期运行任务](#后台长期运行任务)
4. [📂 文件传输](#-文件传输)

---

## 👋 第一步：账户与登录

### 1. 获取账户
联系管理员 `[你的名字]` 获取你的服务器用户名和初始密码。

### 2. 通过SSH登录
你需要一个SSH客户端。
- **Windows**: 可以使用 [MobaXterm](https.mobaxterm.mobatek.net/) 或 Windows Terminal/PowerShell 自带的`ssh`。
- **macOS / Linux**: 直接使用系统自带的终端 (Terminal)。

执行以下命令登录，将`username`换成你的用户名，`server_ip_or_hostname`换成服务器的IP地址或域名。

```bash
ssh username@server_ip_or_hostname
```
*服务器地址1: `[服务器1的IP或主机名]`*
*服务器地址2: `[服务器2的IP或主机名]`*

### 3. ❗️ 修改初始密码
首次登录后，为了你的账户安全，**必须立即修改密码**！

```bash
passwd
```
系统会提示你输入当前密码，然后输入两次新密码。

---

## 💻 环境管理：使用Conda

我们使用 [Miniconda](https://docs.conda.io/en/latest/miniconda.html) 来管理每个人的开发环境，以避免软件包依赖冲突。

#### Conda 基础工作流

```bash
# 1. 创建一个新的环境 (例如，名为 my_env，使用 Python 3.9)
# 你可以为你自己的每个项目创建一个独立的环境
conda create -n my_env python=3.9

# 2. 激活环境 (进入该环境)
conda activate my_env
# 激活后，你的命令行提示符前会出现 (my_env) 字样

# 3. 在环境中安装你需要的包 (例如 PyTorch)
# !! 确保你已在激活的环境中 !!
pip install torch torchvision torchaudio

# 4. 运行你的Python脚本
python your_script.py

# 5. 当你工作完成后，退出环境
conda deactivate

# --- 其他常用命令 ---

# 查看所有已创建的环境
conda env list

# 删除不再需要的环境
conda env remove -n my_env
```

---

## 💡 日常核心命令

### 查看GPU状态

这是你登录后最先要执行的命令。

```bash
# 查看当前GPU的实时状态
nvidia-smi

# 使用watch命令每秒自动刷新，方便监控
watch -n 1 nvidia-smi
```
**如何读懂 `nvidia-smi`:**
- `Fan`: 风扇转速
- `Temp`: GPU温度 (请注意，持续高于85°C可能有风险)
- `Memory-Usage`: **显存占用** (这是判断GPU是否空闲的关键指标)
- `GPU-Util`: **GPU利用率** (显示GPU核心的繁忙程度)

我们还提供了一个更友好的脚本，可以直接看到是谁在用GPU：
```bash
# [如果已将check_gpu.sh放在公共路径]
check_gpu.sh

# [或者在脚本所在目录执行]
./check_gpu.sh
```

### 运行你的代码

**关键：** 使用 `CUDA_VISIBLE_DEVICES` 来选择GPU。GPU的ID从`0`开始。

```bash
# 示例1：指定在 GPU 2 上运行你的脚本
CUDA_VISIBLE_DEVICES=2 python train.py

# 示例2：指定使用 GPU 0 和 GPU 3 (多卡训练)
CUDA_VISIBLE_DEVICES=0,3 python train_multi_gpu.py

# 错误的做法 (会默认占用所有卡或第一张空闲卡，造成冲突)
# ❗️ 不要这样做: python train.py
```

### 后台长期运行任务

对于需要运行数小时或数天的训练任务，你不能一直开着SSH连接。你需要使用 `nohup` 或 `tmux`/`screen`。

#### 方法一：`nohup` (简单直接)
`nohup` 可以让你的程序在退出终端后继续运行，并将输出重定向到文件。

```bash
# 命令格式: nohup [你的命令] > [输出文件名] 2>&1 &
nohup CUDA_VISIBLE_DEVICES=1 python train.py > training.log 2>&1 &
```
- `>` `training.log`: 将标准输出保存到 `training.log` 文件中。
- `2>&1`: 将错误输出也一并保存到该文件中。
- `&`: 让命令在后台执行。

执行后，你可以使用 `jobs -l` 查看后台任务，或 `ps aux | grep python` 查看进程。

#### 方法二：`tmux` (强烈推荐)
`tmux` 是一个终端复用工具，它能创建持久的会-话，即使你断开连接，会话和里面的程序也依然在运行。

```bash
# 1. 创建一个新会话 (例如，名为 train_session)
tmux new -s train_session

# 2. 在 tmux 会话中，正常运行你的程序 (不需要 &)
# 这个窗口就像你的本地终端一样
conda activate my_env
CUDA_VISIBLE_DEVICES=1 python train.py

# 3. 分离会话 (程序会继续在后台运行)
# 按下快捷键: Ctrl+b 然后再按 d (detach)

# --- 当你重新登录服务器后 ---

# 4. 查看所有正在运行的 tmux 会话
tmux ls

# 5. 重新连接到你的会话
tmux attach -t train_session
# 你会看到你的程序还在愉快地运行！
```

---

## 📂 文件传输

使用 `scp` 命令可以在你的本地电脑和服务器之间传输文件。

```bash
# 从本地上传文件到服务器
# scp [本地文件路径] username@server_ip:[服务器目标路径]
scp ./my_code.zip zhangsan@192.168.1.100:/home/zhangsan/projects/

# 从本地上传整个文件夹 (-r 表示递归)
scp -r ./my_project zhangsan@192.168.1.100:/home/zhangsan/projects/

# 从服务器下载文件到本地
# scp username@server_ip:[服务器文件路径] [本地目标路径]
scp zhangsan@192.168.1.100:/home/zhangsan/output.log ./
```

---
## cuda版本更换

目前系统默认cuda版本是12.8，如需要更换，请自行安装，并按照如下步骤操作：
1、在`～/.bashrc`末尾添加如下三行，并保存退出（建议用vim）：
```shell
export CUDA_HOME=/usr/local/cuda118
export PATH=$CUDA_HOME/bin:$PATH
export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH
```
2.更新`～/.bashrc`并检验：
```shell
source ~/.bashrc
nvcc -V          # 看是否能找到编译器（可选）
echo $CUDA_HOME  # 看路径是否设置正确
```
