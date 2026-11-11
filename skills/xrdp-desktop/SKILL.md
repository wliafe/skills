---
name: xrdp-desktop
description: 在已配好 xrdp 和 XFCE 的 Linux 机器上管理「需要时才开」的远程桌面：默认不开机自启，用前手动开并清掉上次残留会话，空闲超过设定时间自动关。用户说要开桌面、关桌面、看远程桌面状态、连上就断、闪退，或要把远程桌面改成按需开启时就用。
compatibility: 目标机器已装 xrdp、xorgxrdp、xfce4，并能 sudo。本机能 SSH 登录该机。
---

# 远程桌面（xrdp）

把已经装好的 xrdp 改成「需要时才开」。平时不监听，用前手动开，空闲超过设定时间自动关。

这个技能不记录机器地址、端口、用户名或密码。连哪台机、端口号、空闲多久，以当前对话和机器上的实际配置为准。

## 先确认机器

改之前先看这台机是不是已经有桌面：

```sh
systemctl is-enabled xrdp
systemctl is-active xrdp
dpkg -l xrdp xorgxrdp xfce4 | awk '/^ii/{print $2,$3}'
grep -n '^port=' /etc/xrdp/xrdp.ini | head -1
```

缺 xrdp 或 xfce4 就停下，告诉用户这台机还没装桌面，不要自己装。端口以 `xrdp.ini` 第一个 `port=` 为准，下面命令里的 `33890` 都换成实际端口。

## 文件放哪

脚本放在登录用户的家目录，不放 `/tmp`（重启清空），也不放 `/usr/local/bin`（系统更新会碰）：

```
~/.local/bin/xrdp-ctl
~/.local/bin/xrdp-idle-watcher
```

把 `~/.local/bin` 加进 `~/.bashrc` 的 `PATH`，之后登录就能直接打 `xrdp-ctl`。改之前先看 `~/.bashrc` 里有没有这一行，有就不要重复加。

## xrdp-ctl

三个动作：`start`、`stop`、`status`。

`start` 要先清掉上次没退掉的会话再启动。xrdp 每次重连都分配新的显示号，旧会话进程还在时，新会话的窗口管理器会立刻退出，表现就是连上就断。

```sh
#!/bin/bash
action="${1:-status}"
PORT=$(awk -F= '/^port=/{print $2; exit}' /etc/xrdp/xrdp.ini)

start_xrdp() {
    sudo pkill -9 -f "xfce4-session|xfwm4|xfdesktop|xfce4-panel|Xorg :" 2>/dev/null || true
    sleep 2
    sudo systemctl start xrdp
    sleep 1
    systemctl is-active --quiet xrdp && ss -lnt | grep ":$PORT" || return 1
}

stop_xrdp() {
    sudo systemctl stop xrdp
    sleep 1
    systemctl is-active --quiet xrdp && return 1
}

status_xrdp() {
    echo "Enabled: $(systemctl is-enabled xrdp 2>&1)"
    echo "Active: $(systemctl is-active xrdp 2>&1)"
    if systemctl is-active --quiet xrdp; then
        ss -lnt | grep ":$PORT" || echo "port $PORT not listening"
        ss -tn state established "( sport = :$PORT or dport = :$PORT )" | tail -n +2
    fi
}

case "$action" in
    start) start_xrdp ;;
    stop) stop_xrdp ;;
    status) status_xrdp ;;
    *) echo "Usage: $0 {start|stop|status}"; exit 1 ;;
esac
```

## 空闲自动关

监控脚本每 5 分钟看一次端口上有没有已建立的连接。没有连接持续超过设定分钟数就停掉 xrdp。分钟数用用户说的，没说就用 30。

```sh
#!/bin/bash
IDLE_MINUTES=30
IDLE_SECONDS=$((IDLE_MINUTES * 60))
PORT=$(awk -F= '/^port=/{print $2; exit}' /etc/xrdp/xrdp.ini)
last_active=$(date +%s)

while true; do
    sleep 300
    systemctl is-active --quiet xrdp || continue
    count=$(ss -tn state established "( sport = :$PORT or dport = :$PORT )" | tail -n +2 | wc -l)
    now=$(date +%s)
    if [ "$count" -gt 0 ]; then
        last_active=$now
    elif [ $((now - last_active)) -ge $IDLE_SECONDS ]; then
        systemctl stop xrdp
        last_active=$now
    fi
done
```

用 systemd 让它开机自启。`ExecStart` 写绝对路径，不要写 `~`，也不要写 `/tmp`。`User` 用实际登录用户：

```ini
[Unit]
Description=XRDP idle watcher
After=network.target

[Service]
Type=simple
ExecStart=/home/<user>/.local/bin/xrdp-idle-watcher
Restart=always
RestartSec=10
User=<user>

[Install]
WantedBy=multi-user.target
```

写完跑：

```sh
sudo systemctl daemon-reload
sudo systemctl enable --now xrdp-idle-watcher
```

## 开机行为

```sh
sudo systemctl disable --now xrdp
```

xrdp 不开机自启，监控服务开机自启。监控服务只在 xrdp 跑着的时候才会停它，平时不占内存。

## 改完要验证

跑一遍 `xrdp-ctl start`，确认端口在监听；再跑 `xrdp-ctl stop`，确认没有监听。然后 `systemctl is-active xrdp-idle-watcher` 应该是 `active`，`systemctl show -p ExecStart` 应该指向家目录下的脚本。

## 连不上时先看日志

不要靠猜。两份日志对得上才能下结论：

```sh
sudo tail -30 /var/log/xrdp.log
sudo tail -30 /var/log/xrdp-sesman.log
```

常见的几种：

- `login failed` 加 `pam_authenticate failed`：密码验不过。用 `passwd -S <用户>` 看状态，`L` 是锁住的，`/etc/shadow` 哈希以 `!` 开头也是锁住的。换机器后密码不会跟过去，需要重新设。
- `exited quickly` 加显示号一次比一次大：旧会话没退。跑 `xrdp-ctl start`，它会先清残留。
- 端口没有监听：服务没开。跑 `xrdp-ctl start`。

## 告诉用户怎么用

改完之后告诉用户三件事：开桌面前先跑 `xrdp-ctl start`，空闲多久会自动关，连接地址和端口是什么。手机上 iOS 用 Windows App，安卓用 Microsoft Remote Desktop 或 aFreeRDP，分辨率建议 1280×720。手机切换网络会断开并留下残留会话，下次连不上就再跑一次 `xrdp-ctl start`。
