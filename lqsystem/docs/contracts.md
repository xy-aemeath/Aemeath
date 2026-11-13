# 统一规定（唯一事实来源）

> **所有"名字"和"数字"都在这一份。** 其他文件只引用，不重复。
> 改这里的任何一条，必须走第 9 节的流程。

---

## 0. 自检（每次改完必须逐条跑，有一条不过就不许说"完成"）

| # | 检查什么 | 怎么查 | 不过怎么办 |
|---|---|---|---|
| 1 | 只改了被指定的文件 | `git status --short` | 多余的文件立刻 `git checkout -- <文件名>` |
| 2 | 没有新增依赖 | 读 `requirements.txt` | 只允许 `fastapi`、`uvicorn` |
| 3 | 列名合法 | 检查你写的每一条 SQL 的列名 | 出现 `state` → 改 `status`；出现 `userId` → 改 `user_id` |
| 4 | 状态值合法 | 搜索所有字符串常量 | 只允许第 4 节那 6 个词 |
| 5 | class 名合法 | 搜索 `class="` | 只允许第 6 节那 9 个 |
| 6 | 返回格式合法 | 检查每一个 `return` | 裸 list/dict → `{"ok":true,"data":...}`；出错 → `{"ok":false,"msg":"中文说明"}` |
| 7 | 没碰别人的文件 | 对照第 7 节 | 立刻还原 |

**七条全过之后，才允许输出"完成"。任何一条不过，先说"未完成，卡在第 N 条"。**

### 0.1 禁止（违反即任务失败）

- 禁止执行 `git add` / `git commit` / `git push`
- 禁止安装任何新包（`pip install` 只允许 `requirements.txt` 里已有的）
- 禁止引入 React / Vue / TypeScript / ORM / UI 框架 / 构建工具
- 禁止修改 `CLAUDE.md`、`.gitignore`、`docs/` 下任何文件
- 禁止创建单元测试文件
- 禁止注释掉报错的代码来"让它先跑起来"
- 禁止一次修改超过 2 个文件（除非我明确要求）
- 禁止返回裸数组
- 禁止用中文作状态值或列名

### 0.2 冲突时的优先级

```
1. 本文件  2. 我这次的口头要求  3. 你自己的建议
```

如果你判断必须违反本文件才能实现需求，**停下，不要写代码**，先说：

```
这需要违反规定第 X 条（<原文>），因为 <原因>。要我继续吗？
```

### 0.3 交付前必须给出三样

1. 改了哪些文件（一行一个）
2. 跑过的验证命令 + **真实输出**（复制过来的，不是描述）
3. 有没有违反规定的地方（有就明说，没有就说"无"）

---

## 1. 技术栈（不许换）

```
Python 3.11 / 3.12 + FastAPI + SQLite（Python 自带 sqlite3）+ 原生 HTML/JS
```

**禁止**：React、Vue、TypeScript、任何 ORM（SQLAlchemy 等）、任何 UI 库（Bootstrap/Tailwind）、任何构建工具、微服务、缓存、消息队列。

**依赖锁版本**：1 号装好后跑 `pip freeze > requirements.txt`，只保留 fastapi / uvicorn 两行。
其他人 `pip install -r requirements.txt`，保证三个人装到的版本完全一致。

---

## 2. 骨架与运行

| 项 | 规定 |
|---|---|
| 谁生成骨架 | **只有 1 号**，跑 `docs/prompts/0-骨架.md` |
| 其他人 | `git pull` 拿同一份，**不许自己再生成一遍** |
| 端口 | `8000`，写死 |
| 启动 | 双击 `start.bat` |
| 地址 | `http://127.0.0.1:8000` |
| 健康检查 | `GET /health` → `{"ok": true}` |
| 数据库文件 | `data.db`，启动时自动创建，**绝不提交**（已在 `.gitignore`） |
| 目录 | `app.py`、`db.py`、`routes_apply.py`、`routes_mentor.py`、`routes_admin.py`、`static/` |

**三个人各生成一次骨架 = 三个不同的项目。** 这是最严重的错。

---

## 3. 数据库表结构

```
mentors    : id, name, lab, quota
students   : id, name, sno, accept_inner, accept_cross
rounds     : id, no, name, status
intentions : id, student_id, mentor_id, round
interviews : id, student_id, mentor_id, round,
             score_attitude, score_logic, score_code, score_time, note
offers     : id, student_id, mentor_id, round, status, created_at
waitlist   : id, mentor_id, student_id, round, rank
```

建表照这个格式：

```sql
CREATE TABLE offers (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  student_id INTEGER NOT NULL,
  mentor_id INTEGER NOT NULL,
  round INTEGER NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending',
  created_at TEXT NOT NULL
);
```

**不许自己起新名字。** 要加字段走第 9 节。

---

## 4. 状态值（只用这 6 个词）

| 词 | 含义 | 用在哪 |
|---|---|---|
| `pending` | 待确认 | offers.status |
| `confirmed` | 已确认 | offers.status |
| `declined` | 已放弃 | offers.status |
| `expired` | 已失效 | offers.status |
| `open` | 可填报 | rounds.status |
| `closed` | 已关闭 | rounds.status |

调剂开关用 `1`（愿意）/ `0`（不愿意）。

**禁止**：中文状态、`waiting`、`success`、`fail`、`done`、`1/2/3` 当状态。

---

## 5. 接口

### 5.1 返回格式（只有这一种）

```json
成功：{"ok": true,  "data": ...}
失败：{"ok": false, "msg": "名额已满"}
```

- `data` 可以是数组、对象、数字
- `msg` 用中文，写给人看（"名额已满"而不是 `QUOTA_FULL`）
- **禁止返回裸数组**

### 5.2 路径前缀（避免撞车）

| 谁 | 前缀 |
|---|---|
| 1 号 | `/api/intentions`、`/api/mentors`、`/api/debug` |
| 2 号 | `/api/mentor/...`、`/api/student/...`、`/api/offers/...` |
| 3 号 | `/api/admin/...`、`/api/public/...` |

**各人的接口只有自己用，不跨人调用。** 三个人共享的是**数据库表**，不是接口。

---

## 6. 页面 class（只用这 9 个）

| 用途 | 写法 |
|---|---|
| 页面外壳 | `<div class="page">` |
| 标题 | `<h1 class="title">` |
| 卡片 | `<div class="card">` |
| 按钮 | `<button class="btn">` |
| 危险按钮 | `<button class="btn danger">` |
| 表格 | `<table class="table">` |
| 状态标签 | `<span class="tag ok">` / `<span class="tag bad">` |

1 号、2 号**只写 class，不写 `<style>`，不改颜色**。样式只在 `static/style.css`，3 号改。

---

## 7. 文件所有权

| 文件 | 主人 |
|---|---|
| `app.py`、`db.py` | 1 号 |
| `routes_apply.py`、`static/apply.html` | 1 号 |
| `CLAUDE.md`、`.gitignore`、`docs/*` | 1 号 |
| `routes_mentor.py`、`static/mentor.html`、`static/my-offers.html` | 2 号 |
| `routes_admin.py`、`static/admin.html`、`static/public.html` | 3 号 |
| `static/style.css`、`start.bat` | 3 号 |

**只改自己名下的文件。** 需要别人的功能就假设它已有、直接调用，不要替他实现。

---

## 8. 数据规模（只规定比例）

| 项 | 值 |
|---|---|
| 导师 | **6 个**，每人 `quota = 3` → 合计 **18**（**固定，不随轮次变**） |
| 学生 | **18 个**（= 总名额，保证人人有位置） |
| 轮次 | **3 轮**，初始只有第 1 轮 `open` |
| 实训室 | **3 个**（至少 2 个，否则跨室调剂没得演） |

**硬约束（代码里不许写死数字）：**

- `total = SUM(quota)`，**不许写成 18**
- `剩余名额 = quota - (pending + confirmed)`，**是算出来的，不是"加出来的"**
- 流程结束时 **未安置人数必须为 0**

姓名、学号由 1 号 seed 时自定（张三李四这种假名，学号 `2023001` 起）。
**严禁真实姓名、学号、手机号。**

---

## 9. 改规定的流程

```
群里说一句要改什么
   ↓
1 号改本文件（只有他能改）
   ↓
1 号在群里通知"已改，请 pull"
   ↓
其他人 git pull
```

跳过任何一步，就会出现"两个人名字不一样"，然后页面白屏、看板显示 0，查一整天。
