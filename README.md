# QWEN2.5 GRPO Economic Agent Training

## 项目结构

```
.
├── RL/
│   ├── reward.py              # Reward 函数 (Buffer Stock Theory)
│   ├── prepare_verl_data.py   # 数据准备脚本
│   └── config/
│       └── econ_grpo_small.yaml  # 训练配置
├── ai_economist/
│   └── foundation/            # 仿真环境核心模块
│       ├── agents/            # Agent 定义
│       ├── base/              # 基础类
│       ├── components/        # 经济行为组件
│       ├── entities/          # 实体定义
│       └── scenarios/         # 场景定义
├── data/
│   └── verl_dataset_small/
│       ├── train.parquet      # 训练数据
│       └── val.parquet        # 验证数据
├── simulate.py                # 仿真主脚本
├── simulate_utils.py          # 仿真工具函数
├── plot_training.py           # 训练曲线绘图
├── config.yaml                # 仿真配置
├── requirements.txt           # 依赖包
└── GRPO_requirements.txt      # GRPO 完整依赖
```

## 快速开始

### 0. 安装依赖

```bash
pip install -r requirements.txt
# 或完整安装
pip install -r GRPO_requirements.txt
```

### 1. 运行 GRPO 训练

```bash
cd RL
python -m verl.trainer.main_ppo --config-path=config --config-name=econ_grpo_small
```

### 2. 使用 tmux 后台训练

```bash
tmux new-session -d -s grpo "cd RL && python -m verl.trainer.main_ppo --config-path=config --config-name=econ_grpo_small 2>&1 | tee train.log"
```

### 3. 运行仿真

```bash
python simulate.py --policy_model gpt --num_agents 100 --episode_length 240
```

### 4. 绘制训练曲线

```bash
python plot_training.py --log_dir checkpoints_v11/
```

## 配置说明

### 训练配置 (econ_grpo_small.yaml)

- 模型: Qwen2.5-7B-Instruct
- LoRA: rank=8, alpha=16
- 总步数: 300
- batch_size: 2
- learning_rate: 1e-5
- validation 频率: 每 300 步
- checkpoint 保存: 每 300 步

### Reward 设计 (reward.py)

核心设计基于 **Buffer Stock Theory**:

1. **Work Reward**:
   - `pref = 1 - 2*alpha` (穷人偏好高工作，富人偏好低工作)
   - `direction_bonus = K_DIR * pref * work_center`
   - `plateau_pen`: 惩罚 work 停留在 0.5 附近

2. **Consumption Reward**:
   - 数据驱动的 target: `CONS_POOR=0.3760`, `CONS_RICH=0.1993`
   - Regime 响应: recession 减少消费，boom 增加消费

3. **权重配置**:
   - `W_WORK = 0.35`
   - `W_MACRO = 0.35` (包含 consumption)
   - `W_SR = 0.15` (saving rate)
   - `W_PEN = 0.15`

## 注意事项

1. 需要预先下载 Qwen2.5-7B-Instruct 模型到 `/workspace/models/`
2. 训练需要 ~70GB GPU 显存
3. checkpoint 保存在 `checkpoints_v11/`
4. `ai_economist/foundation` 是仿真环境，运行 simulate.py 必需

## 新实例快速部署

```bash
# 1. 克隆仓库
git clone https://github.com/uqcxu9/qwen2.5_300step_NEW_CONSIS_2.git
cd qwen2.5_300step_NEW_CONSIS_2

# 2. 安装依赖
pip install -r GRPO_requirements.txt

# 3. 下载模型 (如需要)
# huggingface-cli download Qwen/Qwen2.5-7B-Instruct --local-dir /workspace/models/Qwen2.5-7B-Instruct

# 4. 开始训练
cd RL
tmux new-session -d -s grpo "python -m verl.trainer.main_ppo --config-path=config --config-name=econ_grpo_small"
```
