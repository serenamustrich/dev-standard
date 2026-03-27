# Git工作流规范

**文档版本**：v2.0
**创建日期**：2026-03-27
**作者**：技术总监

---

## 1. 分支管理策略

### 1.1 主要分支

| 分支 | 用途 | 保护规则 |
|------|------|----------|
| `main` | 生产环境代码，只接受合并 | 必须通过PR，禁止强制推送 |
| `test` | 测试环境代码，所有开发在此测试 | 必须通过PR，禁止强制推送 |

### 1.2 分支命名规范

```
# 主分支（固定）
main      - 生产环境
test     - 测试环境

# SubAgent开发分支
sub/<Agent类型>/<任务描述>
示例：
sub/dev/fix-login-bug
sub/dev/add-wms-feature
sub/qa/test-requirement-module

# 临时分支（用完即删）
tmp/<开发者>/<日期>
示例：
tmp/agent/20260327
```

---

## 2. SubAgent开发流程

### 2.1 标准流程

```
用户需求 → 主Agent分解任务 → SubAgent创建小分支 → 开发测试 → 合并到test → 等待指令 → 合并到main
```

### 2.2 详细步骤

#### 步骤1：创建SubAgent开发分支

```bash
# 1. 确保在test分支最新状态
git checkout test
git pull origin test

# 2. 创建SubAgent专属分支
git checkout -b sub/<类型>/<任务描述>

# 示例
git checkout -b sub/dev/add-wms-manual-feature
```

#### 步骤2：在小分支开发

```bash
# 开发代码...
git add .
git commit -m "feat: 详细描述"

# 多次提交都可以在小分支
git push origin sub/dev/add-wms-manual-feature
```

#### 步骤3：合并到test分支

```bash
# 1. 确保小分支是最新的
git checkout test
git pull origin test

# 2. 合并小分支到test
git merge sub/<类型>/<任务描述>

# 3. 推送到远程test分支
git push origin test

# 4. 删除小分支（可选）
git branch -d sub/<类型>/<任务描述>
```

#### 步骤4：等待用户指令

**⚠️ 关键规则：没有用户指令，禁止合并到main！**

```
SubAgent完成开发 → 合并到test → 测试验证 → 等待用户指令 → 合并到main
```

#### 步骤5：合并到main分支

**必须获得用户明确指令后执行！**

```bash
# 1. 确保test分支最新
git checkout test
git pull origin test

# 2. 切换到main分支
git checkout main
git pull origin main

# 3. 合并test到main
git merge test

# 4. 自动更新版本号（见第3节）
mvn versions:set -DnewVersion=X.X.X
# 或手动更新pom.xml和application.yml

# 5. 提交版本更新
git add .
git commit -m "chore: 升级版本号到 X.X.X"

# 6. 推送到远程main
git push origin main
```

---

## 3. 版本号管理

### 3.1 版本号格式

```
主版本号.次版本号.修订号
<major>.<minor>.<patch>

示例：
2.4.0 → 2.4.1 → 2.4.2
2.4.9 → 2.5.0 (次版本号递增)
2.9.9 → 3.0.0 (主版本号递增)
```

### 3.2 自动版本号更新规则

**⚠️ 强制规则：每次合并到main时，版本号必须+0.0.1**

需要更新的文件：

| 文件 | 更新位置 | 规则 |
|------|----------|------|
| `pom.xml` | `<version>` | +0.0.1 |
| `application.yml` | `app.version` | +0.0.1 |

### 3.3 版本号递增脚本

```bash
#!/bin/bash
# 合并到main后执行

# 获取当前版本号
VERSION=$(grep -oP '<version>\K[^<]+' pom.xml)

# 分割版本号
IFS='.' read -ra VER <<< "$VERSION"
MAJOR=${VER[0]}
MINOR=${VER[1]}
PATCH=${VER[2]}

# 递增修订号
NEW_PATCH=$((PATCH + 1))
NEW_VERSION="${MAJOR}.${MINOR}.${NEW_PATCH}"

# 更新pom.xml
sed -i '' "s/<version>${VERSION}/<version>${NEW_VERSION}/g" pom.xml

# 更新application.yml
sed -i '' "s/version: ${VERSION}/version: ${NEW_VERSION}/g" application.yml

# 提交
git add .
git commit -m "chore: 升级版本号到 ${NEW_VERSION}"
```

---

## 4. 强制规则

### 4.1 上传规则

**⚠️ 关键规则：没有用户允许，禁止上传代码到远程分支！**

**允许的操作**：
- ✅ 在本地分支开发
- ✅ 在本地提交
- ✅ 提交给用户审查

**禁止的操作**：
- ❌ `git push` 到远程（除非获得明确允许）
- ❌ `git push origin test`（除非获得明确允许）
- ❌ `git push origin main`（除非获得明确允许）

### 4.2 合并规则

**⚠️ 关键规则：没有用户指令，禁止合并到main！**

```
test → main: 必须获得用户指令
sub → test: SubAgent可自行合并（经过测试）
```

### 4.3 代码审查规则

每次合并前需要：
- [ ] 代码已完成自测
- [ ] 没有遗留调试代码
- [ ] 版本号已更新（如需要）
- [ ] 提交信息符合规范

---

## 5. 常见问题

### Q: SubAgent可以直接push到远程吗？

**禁止！** 除非获得用户明确允许。

### Q: 合并到test需要用户批准吗？

不需要。SubAgent完成开发测试后可以自行合并到test，但必须经过测试。

### Q: 如何请求合并到main？

在需求管理系统中创建评论或请求，说明：
- 需求编号
- 完成的功能
- 测试情况
- 请求合并到main

### Q: 忘记更新版本号怎么办？

```bash
# 查看当前版本
git log --oneline main -1

# 查看test分支版本
git log --oneline test -1

# 如果发现版本号未更新，在main分支补上
git checkout main
git cherry-pick <版本更新commit>
```

---

## 6. 分支操作速查

```bash
# 创建SubAgent分支
git checkout -b sub/dev/任务名称

# 合并到test（SubAgent自行执行）
git checkout test
git merge sub/dev/任务名称
git push origin test

# 合并到main（需要用户指令）
git checkout main
git merge test
# 自动更新版本号
git push origin main

# 删除本地分支
git branch -d sub/dev/任务名称

# 查看所有分支
git branch -a
```

---

**最后更新**：2026-03-27
