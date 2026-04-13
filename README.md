# meta-harness

基于 [Harbor](https://github.com/ljvmiranda921/harbor) Terminus2 的 LLM 智能体脚手架，可通过 pip 安装。源自 [stanford-iris-lab/meta-harness-tbench2-artifact](https://github.com/stanford-iris-lab/meta-harness-tbench2-artifact)，在此基础上封装为标准 Python 包。

## 特性

- **原生工具调用**：通过 `execute_commands` / `task_complete` / `image_read` 三个工具与沙箱交互，不依赖 JSON/XML 解析
- **环境快照注入**：Agent 循环启动前自动采集沙箱状态（工作目录、文件列表、可用语言与包管理器、内存），注入初始提示词，减少探索轮次
- **标记轮询执行**：每条命令末尾追加唯一 echo 标记，以 0.5s 间隔轮询 tmux 输出，命令完成即退出等待，避免固定 sleep 浪费时间
- **二次确认完成**：首次调用 `task_complete` 时触发多角度核查清单，第二次确认后才真正结束任务
- **Anthropic 提示词缓存**：对 Anthropic / Claude 系模型自动为最近 3 条消息添加 `cache_control`
- **上下文长度恢复**：超出上下文窗口时自动 unwind + summarize，失败则截断后重试

## 安装

```bash
# 直接安装
pip install git+https://github.com/circlecrystal/meta-harness-pkg.git

# 可编辑安装（开发用）
git clone https://github.com/circlecrystal/meta-harness-pkg.git
pip install -e meta-harness-pkg/
```

**依赖**：`harbor`（需单独安装）、`litellm`、`anthropic`、`tenacity`

## 使用

```python
from meta_harness import AgentHarness

agent = AgentHarness(
    logs_dir="./logs",
    model_name="anthropic/claude-opus-4-6",   # litellm 格式：provider/model
    max_turns=30,
    temperature=1.0,
    model_info={"max_input_tokens": 200000, "max_output_tokens": 16000},
)

# env 为 Harbor BaseEnvironment 实例，ctx 为 AgentContext
await agent.setup(env)
result = await agent.run(instruction="完成某项终端任务", environment=env, context=ctx)
```

## 与上游的关系

本仓库与上游 `stanford-iris-lab/meta-harness-tbench2-artifact` 的核心逻辑完全一致，仅做了包化改造：

| 变更点 | 说明 |
|---|---|
| 目录结构 | 源码移入 `meta_harness/` 包目录，添加 `__init__.py` |
| import 路径 | `from anthropic_caching` → `from .anthropic_caching`（相对导入） |
| `pyproject.toml` | 添加 hatchling 构建配置；`harbor` 依赖由外部管理，`litellm` 去除版本上限 |
