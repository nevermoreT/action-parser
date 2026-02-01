---
name: ci-results-analyze-fix
description: Fetches CI/build results (GitHub checks or local build output), analyzes failure causes (compile, test, lint), and applies code or config fixes. Use when CI is failing, when the user asks to fix CI or build errors, or when reviewing workflow run or build logs.
---

# CI 结果获取与自动修复

## 何时使用

- 用户说 CI 红了、流水线失败、build 失败
- 用户要求“看下 CI 报错并修掉”
- 用户粘贴了 GitHub Actions / 其他 CI 的日志
- 用户有 PR，需要根据 checks 状态排查并修复

## 工作流概览

```
1. 获取 CI 结果 → 2. 分析错误类型与位置 → 3. 制定修复方案 → 4. 修改代码/配置 → 5. 验证（本地或提示重跑 CI）
```

---

## 1. 获取 CI 结果

### 方式 A：GitHub PR 的 checks 状态

当用户提到某条 PR 或提供了 `owner/repo` 和 PR 号时：

- 使用 GitHub MCP 的 `pull_request_read`，`method`: `get_status`，获取该 PR 头提交的 **build/check 状态**。
- 若返回中有失败 check 的名称、摘要或详情链接，据此判断是哪个 job/step 失败。

若 MCP 未返回完整日志，可请用户从 GitHub Actions 页面复制失败 job 的日志，或进入下一步在本地复现。

### 方式 B：本地复现（推荐用于拿到完整错误信息）

根据项目类型执行与 CI 相同的命令，捕获标准输出和标准错误：

- **Maven (Java)**：`mvn clean compile` 或 `mvn clean test`（与 `.github/workflows/*.yml` 中一致）。
- **Gradle**：`./gradlew build` 或 `./gradlew test`。
- **Node/npm**：`npm ci` 或 `npm run build`，`npm test`。
- **其他**：先读仓库中的 `.github/workflows/*.yml` 或 `Jenkinsfile` 等，再执行对应命令。

把终端完整输出作为“CI 结果”用于下一步分析。

### 方式 C：用户已粘贴的日志

用户直接粘贴的 CI 日志视为“CI 结果”，无需再获取，直接进入分析。

---

## 2. 分析错误类型与位置

从日志中识别：

| 类型 | 典型特征 | 要提取的信息 |
|------|----------|--------------|
| 编译错误 | `error:`, `compilation failed`, `BUILD FAILURE` | 文件路径、行号、错误信息 |
| 测试失败 | `FAILURE`, `Tests run: X, Failures: Y`, `AssertionError` | 测试类/方法名、失败原因、堆栈中的文件:行号 |
| 依赖/解析失败 | `Could not resolve`, `404`, `dependency not found` | 依赖名、版本、仓库配置 |
| 语法/风格 | `Parse error`, linter 名（Checkstyle, ESLint 等） | 文件、行号、规则名 |
| 环境/配置 | `JAVA_HOME`, `command not found`, 超时 | 缺失的配置或环境 |

- **Maven**：关注 `[ERROR]` 行，路径常为 `src/...` 或 `pom.xml`；测试失败会带 `<<< FAILURE!` 和类名。
- **Gradle**：关注 `FAILED` 和 `Caused by:` 后的异常与文件位置。
- **npm/Node**：关注 `Error:` 与堆栈中的文件名和行号。

将“错误类型 + 文件:行号 + 原文摘要”整理成简短列表，便于制定修复方案。

---

## 3. 制定修复方案与自动修复

- **编译错误**：打开对应文件，根据报错修语法、类型、缺失方法/导入等。
- **测试失败**：  
  - 若断言或业务逻辑错误：改生产代码或测试逻辑。  
  - 若环境/顺序依赖：改测试写法或加/改 `@Before`、资源清理。
- **依赖问题**：改 `pom.xml` / `build.gradle` / `package.json` 的依赖或仓库配置，必要时查兼容版本。
- **语法/风格**：按 linter 规则改代码或配置（如禁用某规则需说明原因）。
- **环境/配置**：改 CI 配置（如 `.github/workflows/*.yml`）或文档中的环境说明。

修复时：

- 一次只改与当前错误直接相关的部分，避免大范围无关修改。
- 若同一日志中有多处错误，按“阻塞顺序”修（例如先编译再测试）。

---

## 4. 验证

- **本地**：再次执行与 CI 相同的命令（如 `mvn clean test`），确认通过。
- **未在本地跑**：说明已做修改并建议用户提交后查看 CI 或提供最新日志以便继续排查。

---

## 简要检查清单

- [ ] 已通过 PR status、本地复现或用户粘贴获取到失败信息
- [ ] 已区分错误类型并定位到文件与行号
- [ ] 已按类型选择修代码、依赖、配置或 CI
- [ ] 已本地验证或已建议用户重跑 CI

更多常见错误模式与对应修复策略见 [reference.md](reference.md)。
