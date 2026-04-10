# GEO 子项目启动说明

## 子项目目标

在 `GEO/` 目录下落地一个 GEO 营销文章智能体 MVP。

第一版重点验证：

- 关键词输入
- 关键词理解
- 内容规划
- 知识库取证
- 文章生成
- 基础审核

## 当前阶段

当前还处于“文档规划完成、代码骨架未开始”的阶段。

已存在文档：

- `GEO/README.md`
- `GEO/docs/2026-04-08-geo-agent-mvp.md`
- `talkings/geo/2026-04-11-geo-agent-handoff.md`

## 计划技术路线

- Python
- FastAPI
- SQLite
- 本地知识文件
- 单任务文章生成 API

## 推荐启动顺序

1. 初始化 `pyproject.toml`
2. 创建 `src/geo_agent/`
3. 创建 `tests/`
4. 先实现最小 jobs API
5. 再实现理解、规划、取证、生成、审核服务

## 当前不做的内容

- 前端 UI
- 多用户系统
- 定时任务
- 自动发布
- 复杂 analytics

## 继续开发前建议先读

1. `GEO/docs/2026-04-08-geo-agent-mvp.md`
2. `talkings/project-overview.md`
3. `talkings/geo/2026-04-11-geo-agent-handoff.md`
