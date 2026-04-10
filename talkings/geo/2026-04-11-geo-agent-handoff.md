# GEO 子项目交接说明

## 子项目定位

`GEO/` 是当前仓库下的一个 AI 相关子项目，目标是做一个：

- 根据关键词、产品上下文和知识库内容
- 自动生成 GEO 营销文章
- 并输出审核建议

## 当前已完成内容

### 文档与目录

- 已把 GEO 相关材料整理进 `GEO/`
- 已新增 `GEO/README.md`
- 已保存 MVP 计划：
  - `GEO/docs/2026-04-08-geo-agent-mvp.md`
- 已新增启动说明：
  - `GEO/docs/setup.md`

### 方案层共识

当前已经确认：

- 先做 API-first MVP，不先做前端
- 先验证“输入 -> 理解 -> 规划 -> 取证 -> 生成 -> 审核”这条链路
- 优先支持问题型、场景型、参数特征型内容
- 品牌/竞品型内容放到后续版本

## 当前未完成内容

GEO 的实际代码骨架还没开始落地，尚未创建：

- `pyproject.toml`
- `src/geo_agent/...`
- `tests/...`
- `storage/knowledge/...`

## 下一个最合理动作

下一步建议直接做：

1. 在 `GEO/` 下初始化 Python 项目骨架
2. 搭 FastAPI 入口
3. 加入最小的 jobs API
4. 接入本地知识文件读取
5. 用 mock 方式先跑通整条流水线

## 恢复时推荐阅读

1. `GEO/README.md`
2. `GEO/docs/setup.md`
3. `GEO/docs/2026-04-08-geo-agent-mvp.md`
