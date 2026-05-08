# rmount-ctx — Claude Code 安装助手

本目录是 rmount-ctx 的分发项目。当用户打开此目录并要求安装时，你（Claude）应帮用户完成安装配置。

## 安装流程

1. **检查前置依赖**
   - `rclone` — 必须，用于 SFTP ↔ WebDAV 转换
   - `ssh` — 必须，用户的 SSH config (`~/.ssh/config`) 中应有远程主机别名
   - macOS 原生工具：`mount_webdav`, `diskutil`, `lsof`
   - 所有检查项缺失时，指导用户安装

2. **安装核心脚本**
   - 将 `rmount-ctx` 拷贝到 `~/bin/` 并 `chmod +x`
   - 确保 `~/bin` 在 `PATH` 中

3. **注册 Skill**
   - 将 `SKILL.md` 拷贝到用户的 Claude Code skills 目录
   - 通常为 `~/.claude/skills/rmount-ctx/SKILL.md` 或用户的插件目录

4. **配置 SessionStart Hook**
   - 参考 `hooks/settings.json.hook` 的内容
   - 合并到用户 `~/.claude/settings.json` 的 `hooks.SessionStart` 数组中
   - 如果用户没有该文件，帮助创建
   - **注意：** 保留用户现有的其他 hook 配置，只追加，不覆盖

5. **配置 Zsh chpwd Hook**
   - 参考 `hooks/zshrc.hook` 的内容
   - 追加到用户的 `~/.zshrc` 末尾
   - 执行 `source ~/.zshrc` 使其生效

6. **验证安装**
   - 运行 `~/bin/rmount-ctx list` 确认脚本可用
   - 提醒用户安装完成，使用示例见 README.md

## 注意事项

- 不要修改用户现有的 SSH config
- 不要覆盖用户现有的 settings.json 中非 rmount-ctx 的配置
- 如果用户遇到问题，引导他们查看 README.md 的故障排除部分
- 此项目的完整文档在 README.md 中
