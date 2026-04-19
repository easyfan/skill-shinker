# skill-shrinker

将臃肿的 Claude Code skill/agent/command 文件压缩为干净的三层结构。

## 功能

分析目标 SKILL.md（或 command/agent .md），使用 **ABCD 四类提取体系**进行压缩，
支持两种目标轨道：

**ABCD 四类说明：**

| 类别 | 目标位置 | 判据 |
|------|---------|------|
| A | 保留在 SKILL.md | 编排逻辑、流程步骤、分支决策 |
| B | `scripts/` | >3 行的 bash 代码块，可独立调用 |
| C | `manifest-schema.json` | 跨 Phase 共享参数（多 Phase 消费且必须一致）|
| D | `DESIGN.md` | 设计背景、推导过程、补充说明（不加载至上下文）|

**两种目标轨道：**

| 轨道 | 目标 | 适用场景 |
|------|------|---------|
| Option A — 稳定型 | SKILL.md ≤ 220 行 | 单阶段或稳定 skill |
| Option B — 成长型 | SKILL.md ≤ 80 行（纯协调者）| Phase ≥ 3、持续增长的 skill |

Option B 为 **Proposal-only**：skill-shrinker 输出 agent 化方案和目录结构建议，
实际拆分由用户手动执行。

**三种运行模式（按文件行数）：**

| 行数 | 操作 |
|------|------|
| < 200 | 告知无需压缩 |
| 200–500 | 仅输出 **Proposal**（ABCD 分类 + 预估结果，不修改文件）|
| > 500 | 完整分析，用户确认 A 或 B 轨道后执行 |

## 被 skill-review 依赖

skill-review（v1.5.0+）设有 **400 行强制拦截**：任何超过 400 行的目标文件将被拒绝审查，
并提示先运行 `/skill-shrink`。安装 skill-shrinker 后可解锁对大型 skill/agent 文件的审查能力。

```
⛔ skill-review 拒绝审查超过 400 行的文件。
   请先运行 /skill-shrink <文件>，再重新触发审查。
```

## 安装

### 方式一：从 marketplace 安装

```
/plugin marketplace add skill-shrinker
/plugin install skill-shrinker@latest
```

### 方式二：手动安装

```bash
bash install.sh
```

自定义 Claude 配置目录：

```bash
bash install.sh --target=/path/to/.claude
```

预览（不写入文件）：

```bash
bash install.sh --dry-run
```

卸载：

```bash
bash install.sh --uninstall
```

如需覆盖 `CLAUDE_DIR`：

```bash
CLAUDE_DIR=/custom/path bash install.sh
```

## 用法

安装后，在 Claude Code 中触发：

```
/skill-shrink my-skill
shrink ~/.claude/skills/my-skill/SKILL.md
这个 skill 太大了，帮我拆一下
```

skill-review 在目标文件超过 400 行时也会自动引导你执行 `/skill-shrink`。

## 输出结构

**Option A（稳定型）：**

```
my-skill/
├── SKILL.md              ← ≤220 行，仅编排指令
├── manifest-schema.json  ← 跨 Phase 共享参数（init 阶段写入，后续只读）
├── scripts/              ← 提取的 bash 脚本（各脚本可独立调用）
│   └── init_something.sh
└── DESIGN.md             ← 设计说明、推导过程、背景信息
```

**Option B（成长型，Proposal-only）：**

```
my-skill/
├── SKILL.md              ← ≤80 行，纯协调者
├── manifest-schema.json  ← 跨 Phase 共享参数
├── agents/               ← 每个 Phase 对应一个 agent 文件
│   ├── phase-0-init.md
│   └── phase-1-xxx.md
└── DESIGN.md
```

## 评测用例

`evals/evals.json` 包含 5 个测试用例，覆盖所有核心决策路径：

| ID | 场景 | 预期行为 |
|----|------|---------|
| 1 | 大型 skill（>500 行，多 Phase）| ABCD 分析 + Phase ≥ 3 时推荐 Option B |
| 2 | 中等 skill（200–500 行）| Proposal 模式：ABCD 分类，等待 y/n/B 确认 |
| 3 | 小型 skill（<200 行）| 拒绝执行，告知无需压缩 |
| 4 | 成长型 skill（4 个 Phase）| Option B Proposal-only：agent 化方案 + manifest 字段列表 |
| 5 | C vs D 边界（LUFS 目标值 + 推导公式）| LUFS → C 类（manifest）；推导公式 → D 类（DESIGN.md）|

## 更新日志

### v0.3.0 (2026-04-20)

重大设计升级 — ABCD 四类提取体系 + Option A/B 双轨设计：

| 变更项 | 说明 |
|--------|------|
| 分类体系 | ABC 三类升级为 ABCD 四类；C 拆分为 C（manifest）和 D（DESIGN.md）|
| manifest | 新增 `manifest-schema.json` 用于跨 Phase 共享参数，解决 context drift 问题 |
| Option B | 成长型轨道：Phase agent 化（Proposal-only），SKILL.md ≤ 80 行 |
| 决策树 | 新增 C vs D 二步决策树及边界案例对照表 |
| 容错处理 | 文件存在性检查、无效输入处理、脚本语法检查失败提示 |
| 评测用例 | 新增 5 个测试用例覆盖所有决策路径 |

### v0.2.0 (2026-04-14)

skill-review 集成 — skill-shrinker 成为 skill-review v1.5.0+ 的必要伴侣：

| 变更项 | 说明 |
|--------|------|
| 依赖关系 | skill-review v1.5.0 新增 400 行强制拦截，超限文件必须先经 `/skill-shrink` 处理 |
| 安装提示 | install.sh 安装完成后检测 skill-review，输出伴侣激活确认 |
| README | 新增"被 skill-review 依赖"章节 |

### v0.1.0 (2026-03-01)

初始版本。

## License

MIT
