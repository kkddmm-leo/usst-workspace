# 🎓 USST Study Team

Welcome to our research collaboration workspace!

本仓库用于团队成员之间的**学术资源共享、研究想法交流及个人工作记录**，希望通过持续的知识积累与开放交流，提高团队的科研效率，促进共同进步。

## 📂 Repository Structure

```text
usst-workspace/
│
├── README.md
│
├── Paper Repository/           # 论文共享库
│   └── README.md
│
├── Sharing Ideas/              # 研究想法共享区
│   └── README.md
│
├── lzy/                        # 个人工作空间
├── ry/
└── wh/
```

## 📚 Paper Repository

**论文共享库**

用于收集和分享值得阅读的学术论文，包括相关领域的最新研究成果、优秀研究方法以及具有参考价值的文献。

- 所有成员均可分享论文。
- 论文采用统一命名格式，注明分享者。
- 新增论文后，更新目录中的 README 索引。
- 鼓励分享论文的核心贡献及推荐理由。

**文件命名规范：**

`年份-期刊简写_论文简称_分享者.pdf`

例如：

`2025-TRO_UAVRouting_lzy.pdf`

📖 详细说明：[Paper Repository](./Paper%20Repository/README.md)

## 💡 Sharing Ideas

**研究想法共享区**

用于记录、分享和讨论团队成员的研究想法。

无论是初步研究构思、算法改进方案、论文阅读启发，还是新的实验设计，都欢迎在这里分享。

- 每个 Idea 使用独立的 Markdown 文件。
- 文件名注明提出人的姓名缩写。
- 在 README 中维护 Idea 索引。
- 鼓励成员相互讨论、补充和完善想法。

**文件命名规范：**

`IDEA-编号-想法名称-提出人.md`

例如：

`IDEA-001-公交辅助无人机配送-lzy.md`

💡 详细说明：[Sharing Ideas](./Sharing%20Ideas/README.md)

## 👤 Personal Workspace

**个人工作空间**

每位成员拥有独立的文件夹，用于记录个人学习与科研工作。

| Member | Workspace |
|---|---|
| lzy | [lzy](./lzy/) |
| ry | [ry](./ry/) |
| wh | [wh](./wh/) |
| zjq | [zjq](./zjq/) |
| wmx | [wmx](./wmx/) |
| phw | [phw](./phw/) |

个人文件夹可用于存放：

- 每周工作记录与总结
- 科研进展与阶段性成果
- 学习笔记与技术文档
- 实验记录与后续计划

成员主要维护各自的文件夹，文件组织方式可根据个人习惯灵活调整。

## 🤝 Collaboration Guidelines

为了方便团队协作，请遵循以下约定：

**1. 资源共享**

欢迎分享高质量论文和有价值的研究想法。共享内容应尽量按照统一规范命名，并更新对应的 README 索引。

**2. 个人记录**

每位成员在自己的文件夹中维护工作记录，尽量避免随意修改其他成员的个人文件。

**3. Git 协作**

- 修改前先拉取远程仓库的最新内容。
- 完成修改后，提交并推送到 GitHub。
- 提交信息尽量简洁、明确。
- 修改其他成员的内容时，建议提前沟通。
- 遇到文件冲突时，先确认修改内容，再进行合并。

**4. 开放交流**

鼓励团队成员积极分享研究思路、提出问题、交流经验，共同维护和完善本仓库。

## 🚀 Quick Start

首次使用时，克隆仓库：

```bash
git clone git@github.com:kkddmm-leo/usst-workspace.git
```

进入仓库：

```bash
cd usst-workspace
```

日常更新：

```bash
# 获取最新内容
git pull origin main

# 添加修改
git add .

# 提交修改
git commit -m "Update research notes"

# 推送到远程仓库
git push origin main
```

> 使用 SSH 地址前，需要先在 GitHub 中配置 SSH Key。请根据实际使用的分支调整命令。

---

**Share Knowledge · Exchange Ideas · Grow Together**