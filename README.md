import os
import feedparser
import requests

DEEPSEEK_API_KEY = os.environ.get("DEEPSEEK_API_KEY")
WECHAT_WEBHOOK = os.environ.get("WECHAT_WEBHOOK")

def get_ai_summary(news_text):
    url = "https://api.deepseek.com/chat/completions"
    headers = {
        "Authorization": f"Bearer {DEEPSEEK_API_KEY}",
        "Content-Type": "application/json"
    }
    prompt = f"你是一个科技情报分析师。请用中文将以下新闻总结成一份每日简报，分为'产品动态'、'行业观察'两个板块，每条用一句话概括，最后加一段30字左右的今日总结：\n\n{news_text}"
    data = {
        "model": "deepseek-chat",
        "messages": [{"role": "user", "content": prompt}],
        "temperature": 0.7
    }
    response = requests.post(url, headers=headers, json=data, timeout=60)
    response.raise_for_status()
    return response.json()["choices"][0]["message"]["content"]

def main():
    print("每日情报开始运行...")
    if not DEEPSEEK_API_KEY or not WECHAT_WEBHOOK:
        print("错误：缺少 DEEPSEEK_API_KEY 或 WECHAT_WEBHOOK 环境变量。")
        return
    rss_url = "https://sspai.com/feed"
    feed = feedparser.parse(rss_url)
    if not feed.entries:
        print("抓取失败，请检查网络。")
        return
    news_text = ""
    for index, entry in enumerate(feed.entries[:5], 1):
        news_text += f"{index}. {entry.title}\n链接: {entry.link}\n\n"
    print("正在请求 AI 总结...")
    try:
        summary = get_ai_summary(news_text)
        print("AI 总结成功！")
    except Exception as e:
        print(f"AI 调用失败: {e}")
        summary = "AI总结失败，以下是原始新闻：\n\n" + news_text
    markdown_content = f"### 每日情报\n\n{summary}"
    headers = {"Content-Type": "application/json"}
    data = {"msgtype": "markdown", "markdown": {"content": markdown_content}}
    try:
        response = requests.post(WECHAT_WEBHOOK, json=data, headers=headers, timeout=10)
        if response.status_code == 200:
            print("消息已成功发送到企业微信！")
        else:
            print(f"发送失败，状态码：{response.status_code}")
    except Exception as e:
        print(f"发送时出现错误：{e}")

if __name__ == "__main__":
    main()# main.py
