# Mihomo-FPK

![License](https://img.shields.io/badge/license-MIT-blue.svg)

Mihomo 在飞牛 fnOS 系统上的应用打包版本，提供完整的代理客户端功能和网页管理界面。

## 简介

**Mihomo** 是一个功能强大的代理客户端，支持多种协议和灵活的规则引擎。本项目将 Mihomo 封装为飞牛 fnOS 应用包（.fpk），包含以下特性：

- 🚀 支持多种代理协议（HTTP、HTTPS、SOCKS5、Shadowsocks、VMess、VLESS、Trojan 等）
- 🎯 灵活的流量路由规则和分组策略
- 🌍 地理位置数据库支持（GeoIP、GeoSite、MMDB、ASN）
- 💻 现代化的网页管理界面（http://localhost:9090/ui/）
- 🔧 完整的配置系统和 API 接口
- 📱 TUN 模式支持系统代理
- 🔐 安全的 external-controller 管理接口

## 功能特性

### 核心功能
- **多协议支持** - 支持 HTTP、HTTPS、SOCKS5、Shadowsocks、VMess、VLESS、Trojan 等多种代理协议
- **智能路由** - 基于规则的灵活流量分流配置
- **代理分组** - 支持多个代理分组、负载均衡和自动故障转移
- **地域分流** - 使用 GeoIP/GeoSite 数据库进行国家级流量分流
- **DNS 处理** - 增强型 DNS 处理和 FakeIP 支持
- **流量嗅探** - SNI 嗅探识别 HTTPS、TLS、QUIC 等加密流量
- **TUN 模式** - 系统级代理支持，拦截所有流量

### 管理界面
### 地理数据库

注意：为了减小仓库体积，包含 GeoIP/GeoSite/MMDB/ASN 的数据库文件已从本仓库中移除。
应用在首次启动或按需会自动从官方或配置的镜像下载所需的地理数据库，并将其保存到应用运行目录下的服务器数据目录（通常为 /usr/local/apps/@appconf/mihomo/）。

示例（运行时会生成）：
```
/usr/local/apps/@appconf/mihomo/
├── geoip.dat                  # （运行时下载）IP 地址库
├── geosite.dat                # （运行时下载）域名库
├── country-lite.mmdb          # （运行时下载）MMDB 格式 IP 库
└── GeoLite2-ASN.mmdb          # （运行时下载）ASN 库
```

如果希望手动管理这些文件，可以把下载好的数据库放到上述目录；程序会在启动/运行时自动检测并加载它们。
- 正常的网络连接

### 安装步骤

1. **下载应用包**
   - 从应用中心下载 Mihomo-fpk，或
   - 从本仓库 Release 页面下载 `mihomo.fpk` 文件

2. **安装应用**
   - 在飞牛系统中上传并安装 .fpk 文件
   - 应用会自动初始化配置文件和目录结构

3. **首次配置**
   - 安装完成后会显示安装向导
   - 访问网页管理界面：http://localhost:9090/ui/

## 配置

### 配置文件位置
```
/usr/local/apps/@appconf/mihomo/config.yaml
```

### 快速编辑
```bash
nano /usr/local/apps/@appconf/mihomo/config.yaml
```

### 主要配置项

#### 1. 代理订阅配置
编辑 `proxy-providers` 部分，添加你的代理订阅链接：

```yaml
proxy-providers:
  provider1:
    url: "https://your-subscription-url-here"
    type: http
    interval: 86400  # 24小时更新一次
    health-check:
      enable: true
      url: "https://www.gstatic.com/generate_204"
      interval: 300   # 每5分钟健康检查一次
```

#### 2. 端口配置
```yaml
mixed-port: 7890           # 混合协议端口（支持 HTTP 和 SOCKS5）
external-controller: 0.0.0.0:9090  # 管理接口
```

#### 3. 代理分组
预配置了常用的地域分组：
- 香港、台湾、日本、新加坡、美国、其它地区
- 自动选择（基于延迟测试）
- 全部节点

#### 4. 流量规则
按配置顺序匹配：
- 国内 IP（直连）
- 主流国外服务（Google、YouTube、Netflix、Telegram 等）
- 其它国家/地区（代理）
- 默认（其它）

### 地理数据库

地理数据库文件将自动下载到以下目录：
```
/usr/local/apps/@appconf/mihomo/
├── geoip.dat                  # IP地址库
├── geosite.dat               # 域名库
├── country-lite.mmdb         # MMDB 格式 IP 库
└── GeoLite2-ASN.mmdb        # ASN 库
```

这些文件会在应用启动时自动加载。

## 使用

### 启动应用
```bash
# 应用安装后会自动启动
# 或手动启动
systemctl start mihomo
```

### 访问管理界面
打开浏览器访问：
```
http://localhost:9090/ui/
```

### 添加RESTful API访问口令
打开/usr/local/apps/@appconf/mihomo/config.yaml
在geodata-mode: true条目下添加secret: "1234"，引号内为你的访问口令

### 配置系统代理（可选）

**Linux 系统代理**
```bash
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
```

**Firefox 代理设置**
- 设置 → 网络设置 → 手动代理配置
- HTTP 代理：127.0.0.1，端口：7890
- HTTPS 代理：127.0.0.1，端口：7890

### 使用 TUN 模式（高级）
TUN 模式已在配置中启用，可以拦截系统所有流量：

```yaml
tun:
  enable: true
  stack: system
  device: tun
  mtu: 9000
  dns-hijack:
    - "any:53"
    - "tcp://any:53"
  auto-route: true
  auto-detect-interface: true
  strict-route: true
```

## 文件结构

```
mihomo/
├── cmd/                          # 应用生命周期脚本
│   ├── main                      # 主程序入口
│   ├── install_init             # 安装初始化脚本
│   ├── install_callback         # 安装回调脚本
│   ├── upgrade_callback         # 升级回调脚本
│   ├── config_callback          # 配置回调脚本
│   ├── uninstall_callback       # 卸载回调脚本
│   └── ...
├── app/
│   ├── server/                   # Mihomo 服务器文件
│   │   ├── mihomo               # Mihomo 可执行文件
│   └── ui/                       # 网页 UI 文件
│       ├── index.html
│       ├── config.js
│       ├── _nuxt/
│       ├── images/
│       └── ...
├── wizard/
│   ├── install                  # 安装向导 JSON
│   └── uninstall               # 卸载向导 JSON
├── config/                       # 配置文件模板
│   └── privilege                # 权限配置
├── manifest                      # 应用元数据
├── mihomo.fpk                   # 打包后的应用文件
└── README.md                    # 本文件
```

## 安装向导

安装完成后，系统会显示安装向导，引导你：

1. **欢迎使用Mihomo** - 功能介绍和界面地址
2. **配置代理订阅** - 编辑配置文件添加订阅链接
3. **配置说明** - 常用参数解释
4. **重要提示** - 使用注意事项

## 常见问题

### Q: 如何添加代理订阅？
A: 编辑 `/usr/local/apps/@appconf/mihomo/config.yaml`，找到 `proxy-providers` 部分，修改订阅链接，然后重启应用。

### Q: 修改配置后如何生效？
A: 修改 config.yaml 后需要重启应用，可以在网页管理界面或通过命令重启：
```bash
systemctl restart mihomo
```

### Q: 如何查看应用日志？
A: 日志通常存放在：
```bash
tail -f /var/log/mihomo.log
```

### Q: 网页界面无法访问？
A: 检查防火墙设置，确保 9090 端口未被阻止：
```bash
netstat -tlnp | grep 9090
```

### Q: 如何卸载应用？
A: 通过飞牛系统应用管理界面卸载，或使用命令：
```bash
fnapp uninstall mihomo
```

## 故障排除

### 应用无法启动
1. 检查配置文件是否有语法错误（YAML 格式）
2. 查看系统日志：`systemctl status mihomo`
3. 确保数据库文件存在

### 无法连接代理
1. 确认订阅链接有效
2. 检查网络连接
3. 重启应用

### 高 CPU 使用率
1. 检查规则配置是否过复杂
2. 减少健康检查频率
3. 禁用不必要的流量嗅探

## 更新和升级

### 检查版本
网页界面会显示当前版本信息

### 升级应用
- 通过飞牛应用中心检查并安装更新
- 升级过程会保留你的配置文件

## 开发和贡献

### 项目结构说明
- **cmd/** - 飞牛系统生命周期脚本
- **app/** - 应用资源文件
- **wizard/** - 安装向导定义
- **config/** - 配置模板

### 打包命令
```bash
fnpack build
```

### 验证打包
```bash
fnpack verify mihomo.fpk
```

## 许可证

本项目采用 MIT 许可证。详见 [LICENSE](LICENSE) 文件。

## 相关资源

- **Mihomo GitHub** - https://github.com/MetaCubeX/mihomo
- **Mihomo 文档** - https://wiki.metacubex.one/
- **飞牛 fnOS** - https://www.fnnas.com/
- **应用打包文档** - https://docs.fnnas.com/

## 支持和反馈

如有问题或建议，欢迎：
- 在本仓库提交 Issue
- 联系应用开发者
- 查阅官方文档

## 致谢

感谢 [MetaCubeX](https://github.com/MetaCubeX) 团队开发的 Mihomo 项目。

---

**最后更新:** 2026-02-21

## Mihomo飞牛fpk开发过程使用vscode ai进行



