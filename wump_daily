import json
import time
import requests
import jwt
from datetime import datetime, timezone

CONFIG_PATH = "config.json"

# ===================== 加载和保存配置 =====================
def load_config():
    with open(CONFIG_PATH, "r") as f:
        return json.load(f)

def save_config(config):
    with open(CONFIG_PATH, "w") as f:
        json.dump(config, f, indent=2)

# ===================== Token 管理 =====================
def extract_refresh_token(access_token):
    try:
        decoded = jwt.decode(access_token, options={"verify_signature": False})
        return decoded.get("app_metadata", {}).get("discord", {}).get("refresh_token")
    except Exception:
        return None

def is_token_expired(access_token):
    try:
        decoded = jwt.decode(access_token, options={"verify_signature": False})
        exp = decoded.get("exp")
        return datetime.now().timestamp() > (exp - 300)
    except Exception:
        return True

def refresh_access_token(api_key, refresh_token):
    url = "https://api.wump.xyz/auth/v1/token?grant_type=refresh_token"
    headers = {
        "apikey": api_key,
        "Content-Type": "application/json"
    }
    payload = { "refresh_token": refresh_token }
    resp = requests.post(url, json=payload, headers=headers)
    if resp.status_code == 200:
        return resp.json().get("access_token")
    else:
        print("❌ refresh_token 已失效，请更新 config.json")
        return None

def ensure_valid_token(config):
    access_token = config["ACCESS_TOKEN"]
    api_key = config["API_KEY"]

    if is_token_expired(access_token):
        print("⚠️ access_token 过期，尝试刷新...")
        refresh_token = extract_refresh_token(access_token)
        if not refresh_token:
            print("❌ 无法提取 refresh_token，请重新登录更新 config.json")
            return None
        new_token = refresh_access_token(api_key, refresh_token)
        if new_token:
            config["ACCESS_TOKEN"] = new_token
            save_config(config)
            print("✅ 已刷新 access_token 并更新配置")
            return new_token
        else:
            return None
    else:
        print("✅ access_token 有效")
        return access_token

# ===================== 签到相关 =====================
def get_user_tasks(user_id, headers):
    url = f"https://api.wump.xyz/rest/v1/user_tasks?select=*&user_id=eq.{user_id}"
    resp = requests.get(url, headers=headers)
    if resp.status_code == 200:
        return resp.json()
    else:
        print("❌ 获取任务列表失败")
        return []

def is_today_claimed(tasks):
    today = datetime.now(timezone.utc).date()
    for task in tasks:
        if task.get("task_type") == "daily" and task.get("completed_at"):
            dt = datetime.fromisoformat(task["completed_at"].replace("Z", "+00:00"))
            if dt.date() == today:
                return True
    return False

def get_daily_task_id(tasks):
    for task in tasks:
        if task.get("task_type") == "daily":
            return task.get("task_id")
    return None

def claim_daily_task(task_id, headers):
    url = "https://api.wump.xyz/functions/v1/api/tasks/daily"
    payload = {"taskid": task_id}
    resp = requests.post(url, json=payload, headers=headers)
    if resp.status_code == 200:
        result = resp.json()
        if result.get("success"):
            earned = result.get("result", {}).get("earned", 0)
            total = result.get("result", {}).get("total_points", 0)
            print(f"✅ 签到成功！获得 {earned} 分，累计 {total}")
        else:
            print("❌ 签到失败: ", result)
    else:
        print("❌ 签到请求失败: ", resp.text)

# ===================== 主循环 =====================
def main():
    while True:
        print("\n========== [WUMP 签到脚本] ==========")
        try:
            config = load_config()
            token = ensure_valid_token(config)
            if not token:
                print("❌ 无法使用有效 access_token，终止")
                return

            headers = {
                "apikey": config["API_KEY"],
                "Authorization": f"Bearer {token}",
                "Content-Type": "application/json",
                "Origin": "https://wump.xyz",
                "Referer": "https://wump.xyz/",
                "User-Agent": "Mozilla/5.0",
                "x-client-info": "supabase-ssr/0.5.2",
                "x-supabase-api-version": "2024-01-01"
            }

            tasks = get_user_tasks(config["USER_ID"], headers)
            if is_today_claimed(tasks):
                print("✅ 今日已签到，等待明天再试...")
            else:
                task_id = get_daily_task_id(tasks)
                if task_id:
                    claim_daily_task(task_id, headers)
                else:
                    print("⚠️ 未找到每日任务 ID，无法签到")
        except Exception as e:
            print(f"❗ 脚本异常: {e}")

        print("⏳ 进入休眠 24 小时...\n")
        time.sleep(24 * 60 * 60)

# 启动
if __name__ == "__main__":
    main()
