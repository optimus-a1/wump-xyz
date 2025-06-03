# WUMP Daily Auto Check-in

自动化签到脚本，适用于 [https://wump.xyz](https://wump.xyz) 平台，支持多账号、access_token 自动刷新、每日执行一次签到。

---

## 📁 项目结构

| 文件名        | 描述 |
|---------------|------|
| `wump_daily`  | 主执行脚本（Python） |
| `config.json` | 配置文件，包含账号信息 |

---

## 🧩 依赖安装

请先安装 Python 第三方库：

```bash
pip install requests pyjwt
```

---

## ⚙️ 配置文件：`config.json`

支持单账号或多账号结构。

### ✅ 单账号格式：

```json
{
  "API_KEY": "你的_apikey",
  "ACCESS_TOKEN": "你的_access_token",
  "USER_ID": "你的_user_id"
}
```

### ✅ 多账号格式（数组）：

```json
[
  {
    "API_KEY": "账号1_apikey",
    "ACCESS_TOKEN": "账号1_access_token",
    "USER_ID": "账号1_user_id"
  },
  {
    "API_KEY": "账号2_apikey",
    "ACCESS_TOKEN": "账号2_access_token",
    "USER_ID": "账号2_user_id"
  }
]
```

---

## 🔍 如何获取 API_KEY / ACCESS_TOKEN / USER_ID

你可以通过 Chrome 浏览器打开开发者工具（F12） → Network 面板，刷新页面并完成一次登录，然后查看如下信息：

### ✅ `API_KEY`

1. 任意接口请求中（如 `/rest/v1/user_tasks`），请求头中的：
   ```http
   apikey: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...
   ```

### ✅ `ACCESS_TOKEN`

1. 请求头 `Authorization` 字段中：
   ```http
   Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
   ```

### ✅ `USER_ID`

1. 接口请求中 URL 或返回体常包含该字段，如：
   ```http
   /rest/v1/users?id=eq.c108a0af-2f03-4e05-b01b-30c85beb4f6f
   ```
   其中 `c108a0af-2f03-4e05-b01b-30c85beb4f6f` 就是你的 `USER_ID`。

---

## 🚀 启动脚本

```bash
python3 wump_daily
```

脚本将常驻运行，每 24 小时自动执行一次签到，并跳过已签到账号。

---

## ✅ 功能特性

- ✅ 自动签到每日任务（每日一次）
- ✅ 支持多账号轮询签到
- ✅ 自动判断 token 过期
- ✅ 自动提取 refresh_token 并刷新
- ✅ 自动更新 config.json 中的 token
- ✅ 支持无 crontab 情况下常驻运行

---

## 📌 注意事项

- 请确保每个账号 `ACCESS_TOKEN` 有效（首次手动获取即可）
- 每次脚本运行将持续运行（建议配合 `screen` 或 `tmux`）

---

## 👨‍💻 作者

Made with ❤️ by [optimus-a1](https://github.com/optimus-a1)
