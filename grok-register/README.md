# Grok (x.ai) 注册机

## 功能概述

- 自动抓取 x.ai sign-up 页的 site_key / state_tree / action_id
- 使用 Mail.tm 临时邮箱获取验证码
- 支持 YesCaptcha 云解 (优先) 或本地 Turnstile solver
- 并发多线程批量注册，成功后保存 SSO 与账户信息

## 目录结构

- `grok.py`：主入口
- `g/`：服务组件
  - `email_service.py`：Mail.tm 邮箱创建/收信
  - `turnstile_service.py`：YesCaptcha 或本地 solver 取 token
- `api_solver.py`：本地 Turnstile solver（可选）
- `keys/`：输出目录（`grok.txt`、`accounts.txt`）
- `.env`：存放 YesCaptcha key（如需云解）

## 环境准备

1. Python 3.10+，激活你的虚拟环境。
2. 安装依赖：

   ```bash
   pip install -r requirements.txt
   ```

   若没有 requirements.txt，可直接：

   ```bash
   pip install curl_cffi beautifulsoup4 requests python-dotenv quart camoufox patchright rich playwright
   ```

   如使用 chromium solver：

   ```bash
   python -m playwright install chromium
   ```

3. 目录：确保有 `keys/` 目录（保存结果）。

## 配置

### YesCaptcha（推荐）

- 推荐注册链接（支持一下）：https://yescaptcha.com/i/tlkF6o
  在 `.env` 写入：

```
YESCAPTCHA_KEY="你的_yescaptcha_key"
```

`g/turnstile_service.py` 会自动读取，不需其它配置。

### 代理（可选）

编辑 `grok.py` 内的 `PROXIES`：

```python
PROXIES = {
    "http": "http://127.0.0.1:7890",
    "https": "http://127.0.0.1:7890"
}
```

## 运行（云解模式）

```bash
python grok.py
# 输入并发数，回车默认 8
```

日志会显示注册进度，成功后写入：

- `keys/grok.txt`：SSO token 列表
- `keys/accounts.txt`：email:password:SSO

## 运行（本地 solver 模式，可选）

1. 启动 solver（默认 headless chromium）：

```bash
python api_solver.py --browser_type chromium --thread 2 --debug
```

如需可视化：加 `--no-headless`；随机 UA：加 `--random`；若需代理：`--proxy` 并导出 http_proxy/https_proxy。

2. 另开终端运行 grok：

```bash
python grok.py
```

## 常见问题

- **Mail.tm 超时**：重试或加代理。
- **YesCaptcha 错误码 ERROR_KEY_DOES_NOT_EXIST**：检查 key 是否正确/有余额/未被空格截断。
- **CAPTCHA_FAIL**：提升通过率的方法：用云解；本地 solver 用 `--no-headless --random` 且配住宅代理；或降低并发。
