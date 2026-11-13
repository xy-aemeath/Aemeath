# 给 AI 的约定

> 每次开工第一句：**先读 CLAUDE.md、docs/contracts.md、docs/roles/当前号.md。**

## 项目是什么

学生实训室进组选拔录取系统的演示版。核心规则见 `docs/rules.md`。
三个人并行开发，每人只负责自己名下的一批文件。

## 技术栈（不许换）

Python 3.11 + FastAPI + SQLite（原生 `sqlite3`）+ 原生 HTML/JS。

- 不许引入 React / Vue / TypeScript / 任何前端框架
- 不许引入 ORM（SQLAlchemy 等），直接写 SQL
- 不许引入任何 UI 库（Bootstrap / Tailwind 等）
- 不许拆成微服务，不许加缓存、消息队列

## 文件所有权

| 文件 | 主人 |
|---|---|
| `app.py`、`db.py`、`routes_apply.py`、`static/apply.html` | 1 号 |
| `routes_mentor.py`、`static/mentor.html`、`static/my-offers.html` | 2 号 |
| `routes_admin.py`、`static/admin.html`、`static/public.html`、`static/style.css`、`start.bat` | 3 号 |
| `CLAUDE.md`、`.gitignore`、`docs/*` | 1 号 |

**改别人的文件是严重错误。** 需要别人的功能时，假设它已经存在，直接调用；不要替别人实现。

## 硬规矩

1. **只改我名下文件。** 我要的东西如果必须动别人的文件，先告诉我"这需要改 X 文件"，等我决定。
2. **每次任务结束，列出你改了哪些文件。** 我会核对。
3. **字段名、状态值、接口返回格式，一律查 `docs/contracts.md`，不要自己起名字。**
4. **一次只做一件事。** 做完停下，等我看过再说下一步。
5. **不要"顺手优化"别人的代码，也不要顺手加功能。**
6. **先说明方案，再写代码。** 除非我已经确认过方案。
7. 报错时不要注释掉代码让它不报错。

## 交付纪律

**改完代码，先读 `docs/contracts.md` 的第 0 节，逐条跑那 7 条自检。**
七条全过才允许说"完成"；任何一条不过，回一句"未完成，卡在第 N 条"。

## 输出风格

- 中文注释，简短
- 不要写单元测试（这个项目用手动点检）
- 不要在代码里留 `TODO` 假装做完了
- 做完一句话总结改了什么，不要长篇解释
