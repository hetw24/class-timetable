# 📚 飞书课程表自动提醒服务 (Cloudflare Worker)

基于 Cloudflare Workers 和 Cloudflare KV 构建的轻量级课程表后台与飞书机器人提醒服务。通过 Cron Trigger 定时任务，在每节课开课前自动匹配课程并推送飞书卡片消息。

---

## ✨ 项目特性

* 🚀 **极速响应**：部署在 Cloudflare Worker 边缘网络，毫秒级响应。
* ⏰ **智能课前提醒**：支持自适应推导提醒时刻（默认课前 5 分钟），开课时间调整时无需手动计算提醒点。
* 🗄️ **KV 解耦设计**：课程数据与前端/ Worker 共享同一个 KV 数据库，支持灵活的课程节点匹配。
* 📱 **富文本卡片**：使用飞书 Interactive 消息卡片，清晰展示课程名称、时间、教室地点、任课教师及周次信息。
* 🔍 **完善的调试路由**：自带诊断与测试接口，开箱即用。

---

## 🛠️ 快速开始

### 1. 准备工作

* 一个 [Cloudflare](https://dash.cloudflare.com/) 账号。
* 一个飞书自定义机器人 Webhook URL（在飞书群组内添加自定义机器人即可获得）。

### 2. 配置说明

修改 `index.js` 中的基础配置：

```javascript
// 1. 填入你的飞书机器人 Webhook 链接
const FEISHU_WEBHOOK_URL = "https://open.feishu.cn/open-apis/bot/v2/hook/xxx-xxx-xxx"; 

// 2. 基础节次与上课时间映射表 (开课时间 start，remTime 自动倒推 5 分钟)
const PERIOD_CONFIG = {
  1: { start: "08:00", name: "第 1-2 节" },
  2: { start: "10:00", name: "第 3-4 节" },
  3: { start: "13:20", name: "第 5-6 节" },
  4: { start: "15:20", name: "第 7-8 节" },
  5: { start: "18:00", name: "第 9-10 节" },
  6: { start: "19:50", name: "第 11 节" }
};

```

### 3. KV 绑定配置

Worker 会依次从绑定的 KV 空间中尝试读取以下 Key 中的课程 JSON 数据：

1. `freshman-1`
2. `TIMETABLE_DB`
3. `timetable`

#### 课程 JSON 数据结构示例：

```json
[
  {
    "id": "1784602834435",
    "name": "高等数学",
    "day": 1,
    "period": 2,
    "weeks": "1-16",
    "room": "教二 201",
    "teacher": "张老师",
    "color": "#0072bd"
  }
]

```

### 4. Cloudflare Worker 定时器配置 (Cron Triggers)

为了确保能在对应的提醒节点准确触发，请在 Cloudflare Worker 控制台或 `wrangler.toml` 中添加 Cron 定时任务：

```toml
# 建议配置为每分钟触发一次，由 Worker 内部逻辑精确匹配时间节点
[triggers]
crons = ["* * * * *"]

```

---

## 📡 调试与路由接口

部署完成后，可以通过以下路由检查服务状态：

| 路由地址 | 请求方式 | 功能描述 |
| --- | --- | --- |
| `/` | `GET` | 检查 Worker 基础运行状态 |
| `/debug` | `GET` | 查看当前北京时间、KV 绑定状态、自适应提醒时间表与课程数据预览 |
| `/test-notify` | `GET` | 手动触发一条默认测试卡片到飞书，验证 Webhook 连通性 |

---

## 📄 开源协议

本项目采用 [MIT License](https://www.google.com/search?q=LICENSE) 协议。
