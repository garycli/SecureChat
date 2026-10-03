# SecureChat
SecureChat 桌面加密通信原型，包含 PKI 证书、TOTP 多因素认证、证书指纹验证和客户端网络通信代码。

GUI 入口：[test.py](test.py)（该文件是客户端程序，不是自动化测试套件）。

[![Status](https://img.shields.io/badge/Status-Experimental-yellow)](test.py)

**客户端加密、PKI 与 MFA 的研究演示原型**
**Client-Side Encryption, PKI and MFA Prototype**

**验证边界：** 本文以已提交源码为依据。当前仓库未提供独立安全审计、合规认证或可复现的性能、成功率报告，不能据此声明生产就绪。

---

## 🎯 核心特性 Core Features

### 🔐 加密与签名
- **AES-256-GCM** - 消息加密实现（不代表加密模块获得 FIPS 认证）
- **Ed25519** - 签名和验签接口
- **X25519** - ECDH密钥交换；基础模块使用持久化密钥，未验证完美前向保密
- **SHA-256** - 加密哈希函数

### 🏛️ PKI证书基础设施
- **RSA-4096** - CA证书颁发机构
- **X.509 v3** - 标准数字证书
- **吊销记录** - 本地 JSON 序列号列表，未实现标准签名 CRL
- **指纹验证** - 人工比对证书指纹
- **信任管理** - 本地可信对等方记录

### 🔑 多因素认证 (MFA)
- **TOTP** - 时间基础一次性密码（RFC 6238）
- **认证器配置** - 提供 otpauth URI 和 Base32 密钥
- **恢复码** - 10个备用恢复码

### 🛡️ 安全策略
- **密码规则** - GUI 注册路径要求最低16字符，包含大小写、数字、特殊字符
- **输入验证** - 用户名、消息等字段的校验接口
- **会话参数** - 策略包含30分钟超时和登录次数配置，GUI 尚未强制执行超时或失败锁定
- **事件日志** - 增强加密模块提供本地日志；当前 GUI 使用基础加密模块
- **防重放攻击** - 序列号机制

### 🌐 网络连接
- **WebSocket** - `test.py` 默认使用 `ws://`，未配置 TLS
- **信令与中继** - 联网依赖外部服务，当前仓库未提交服务端实现
- **NAT与UDP打洞** - 提供客户端模块，跨网络可靠性尚未验证

---

## 📊 实现证据 Implementation Evidence

| 模块 | 已提交源码 | 范围 |
|------|------------|------|
| 加密与签名 | [crypto_module.py](client/crypto_module.py) | AES-GCM、Ed25519、X25519及序列号检查代码 |
| PKI与信任 | [pki_manager.py](client/pki_manager.py) | 本地证书、密钥存储、吊销记录和指纹管理 |
| MFA | [mfa_module.py](client/mfa_module.py) | TOTP与恢复码 |
| 输入与策略 | [security_policy.py](common/security_policy.py) | 校验接口和策略参数 |
| 增强加密 | [crypto_module_enhanced.py](client/crypto_module_enhanced.py) | 双棘轮和事件日志的实验性实现 |

源码存在不等于安全性、集成完整性或合规性已验证。

---

## 🚀 快速开始 Quick Start

### 1. 安装依赖

```bash
python3 -m pip install cryptography pycryptodome websockets qrcode pillow
```

需要带 Tkinter 的 Python 桌面环境。

### 2. 运行已提交入口

```bash
# 增强版 GUI 客户端
python3 test.py

# 另一套 GUI（使用 client/p2p_client.py，不含相同的PKI/MFA登录流程）
python3 main.py
```

联网前请配置 `test.py` 中的 `SIGNALING_SERVER` 并准备可用服务端；当前仓库未提交服务端。增强客户端目前从本机 `~/.securechat_military/keys/` 读取对端证书及公钥，跨机器密钥分发流程仍需补齐。这些启动命令不代表完整通信流程已验证。

**首次使用**:
1. 输入用户名（3-32字符）
2. 设置强密码（≥16字符）
3. 将显示的密钥导入认证器并完成MFA验证
4. 保存恢复码

**日常使用**:
1. 输入用户名和密码
2. 输入认证器中的6位数字
3. 连接对等方并验证指纹
4. 开始加密聊天

---

## 📁 项目结构

当前已提交文件：

```
SecureChat/
├── client/
│   ├── crypto_module.py
│   ├── crypto_module_enhanced.py
│   ├── hole_puncher.py
│   ├── mfa_module.py
│   ├── nat_detector.py
│   ├── p2p_client.py
│   └── pki_manager.py
├── common/
│   └── security_policy.py
├── README.md
├── main.py
├── run.sh
└── test.py
```

---

## 📚 文档导航

### 已提交入口与模块

| 内容 | 入口 | 用途 |
|------|------|------|
| 增强客户端 | [test.py](test.py) | PKI、MFA与聊天GUI实现 |
| 基础客户端 | [main.py](main.py) | 另一套GUI与P2P客户端接入 |
| PKI模块 | [pki_manager.py](client/pki_manager.py) | 证书与本地密钥管理 |
| MFA模块 | [mfa_module.py](client/mfa_module.py) | TOTP与恢复码实现 |
| 项目概览 | [README.md](README.md) | 当前公开范围与限制 |

### 快速链接

- 🎮 **[test.py](test.py)** - 增强GUI入口
- 🌐 **[p2p_client.py](client/p2p_client.py)** - 客户端连接逻辑
- 🔐 **[crypto_module.py](client/crypto_module.py)** - 基础加密实现
- 🧪 **[crypto_module_enhanced.py](client/crypto_module_enhanced.py)** - 实验性增强模块

---

## 🎓 使用场景 Use Cases

### 原型展示与学习

- 加密、证书和TOTP模块的学习与代码研究
- 桌面GUI和异步网络通信的项目展示
- 受控环境中的一对一通信实验

### 尚未验证的场景

- 生产环境及需要独立安全审计、认证或合规验收的应用
- 高吞吐量、多用户并发或复杂NAT环境
- 移动端和带宽受限环境

---

## 🔒 安全特性详解

### 客户端加密流程

**源码中的流程**：
- 客户端先调用 AES-GCM 加密，再向网络发送密文
- 增强GUI从本地密钥存储读取身份与DH密钥
- 密钥分发、对端身份绑定和端到端安全性仍需进一步验证

### PKI信任链

```
    [Root CA]
       |
       |-- 签名 ---> [Alice证书]
       |              (有效期1年)
       |
       |-- 签名 ---> [Bob证书]
                      (有效期1年)

验证流程:
1. 客户端选择对等方（当前读取本机记录）
2. 读取对端证书及公钥
3. 验证CA签名
4. 计算并检查已记录的证书指纹
5. 用户手动确认指纹 ✓
6. 添加到信任数据库
```

### 多因素认证 (MFA)

```
登录流程:
1. 用户名 + 密码 -----------> [第一因素]
2. TOTP 6位数字 -----------> [第二因素]
   (30秒周期，允许相邻时间步)
3. 可选: 恢复码 -----------> [备用因素]

说明:
- TOTP默认30秒周期，验证窗口允许相邻时间步
- 恢复码用于备用登录，不能替代私钥及配置备份
```

---

## 🛠️ 技术栈 Technology Stack

### 加密库
- **cryptography** - 密钥、证书和签名实现所用的 Python 库
- **pycryptodome** - AES-GCM 实现
- **hashlib/hmac** - 当前 TOTP 模块使用的标准库

### 网络
- **websockets** - WebSocket协议
- **asyncio** - 异步网络通信；当前默认连接未启用 TLS

### 界面
- **tkinter** - GUI框架（内置）

### 其他
- **qrcode** - 二维码生成（MFA设置）
- **pillow** - 图像处理

---

## 📈 性能指标 Performance

当前仓库未提供基准脚本、测试环境或可复现结果。加解密耗时、吞吐量、网络延迟与NAT穿透开销均需实测，本文不列出未经验证的数值。

---

## 🔐 合规性 Compliance

当前仓库未提供认证证书、第三方审计报告或合规验收材料，不声明已经取得 FIPS 140-2、SOC 2、ISO 27001 等认证或满足 GDPR、HIPAA、PCI DSS 等合规要求。

---

## 🐛 已知限制 Known Limitations

### 功能限制

1. **群聊** - 当前代码围绕一对一通信，未包含群聊实现
2. **文件传输** - 存在附件界面入口，未见完整文件传输协议实现
3. **移动客户端** - 当前仅有桌面版
4. **视频通话** - 未实现实时音视频

### 性能限制

1. **并发连接** - 未提交服务端和容量基准，无法给出并发上限
2. **消息长度** - 输入校验器默认限制10,000字符，不代表网络消息的字节上限
3. **带宽与NAT** - 未提供跨网络可靠性或低带宽测试结果

### 实验性功能

1. **双棘轮算法** - `client/crypto_module_enhanced.py` 为实验性实现，需要进一步验证
   - GUI 当前使用 `client/crypto_module.py`；不据此宣称其已通过安全审计

---

## 🔄 更新日志 Changelog

### v2.0 (2025-11-25) - PKI与MFA模块

**新功能**:
- ✅ PKI证书模块（X.509, CA, 本地吊销记录）
- ✅ 多因素认证（TOTP, 恢复码）
- 客户端 WebSocket 通信代码（服务端未提交）
- ✅ 安全策略（输入验证, 密码要求）
- ✅ 本地信任管理（指纹比对）
- 增强加密模块的本地事件日志（实验性）
- ✅ GUI客户端（日常使用）

**实现范围**:
- 提供加密密钥保存与加载接口
- 提供证书、指纹验证和MFA登录接入
- 独立部署指南、安全审计和实现状态文档尚未提交

### 基础客户端路径（main.py / client/p2p_client.py）

**功能**:
- 客户端AES-256-GCM加密代码
- STUN NAT检测与UDP打洞代码
- WebSocket信令客户端

**与增强GUI的差异**:
- 未接入同样的PKI、MFA与本地密钥存储流程
- 仍依赖外部信令服务，不包含服务端认证实现

---

## 🤝 贡献 Contributing

### 报告问题

如发现安全漏洞，请**私密**报告：
- 不要公开披露
- 发送到安全邮箱（如有）
- 提供详细复现步骤

### 改进建议

欢迎提交改进建议：
1. 性能优化
2. 功能增强
3. 文档改进
4. 测试用例

---

## 📄 许可证 License

当前 README 声明采用 MIT；仓库尚未提供 `LICENSE` 文件，许可证正文待补充。

---

## ⚠️ 免责声明 Disclaimer

1. **使用风险自负** - 本系统按"原样"提供
2. **需要专业审计** - 生产部署前建议第三方安全审计
3. **合规责任** - 用户需确保符合当地法律法规
4. **出口管制** - 强加密可能受到出口管制

---

## 📞 技术支持 Support

### 获取帮助

1. **查看文档** - 首先查阅相关文档
2. **检查入口** - `python3 test.py`（GUI 客户端，不是自动化测试）
3. **查看输出** - 当前客户端的控制台输出；增强模块日志尚未接入该 GUI

### 常见问题

**Q: 证书验证失败？**
A: 检查系统时间、证书有效期、CA证书

**Q: MFA验证失败？**
A: 同步手机时间、等待新的30秒周期

**Q: 连接超时？**
A: 检查网络、防火墙及 `test.py` 中的 `SIGNALING_SERVER`；当前仓库未提供服务端

**Q: 如何备份数据？**
A: 备份整个 `~/.securechat_military/` 目录

**Q: 如何恢复MFA？**
A: 使用注册时保存的10个恢复码

---

## 🎯 下一步 Next Steps

### 立即开始

```bash
# GUI入口；联网需额外准备服务端
python3 test.py

# 阅读已提交模块
cat client/crypto_module.py
cat client/pki_manager.py client/mfa_module.py
```

### 深入了解

1. 📖 阅读 [pki_manager.py](client/pki_manager.py) - 本地证书和密钥管理
2. 🔍 查看 [crypto_module.py](client/crypto_module.py) - 加密与序列号检查
3. 🛠️ 研究 [mfa_module.py](client/mfa_module.py) - TOTP与恢复码
4. 🧪 查看 [crypto_module_enhanced.py](client/crypto_module_enhanced.py) - 实验性增强实现

---

## 🌟 当前可查看的实现

- PKI证书、本地密钥存储、指纹及信任记录
- TOTP与恢复码模块
- 桌面GUI与客户端加密通信代码
- 实验性双棘轮及事件日志模块

完整通信成功率、性能和生产部署结果仍需可复现验证。

---

<div align="center">

**SecureChat - 桌面加密通信原型**

[![Status](https://img.shields.io/badge/Status-Experimental-yellow)](test.py)

*客户端加密 · PKI证书 · 多因素认证 · 实验性原型*

</div>

---

**最后更新**: 2026-10-03
**版本**: 2.0
**状态**: 实验性原型，尚未完成生产验证
