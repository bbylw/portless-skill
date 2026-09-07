# Portless Agent Skill

An open, cross-platform **agent skill** for working with [portless](https://github.com/vercel-labs/portless) — the CLI that replaces random `localhost:<port>` dev-server ports with stable, named HTTPS URLs like `https://myapp.localhost`.

It teaches AI coding agents (and humans) how to run dev servers and static previews through portless, what flags it injects per framework, and how to diagnose routing/HTTPS problems — verified against the real CLI on Windows, macOS, and Linux.

This is a community, **workflow-focused** take: it keeps what an agent actually runs day to day and stays lean. For the canonical, full reference (CLI tables, `service`, Tailscale/ngrok, config fields, `oauth`), use the official skills in the [portless repository](https://github.com/vercel-labs/portless/tree/main/skills).

> **中文版本见下方** — English first, 中文在下。

---

## English

### What's inside

```
.agents/skills/portless/SKILL.md   # the skill itself (self-contained)
```

The skill covers:

- Zero-config usage (`portless` → runs the `"dev"` script), named runs, subdomains, git worktrees, monorepos, static sites (`npx serve`), and `alias` routes for Docker/external services.
- Framework flag injection (Vite, VitePlus, Astro, React Router, Angular, Expo, React Native), `--strictPort`, the 508 loop fix, and the 502 cases portless cannot rewrite.
- Environment variables, verification ladder (`list`/`doctor`/`prune`/`trust`/`hosts sync`), and platform notes for Windows, macOS, Linux, WSL, and custom TLDs.

### Try it in 60 seconds

With portless installed (`npm install -g portless`), serve this repo's example folder through a named HTTPS URL:

```bash
git clone https://github.com/bbylw/portless-skill
cd portless-skill/examples/static-site
portless demo npx -y serve .
# -> open https://demo.localhost
```

You should see the page with no certificate warning. Press `Ctrl+C` to stop; the route cleans up automatically. More in [`examples/static-site`](examples/static-site).

### Install for your agent

Point your agent tool at this repo and it will pick up the skill automatically (project-level `.agents/skills`). To make it available machine-wide:

```bash
# Codebuff / Claude Code style user-level install
cp -r .agents/skills/portless ~/.agents/skills/
```

or reference this repository directly with a skill installer, e.g.:

```bash
npx skills add <owner>/<repo> --skill portless --yes
```

### Requirements

The skill is just instructions; the tool itself is required on the machine that runs it:

```bash
npm install -g portless   # Node.js 24+
portless doctor           # read-only health check
```

Behavior notes assume **portless ≥ 0.15**.

### Relationship to the official skills

The [upstream portless repo](https://github.com/vercel-labs/portless/tree/main/skills) ships its own skills — a full `portless` reference and an `oauth` companion — and they are the canonical source. This repo is intentionally different:

- **Bilingual** — English and Chinese; upstream is English-only.
- **Workflow-focused, not a manual** — covers the daily agent flow (zero-config, named runs, static sites, monorepos, framework injection, diagnostics) and stays lean. For the exhaustive reference (every command, `service install`, Tailscale/ngrok, `--wildcard`, config fields, OAuth setup), see the official skills.

Install one or the other — they share the same skill name (`portless`) and would conflict if both are loaded.

### License

MIT

---

## 中文

### 这是什么

一份开源的、跨平台的 **AI 代理技能（agent skill）**，用于配合 [portless](https://github.com/vercel-labs/portless) 使用——它能把本地开发服务器随机的 `localhost:<端口>` 换成稳定、有名字的 HTTPS 地址，例如 `https://myapp.localhost`。

它教会 AI 编程代理（以及人类开发者）如何通过 portless 运行开发服务器和静态预览、portless 会按框架注入哪些参数、以及如何排查路由/HTTPS 问题——内容已在 Windows、macOS、Linux 上对照真实 CLI 验证过。

### 目录结构

```
.agents/skills/portless/SKILL.md   # 技能本体（自包含）
```

技能涵盖：

- 零配置用法（`portless` → 运行 `"dev"` 脚本）、命名运行、子域名、git worktree、monorepo、静态站点（`npx serve`），以及面向 Docker/外部服务的 `alias` 路由。
- 框架参数注入（Vite、VitePlus、Astro、React Router、Angular、Expo、React Native）、`--strictPort`、508 循环修复、以及 portless 无法改写而返回 502 的场景。
- 环境变量、诊断阶梯（`list`/`doctor`/`prune`/`trust`/`hosts sync`）、以及 Windows、macOS、Linux、WSL 与自定义 TLD 的平台注意事项。

### 60 秒上手

装好 portless 后（`npm install -g portless`），把本仓库的示例目录跑成一个有名字的 HTTPS URL：

```bash
git clone https://github.com/bbylw/portless-skill
cd portless-skill/examples/static-site
portless demo npx -y serve .
# -> 打开 https://demo.localhost
```

页面应正常显示且无证书告警。按 `Ctrl+C` 停止，路由自动清理。详见 [`examples/static-site`](examples/static-site)。

### 安装到你的 AI 代理

把本仓库指向你的代理工具，即可自动加载该技能（项目级 `.agents/skills`）。若要全机器可用：

```bash
# Codebuff / Claude Code 风格的用户级安装
cp -r .agents/skills/portless ~/.agents/skills/
```

或直接用技能安装器引用本仓库，例如：

```bash
npx skills add <owner>/<repo> --skill portless --yes
```

### 环境要求

技能本身只是说明文档；真正执行需要在本机安装工具：

```bash
npm install -g portless   # 需要 Node.js 24+
portless doctor           # 只读健康检查
```

行为说明假设 **portless ≥ 0.15**。

### 与官方技能的关系

[上游 portless 仓库](https://github.com/vercel-labs/portless/tree/main/skills)自带技能——完整的 `portless` 参考版和一个 `oauth` 配套技能——它们才是权威来源。本仓库刻意与之不同：

- **中英双语**——上游只有英文。
- **面向工作流而非手册**——聚焦日常 agent 流程（零配置、命名运行、静态站点、monorepo、框架注入、诊断），保持精简。需要完整参考（全部命令、`service install`、Tailscale/ngrok、`--wildcard`、配置字段、OAuth 配置）时，请看官方技能。

两者安装其一即可——它们共用同一个技能名（`portless`），同时加载会冲突。

### 许可证

MIT
