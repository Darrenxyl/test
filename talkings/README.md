# Talkings

这个目录专门放“对话记录、交接文档、阶段总结、恢复说明”。

## 目录结构

- `project-overview.md`
  仓库级总览，切换电脑或新开窗口时建议先读。
- `geo/`
  GEO 子项目相关的 handoff 和阶段记录。
- `git/`
  分支同步、仓库整理、Git 状态相关记录。
- `codex/`
  Codex 工作站、规则、skills、环境说明。
- `product/`
  早期产品讨论或历史会话纪要。
- `templates/`
  以后新增 handoff 文档时可直接复用的模板。

## 推荐阅读顺序

### 如果你要恢复整个仓库上下文

1. `README.md`
2. `talkings/project-overview.md`
3. 按主题进入对应子目录继续看

### 如果你要继续 GEO 子项目

1. `GEO/README.md`
2. `GEO/docs/setup.md`
3. `GEO/docs/2026-04-08-geo-agent-mvp.md`
4. `talkings/geo/2026-04-11-geo-agent-handoff.md`
5. `talkings/codex/2026-04-11-codex-workstation-handoff.md`

## 维护规则

- 一次重要对话或一个独立主题，尽量生成一份独立 handoff 文档
- 文件名统一用：`YYYY-MM-DD-主题-handoff.md`
- 如果内容偏“长期稳定信息”，不要放在 handoff 里，放到 `project-overview.md` 或对应的 setup 文档里
