# Telegram 自动签到助手 (Telegram Auto Checkin)

本项目基于 GitHub Actions 和 Python Telethon 库，实现 Telegram 机器人的自动化签到。

目前支持以下服务的自动签到：
1.  **Doubledou Bot** (`dounai.py`)：通过发送 `/checkin` 指令签到。
2.  **ICMP9 Bot** (`icmp9.py`)：通过点击最新的按钮进行签到。

> **注意**：如果你只需要使用其中某一个功能（例如只签到 Doubledou），请参考下文的 [配置说明](#4-可选只运行特定的签到脚本)。

---

## 🚀 快速开始

### 1. Fork 本仓库
点击页面右上角的 **Fork** 按钮，将本项目克隆到你自己的 GitHub 账号下。

### 2. 获取 Telegram API 凭证
为了让脚本能登录你的账号，你需要申请 API ID：
1.  登录 [my.telegram.org](https://my.telegram.org)。
2.  进入 **API development tools**。
3.  任意填写 `App title` 和 `Short name`，平台选择 `Other`。
4.  点击创建，记下 **App api_id** 和 **App api_hash**。

### 3. 获取 Session String (最关键一步)
由于 GitHub Actions 无法进行交互式登录（输入验证码），你需要提前生成一串登录密钥（Session String）。

**方法：**
在你的本地电脑（需安装 Python）或 [Google Colab](https://colab.research.google.com/) 上运行以下代码：

```python
# 首先安装依赖:
!pip install telethon
!pip install asyncio
from telethon import TelegramClient
from telethon.sessions import StringSession
import asyncio

# 替换为你的 api_id 和 api_hash
api_id = 替换数字
api_hash = '替换字符'

async def generate_session():
    # 初始化 Client
    client = TelegramClient(StringSession(), api_id, api_hash)

    # 使用 await 启动，这会触发登录流程
    # 注意：运行后，Colab 会在输出框出现输入框，让你输入手机号和验证码
    await client.start()

    print("\n请将下方生成的 String Session 复制并保存到 GitHub Secrets：")
    print("------------------------------------------------------")
    print(client.session.save())
    print("------------------------------------------------------")

    # 断开连接
    await client.disconnect()

# 在 Colab/Jupyter 中，直接 await 函数即可运行
await generate_session()

```
运行后输入手机号和验证码，终端会打印出一长串字符，这就是 `TELEGRAM_SESSION`。

### 4. 配置 GitHub Secrets
回到你 Fork 后的 GitHub 仓库，依次点击：
`Settings` -> `Secrets and variables` -> `Actions` -> `New repository secret`。

添加以下变量：

| Secret Name | Value (填入的内容) | 说明 |
| :--- | :--- | :--- |
| `API_ID` | `123456` | 你的 API ID |
| `API_HASH` | `xxxxxx` | 你的 API Hash |
| `TELEGRAM_SESSION` | `1BVts...` | 上一步生成的长字符串 |
| `BOT_USERNAME` | `ICMP9_Bot` | **(可选)** 如果你需要运行 icmp9 签到则必须填，否则可忽略 |

---

### 5. (可选) 只运行特定的签到脚本

如果你**只需要**签到 `doubledoubot`，而不需要 `icmp9`，建议修改 Workflow 文件以避免报错。

1.  在你的仓库中找到 `.github/workflows/checkin.yml` 文件。
2.  点击铅笔图标进行编辑。
3.  找到 `Run icmp9 Checkin Script` 这一段，将其**注释掉**或**删除**。

**修改后的 `jobs` 部分示例（只保留 dounai）：**

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install telethon

      # --- 如果不需要 ICMP9，请删除或注释以下部分 ---
      # - name: Run icmp9 Checkin Script
      #   env:
      #     API_ID: ${{ secrets.API_ID }}
      #     API_HASH: ${{ secrets.API_HASH }}
      #     TELEGRAM_SESSION: ${{ secrets.TELEGRAM_SESSION }}
      #     BOT_USERNAME: ${{ secrets.BOT_USERNAME }}
      #   run: python icmp9.py
      # ---------------------------------------------

      - name: Run Command Checkin (doubledoubot)
        env:
          API_ID: ${{ secrets.API_ID }}
          API_HASH: ${{ secrets.API_HASH }}
          TELEGRAM_SESSION: ${{ secrets.TELEGRAM_SESSION }}
        run: python dounai.py
```
4.  点击 **Commit changes** 保存。

---

## ⏰ 运行时间
*   **自动运行**：脚本默认在每天 **北京时间上午 9:00** (UTC 1:00) 自动执行。
*   **手动触发**：进入 `Actions` 页面 -> 选择 `Telegram Auto Checkin` -> 点击 `Run workflow` 按钮可随时测试。

## ⚠️ 免责声明
*   本项目仅供学习和技术研究使用。
*   请勿将脚本用于群发广告、骚扰他人等违反 Telegram ToS 的行为。
*   使用 Userbot 存在一定的账号风险，请使用小号或自行承担后果。
