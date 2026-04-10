# Codex 工作站交接说明

## 文档作用

这份文档专门记录和 Codex 本机环境相关的上下文，方便你在两台电脑之间切换时，快速恢复工具能力和工作方式。

如果你要继续 GEO 子项目，请优先配合下面这些文档一起看：

1. `README.md`
2. `GEO/README.md`
3. `GEO/docs/2026-04-08-geo-agent-mvp.md`
4. `GEO/docs/setup.md`
5. `talkings/project-overview.md`

## 本机 Codex 能力

### 已安装的增强能力

- 已安装 `Superpowers`
- 主要用于增强规划、调试、验证、头脑风暴等工作流

相关路径：

- 仓库本体：`C:\Users\82442\.codex\superpowers`
- skills 入口：`C:\Users\82442\.agents\skills\superpowers`

### 当前常用 skills

- `writing-plans`
- `systematic-debugging`
- `verification-before-completion`
- `brainstorming`

## 本机规则偏好

### 已写入规则

已写入一条用户规则：

- 生成、修改、补充代码时，默认添加必要的中文注释

规则文件位置：

- `C:\Users\82442\.codex\rules\user-preferences.rules`

## 跨电脑切换时的推荐动作

1. 当前电脑先 `git push`
2. 另一台电脑先 `git pull`
3. 然后检查：
   - `Superpowers` 是否已安装
   - `user-preferences.rules` 是否已存在
   - Python / Git / 其他依赖是否可用
4. 最后再让 Codex 继续执行

## 可直接发给 Codex 的续接提示词

```text
请先阅读：
1. README.md
2. GEO/README.md
3. GEO/docs/2026-04-08-geo-agent-mvp.md
4. GEO/docs/setup.md
5. talkings/project-overview.md
6. talkings/codex/2026-04-11-codex-workstation-handoff.md

这是一个 AI 相关仓库，其中 GEO/ 是 GEO 营销文章智能体子项目。
当前已经完成目录整理、MVP 计划和文档体系搭建，但代码骨架还没完成。
请基于这些文档继续推进，并默认给代码添加必要的中文注释。
```
