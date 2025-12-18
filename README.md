# QWEN2.5 GRPO Economic Agent Training

## 项目结构

```
.
├── RL/
│   ├── reward.py              # Reward 函数
│   ├── prepare_verl_data.py   # 数据准备脚本
│   └── config/
│       └── econ_grpo_small.yaml  # 训练配置
├── data/
│   └── verl_dataset_small/
│       ├── train.parquet      # 训练数据
│       └── val.parquet        # 验证数据
├── simulate.py                # 仿真主脚本
├── simulate_utils.py          # 仿真工具函数
└── config.yaml                # 仿真配置
```

## 快速开始

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

## 配置说明

### 训练配置 (econ_grpo_small.yaml)

- 模型: Qwen2.5-7B-Instruct
- LoRA: rank=8, alpha=16
- 总步数: 300
- batch_size: 2
- learning_rate: 1e-5

### Reward 设计

- Buffer Stock Theory: 穷人多工作，富人少工作
- Consumption: 数据驱动的 target
- Regime 响应: recession/boom 调整消费

## 注意事项

1. 需要预先下载 Qwen2.5-7B-Instruct 模型到 `/workspace/models/`
2. 训练需要 ~70GB GPU 显存
3. checkpoint 保存在 `checkpoints_v11/`

