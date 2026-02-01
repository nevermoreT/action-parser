# CI 错误模式与修复参考

## Maven / Java

### 编译错误

- `cannot find symbol` / `package X does not exist`  
  → 补全 import 或修正类名；检查 `pom.xml` 是否声明依赖并 `mvn dependency:resolve`。
- `incompatible types` / `required: X, found: Y`  
  → 改类型、强转或改方法返回类型。
- `method X in class Y cannot be applied to given types`  
  → 修正参数类型或个数，或改用正确重载。
- `duplicate class`  
  → 删重复类或合并模块，避免同名类出现在同一 classpath。

### 测试失败

- `AssertionError` / `expected: X but was: Y`  
  → 判断是断言写错还是实现错：改测试或改生产代码。
- `NullPointerException` 在测试中  
  → 补 mock 或初始化，或修被测代码的空指针。
- `NoSuchBeanDefinitionException` / Spring 上下文失败  
  → 补 `@Bean` 或 `@ComponentScan`，或加测试配置类。
- 超时 / `TimeoutException`  
  → 提高超时时间或优化慢逻辑；集成测试可调大 `@Timeout`。

### 依赖与仓库

- `Could not resolve artifact` / `404`  
  → 检查 `pom.xml` 中 `groupId/artifactId/version` 和 `repository` 配置；私服需 URL 和认证正确。
- 版本冲突 / `ConvergenceError`  
  → 在 `pom.xml` 用 `<dependencyManagement>` 或 `<exclusion>` 统一版本。

---

## Gradle

- `Could not find X`  
  → 检查 `repositories` 和依赖坐标；必要时加 `mavenCentral()` 或对应仓库。
- `Task X failed`  
  → 看堆栈中的 `Caused by:`，按编译/测试/资源错误同上处理。
- 配置缓存 / 并行导致偶发失败  
  → 可先关配置缓存或并行复现；再修非幂等或共享状态。

---

## npm / Node

- `Module not found`  
  → 补依赖到 `package.json` 并 `npm install`；或修正 import 路径。
- `EADDRINUSE` / 端口占用  
  → 改端口或先关占用进程；CI 中可用不同端口或随机端口。
- 测试 `AssertionError`  
  → 同“测试失败”：改断言或改实现。
- ESLint / 类型检查失败  
  → 按报错文件与规则修代码或适当调规则。

---

## GitHub Actions

- `Permission denied` / 无法写仓库  
  → 给 workflow 写权限：`permissions: contents: write` 等。
- `Resource not accessible by integration`  
  → 检查 token 权限或改用 `GITHUB_TOKEN` 允许的 API。
- 超时 / 挂起  
  → 缩短超时、检查是否有等待输入；必要时加 `timeout-minutes`。
- Java/Node 版本不匹配  
  → 将 `actions/setup-java` 或 `actions/setup-node` 的版本与本地一致。

---

## 通用原则

1. **先定位再改**：确认文件与行号再改，避免盲目全局替换。
2. **最小改动**：只改与当前错误相关的代码或配置。
3. **本地复现**：能本地跑 CI 同款命令就先本地验证再提交。
4. **一次一类**：多处错误时先修阻塞项（如编译），再修测试或 lint。
