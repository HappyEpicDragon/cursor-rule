## 一、总体设计（3 层）

- **第 1 层：配置选择（Hydra）**  
  用命令行选择一个“实验配置文件”（如 `simulation=<domain>/<mode>`），并支持 override。
- **第 2 层：仿真类型选择（Factory）**  
  用配置中的 `name`（或 `sim_type`）从工厂创建具体 Simulator 类。
- **第 3 层：任务路由（mode dispatch）**  
  Simulator 内部根据 `mode` 分发到 `train/test/eval/plot/...` 等方法。

这三层解耦后，新增功能时不需要改主入口逻辑。

---

## 二、Hydra 参数管理通用规范

- **根配置最小化**：只放 defaults、输出目录模板、全局日志等。
- **模式配置独立文件**：每个模式一个 yaml，且都声明：
  - `name: <sim_type>`
  - `mode: <mode_name>`
  - `<mode_name>:` 子树作为该模式的参数命名空间
- **公共配置复用**：环境、模型、数据路径等放成可复用组（`defaults` 引入）。
- **覆盖规则统一**：所有运行时变化只走 Hydra override，不在代码里写死。
- **参数读取单一来源**：业务函数只从 `cfg.<mode_name>` 读取本模式参数，避免跨模式串读。

---

## 三、Factory 通用规范

- **注册式工厂**：`@Factory.register("xxx")` 自动登记类型。
- **创建接口固定**：`Factory.create(sim_type, cfg)` 返回实例。
- **错误可诊断**：未注册时报错并列出可选类型。
- **入口稳定**：主程序永远只做两件事：读配置、交给工厂。

---

## 四、Simulator（调度器）通用规范

- **run() 只做路由，不做业务**：用 `match/case` 或映射表分发模式。
- **每个模式一个方法**：方法内调用对应模块函数，避免把训练逻辑写进调度器。
- **延迟导入**：模式方法内再 import，减少启动耦合与依赖冲突。
- **统一签名**：业务入口建议统一 `fn(cfg, runtime_ctx)`。

---

## 五、新增模式的标准 Checklist（可复用）

- 新建 `<mode>.yaml`（声明 `name`/`mode`/`<mode>:`）。
- 实现业务函数 `fn(cfg, runtime_ctx)`。
- 在 Simulator 的 mode 路由中注册该模式。
- 做一次 smoke test（`--help` 或最小样例运行）。
- 补文档（模式用途、关键参数、示例命令）。

---

## 六、可维护性与复现性规则

- **路径不要硬编码**：都来自配置；运行时路径由统一 output 模板生成。
- **模式命名稳定**：`train_xxx` / `test_xxx` / `eval_xxx` 语义固定。
- **版本化输出**：训练结果建议 run-id 目录 + 可选 `latest` 指针。
- **失败即显式报错**：输入缺失、模型缺失、参数冲突都应 fail-fast。
- **避免“脚本漂移”**：一次性命令也尽量沉淀成可注册模式，而不是仅 shell 脚本。