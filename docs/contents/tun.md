# TUN 模式

对于不遵循系统代理的软件，TUN 模式可以接管其流量并交由 CFW 处理，在 Windows 中，TUN 模式性能比 TAP 模式好

## Windows

启动 TUN 模式需要进行如下操作：

1. 点击`General`中`Service Mode`右边`Manage`，在打开窗口中安装服务模式，安装完成应用会自动重启，Service Mode 右边地球图标变为`绿色`即安装成功（无法安装参考：[这里](./questions.md#service-mode-无法安装-windows)）
2. 在使用的**配置文件**中加入如下内容：

```yaml
dns:
  enable: true
  enhanced-mode: fake-ip
  nameserver:
    - 8.8.8.8 # 真实请求DNS，可多设置几个
    - 114.114.114.114
# interface-name: WLAN # 出口网卡名称，或者使用下方的自动检测
tun:
  enable: true
  stack: gvisor # 使用 system 需要 Clash Premium 2021.05.08 及更高版本
  dns-hijack:
    - 198.18.0.2:53 # 请勿更改
  auto-route: true
  auto-detect-interface: true # 自动检测出口网卡
```

::: tip NOTICE
从`Clash Premium 2021.05.08`开始，使用`auto-*`代替`macOS-auto-*`，往后数个版本将暂时兼容旧字段名。此版本同时添加了`system stack`支持。[参考](https://github.com/Dreamacro/clash/releases/tag/premium)
:::

### 注意事项

当`enhanced-mode`设置为`fake-ip`时，会出现系统检测到网卡无法联网，微软系 APP 无法登陆使用等问题，可以通过添加`fake-ip-filter`解决：

```yaml
dns:
  enable: true
  enhanced-mode: fake-ip
  nameserver:
    - 114.114.114.114
  fake-ip-filter:
    - "dns.msftncsi.com"
    - "www.msftncsi.com"
    - "www.msftconnecttest.com"
```

::: tip
TUN 模式更推荐使用 fake-ip 模式
:::

## macOS

启动 TUN 模式需要进行如下操作：

1. 点击`General`中`Service Mode`右边`Manage`，在打开窗口中安装服务模式，安装完成应用会自动重启，Service Mode 右边地球图标变为`绿色`即安装成功
2. 在使用的**配置文件**中加入如下内容：

```yaml
dns:
  enable: true
  enhanced-mode: fake-ip
  nameserver:
    - 114.114.114.114 # 真实请求DNS，可多设置几个
# interface-name: en0 # 出口网卡名称，或者使用下方的自动检测
tun:
  enable: true
  stack: system # 或 gvisor
  dns-hijack: # DNS劫持设置为系统DNS
    - 114.114.114.114 # 可任意设置，但为了保证CFW关闭后能不影响联网，建议设置真实能访问的DNS服务器
  auto-route: true
  auto-detect-interface: true # 自动检测出口网卡
```

::: tip NOTICE
从`Clash Premium 2021.05.08`开始，使用`auto-*`代替`macOS-auto-*`，往后数个版本将暂时兼容旧字段名。[参考](https://github.com/Dreamacro/clash/releases/tag/premium)
:::

::: tip
dns-hijack 不可以劫持局域网地址的 DNS，如 192.168.0.0/16，请务必手动设置系统 DNS
:::

::: tip
若要将此 Mac 设置为代理网关，打开 IP 转发即可：

```
sudo sysctl -w net.inet.ip.forwarding=1
```

这种做法将在机器下次重启后失效，如果想要永久保存，编辑文件`/etc/sysctl.conf`，配置下面变量：

```
net.inet.ip.forwarding=1
net.inet6.ip6.forwarding=1
```

或者使用 LaunchDaemons 进行配置：

1. 新建 `network.forwarding.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>Network Forwarding</string>
    <key>UserName</key>
    <string>root</string>
    <key>GroupName</key>
    <string>wheel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/sbin/sysctl</string>
        <string>-w</string>
        <string>net.inet.ip.forwarding=1</string>
        <string>net.inet6.ip6.forwarding=1</string>
    </array>
    <key>KeepAlive</key>
    <false/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

2. 将文件添加进 `/Library/LaunchDaemons`
3. `sudo launchctl load /Library/LaunchDaemons/network.forwarding.plist`
   :::

## Linux

启动 TUN 模式需要进行如下操作：

1. 点击`General`中`Service Mode`右边`Manage`，在打开窗口中安装服务模式，安装完成应用会自动重启（某些系统需要手动重启 APP），Service Mode 右边地球图标变为`绿色`即安装成功
2. 在使用的**配置文件**中加入如下内容：

```yaml
dns:
  enable: true
  enhanced-mode: fake-ip
  nameserver:
    - 114.114.114.114 # 真实请求DNS，可多设置几个
tun:
  enable: true
  stack: system # 或 gvisor
  dns-hijack:
    - 1.0.0.1:53 # 请勿更改
```

::: tip
Linux 下一般不需要设置`interface-name`字段
:::

::: tip
Service Mode 安装脚本使用 [Kr328/clash-premium-installer](https://github.com/Kr328/clash-premium-installer)
:::

## 配置文件参考

[Clash Wiki](https://github.com/Dreamacro/clash/wiki/Premium-Core-Features)

## 技巧

::: tip
你可以使用 [mixin](/contents/mixin.md) 将上面内容合并至所有配置文件中，并使用 Mixin 开关 TUN 模式，以 windows 配置为例：
```yaml
# windows 的 tun 示例配置
mixin:
  dns:
    enable: true
    enhanced-mode: fake-ip
    nameserver:
      - 8.8.8.8 # 真实请求DNS，可多设置几个
      - 114.114.114.114
  # interface-name: WLAN # 出口网卡名称，或者使用下方的自动检测
  tun:
    enable: true
    stack: gvisor # 使用 system 需要 Clash Premium 2021.05.08 及更高版本
    dns-hijack:
      - 198.18.0.2:53 # 请勿更改
    auto-route: true
    auto-detect-interface: true # 自动检测出口网卡
```
:::
