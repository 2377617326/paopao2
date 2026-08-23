# 竞赛平台房间自动化 Bot（paopao2 部署）

本仓库为 `2377617326/paopao2`，是原 `room-bot` 自动化的完整复制部署。

自动完成: 登录 → 创建房间 → 检测人数 → 开始实验(翻期) → 结束实验。

## 文件说明
- `room_scheduler.py` — 全自动调度器（建房/接管/翻期/提交决策/结束，核心脚本）
- `room_bot.py` — 主脚本, 本地/服务器均可运行
- `.github/workflows/room_bot.yml` — GitHub Actions 定时任务配置(每2小时跑一次 + 自动重启，可自行改 cron)

## 本地运行(可选)
```bash
pip install requests

# 查看积分/签到
python room_bot.py --login 账号 密码 --status

# 创建房间
python room_bot.py --login 账号 密码 --create --room-name 房间名

# 创建房间并自动监控人数 -> 满3人开始实验 -> 翻期 -> 结束
python room_bot.py --login 账号 密码 --create --monitor --min-players 3 --max-wait 3600

# 手动: 开始/翻期/结束
python room_bot.py --login 账号 密码 --start --room-id 房间ID
python room_bot.py --login 账号 密码 --period --room-id 房间ID
python room_bot.py --login 账号 密码 --finish --room-id 房间ID
```

## GitHub Actions 部署
见下方步骤。核心是: 把这两个文件放进 GitHub 仓库, 在仓库 Secrets 里设置 `BOT_USER` 和 `BOT_PASS`, 之后自动定时执行。

## 当前配置 (paopao2)
- **账号**: `云泽杯-1` / `1234546@a`（对应仓库 Secrets: `BOT_USER=云泽杯-1`, `BOT_PASS=1234546@a`）
- **房间名模板**: `云泽杯自动比赛 自动测试{HH:MM}开`（HH:MM = 建房时间 + 40 分钟）
- **运行时间**: 北京时间 **07:00 – 22:20** 建房开赛；其余时间仅检查是否有遗漏(未结束)房间并处理
- **concurrency group**: `paopao2`（与其他仓库的调度互不抢占）
- 建房参数: 4 季度、每周期 20 分钟、密码 `123`、建房后 40 分钟强制开赛
- 决策: 每季度提交 8 类决策（type7=9999、type8 state=2 终审），不提交会封号

## 接口返回码速查
| 接口 | 返回码 | 含义 |
| --- | --- | --- |
| addRoom | 0 | 房间已爆满 |
| addRoom | 2 | 有未结束的实验 |
| addRoom | 3 | 非法数据 |
| startRoomExp type=1 | 1 | 实验已开始 |
| startRoomExp type=2 | 1 | 翻期成功 |
| startRoomExp type=3 | 1 | 实验已结束 |