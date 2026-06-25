---
layout: default
title: Termux 所有主流玩法
parent: 🤖 AI 实战笔记      # 极其关键！名字必须和上面的顶级目录 title 完全一致
nav_order: 4                # 在子目录里的排序
---

下面我把 **Termux 所有主流玩法**，从入门→进阶→极客→黑科技，一次性全给你列满（尽量不重复、覆盖最广），你可以直接按分类挑着玩。

---

## 📱 一、基础环境 & 美化（必玩）
1. **更新源 + 换国内源（更快）**
   ```bash
   pkg update && pkg upgrade
   ```
2. **改终端样式（zsh + oh-my-zsh + 配色）**
   - bash/zsh/fish 切换
   - oh-my-zsh、powerlevel10k 主题
   - 终端配色、透明、字体美化
3. **文件管理**
   - `nano` / `vim` 编辑
   - `ranger` 命令行文件管理器（超好用）
   - 访问 `/sdcard` 手机存储
4. **常用 Linux 命令练习**
   - ls、cd、grep、find、tar、rsync、chmod、ssh
   - 写 shell 脚本（.sh）练自动化

---

## 🧑‍💻 二、手机写代码/开发（生产力）
### 1）Python 全栈
```bash
pkg install python
pip install numpy pandas requests jupyterlab
jupyter-lab --ip=0.0.0.0
```
- 写爬虫、数据分析、脚本
- Jupyter 笔记本（浏览器打开）
- 运行 Python 小游戏、AI 模型（轻量）

### 2）Node.js / Web
```bash
pkg install nodejs
npm install -g http-server vue-cli
```
- 写前端、Vue/React
- 手机起 Web 服务器、静态站

### 3）C/C++/编译
```bash
pkg install g++ gcc clang make
```
- 编译小程序、练算法
- 写 C 语言、数据结构

### 4）Git + GitHub
```bash
pkg install git
```
- 手机直接 push/pull、管理仓库
- 写 README、练 Git 工作流

### 5）Java / Kotlin（简单）
```bash
pkg install openjdk
```
- 编译运行 Java 代码

---

## 🌐 三、网络 & 服务器（把手机当服务器）
1. **SSH 远程连接手机（电脑控手机）**
   ```bash
   pkg install openssh
   sshd
   ```
   - 电脑 `ssh -p 8022 用户名@手机IP`

2. **手机当 Web 服务器**
   ```bash
   pkg install nginx lighttpd
   ```
   - 局域网访问手机网页/文件

3. **内网穿透（外网访问手机服务）**
   - frp、ngrok
   - 把手机服务暴露到公网

4. **网络工具/抓包**
   ```bash
   pkg install nmap tcpdump wireshark
   ```
   - 扫描局域网、抓包分析

5. **代理 / 翻墙客户端**
   - v2ray、clash、trojan（Termux 版）
   - 手机做代理服务器给其他设备用

---

## 🤖 四、手机自动化 + Termux:API（超级强）
先装：`pkg install termux-api` + 安卓端同名 APP

1. **读取短信、通知、剪贴板**
   - `termux-sms-list`
   - `termux-notification`
   - `termux-clipboard-get`

2. **调用摄像头、拍照、录像**
   ```bash
   termux-camera-photo /sdcard/photo.jpg
   ```

3. **获取 GPS 定位、传感器**
   ```bash
   termux-location
   ```

4. **控制手机：震动、亮度、音量、屏幕**
   - `termux-vibrate`
   - `termux-brightness`

5. **定时任务（cron）**
   ```bash
   pkg install cronie
   ```
   - 定时备份、定时发通知、定时上传

6. **脚本自动化举例**
   - 监控短信验证码自动复制
   - 电量低于 20% 发通知+震动
   - 每天自动备份相册到网盘

---

## 🐧 五、运行完整 Linux（Ubuntu/Debian，不用 root）
### proot-distro（最稳）
```bash
pkg install proot proot-distro
proot-distro list
proot-distro install ubuntu
proot-distro login ubuntu
```
- 得到完整 Ubuntu（root 权限、apt、所有 Linux 软件）
- 装桌面、浏览器、QQ、微信（Linux 版）
- 运行 Docker（部分机型）
- 做渗透、做服务器、做开发环境

### 图形桌面（VNC）
- Ubuntu 里装 xfce4 + tightvnc
- 手机/电脑 VNC 连手机桌面

---

## 🎮 六、玩游戏（终端/复古/模拟器）
1. **终端小游戏**
   - `nethack`、`cmatrix`、`hollywood`、`sl`（小火车）
   - `neofetch` 系统信息装逼
2. **复古游戏模拟器**
   - `pkg install retroarch`（FC/SFC/MD/GBA）
3. **跑 Minecraft（Linux 版）**
4. **玩 2048、贪吃蛇、俄罗斯方块（命令行版）**

---

## 🔐 七、安全/渗透/黑客向（合法练手）
1. **信息收集**
   - nmap、whois、dig、curl
2. **弱口令/爆破（仅自己靶机）**
   - hydra、patator
3. **SQL 注入（本地靶场）**
   - sqlmap
4. **密码工具**
   - hashcat、john
5. **漏洞扫描**
   - nikto、openvas（轻量）
6. **VPN/代理/流量分析**
   - proxychains、tcpdump

> 注意：**仅用于自己设备/授权靶场**，违法必究。

---

## ☁️ 八、私有云 / NAS / 网盘
1. **Nextcloud（手机私有云）**
   - proot 里装 Nextcloud
   - 手机当个人网盘、同步相册
2. **samba 共享（局域网文件）**
   - 电脑访问手机文件
3. **rsync 自动备份**
   - 备份手机数据到电脑/服务器

---

## 🧠 九、AI / 大模型 / 机器学习（轻量）
1. **本地跑 LLM（小模型）**
   - llama.cpp、ollama
   - 手机跑 7B 模型（中低端也能跑）
2. **Python AI 库**
   - transformers、torch（CPU）
   - 文字生成、翻译、图片识别

---

## 🛠️ 十、系统/硬件级玩法（无 root）
1. **模拟 root（proot/chroot）**
   ```bash
   termux-chroot
   ```
2. **CPU 监控、温控、性能**
   - htop、sensors
3. **USB 外设（部分机型）**
   - 接键盘、鼠标、U盘
4. **挂载手机分区（有限）**

---

## ✨ 十一、装逼 / 美化 / 有趣小工具
1. **终端特效**
   - cmatrix（黑客帝国）
   - hollywood（好莱坞特效）
   - neofetch、screenfetch
2. **ASCII 艺术**
   - figlet、toilet、cowsay
3. **音乐/播放器**
   - mpv、cmus（命令行音乐）
4. **视频播放（命令行）**
   - mpv

---

## 📦 十二、其他实用玩法
- 手机定时关机/重启（配合 API）
- 手机当闹钟、番茄钟
- 命令行 RSS 阅读器
- 命令行邮件客户端
- 手机搭建 IRC、Mumble、TeamSpeak 服务器
- 运行 BitTorrent 客户端（transmission）

---

## ✅ 快速入门建议（新手直接照做）
1. 先更新：`pkg update && pkg upgrade`
2. 装常用工具：`pkg install git vim curl wget htop`
3. 装 API：`pkg install termux-api` + 安卓 APP
4. 玩 proot Ubuntu：`pkg install proot proot-distro`
