# Impeccable 中文快速上手

> 原项目：https://github.com/pbakaus/impeccable（Apache-2.0）
> 本文件是 Kilganon725 fork 中新增的中文说明，只翻译和整理上游 README 中已验证的命令，不改动上游代码、规则与默认值。
> 适用对象：已经有一个界面、想让 AI 生成的页面不再千篇一律的开发者。

## 一句话理解

不要再对 AI 说“好看一点”。用有名有姓的命令指着一个具体组件说：
`bolder`（更大胆）、`critique`（打分评审）、`audit`（技术体检），
再用不需要大模型的 62 条规则检测器把“AI 味”拦下来。

## 1. 安装（任选一种）

```bash
# 方式 A：npx 安装器（推荐，装完记得按提示重载你的编程工具）
npx impeccable install

# 之后更新
npx impeccable update
```

```bash
# 方式 B：只想手动复制 skill，不用 Node
# 把仓库里对应工具的文件夹复制到你的项目或全局 skill 目录，例如 Claude Code：
mkdir -p your-project/.claude/skills && cp -r .claude/skills/impeccable your-project/.claude/skills/
```

注意：手动复制会跳过设计钩子（hook）和子代理；想让每次改 UI 文件都自动被检查，
请用方式 A 安装，并在 Codex 里批准项目钩子、Grok Build 里执行 `/hooks-trust`、
Hermes Agent 里执行 `hermes plugins enable impeccable-design-hook`。

插件市场安装（按你的工具二选一即可）：

- Claude Code：`/plugin marketplace add pbakaus/impeccable`，再 `/plugin` 安装 Impeccable。
- GitHub Copilot（VS Code）：安装 Impeccable 扩展，或 `code --install-extension renaissance-geek.impeccable`（需 VS Code 1.109.3+、Copilot Chat、可信工作区）。
- Grok Build：`grok plugin install pbakaus/impeccable#plugin --trust`。

## 2. 初始化你的项目（每个项目做一次）

在你的编程工具里执行：

```
/impeccable init
```

它会读你的代码，只问它推断不出来的事（产品给谁用、要做什么、有什么约束），
然后写出 `PRODUCT.md`。之后每条命令都会读这份文件。

再执行一次，把你现有的视觉系统固定下来，避免每次改动都跑偏：

```
/impeccable document
```

这会生成 `DESIGN.md`。常用命令可以钉住，例如 `/impeccable pin audit` 之后
就能直接用 `/audit`。

## 3. 实战示例：评审一张定价卡片

假设你的页面已经有一组定价卡片，但看起来就是默认的 AI 风格。按这个顺序做：

```text
第 1 步：评审并拿到排序后的修改清单
/impeccable critique the pricing cards

第 2 步：按清单先改最要紧的几项（不要一次全改）
/impeccable polish the pricing cards

第 3 步：如果还不够大胆，只针对这一处加码
/impeccable bolder the pricing cards

第 4 步：上线前做一次技术体检（无障碍、性能、主题、响应式，P0 阻塞 → P3 打磨）
/impeccable audit the pricing cards
```

经验法则（来自上游 README）：一次只盯一个组件、盯仔细，比一眼扫过整页有效；
新页面从零生成是较难的模式，先有界面再迭代效果最好。

## 4. 不用大模型的检测器：62 条规则

检测器不需要 LLM、不需要 API key，适合放进终端和 CI：

```bash
# 扫描一个目录（源码文件只跑模式类规则）
npx impeccable detect src/

# 扫描一个已渲染的页面（低对比度、文字过小这类规则需要渲染后的页面）
npx impeccable detect https://example.com

# 机器可读输出
npx impeccable detect --json .

# GitHub Actions 注释格式
npx impeccable detect --github src/

# 查看全部规则的编号与说明
npx impeccable detect --list-rules
```

退出码含义：`2` = 发现问题；`1` = 有目标扫不出来。

有意的品牌选择不要硬扛规则，显式放行并写清理由，例如全站都用 Inter：

```bash
npx impeccable ignores add-value overused-font Inter --reason "Brand font"
```

只对单个文件关一条规则，用注释：

```html
<!-- impeccable-disable overused-font: exported brand doc -->
```

## 5. 在浏览器里实时迭代（可选）

```text
/impeccable live
```

它会在你本地的开发服务器（Vite、Next、SvelteKit、Astro、Nuxt 或纯 HTML）
上加一层覆盖：在浏览器里点中一个元素、选一个方向，就能原地切换多个变体，
接受后写回源码。注意两条红线：只改本地 checkout，不要往已部署站点注入；
应用文案修改时会跑你 `package.json` 里的 `impeccable:manual-edit-validate`
脚本，不熟悉的仓库先读一下这个脚本。

## 6. 24 条命令速查

| 类别 | 命令 |
| --- | --- |
| 指挥方向 | `bolder` 更大胆 · `quieter` 收敛 · `distill` 做减法 · `colorize` 上色 · `typeset` 修字体层级 · `layout` 修间距节奏 · `animate` 加动效反馈 · `delight` 加个性 · `overdrive` 上 shader、弹簧物理与滚动特效 |
| 评审 | `critique` 打分评审 · `audit` 技术体检（P0–P3） |
| 收尾 | `polish` 上线前最后一遍 · `harden` 错误/i18n/溢出等边界 · `clarify` 改写不清的文案 · `adapt` 适配屏幕尺寸 · `optimize` 修加载性能 · `onboard` 首 run 流程与空状态 |
| 实时与规划 | `live` 浏览器里选元素迭代 · `generate` 为指定元素生成变体 · `shape` 动工前先规划体验 · `init` 写 `PRODUCT.md` · `document` 写 `DESIGN.md` · `extract` 把重复模式抽成设计系统 |

## 7. 检查清单：第一次跑通的标志

- [ ] `npx impeccable install` 完成，编程工具已重载。
- [ ] `/impeccable init` 生成了 `PRODUCT.md`。
- [ ] `/impeccable critique` 对一个真实组件给出了可执行的排序清单。
- [ ] `npx impeccable detect src/` 能跑通，并能读懂退出码 2 与 1 的含义。
- [ ] 任何品牌特例都已用 `ignores` 或注释显式放行，而不是被忽略。

## 声明

本文件与同批提交的 `UNDERSTANDING.md`、`covers/2026-10-31-cover.png`
均为 Kilganon725 fork 的新增内容；原项目作者与 Apache-2.0 许可证保持不变。
