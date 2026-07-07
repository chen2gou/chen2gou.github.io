---
title: OpenSpec、Superpower 与 Claude Code 协作开发模式说明
date: 2026-07-07 16:50:00 +0800
tags: AI ClaudeCode OpenSpec 开发流程
---
## 1. 目标

本文档用于说明当前项目推荐的开发协作模式，重点明确：

- OpenSpec 的作用和边界
- Superpower 的作用和边界
- Claude Code 在开发过程中的配合方式
- 以本次“动态数据权限与按钮控制”需求为例，说明完整流程

核心原则：

- OpenSpec 负责“需求和规格是什么”
- Superpower 负责“如何拆解和执行开发任务”
- Claude Code 负责“理解上下文、生成方案、修改代码、运行验证”
- Java 项目代码和正式文档进入 git
- AI 临时过程文件不进入 git

---

## 2. OpenSpec 的作用和边界

### 2.1 OpenSpec 是什么

OpenSpec 用于维护项目的正式规格说明，描述需求目标、行为约束、数据模型、接口契约和验收标准。

它不是临时计划，也不是聊天记录，而是可以长期维护、参与评审、随代码一起演进的项目资产。

项目地址：

| 项目 | Git 地址 | 说明 |
| --- | --- | --- |
| OpenSpec | https://github.com/Fission-AI/OpenSpec | 当前项目使用的 `openspec` CLI 来源，用于维护 spec-driven development 的 proposal、design、tasks 和 specs。 |

### 2.2 OpenSpec 适合记录什么

OpenSpec 适合记录以下内容：

- 需求背景
- 业务目标
- 功能范围
- 核心规则
- 权限模型
- 状态流转
- 数据结构约束
- 接口行为约束
- 验收标准
- 与旧逻辑的兼容关系

以本次需求为例，OpenSpec 可以记录：

- 需要支持不同实体的动态按钮权限
- 按钮权限由规则表 `sys_action_rule` 驱动
- 权限结果需要根据当前用户和实体关系人动态计算
- `sys_action_rule` 只存静态规则，不缓存最终按钮权限结果
- Story 列表和详情页需要接入按钮权限
- 评论使用通用主体 `COMMENT`
- Story 的 `关闭`、`重新打开` 在 SQL 中保持通用动作名，页面展示文案由前端按场景处理

### 2.3 OpenSpec 不适合记录什么

OpenSpec 不适合记录以下内容：

- AI 临时执行计划
- 每一步具体改哪个文件
- 中间调试过程
- 临时对话结论
- 一次性草稿
- 未确认的实现细节

这些内容更适合放在 Superpower 的 plan 或 Claude Code 对话中。

### 2.4 OpenSpec 是否进入 git

建议进入 git。

原因：

- OpenSpec 是正式规格，属于项目资产
- 需求变更可以随代码一起追踪
- Code Review 时可以同时审查规格和实现
- 后续维护人员可以通过规格理解代码设计原因

推荐路径：

```text
openspec/
  changes/
    add-dynamic-entity-action-permissions/
      proposal.md
      tasks.md
      specs/
        dynamic-action-permissions/
          spec.md
```

或者后续也可以统一收敛到：

```text
docs/openspec/
```

当前项目已经使用 `openspec/`，因此可以继续保留该目录并纳入 git。

---

## 3. Superpower 的作用和边界

### 3.1 Superpower 是什么

Superpower 是 Claude Code 的开发流程辅助工具，主要用于把已经明确的需求拆解成可执行的开发计划，并约束执行过程。

它关注的是“如何开发”，而不是“需求本身是什么”。

项目地址：

| 项目 | Git 地址 | 说明 |
| --- | --- | --- |
| Superpowers | https://github.com/obra/superpowers | Claude Code 等 coding agent 的技能与开发方法论插件，覆盖 brainstorming、TDD、debugging、plan execution、code review 等流程。 |

### 3.2 Superpower 适合做什么

Superpower 适合用于：

- 需求澄清后的方案设计
- 编写实现计划
- 拆分开发任务
- 强制 TDD 流程
- 执行计划
- 代码评审
- 验证完成情况

以本次需求为例，Superpower plan 可以拆成：

1. 新增按钮权限枚举和上下文模型
2. 新增 `sys_action_rule` 表和初始化 SQL
3. 新增按钮权限计算工具
4. Story 列表接入按钮权限
5. Story 详情接入按钮权限
6. 新增 Redis 缓存服务
7. 调整 `checkAllowed()` 走缓存
8. 补充单元测试

### 3.3 Superpower 不适合做什么

Superpower 不适合作为正式项目文档长期维护。

原因：

- 它偏向执行过程，而不是业务规格
- 内容可能包含阶段性计划、临时结论、调试记录
- 会随着 AI 执行方式变化而变化
- 不一定适合人工长期阅读和评审

因此，Superpower plan 建议作为本地临时文件，不进入 git。

推荐路径：

```text
.superpower/
  plans/
    2026-07-01-action-rule-redis-cache.md
    2026-07-01-story-action-permissions.md
```

`.gitignore` 中应包含：

```text
.superpower/
.superpowers/
.claude/
```

---

## 4. OpenSpec 和 Superpower 的边界

二者可以这样区分：

| 项目 | OpenSpec | Superpower |
| --- | --- | --- |
| 核心问题 | 要做什么 | 怎么做 |
| 面向对象 | 产品、研发、评审、维护人员 | Claude Code、开发执行者 |
| 生命周期 | 长期维护 | 阶段性使用 |
| 是否进 git | 建议进入 | 不建议进入 |
| 内容类型 | 规格、规则、验收标准 | 实施步骤、任务拆解、执行记录 |
| 变更方式 | 随需求变更更新 | 随开发过程生成和调整 |
| 示例 | 按钮权限规则定义 | 第一步改枚举，第二步写测试 |

简单理解：

```text
产品文档 / 需求讨论
        ↓
OpenSpec：沉淀正式规格
        ↓
Superpower：拆成可执行任务
        ↓
Claude Code：按任务修改代码和验证
        ↓
代码、测试、OpenSpec 一起进入 git
```

---

## 5. 需求探讨阶段怎么选择工具

### 5.1 需求不明确时怎么做

需求不明确时，优先直接和 Claude Code 交互探讨。

例如：

```text
这个按钮权限需求，状态机和动态权限怎么分边界？
```

```text
sys_action_rule 和 sys_action_transition 是否都需要？帮我分析下。
```

这类问题适合直接和 Claude Code 聊，因为 Claude Code 可以：

- 读取现有代码
- 读取产品文档
- 对比当前实现
- 给出多个方案
- 逐步收敛设计

这一步不一定马上落文档。只有形成稳定结论后，才需要沉淀到正式规格或 `docs/` 中。

### 5.2 什么时候使用 `/opsx:explore`

`/opsx:explore` 也可以用于探讨问题，但它更适合做“围绕 OpenSpec 上下文的需求探索和规格分析”。

它不是最终规格文档本身，而是进入 OpenSpec 工作流前后的探索入口，适合在以下场景使用：

- 已经准备使用 OpenSpec 管理这个需求
- 需要围绕现有变更来讨论方案
- 需要检查某个需求是否已经被已有规格覆盖
- 需要分析新需求会影响哪些 capability、spec 或 change
- 需要把模糊需求逐步收敛到可写入规格的程度

简单理解：

```text
直接 Claude Code 交互：自由讨论，最快
/opsx:explore：结合 OpenSpec 上下文做需求探索
Superpower brainstorming：完整设计流程，最重
```

`/opsx:explore` 适合用来回答：

```text
这个需求应该落到哪个 change？
```

```text
当前规格是否已经覆盖这个按钮权限规则？
```

```text
这个需求会影响哪些 spec？
```

```text
这个变更应该拆成一个 change，还是多个 change？
```

它可以参与需求探讨，但讨论重点更偏 OpenSpec 体系内的规格分析，而不是完整产品设计流程。

### 5.3 什么时候使用 Superpower brainstorming

当需求较大、边界不清、可能影响多个模块时，可以使用 Superpower 的 `brainstorming`。

适合场景：

- 设计一个动态权限体系
- 设计通用状态机体系
- 状态流转、按钮权限、数据权限需要分层
- 多个模块职责边界不清
- 需要拆分一期、二期或多个子项目

推荐流程：

```text
Superpower brainstorming
  ↓
输出设计文档
  ↓
正式规格沉淀
  ↓
Superpower writing-plans 拆任务
  ↓
Claude Code 实现和验证
```

### 5.4 三者在需求探讨阶段的分工

| 能力 | 直接 Claude Code 交互 | `/opsx:explore` | Superpower brainstorming |
| --- | --- | --- | --- |
| 需求发散 | 强 | 中 | 强 |
| 多方案比较 | 强 | 中 | 强 |
| 结合现有代码分析 | 强 | 中 | 强 |
| 结合现有规格分析 | 中 | 强 | 中 |
| 强制澄清问题 | 中 | 中 | 强 |
| 设计流程约束 | 弱 | 中 | 强 |
| 规格沉淀前准备 | 弱 | 强 | 中 |
| 流程成本 | 低 | 中 | 高 |
| 适合场景 | 小问题和快速分析 | OpenSpec 相关需求探索 | 大中型需求设计 |

更具体地说：

| 场景 | 推荐工具 |
| --- | --- |
| 想快速讨论一个技术或产品问题 | 直接 Claude Code 交互 |
| 想确认一个需求在规格体系里怎么落位 | `/opsx:explore` |
| 想检查现有 spec 是否覆盖某个需求 | `/opsx:explore` |
| 需求很大、方案很多、边界不清 | Superpower brainstorming |
| 已经明确要实现，需要拆任务 | Superpower writing-plans |

一句话：

```text
Claude Code 用来快速想，/opsx:explore 用来结合规格想，brainstorming 用来系统想，Superpower plan 用来做。
```

### 5.5 OpenSpec 与 Superpower 常用技能/命令

OpenSpec 在当前项目中主要通过 `/opsx:*` 命令使用：

| 命令 | 作用 | 适合场景 | 是否修改业务代码 |
| --- | --- | --- | --- |
| `/opsx:explore` | 进入探索模式，围绕需求、现有代码和 OpenSpec 上下文分析问题 | 需求不清晰、需要讨论方案、需要判断需求落到哪个 spec/change | 否 |
| `/opsx:propose` | 创建一个新的 OpenSpec change，并生成 proposal、design、tasks 等工件 | 需求已基本明确，需要正式沉淀为可评审的规格变更 | 否，主要写规格工件 |
| `/opsx:apply` | 按 OpenSpec change 中的 tasks 执行实现 | 规格和任务已经确认，需要开始编码实现 | 是 |
| `/opsx:sync` | 将 change 中的规格变更同步回正式 specs | 实现完成或需求稳定后，需要把 delta spec 合并到长期规格 | 否，主要改规格文件 |
| `/opsx:archive` | 归档已完成的 change | change 已完成并同步，需要从 active changes 中归档 | 否，主要移动规格工件 |

Superpower 常用 skill 可以按开发阶段理解：

| Skill | 作用 | 适合场景 | 关键约束/产出 |
| --- | --- | --- | --- |
| `using-superpowers` | 启动 Superpowers 的总入口，判断是否需要调用具体 skill | 任意任务开始前 | 有适用 skill 时必须先使用 |
| `brainstorming` | 将模糊想法收敛为设计方案 | 大中型需求、边界不清、方案需要比较 | 用户确认设计前不写代码 |
| `writing-plans` | 把已确认设计拆成可执行实现计划 | 规格或设计已经明确，需要进入开发排期 | 输出具体到文件、步骤、测试命令的 plan |
| `subagent-driven-development` | 用子代理按任务实现、评审、修复 | 已有完整 plan，任务可分阶段执行 | 每个任务后做代码评审，适合较大改动 |
| `executing-plans` | 在当前会话中按 plan 顺序执行 | 已有完整 plan，但不使用子代理模式 | 按任务逐项执行并验证 |
| `test-driven-development` | 强制先写失败测试，再写最小实现 | 新功能、Bug 修复、行为变更 | 没有失败测试前不写生产代码 |
| `systematic-debugging` | 系统化定位根因再修复 | 测试失败、线上 Bug、构建失败、集成问题 | 先找根因，不做猜测式修复 |
| `requesting-code-review` | 请求代码评审 | 完成任务、重要功能、合并前 | 让独立视角检查实现风险 |
| `receiving-code-review` | 处理代码评审反馈 | 收到评审意见后 | 先核实再修改，不盲从也不表演式认同 |
| `verification-before-completion` | 完成前做新鲜验证 | 声称测试通过、构建成功、任务完成前 | 没有最新验证证据就不声明完成 |
| `finishing-a-development-branch` | 完成开发分支收尾 | 所有任务完成后 | 做最终验证、整理交付说明 |
| `using-git-worktrees` | 使用 git worktree 隔离开发 | 明确需要隔离分支或并行工作区时 | 当前项目未明确要求时不主动使用 |
| `dispatching-parallel-agents` | 并行分派多个 agent 做独立工作 | 多个互不冲突的搜索、分析或实现任务 | 需要明确任务边界，避免并行冲突 |
| `writing-skills` | 编写或修改 Superpowers skill | 需要开发新的 skill 或调整既有 skill | 需要评估 skill 行为，不适合普通业务需求 |

---

## 6. Claude Code 的角色

Claude Code 在当前模式中承担执行助手角色，主要负责：

1. 阅读产品文档、OpenSpec 和现有代码
2. 帮助澄清需求和边界
3. 基于 OpenSpec 生成开发方案
4. 使用 Superpower 拆分任务
5. 按 TDD 流程新增或修改代码
6. 运行 Maven 单测验证
7. 根据反馈调整实现
8. 生成提交信息和变更说明

Claude Code 不应直接把一次对话中的临时结论当作正式规格。

重要需求结论应优先沉淀到 OpenSpec 或 `docs/` 下的正式文档中。

---

## 7. 当前需求示例：动态数据权限与按钮控制

### 7.1 原始输入

本次需求来源包括：

```text
docs/ITP优化.md
docs/动态数据权限与按钮控制架构宣讲.md
```

这些文档属于产品和架构输入，描述了业务背景、页面按钮、动态权限和状态流转等内容。

### 7.2 规格沉淀

基于原始文档讨论：

![OpenSpec 需求讨论示意](</img/2026/07/openspec-superpower-claude-code-3.png>)

将需求沉淀到 OpenSpec：

![OpenSpec 规格沉淀示意](</img/2026/07/openspec-superpower-claude-code-2.png>)

```text
openspec/changes/add-dynamic-entity-action-permissions/
  proposal.md
  tasks.md
  specs/dynamic-action-permissions/spec.md
```

OpenSpec 中应明确：

- 支持实体维度的按钮权限规则
- 支持关系人维度的权限判断
- 支持 Story 列表和详情按钮权限
- 权限规则可通过 SQL 初始化
- 权限规则表 `sys_action_rule` 使用 `entity_type + action_code` 做唯一约束
- 按钮名称保持通用，不在同一个规则里区分页面场景
- Story 详情页特殊文案由前端处理

### 7.3 任务拆解

确认规格后，使用 Superpower 生成实现计划。

![Superpower 任务拆解示意](</img/2026/07/openspec-superpower-claude-code-1.png>)

示例计划文件：

```text
.superpower/plans/2026-07-01-story-action-permissions.md
.superpower/plans/2026-07-01-action-rule-redis-cache.md
```

计划关注具体执行，例如：

- 修改哪些 Java 类
- 新增哪些测试
- 先写哪个失败测试
- 再写哪个最小实现
- 执行哪些 Maven 命令验证

### 7.4 编码实现

Claude Code 根据计划修改代码。

本次主要实现包括：

```text
itp-common/src/main/java/com/robotees/itp/enums/EntityTypeEnum.java
itp-common/src/main/java/com/robotees/itp/enums/ActionTypeEnum.java
itp-common/src/main/java/com/robotees/itp/enums/ActionRelationEnum.java
itp-common/src/main/java/com/robotees/itp/util/ActionPermissionUtils.java
itp-common/src/main/java/com/robotees/itp/util/ActionRelationContext.java
itp-common/src/main/java/com/robotees/itp/utils/CacheKeyUtils.java
itp-admin/src/main/java/com/robotees/itp/modules/biz/entity/SysActionRule.java
itp-admin/src/main/java/com/robotees/itp/modules/biz/mapper/SysActionRuleMapper.java
itp-admin/src/main/java/com/robotees/itp/modules/biz/service/ActionPermissionService.java
itp-admin/src/main/java/com/robotees/itp/modules/biz/service/ActionRuleCacheService.java
itp-admin/src/main/java/com/robotees/itp/modules/biz/service/impl/ActionPermissionServiceImpl.java
itp-admin/src/main/java/com/robotees/itp/modules/biz/service/impl/ActionRuleCacheServiceImpl.java
itp-admin/src/main/java/com/robotees/itp/modules/biz/service/impl/StoryServiceImpl.java
itp-admin/src/main/resources/db.migration/V3__add_sys_action_rule.sql
```

### 7.5 测试验证

本次配套测试包括：

```text
itp-common/src/test/java/com/robotees/itp/util/ActionPermissionUtilsTest.java
itp-common/src/test/java/com/robotees/itp/util/ActionRuleSqlTest.java
itp-admin/src/test/java/com/robotees/itp/modules/biz/service/impl/ActionPermissionServiceImplTest.java
itp-admin/src/test/java/com/robotees/itp/modules/biz/service/impl/ActionRuleCacheServiceImplTest.java
itp-admin/src/test/java/com/robotees/itp/modules/biz/service/impl/StoryServiceImplPermissionTest.java
```

推荐使用当前项目指定 Maven 执行测试：

```bash
/usr/local/apache-maven-3.3.9/bin/mvn \
  -s /usr/local/apache-maven-3.3.9/conf/settings.xml \
  -o \
  -f pom.xml \
  -pl itp-admin \
  -am \
  -Dtest=ActionRuleCacheServiceImplTest,ActionPermissionServiceImplTest,StoryServiceImplPermissionTest \
  -DfailIfNoTests=false \
  test
```

common 模块测试可执行：

```bash
/usr/local/apache-maven-3.3.9/bin/mvn \
  -s /usr/local/apache-maven-3.3.9/conf/settings.xml \
  -o \
  -f pom.xml \
  -pl itp-common \
  -Dtest=ActionRuleSqlTest,ActionPermissionUtilsTest \
  -DfailIfNoTests=false \
  test
```

---

## 8. 推荐协作流程

### 8.1 新需求开发流程

推荐流程如下：

```text
1. 收集原始产品文档
2. Claude Code 阅读 docs 和现有代码
3. 用户澄清需求边界
4. 生成或更新 OpenSpec
5. 用户确认 OpenSpec
6. 使用 Superpower 生成实现计划
7. 用户确认实现计划
8. Claude Code 按计划 TDD 实现
9. 运行单测和必要验证
10. 更新正式文档
11. 生成提交信息
12. 提交代码
```

### 8.2 变更处理流程

开发过程中如果发现需求变化，按以下方式处理：

```text
小的实现细节变化：
  直接调整代码和测试，必要时同步更新 Superpower plan

影响行为或验收标准的变化：
  先更新 OpenSpec，再调整代码

影响产品理解的变化：
  先更新 docs/product 或相关说明，再更新 OpenSpec
```

### 8.3 文件入库建议

建议进入 git：

```text
openspec/
itp-common/
itp-admin/
pom.xml
.gitignore
```

不建议进入 git：

```text
.claude/
.superpower/
.superpowers/
```

---

## 9. 当前项目建议目录结构

```text
docs/
  ITP优化.md
  动态数据权限与按钮控制架构宣讲.md
  开发模式说明.md

openspec/
  changes/
    add-dynamic-entity-action-permissions/
      proposal.md
      tasks.md
      specs/
        dynamic-action-permissions/
          spec.md

.superpower/
  plans/
    2026-07-01-action-rule-redis-cache.md
    2026-07-01-story-action-permissions.md
```

说明：

- `docs/` 保存产品文档和项目级说明
- `openspec/` 保存正式规格和变更说明
- `.superpower/` 保存 AI 辅助开发过程文件，不进入 git

---

## 10. 实践原则

### 10.1 先规格，后实现

不要直接从一句需求开始写代码。

应该先明确：

- 业务目标是什么
- 不做什么
- 数据边界是什么
- 谁来消费这个能力
- 验收标准是什么

### 10.2 规格和计划分离

OpenSpec 写“系统应该具备什么行为”。

Superpower 写“为了实现这个行为，具体怎么改代码”。

不要把实现步骤写进 OpenSpec，也不要把正式业务规则只留在 Superpower plan 中。

### 10.3 临时文件不入库

AI 执行过程中产生的临时计划、工作树、对话上下文不进入 git。

它们可以帮助开发，但不是项目正式资产。

### 10.4 测试结果必须明确

Claude Code 完成修改后，应明确说明：

- 执行了什么测试命令
- 测试是否通过
- 如果失败，失败原因是什么
- 是否有未验证的内容

---

## 11. 一句话总结

当前推荐模式是：

```text
用 docs 保存原始资料，用 OpenSpec 固化正式规格，用 Superpower 拆解执行计划，用 Claude Code 按计划编码和验证。
```
