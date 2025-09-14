# How to Use CloseAirCombatPettingZoo: Complete Guide

## What is This Repository?

This is **Light Aircraft Game** - a lightweight, scalable aircraft competitive environment for training reinforcement learning agents in air combat scenarios. It provides:

- **Realistic flight dynamics** using JSBSim flight simulator
- **Missile dynamics** with proportional guidance implementation
- **Multiple combat scenarios**: 1v1 and 2v2 air combat
- **RL algorithms**: PPO and MAPPO implementations
- **Self-play training** capabilities
- **3D visualization** using TacView

## Repository Structure

```
CloseAirCombatPettingZoo/
├── envs/JSBSim/           # Main environments and configs
├── scripts/               # Training and rendering scripts  
├── algorithms/            # PPO and MAPPO implementations
├── runner/                # Training runners
├── renders/               # Visualization scripts
└── docs/                  # Documentation
```

## Available Environments

### 1. SingleControl Environment
**Purpose**: Train basic flight control (heading, altitude, velocity)
- **Use case**: Foundation training for combat agents
- **Config**: `envs/JSBSim/configs/1/heading.yaml`
- **Script**: `scripts/train_heading.sh`

### 2. SingleCombat Environment (1v1)
**Purpose**: Two aircraft dogfighting

#### A. NoWeapon Tasks
- Goal: Achieve positional advantage (get behind opponent)
- **Self-play config**: `envs/JSBSim/configs/1v1/NoWeapon/Selfplay.yaml`
- **vs-Baseline config**: `envs/JSBSim/configs/1v1/NoWeapon/vsBaseline.yaml`

#### B. DodgeMissile Tasks  
- Goal: Learn to dodge incoming missiles
- **Config**: `envs/JSBSim/configs/1v1/DodgeMissile/Selfplay.yaml`

#### C. ShootMissile Tasks
- Goal: Learn to shoot missiles and engage in combat
- **Config**: `envs/JSBSim/configs/1v1/ShootMissile/HierarchySelfplay.yaml`

### 3. MultipleCombat Environment (2v2)
**Purpose**: Four aircraft team-based combat
- **Config**: `envs/JSBSim/configs/2v2/NoWeapon/HierarchySelfplay.yaml`

## Quick Start Guide

### Step 1: Installation (if not done)
```bash
# Install dependencies  
pip install torch pymap3d jsbsim==1.1.6 geographiclib wandb icecream setproctitle
```

### Step 2: Choose Your Training Scenario

Navigate to the scripts directory:
```bash
cd /Users/mustafa/Desktop/wvr/CloseAirCombatPettingZoo/scripts
```

#### Option A: Basic Flight Control Training
```bash
bash train_heading.sh
```
This trains an agent to follow basic flight commands (good starting point).

#### Option B: 1v1 Combat vs Baseline
```bash 
bash train_vsbaseline.sh
```
Train against pre-programmed opponents.

#### Option C: 1v1 Self-Play Combat
```bash
bash train_selfplay.sh  
```
Agents train by fighting against copies of themselves.

#### Option D: 1v1 Missile Combat
```bash
bash train_selfplay_shoot.sh
```
Advanced: agents learn to shoot missiles.

#### Option E: 2v2 Team Combat
```bash
bash train_share_selfplay.sh
```
Team-based multi-agent training.

### Step 3: Monitor Training

If you enable wandb (set `--use-wandb` in scripts), you can monitor training progress online.

Training results are saved in:
```
scripts/results/[ENV_NAME]/[SCENARIO]/[ALGORITHM]/[EXPERIMENT_NAME]/
```

## Using Different Environments

### JSBSim Environments (Recommended)
These use realistic flight dynamics:

```bash
# Train with JSBSim physics
python train/train_jsbsim.py --env-name SingleCombat --scenario-name "1v1/NoWeapon/Selfplay" --algorithm-name ppo
```

### Gym Environments (For Testing)
We fixed the Gymnasium compatibility issues, so you can also use:

```bash
# Train with simple Gym environments (like CartPole)
python train/train_gym.py --scenario-name CartPole-v1
```

## Key Parameters You Can Modify

When running training scripts, you can customize:

- `--env-name`: SingleControl, SingleCombat, MultipleCombat
- `--scenario-name`: Path to config file (without .yaml)
- `--algorithm-name`: ppo (single agent) or mappo (multi-agent)  
- `--experiment-name`: Your experiment name
- `--seed`: Random seed
- `--n-rollout-threads`: Parallel environments
- `--num-env-steps`: Total training steps
- `--use-selfplay`: Enable self-play training
- `--use-wandb`: Enable wandb logging

## Visualization and Rendering

After training, visualize your agents:

```bash
cd renders
python render_jsbsim.py
```

This generates `.acmi` files that can be opened with [TacView](https://www.tacview.net/) for 3D visualization of air combat.

## Example: Complete Training Pipeline

1. **Start with basic control**:
   ```bash
   cd scripts
   bash train_heading.sh
   ```

2. **Move to combat training**:
   ```bash
   bash train_selfplay.sh
   ```

3. **Visualize results**:
   ```bash
   cd renders  
   python render_jsbsim.py
   ```

4. **Open in TacView** to watch your trained agents fight!

## What You'll See

- **Training progress** in terminal showing episode rewards
- **Model checkpoints** saved in results directory
- **ACMI files** for 3D visualization
- **Wandb dashboards** (if enabled) for detailed metrics

## Advanced Usage

### Custom Scenarios
Edit YAML files in `envs/JSBSim/configs/` to create custom scenarios.

### Custom Algorithms  
The algorithms are in `algorithms/ppo/` and `algorithms/mappo/`.

### Custom Environments
The JSBSim environments are in `envs/JSBSim/envs/`.

## Troubleshooting

### Common Issues:
1. **CUDA out of memory**: Reduce `--n-rollout-threads`
2. **Training too slow**: Enable `--cuda` if you have GPU
3. **Visualization issues**: Make sure TacView is installed

### Environment Issues:
- We've fixed the Gymnasium API compatibility issues
- If you encounter "too many values to unpack" errors, the fix is already applied

## What Makes This Special

This isn't just another RL environment - it's a **realistic air combat simulator** that:
- Uses actual flight dynamics (JSBSim is used by aerospace industry)
- Implements realistic missile physics
- Supports advanced training techniques (self-play, hierarchy)
- Provides professional-grade visualization tools

Perfect for researching:
- Multi-agent reinforcement learning
- Self-play algorithms  
- Hierarchical control
- Combat AI systems

## Next Steps

1. **Start simple**: Train basic flight control first
2. **Progress gradually**: Move to 1v1 combat, then missiles, then 2v2
3. **Experiment**: Try different algorithms and hyperparameters
4. **Visualize**: Always render and watch your agents to understand behavior
5. **Research**: This is a great platform for multi-agent RL research!

The repository provides a complete research platform for air combat AI - from basic flight control to advanced multi-agent combat scenarios!