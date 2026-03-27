# Git工作流规范

**文档版本**：v1.0
**创建日期**：2026-03-26
**作者**：开发团队

---

## 1. 分支管理

### 1.1 分支类型

| 分支类型 | 分支名称 | 说明 | 保护规则 |
|---------|---------|------|---------|
| 主分支 | `main` | 生产环境代码，只接受合并 | 必须通过PR，禁止强制推送 |
| 测试分支 | `test` | 开发测试分支，所有开发在此进行 | 必须通过PR，禁止强制推送 |
| 功能分支 | `feature/功能名称` | 新功能开发 | 完成后合并到test |
| 修复分支 | `fix/问题描述` | Bug修复 | 完成后合并到test |

### 1.2 分支命名规范

```
# 功能分支
feature/wms-manual-flowchart
feature/container-management

# 修复分支
fix/login-redirect-issue
fix/table-sort-bug

# 发布分支（如果需要）
release/v2.4.0
```

---

## 2. 开发流程

### 2.1 标准开发流程

```
1. 从test分支创建功能分支
   git checkout test
   git pull origin test
   git checkout -b feature/功能名称

2. 在功能分支上开发
   - 编写代码
   - 运行测试
   - 提交代码

3. 推送到远程
   git push origin feature/功能名称

4. 创建Pull Request
   - Source: feature/功能名称
   - Target: test

5. 代码评审后合并到test

6. 测试通过后，合并到main
   git checkout test
   git pull origin test
   git checkout main
   git pull origin main
   git merge test
   git push origin main

7. 删除功能分支（可选）
   git branch -d feature/功能名称
```

### 2.2 紧急修复流程

```
1. 从main创建修复分支
   git checkout main
   git pull origin main
   git checkout -b hotfix/问题描述

2. 修复并测试

3. 合并到main
   git checkout main
   git merge hotfix/问题描述
   git push origin main

4. 合并到test（保持同步）
   git checkout test
   git merge hotfix/问题描述
   git push origin test

5. 删除修复分支
   git branch -d hotfix/问题描述
```

---

## 3. 禁止规则

### ⚠️ 重要禁止事项

1. **禁止直接在main分支开发**
   - 违反者将被要求重新在test分支开发
   - 除非是紧急 hotfix，且必须事后补全流程

2. **禁止强制推送main和test分支**
   ```bash
   # 禁止执行
   git push -f origin main
   git push -f origin test
   ```

3. **禁止未测试的代码合并**
   - 所有合并必须经过测试
   - 提交前必须运行 lint 和测试

---

## 4. 提交规范

### 4.1 提交信息格式

```
<类型>(<范围>): <简短描述>

[可选的详细描述]

[可选的关联需求]
```

### 4.2 示例

```
feat(wms-manual): 添加mermaid流程图支持

- 使用mermaid渲染18个流程图
- 修复{#section}显示问题
- 优化目录导航样式

关联需求: PRD-2026-002
```

### 4.3 类型说明

| 类型 | 说明 |
|------|------|
| `feat` | 新功能 |
| `fix` | Bug修复 |
| `docs` | 文档变更 |
| `style` | 代码格式（不影响功能） |
| `refactor` | 重构（不影响功能） |
| `perf` | 性能优化 |
| `test` | 测试相关 |
| `chore` | 构建/工具变更 |

---

## 5. 代码评审

### 5.1 评审要求

- 所有合并到test或main的代码必须经过评审
- 评审者需检查：
  - 代码质量和风格
  - 功能完整性
  - 是否有测试覆盖
  - 是否符合项目规范

### 5.2 评审通过标准

- 至少1人批准
- 无阻塞性评论
- CI/CD测试全部通过

---

## 6. 版本标签

### 6.1 标签命名

```
v<主版本号>.<次版本号>.<修订号>

示例：
v2.4.0
v2.4.1
v3.0.0
```

### 6.2 发布流程

```bash
# 1. 确保main分支是要发布的版本
git checkout main

# 2. 创建标签
git tag -a v2.4.0 -m "版本2.4.0发布"

# 3. 推送标签
git push origin v2.4.0
```

---

## 7. 常见问题

### Q: 误在main分支开发了怎么办？

A: 将更改 cherry-pick 到 test 分支，然后 reset main。
```bash
# 保存更改
git checkout main
git log --oneline -1  # 记录commit ID
git checkout -b recovery
git cherry-pick <commit ID>

# 推送到test
git push origin recovery:test

# 恢复main
git checkout main
git reset --hard origin/main
```

### Q: 如何撤销上一次的提交？

```bash
# 撤销提交但保留更改
git reset --soft HEAD~1

# 撤销提交并删除更改
git reset --hard HEAD~1
```

---

**最后更新**：2026-03-26
