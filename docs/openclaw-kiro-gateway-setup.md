# OpenClaw + Kiro Gateway 完整安装指南

> 零成本使用 Claude Sonnet 4.5 的完整方案

## 📋 目录

- [为什么需要这个方案](#为什么需要这个方案)
- [准备工作](#准备工作)
- [第一步：安装 Kiro](#第一步安装-kiro)
- [第二步：安装 Kiro Gateway](#第二步安装-kiro-gateway)
- [第三步：配置凭证](#第三步配置凭证)
- [第四步：启动并测试 Gateway](#第四步启动并测试-gateway)
- [第五步：配置 OpenClaw](#第五步配置-openclaw)
- [第六步：验证安装](#第六步验证安装)
- [常见问题](#常见问题)
- [进阶配置](#进阶配置)

---

## 为什么需要这个方案？

**问题**：OpenClaw 是一个强大的 AI 助手，但官方 Claude API 需要付费，每月成本可能达到数百元。

**解决方案**：通过 Kiro Gateway，你可以免费使用 Claude Sonnet 4.5、Haiku 4.5 等模型。

**核心优势**：
- ✅ **完全免费**：利用 Kiro 的免费额度
- ✅ **本地代理**：数据经过你自己的服务器，更安全
- ✅ **OpenAI 兼容**：无缝接入 OpenClaw 和其他工具
- ✅ **多模型支持**：Sonnet 4.5、Haiku 4.5、DeepSeek-V3.2 等

---

## 准备工作

### 系统要求

- **操作系统**：Linux / macOS / Windows (WSL2)
- **Python**：3.10 或更高版本
- **Git**：用于克隆仓库
- **网络**：如果在国内，建议准备 VPN/代理

### 检查 Python 版本

```bash
python3 --version
# 应该显示 Python 3.10.x 或更高
```

如果版本过低，请先升级 Python。

---

## 第一步：安装 Kiro

### 方案 A：Kiro IDE（推荐，图形界面）

1. 访问 [kiro.dev](https://kiro.dev/)
2. 下载适合你系统的安装包
3. 安装并启动 Kiro IDE
4. 登录账号（支持免费的 AWS Builder ID）

### 方案 B：Kiro CLI（命令行）

```bash
# macOS/Linux
curl -fsSL https://kiro.dev/install.sh | sh

# 登录
kiro-cli login
```

**重要**：登录后，Kiro 会自动在 `~/.aws/sso/cache/` 目录下生成凭证文件。

---

## 第二步：安装 Kiro Gateway

### 克隆仓库

```bash
# 克隆项目
git clone https://github.com/jwadow/kiro-gateway.git
cd kiro-gateway

# 或者下载 ZIP：访问 GitHub 页面 → Code → Download ZIP → 解压
```

### 安装依赖

```bash
pip install -r requirements.txt
```

**如果遇到权限问题**：

```bash
pip install --user -r requirements.txt
```

---

## 第三步：配置凭证

### 1. 复制配置模板

```bash
cp .env.example .env
```

### 2. 编辑 .env 文件

```bash
nano .env
# 或者用你喜欢的编辑器：vim .env / code .env
```

### 3. 配置两个关键参数

#### 参数 1：KIRO_CREDS_FILE

指向 Kiro 的凭证文件：

```bash
KIRO_CREDS_FILE="~/.aws/sso/cache/kiro-auth-token.json"
```

**如何找到这个文件**：

```bash
ls ~/.aws/sso/cache/
# 应该看到 kiro-auth-token.json 或类似的文件
```

#### 参数 2：PROXY_API_KEY

设置一个密码保护你的代理服务器（自己编一个复杂的密码）：

```bash
PROXY_API_KEY="my-super-secret-password-123"
```

**⚠️ 重要**：这个密码是你自己设置的，后面配置 OpenClaw 时会用到。

### 4. 完整的 .env 示例

```bash
# Kiro 凭证文件路径
KIRO_CREDS_FILE="~/.aws/sso/cache/kiro-auth-token.json"

# 你的代理服务器密码（自己设置）
PROXY_API_KEY="my-super-secret-password-123"

# 可选：如果在国内，配置代理
# VPN_PROXY_URL="http://127.0.0.1:7890"
```

---

## 第四步：启动并测试 Gateway

### 1. 启动服务器

```bash
python main.py
```

**如果端口 8000 被占用**：

```bash
python main.py --port 9000
```

**成功启动的标志**：

```
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
INFO:     Started reloader process
```

### 2. 测试连接（打开新终端）

```bash
curl http://localhost:8000/health
```

**预期输出**：

```json
{
  "status": "healthy",
  "version": "1.0.0"
}
```

### 3. 测试 Claude 模型

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Authorization: Bearer my-super-secret-password-123" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-4-5",
    "messages": [{"role": "user", "content": "你好，请用中文回复"}],
    "stream": false
  }'
```

**如果返回 Claude 的回复，说明配置成功！**

---

## 第五步：配置 OpenClaw

### 1. 找到 OpenClaw 的配置文件

```bash
# 通常在这个位置
~/.openclaw/models.json
```

### 2. 编辑配置文件

```bash
nano ~/.openclaw/models.json
```

### 3. 添加 Kiro Gateway 配置

```json
{
  "models": [
    {
      "id": "kiro1/claude-sonnet-4.5",
      "provider": "openai",
      "baseURL": "http://localhost:8000/v1",
      "apiKey": "my-super-secret-password-123",
      "model": "claude-sonnet-4-5",
      "displayName": "Claude Sonnet 4.5 (Kiro 免费)",
      "contextWindow": 200000
    },
    {
      "id": "kiro1/claude-haiku-4.5",
      "provider": "openai",
      "baseURL": "http://localhost:8000/v1",
      "apiKey": "my-super-secret-password-123",
      "model": "claude-haiku-4-5",
      "displayName": "Claude Haiku 4.5 (Kiro 免费)",
      "contextWindow": 200000
    }
  ]
}
```

**⚠️ 注意**：
- `apiKey` 必须和你在 `.env` 中设置的 `PROXY_API_KEY` 一致
- `baseURL` 必须是 `http://localhost:8000/v1`（注意有 `/v1`）

### 4. 重启 OpenClaw

```bash
openclaw gateway restart
```

---

## 第六步：验证安装

### 1. 在 OpenClaw 中测试

发送一条消息，例如：

```
你好，请介绍一下你自己
```

### 2. 检查模型是否正确

在 OpenClaw 的设置中，确认当前使用的模型是 `kiro1/claude-sonnet-4.5`。

### 3. 查看 Gateway 日志

在运行 `python main.py` 的终端中，应该能看到请求日志：

```
INFO:     127.0.0.1:xxxxx - "POST /v1/chat/completions HTTP/1.1" 200 OK
```

**如果一切正常，恭喜你！已经成功接入免费的 Claude Sonnet 4.5！**

---

## 常见问题

### Q1: Kiro 免费额度有限制吗？

**答**：有使用限制，但对个人使用足够。具体限制取决于你的 Kiro 账号类型（免费 Builder ID 或付费账号）。

### Q2: 需要 VPN 吗？

**答**：如果在国内，建议配置代理。在 `.env` 中添加：

```bash
VPN_PROXY_URL="http://127.0.0.1:7890"
# 或者 SOCKS5
VPN_PROXY_URL="socks5://127.0.0.1:1080"
```

### Q3: 可以用于商业项目吗？

**答**：请遵守 Kiro 的服务条款。建议个人学习使用，商业项目请购买官方 API。

### Q4: Gateway 启动失败怎么办？

**常见原因**：

1. **端口被占用**：换一个端口 `python main.py --port 9000`
2. **Python 版本过低**：升级到 3.10+
3. **依赖安装失败**：重新运行 `pip install -r requirements.txt`

### Q5: OpenClaw 无法连接 Gateway

**检查清单**：

1. Gateway 是否正在运行？（查看终端）
2. `baseURL` 是否正确？（必须是 `http://localhost:8000/v1`）
3. `apiKey` 是否和 `.env` 中的 `PROXY_API_KEY` 一致？
4. 防火墙是否阻止了本地连接？

### Q6: 如何查看 Gateway 的详细日志？

在 `.env` 中添加：

```bash
DEBUG_MODE=errors
```

日志会保存在 `debug_logs/` 目录。

---

## 进阶配置

### 1. 开机自启动 Gateway

#### Linux/macOS (systemd)

创建服务文件：

```bash
sudo nano /etc/systemd/system/kiro-gateway.service
```

内容：

```ini
[Unit]
Description=Kiro Gateway
After=network.target

[Service]
Type=simple
User=你的用户名
WorkingDirectory=/path/to/kiro-gateway
ExecStart=/usr/bin/python3 /path/to/kiro-gateway/main.py
Restart=always

[Install]
WantedBy=multi-user.target
```

启用服务：

```bash
sudo systemctl enable kiro-gateway
sudo systemctl start kiro-gateway
```

### 2. Docker 部署（推荐生产环境）

```bash
# 使用 docker-compose
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止服务
docker-compose down
```

### 3. 多模型配置

在 OpenClaw 的 `models.json` 中添加更多模型：

```json
{
  "models": [
    {
      "id": "kiro1/claude-sonnet-4.5",
      "model": "claude-sonnet-4-5",
      "displayName": "Claude Sonnet 4.5 (平衡)"
    },
    {
      "id": "kiro1/claude-haiku-4.5",
      "model": "claude-haiku-4-5",
      "displayName": "Claude Haiku 4.5 (快速)"
    },
    {
      "id": "kiro1/deepseek-v3",
      "model": "deepseek-v3",
      "displayName": "DeepSeek V3 (开源)"
    }
  ]
}
```

---

## 总结

通过 Kiro Gateway，你可以：

- ✅ **零成本**使用 Claude Sonnet 4.5
- ✅ **大幅降低** AI 助手的运营成本
- ✅ **保持数据隐私**（本地代理）
- ✅ **无缝集成** OpenClaw 和其他工具

这对于个人开发者、小型团队和创业公司来说，是一个非常实用的解决方案。

---

## 相关资源

- [Kiro Gateway GitHub](https://github.com/jwadow/kiro-gateway)
- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [Kiro 官网](https://kiro.dev/)

---

**最后更新**：2026-03-02  
**作者**：小白（OpenClaw 安装服务提供商）

如需帮助，请联系我。
