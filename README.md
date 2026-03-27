# 开发规范仓库

**⚠️ 重要提示：本仓库已迁移到需求管理系统**

---

## 📍 新位置

所有开发规范现已统一管理在需求管理系统中：

**需求管理系统路径**：`05-工作流规范/开发规范/`

包含以下文档：
- [AI开发工作流规范.md](https://github.com/serenamustrich/requirement-management-system/blob/main/05-工作流规范/开发规范/AI开发工作流规范.md)
- [Git工作流.md](https://github.com/serenamustrich/requirement-management-system/blob/main/05-工作流规范/开发规范/Git工作流.md)
- [开发规范.md](https://github.com/serenamustrich/requirement-management-system/blob/main/05-工作流规范/开发规范/开发规范.md)
- [需求管理规范.md](https://github.com/serenamustrich/requirement-management-system/blob/main/05-工作流规范/开发规范/需求管理规范.md)
- [merge_workflow.md](https://github.com/serenamustrich/requirement-management-system/blob/main/05-工作流规范/开发规范/merge_workflow.md)

---

## 📋 规范目录结构

```
需求管理系统/
└── 05-工作流规范/
    ├── 开发规范/           ← 全局开发规范
    │   ├── AI开发工作流规范.md
    │   ├── Git工作流.md
    │   ├── 开发规范.md
    │   ├── 需求管理规范.md
    │   └── merge_workflow.md
    ├── 工作流执行手册.md
    └── 全局工作流规范.md
```

---

## 🚀 快速开始

### 新项目开发流程

```
1. 在需求管理系统创建需求 → 需求管理规范.md
2. 从test分支开发 → Git工作流.md
3. 提交代码 → 开发规范.md
4. 测试通过后申请合并 → merge_workflow.md
```

### 分支策略

```
main    → 生产环境（只接受合并）
test    → 测试环境（所有开发在此进行）
feature/* → 功能分支
fix/*   → 修复分支
```

---

## 📝 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| v1.0 | 2026-03-26 | 初始版本 |
| v2.0 | 2026-03-27 | 迁移到需求管理系统统一管理 |

---

**最后更新**：2026-03-27
