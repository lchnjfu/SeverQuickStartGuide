# 🚀 团队GPU服务器快速入门指南

欢迎加入团队！为了帮助你快速、高效、和谐地使用我们强大的GPU服务器，请仔细阅读本指南。

本文档的目标是：
- 让你在5分钟内完成首次登录和环境配置。
- 掌握日常开发所需的核心命令。
- 理解服务器使用的基本规则，避免资源冲突。

---

## 目录
1. [第一步：账户与登录](#-第一步账户与登录)
2. [📜 **核心使用准则 (必须遵守)**](#-核心使用准则-必须遵守)
3. [💻 环境管理：使用Conda](#-环境管理使用conda)
4. [💡 日常核心命令](#-日常核心命令)
    - [查看GPU状态](#查看gpu状态)
    - [运行你的代码](#运行你的代码)
    - [后台长期运行任务](#后台长期运行任务)
5. [📂 文件传输](#-文件传输)
6. [❓ 遇到问题怎么办？](#-遇到问题怎么办)
7. [附录：服务器管理办法详情](#附录服务器管理办法详情)

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

## 📜 **核心使用准则 (必须遵守)**

为了保证大家都能高效地使用服务器，请务必遵守以下黄金准则：

1.  **先看后用 (Check First)**: 在运行任何代码前，必须使用 `nvidia-smi` 命令查看GPU状态，选择一个空闲的GPU。
2.  **明确指定 (Be Specific)**: **严禁**让程序自动选择GPU。你**必须**使用 `CUDA_VISIBLE_DEVICES` 环境变量来指定你要使用的GPU。
3.  **环境隔离 (Isolate Your Environment)**: **严禁**在系统的全局Python环境中安装包。**必须**使用 `Conda` 或 `venv` 创建独立的虚拟环境。
4.  **及时清理 (Clean Up)**: 调试或短时间占用GPU后，请确保你的进程已结束，释放GPU资源。不要长时间占用GPU而不进行计算。

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

## ❓ 遇到问题怎么办？

1.  **先自查**: 仔细阅读错误信息，尝试在Google或Stack Overflow上搜索。90%的问题都可以通过这种方式解决。
2.  **检查环境**: 确认你是否激活了正确的Conda环境？`which python` 和 `which pip` 指向的是否是你的环境内的路径？
3.  **向管理员求助**: 如果你确信不是自己代码或环境的问题，请带着以下信息联系管理员 `[你的名字]`：
    - **你的目标**: 你本来想做什么。
    - **你的操作**: 你执行了哪些命令。
    - **错误信息**: 完整的错误日志或截图。

---

## 鼓励大家为本指南贡献内容

* 请发送md文件给管理员，统一添加进来。*

## 附录：服务器管理办法详情

*请联系管理员了解《团队GPU服务器管理及使用办法》内容。*
