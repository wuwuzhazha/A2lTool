# A2LTool Git 操作规范

> **版本**: 1.0 | **最后更新**: 2026-05-22
>
> 适用项目: [wuwuzhazha/A2lTool](https://github.com/wuwuzhazha/A2lTool)
> 上游仓库: [DanielT/a2ltool](https://github.com/DanielT/a2ltool)（`upstream`）

---

## 1. 远程仓库配置

```bash
# 查看当前远程配置
git remote -v

# 预期结果：
# origin    https://github.com/wuwuzhazha/A2lTool.git (fetch)
# origin    https://github.com/wuwuzhazha/A2lTool.git (push)
# upstream  https://github.com/DanielT/a2ltool.git (fetch)
# upstream  https://github.com/DanielT/a2ltool.git (push)
```

---

## 2. 分支策略

```
main ────────────────────────────── 发布分支（生产就绪）
  ↑ 合并发布
develop ──────────────────────────── 开发主线（功能集成分支）
  ↑ 功能完成合并
feature/xxx ────────────────────── 功能分支（临时，用完删除）
```

| 分支 | 基线 | 合并目标 | 作用 |
|------|------|---------|------|
| `main` | — | 无 | 发布分支，仅从 `develop` 合入，不直接提交 |
| `develop` | `main` | `main` | 日常开发主线，所有功能在此或从其拉出的 feature 分支上开发 |
| `feature/xxx` | `develop` | `develop` | 功能分支，多处改动或实验性功能时使用，完成后删除 |

---

## 3. 提交信息规范

### 3.1 格式

```
<type>: <中文简短描述>

<详细说明（可选，换行书写）>

<关联引用（可选）>
```

### 3.2 类型定义

| type | 使用场景 | 示例 |
|------|---------|------|
| `feat` | 新功能 | `feat: 新增 --yaml-example 参数` |
| `fix` | Bug 修复 | `fix: 修正 coeffs_linear 物理值计算偏移` |
| `docs` | 文档变更 | `docs: 更新 README 安装说明` |
| `refactor` | 重构（不改功能、不修 Bug） | `refactor: 重命名 yaml_import 为 yaml` |
| `test` | 测试 | `test: 添加 YAML roundtrip 测试用例` |
| `chore` | 杂项（依赖、构建等） | `chore: 升级 serde_yaml 到 0.9` |
| `style` | 格式调整 | `style: 修复 clippy 警告` |

### 3.3 提交原则

- **原子提交**：一个提交只做一件事
- **描述用中文**：简洁明了
- **cherry-pick 上游提交时**：在正文注明来源
- **不混用**：同一提交中不混合 feat + fix 等不同类型的改动

---

## 4. 日常开发流程

### 4.1 简单改动（≤ 3 个文件，直接在 develop 上）

```bash
# 1. 拉取最新 develop
git checkout develop
git pull origin develop

# 2. 修改代码并提交
git add src/xxx.rs
git commit -m "feat: xxx"

# 3. 推送
git push origin develop
```

### 4.2 复杂功能（使用 feature 分支）

```bash
# 1. 从 develop 拉出功能分支
git checkout develop
git pull origin develop
git checkout -b feature/my-feature develop

# 2. 开发过程中频繁提交
git add src/xxx.rs
git commit -m "feat: 部分实现 xxx"
# ...多次提交...

# 3. 开发完成，合并回 develop
git checkout develop
git pull origin develop              # 确保 develop 是最新的
git merge feature/my-feature --no-ff  # --no-ff 保留分支历史
git branch -d feature/my-feature      # 删除本地功能分支
git push origin develop               # 推送 develop

# 4. 删除远程功能分支
git push origin --delete feature/my-feature
```

### 4.3 实验性功能（可能废弃）

```bash
git checkout -b experiment/new-idea develop
# ...开发...
# 如果决定废弃：
git checkout develop
git branch -D experiment/new-idea     # 大写 -D 强制删除
# 远程分支不推送即可
```

---

## 5. 同步上游更新

上游仓库更新时，根据提交类型决定合并策略。

### 5.1 查看上游更新

```bash
git fetch upstream
git log upstream/master --oneline -15
```

### 5.2 提交类型判断

| 上游提交类型 | 建议 | 理由 |
|:-----------:|:----:|------|
| `fix:` 修复 Bug | ✅ 合并 | Bug 修复对本项目也有益 |
| `chore:` 升级依赖 | ✅ 合并 | 依赖更新通常是改进 |
| `feat:` 新增功能 | ⚠ 选择性合并 | 可能与本项目 YAML 功能冲突 |
| `refactor:` 重构 | ⚠ 审慎评估 | 需检查是否影响已有接口 |
| 修改 `Cargo.toml` 版本/作者 | ❌ 跳过 | 已改为你的专属信息 |
| 修改 `README.md` | ❌ 跳过 | README 已重写 |

### 5.3 全量合并（推荐用于 fix / chore 类更新）

```bash
git fetch upstream
git checkout main
git merge upstream/master            # 合并上游 master 到 main
git push origin main

git checkout develop
git merge main                       # 将更新同步到 develop
git push origin develop
```

### 5.4 选择性合并（cherry-pick 单个提交）

```bash
git fetch upstream
git checkout develop
git cherry-pick <commit-hash>        # 挑选需要的提交
git push origin develop
```

如果需要上游修复应用到早于 `develop` 的版本：

```bash
git checkout main
git cherry-pick <commit-hash>        # 将修复也应用到 main
git push origin main
```

### 5.5 合并冲突处理

```bash
# 合并过程中出现冲突时
git status                            # 查看哪些文件冲突
# 编辑冲突文件，解决冲突标记后：
git add <resolved-file>
git merge --continue                  # 继续合并
# 或
git commit -m "fix: 解决合并冲突"
```

**冲突高危文件清单：**

| 文件 | 注意事项 |
|------|---------|
| `Cargo.toml` | 版本号和作者已改为你的，合并时保留你的版本 |
| `src/main.rs` | 可能与你添加的 `--from-yaml`/`--yaml-example` 冲突 |
| `README.md` | 已完全重写，建议用 `git checkout --ours README.md` 跳过 |
| `src/yaml.rs` | 上游没有此文件，不会冲突 |

---

## 6. 发布流程

```bash
# 1. 确认 develop 已包含所有待发布功能
git checkout develop
git log --oneline

# 2. 合并到 main
git checkout main
git merge develop --no-ff
git push origin main

# 3. 打版本标签
git tag -a v1.0.x -m "版本 v1.0.x"

# 4. 推送标签
git push origin v1.0.x
```

### 版本号规则

| 版本变动 | 规则 | 示例 |
|---------|------|------|
| Bug 修复 | 递增第三位 | v1.0.0 → v1.0.1 |
| 新功能（向后兼容） | 递增第二位 | v1.0.0 → v1.1.0 |
| 大版本重构（不兼容） | 递增第一位 | v1.0.0 → v2.0.0 |

---

## 7. 常见操作速查

### 撤销与恢复

```bash
# 撤销未推送的提交，保留改动在工作区
git reset --soft HEAD~1

# 撤销未推送的提交，丢弃改动
git reset --hard HEAD~1

# 补充文件到上一次提交
git add forgotten-file.txt
git commit --amend --no-edit

# 撤销已推送的提交（生成反向提交）
git revert <commit-hash>
git push origin develop
```

### 暂存工作

```bash
# 开发到一半需要切换分支
git stash                           # 暂存
git stash list                      # 查看暂存列表
git stash pop                       # 恢复并删除暂存
git stash drop                      # 删除最近暂存
```

### 分支管理

```bash
# 查看所有分支（含远程）
git branch -a

# 查看已合并的分支（可安全删除）
git branch --merged

# 批量清理本地已合并的分支（保留 main 和 develop）
git branch --merged | findstr -v "\* main develop" | % { git branch -d $_.trim() }

# 重命名分支
git branch -m old-name new-name
```

### 恢复误删

```bash
# 恢复误删的文件
git restore file.txt                 # 未提交的删除
git checkout <commit-hash> -- file.txt  # 已提交的删除

# 恢复误删的分支
git reflog                           # 找到最后一次提交 hash
git checkout -b recovered-branch <commit-hash>
```

### 仓库清理

```bash
# 清理未追踪文件（--dry-run 预览）
git clean -n                         # 预览
git clean -fd                        # 删除未追踪的文件和目录

# 瘦身仓库（极少使用）
git gc --aggressive --prune=now
```

---

## 8. 最佳实践要点

| # | 原则 | 说明 |
|---|------|------|
| 1 | **频繁提交** | 小提交比大提交更容易管理、审查和回滚 |
| 2 | **提交前检查** | 用 `git status` 和 `git diff --staged` 确认改动 |
| 3 | **推送前拉取** | 用 `git pull origin develop --rebase` 避免多余 merge 提交 |
| 4 | **不重写公共历史** | 已推送的分支不要用 rebase 或 reset --hard |
| 5 | **推送用 --force-with-lease** | 远比 `--force` 安全，只在必要时用于个人分支 |
| 6 | **功能分支用完即删** | 避免残留分支堆积 |
| 7 | **上游同步区分类型** | fix/chore 合并，其他选择性处理 |
| 8 | **描述性提交信息** | 让别人（包括未来的你）一眼看懂改动意图 |

---

> **文档版本**: 1.0 | **配套规范**: Git Flow 轻量化变体（main + develop）
