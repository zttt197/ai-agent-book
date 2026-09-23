# 第 1 章 · 阅读顺序与代码执行顺序

> 本文是 `README.md` 的配套速查卡：把「章节推荐的阅读顺序」和「各项目实际可执行的命令顺序」
> 放在一页里，方便随时对照。内容依据 `README.md` 与各子项目代码整理（整理日期 2026-09-23）。

## 一、速览结论

- 阅读顺序**刻意不等于项目编号**：README 让你先读 `web-search-agent`（项目 1-2），再回到 `context`（项目 1-1）。
- 真正的动手起点是**零成本、无需 API Key** 的那条路径：`web-search-agent --provider offline-demo`。
- 每个项目的入口统一是 `main.py`；`learning-from-experience` 例外，入口是 `experiment.py` / `quick_demo.py` / `run_experiment_8_2.py`。

## 二、章节推荐阅读顺序

| 次序 | 项目 | README 原文要求 | 读的时候盯住什么 |
| :--: | --- | --- | --- |
| 1 | `web-search-agent/` | 先看无需凭据的离线搜索轨迹，辨认思考、动作与观察 | 一次「思考 → 调用工具 → 读结果 → 回答」的控制流 |
| 2 | `context/` | 在同一任务中移除一类上下文，分析行为变化 | 变化到底发生在工具调用、还是最终答案 |
| 3 | `image-gen-workflow/` | 比较工作流中的额外步骤是否帮助满足用户需求 | 改写节点是「翻译」还是「篡改」用户约束 |

读完这三个例子后，再查阅其余项目与配置：`search-codegen/`（项目 1-3）、`learning-from-experience/`（实验 7-1 / 7-2）、`EXPERIMENT_LEDGER.md`。

## 三、项目编号与类型

| 编号 | 项目 | 类型 | 一句话说明 |
| :--: | --- | :--: | --- |
| 1-1 | `context/` | ✅ | 上下文消融实验，展示各上下文组件的重要性 |
| 1-2 | `web-search-agent/` | ✅ | 模型即 Agent，基础深度搜索与多轮信息整合 |
| 1-3 | `search-codegen/` | ✅ | 多轮搜索 + 服务端代码执行的 Deep Research 闭环 |
| 1-4 | `image-gen-workflow/` | ✅ | 改写工作流 vs 原生出图的真实对照 |
| 7-1, 7-2 | `learning-from-experience/` | ✅ | Q-learning 与 LLM 上下文学习对比 |

## 四、代码执行顺序（推荐动手路径）

### 第 0 步 · 安装依赖

各项目自带 `requirements.txt`，在项目目录内执行：

```bash
python -m pip install -r requirements.txt
```

### 第 1 步 · `web-search-agent`：先跑零成本离线演示

```bash
python main.py --provider offline-demo            # 回放预写轨迹，无需 API Key
python main.py "2024年诺贝尔物理学奖获得者是谁？"   # 联网单次问答
python main.py                                    # 交互模式
```

`web-search-agent/.env` 已配置可用的 `MOONSHOT_API_KEY`，可走真实搜索。

### 第 2 步 · `context`：单任务 → 消融对照

```bash
python main.py --mode single --context-mode full
python main.py --mode ablation
python main.py --mode ablation --cases 3
python main.py --mode ablation --ablation-modes full no_history --output my_ablation.json
python run_experiment_1_1.py --provider kimi      # 按书里实验 1-1 的口径全跑
```

五种上下文模式：`full` / `no_history` / `no_reasoning` / `no_tool_calls` / `no_tool_results`。

### 第 3 步 · `search-codegen`：先 dry-run，再真跑

```bash
python main.py --backend openai --dry-run --request "..."   # 只看请求体，无需 Key
python main.py --backend dashscope --mode single --request "东盟 10 国首都之间距离最近的两个首都是？"
python run_experiment_1_3.py --backends openai dashscope --reasoning high
```

### 第 4 步 · `image-gen-workflow`：缩小范围再跑全量

```bash
python main.py --requirement windowsill-plant     # 单条需求
python main.py --route workflow                   # 单条路线
python main.py                                    # 3 路线 × 5 需求，较贵
```

### 第 5 步 · `learning-from-experience`：先免费的 RL，再 LLM

```bash
python experiment.py --mode qlearning --rl-episodes 10000 --seed 42
python quick_demo.py
python run_experiment_8_2.py
```

## 五、每个项目内部的代码阅读顺序

- 统一入口：`main.py`（CLI 参数 → 模式分派）。
- 推荐链路：`main.py` → `agent.py` → `config.py` → `tests/` 与 `validation/latest.json`。
- 项目专属补充：
  - `context`：`grounding.py`（答案是否有据）、`calc_sandbox.py`（计算沙箱）。
  - `image-gen-workflow`：`pipeline.py`（路线实现）、`evidence.py`（证据清单）。
  - `learning-from-experience`：`game_environment.py` → `rl_agent.py` → `llm_agent.py`，入口是 `experiment.py` / `quick_demo.py` / `run_experiment_8_2.py`。
- 各子 README 顶部的 `learning-0 … learning-5` 锚点就是作者设计的阅读骨架：
  理解问题与方法 → 准备环境与输入 → 按照步骤完成实验 → 分析结果与形成判断 → 阅读实现与继续探索 → 排查问题与查阅资料。

## 六、本机环境现状（2026-09-23）

| 项目 | `.env` 状态 | 备注 |
| --- | --- | --- |
| `web-search-agent` | 有 `.env`，`MOONSHOT_API_KEY` 为真实值 | 可直接联网 |
| `context` | 有 `.env`，仅 `DEEPSEEK_API_KEY` 真实，`LLM_PROVIDER=deepseek` | 建议 `--provider deepseek` |
| `search-codegen` | 只有 `env.example` | 需先复制为 `.env` |
| `image-gen-workflow` | 只有 `env.example` | 需先复制为 `.env` |
| `learning-from-experience` | 只有 `env.example` | RL 部分无需 Key |

其他注意点：本机 Python 为 3.12.3，`openai` / `dotenv` 等依赖尚未安装。

## 七、判断结论时的提醒

- README 明确指出：实验 1-1 的一次五组对照**没有**观察到「去掉推理内容必然退化」。
  不要把自己预期的现象当成必然复现的结论。
- 比较实验时，把**输入、模型、运行条件和结果**放在一起记录。
- 遇到历史输出，先确认它对应的版本与任务范围，再对照 `EXPERIMENT_LEDGER.md`。

---

> **Agent = LLM + 上下文 + 工具**；Harness 工程才是竞争力。
