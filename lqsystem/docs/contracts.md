# 契约（唯一事实来源）

> 这里写的东西，三个人照抄。**不许自己起名字。**
> 要加字段：群里说 → 1 号改这份文件 → 通知大家。

## 0. 自检（每次改完必须逐条跑，有一条不过就不许说"完成"）

| # | 检查什么 | 怎么查 | 不过怎么办 |
|---|---|---|---|
| 1 | 只改了被指定的文件 | `git status --short` | 多余的文件立刻 `git checkout -- <文件名>` |
| 2 | 没有新增依赖 | 读 `requirements.txt` | 只允许 `fastapi`、`uvicorn` 两个 |
| 3 | 列名合法 | 检查你写的每一条 SQL 的列名 | 出现 `state` → 改 `status`；出现 `userId` → 改 `user_id` |
| 4 | 状态值合法 | 搜索所有字符串常量 | 只允许 `pending` `confirmed` `declined` `expired` `open` `closed` |
| 5 | class 名合法 | 搜索 `class="` | 只允许 `page` `title` `card` `btn` `danger` `table` `tag` `ok` `bad` |
| 6 | 返回格式合法 | 检查每一个 `return` | 裸 list/dict → `{"ok":true,"data":...}`；出错 → `{"ok":false,"msg":"中文说明"}` |
| 7 | 没碰别人的文件 | 对照 `CLAUDE.md` 的文件所有权表 | 立刻还原 |

**七条全过之后，才允许输出"完成"。任何一条不过，先说"未完成，卡在第 N 条"。**

## 0.1 禁止（违反即任务失败）

- 禁止执行 `git add` / `git commit` / `git push`
- 禁止安装任何新包（`pip install` 只允许 `requirements.txt` 里已有的）
- 禁止引入 React / Vue / TypeScript / ORM / UI 框架 / 构建工具
- 禁止修改 `CLAUDE.md`、`.gitignore`、`docs/` 下任何文件
- 禁止创建单元测试文件
- 禁止注释掉报错的代码来"让它先跑起来"
- 禁止一次修改超过 2 个文件（除非用户明确要求）
- 禁止返回裸数组
- 禁止用中文作状态值或列名

## 0.2 冲突时的优先级

```
1. 本文件（契约）
2. 用户这次的口头要求
3. 你自己的建议
```

**如果你判断必须违反契约才能实现需求，停下，不要写代码，先说这一句：**

```
这需要违反契约第 X 条（<原文>），因为 <原因>。
要我继续吗？
```

**不允许自作主张修改契约。**

## 0.3 交付前必须给出三样

1. 改了哪些文件（列表，一行一个）
2. 你跑过的验证命令 + **真实输出**（不是描述，是复制过来的输出）
3. 有没有违反契约的地方（有就明说，没有就说"无"）

## 1. 数据库表结构

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

示例（照这个格式建表）：

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

## 2. 状态值（只用这四个）

| 值 | 含义 |
|---|---|
| `pending` | 待确认 |
| `confirmed` | 已确认 |
| `declined` | 已放弃 |
| `expired` | 已失效 |

轮次状态：`open`（可填报）/ `closed`（已关闭）

调剂开关：`1` 愿意 / `0` 不愿意

## 3. 接口返回格式

```json
成功：{"ok": true,  "data": ...}
失败：{"ok": false, "msg": "名额已满"}
```

前端只判断 `ok`，失败就弹 `msg`。**不许返回裸数组。**

## 4. 页面 class 名（只用这 7 个）

| 用途 | 写法 |
|---|---|
| 页面外壳 | `<div class="page">` |
| 标题 | `<h1 class="title">` |
| 卡片 | `<div class="card">` |
| 按钮 | `<button class="btn">` |
| 危险按钮 | `<button class="btn danger">` |
| 表格 | `<table class="table">` |
| 状态标签 | `<span class="tag ok">` / `<span class="tag bad">` |

**1 号、2 号只写 class，不写 `<style>`，不改颜色。样式只在 `style.css` 里，3 号管。**

## 5. 端口与启动

- 后端：`http://127.0.0.1:8000`
- 前端页面由 FastAPI 直接托管在 `static/` 下，例如 `/apply`、`/mentor`
- 启动：双击 `start.bat`
