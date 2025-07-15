# AutoSync_fork

在 GitHub 中自动同步已 Fork 的上游项目，可以通过以下两种主流方法实现：

---

### 方法一：使用 GitHub Actions（推荐）
**步骤：**
1. **在 Fork 的仓库中创建 Workflow 文件**  
   进入你的 Fork 仓库 → 点击 `Actions` → `New workflow` → `set up a workflow yourself`  
   创建文件（如 `.github/workflows/sync.yml`）并粘贴以下内容：

```yaml
name: Sync Upstream
on:
  schedule:
    # 每天 UTC 时间 0 点同步（根据 cron 语法调整）
    - cron: '0 0 * * *'
  workflow_dispatch: # 允许手动触发

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Sync Fork
        uses: repo-sync/github-sync@v2
        with:
          source_repo: "upstream_owner/upstream_repo"  # 上游仓库（如：torvalds/linux）
          source_branch: "main"                        # 上游分支
          destination_branch: "main"                   # 你的分支
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

2. **保存并启用 Workflow**  
   文件提交后，Actions 会按计划自动运行（每日同步），也可在 `Actions` 标签页手动触发。

---

### 方法二：命令行本地同步（手动/脚本化）
**步骤：**
1. **克隆你的 Fork 仓库到本地**：
   ```bash
   git clone https://github.com/your_username/your_fork.git
   cd your_fork
   ```

2. **添加上游仓库地址**：
   ```bash
   git remote add upstream https://github.com/upstream_owner/upstream_repo.git
   ```

3. **拉取上游更新并合并**：
   ```bash
   git fetch upstream
   git checkout main  # 切换到你的分支
   git merge upstream/main  # 合并上游分支
   git push origin main     # 推送到你的 Fork
   ```

4. **自动化脚本**（保存为 `.sh` 文件定时执行）：
   ```bash
   #!/bin/bash
   cd /path/to/your_fork
   git fetch upstream
   git checkout main
   git merge upstream/main
   git push origin main
   ```

---

### 关键注意事项
1. **权限要求**：
   - GitHub Actions 需要仓库的 `write` 权限（默认使用 `secrets.GITHUB_TOKEN` 已满足）。
   - 如果上游是私有仓库，需在 Secrets 中配置访问令牌（[创建 PAT](https://github.com/settings/tokens)）。

2. **冲突处理**：
   - 如果本地有未同步的修改，合并时可能产生冲突，需手动解决（Actions 会标记失败）。

3. **频率调整**：
   - 修改 `cron` 表达式调整同步频率（如 `0 */6 * * *` 每 6 小时一次）。

4. **分支名称**：
   - 将代码中的 `main` 替换为实际分支名（如 `master`）。

---

### 效果验证
- **GitHub Actions**：在仓库的 `Actions` 标签页查看运行日志。
- **命令行**：执行后检查 `git log` 是否包含上游提交记录。

通过以上方法，即可实现 Fork 仓库与上游的自动同步。推荐使用 **GitHub Actions** 实现全自动化。
