# AI Traffic Signal Control — 4-Way Intersection

A Deep Q-Network (DQN) agent that learns to optimize traffic signal timings at a single 4-way intersection, built with **SUMO** (Simulation of Urban Mobility) and **SUMO-RL**.

## Overview

Traditional traffic signals use fixed-time cycles regardless of actual traffic conditions. This project replaces fixed-time control with a reinforcement learning agent (DQN) that observes real-time traffic state — queue lengths, vehicle densities, current signal phase — and dynamically selects the optimal green phase to minimize vehicle waiting time.

### Key Features

- **Single 4-way intersection** with 4 approaches (North, South, East, West), 2 lanes each
- **Indian left-hand traffic** (LHT) configuration
- **DQN agent** trained via Stable-Baselines3 with experience replay and target networks
- **Reward function**: `diff-waiting-time` — agent is rewarded for reducing cumulative vehicle delay
- **Automated evaluation**: head-to-head comparison of RL agent vs fixed-time signals
- **Result visualization**: training curves, bar charts, and per-episode comparisons

## Architecture

```
┌──────────────┐     TraCI      ┌──────────────┐    Gymnasium    ┌──────────────┐
│     SUMO     │ ◄────────────► │   sumo-rl    │ ◄────────────► │   SB3 DQN    │
│  (Simulator) │                │  (Env Wrap)  │                │   (Agent)    │
└──────────────┘                └──────────────┘                └──────────────┘
```

**Observation space**: queue lengths, vehicle densities, current phase (per lane)
**Action space**: select next green phase for the traffic light
**Reward**: reduction in total cumulative waiting time between decisions

## Project Structure

```
sumo_4lane_intersection/
├── config.py               # All tunable parameters (network, DQN, evaluation)
├── generate_network.py     # Builds SUMO network (.net.xml) + vehicle routes
├── train.py                # DQN training (single-agent, Gymnasium interface)
├── evaluate.py             # RL vs fixed-time baseline comparison
├── plot_results.py         # Training curves + comparison charts (PNG)
├── run_all.py              # One-command full pipeline
├── requirements.txt        # Python dependencies
├── nets/                   # Generated SUMO network and route files
├── models/                 # Saved DQN model checkpoints
├── outputs/                # Training CSVs, TensorBoard logs, evaluation data
└── results/                # Output PNG charts
```

## Prerequisites

1. **SUMO** (Simulation of Urban Mobility)
   - Download: https://sumo.dlr.de/docs/Downloads.php
   - Set the `SUMO_HOME` environment variable:
     ```bash
     # Windows (System Environment Variables)
     SUMO_HOME = C:\Program Files (x86)\Eclipse\Sumo

     # Linux/Mac
     export SUMO_HOME=/usr/share/sumo
     ```

2. **Python 3.10+**

## Installation

```bash
cd sumo_4lane_intersection
pip install -r requirements.txt
```

## Usage

### Full Pipeline (one command)

```bash
python run_all.py
```

This runs all four steps sequentially: generate → train → evaluate → plot.

### Step-by-Step

```bash
# Step 1: Generate the SUMO network and vehicle demand
python generate_network.py

# Step 2: Train the DQN agent (100,000 timesteps)
python train.py

# Step 3: Evaluate RL vs fixed-time signals (5 episodes each)
python evaluate.py

# Step 4: Generate result plots
python plot_results.py
```

### Useful Flags

| Command | Flag | Description |
|---|---|---|
| `train.py` | `--gui` | Open SUMO-GUI to watch vehicles during training |
| `train.py` | `--timesteps N` | Set total training timesteps (default: 100,000) |
| `evaluate.py` | `--gui` | Watch the trained agent control traffic in SUMO-GUI |
| `evaluate.py` | `--episodes N` | Number of evaluation episodes (default: 5) |
| `evaluate.py` | `--no-baseline` | Skip fixed-time comparison |

### Visual Inspection

```bash
# View the intersection in SUMO-GUI (after generating network)
sumo-gui nets/intersection.sumocfg
```

### TensorBoard

```bash
tensorboard --logdir outputs/tb_logs
```

## Results

After evaluation, the comparison between RL (DQN) and Fixed-Time signals:

| Metric | RL (DQN) | Fixed-Time | Improvement |
|---|---|---|---|
| Avg Waiting Time (s) | 1.33 | 4.00 | **+66.6%** |
| Avg Speed (m/s) | 8.43 | 7.96 | **+5.8%** |
| Total Stopped Vehicles | 7.32 | 10.31 | **+29.0%** |
| Total Waiting Time (s) | 56.86 | 155.23 | **+63.4%** |

Result charts are saved to `results/`:
- `training_curves.png` — DQN training progress over 100K steps
- `comparison_metrics.png` — Grouped bar chart with improvement percentages
- `episode_comparison.png` — Per-episode waiting time line plot

## Configuration

All parameters are centralized in `config.py`:

| Parameter | Default | Description |
|---|---|---|
| `NUM_LANES` | 2 | Lanes per direction on each approach |
| `SPEED_LIMIT` | 13.89 m/s | ~50 km/h, typical Indian urban road |
| `LEFT_HAND_TRAFFIC` | True | Indian left-hand driving |
| `DELTA_TIME` | 5s | Seconds between agent decisions |
| `YELLOW_TIME` | 3s | Yellow phase duration |
| `LEARNING_RATE` | 0.001 | DQN learning rate |
| `NET_ARCH` | [256, 256] | MLP hidden layer sizes |
| `TOTAL_TIMESTEPS` | 100,000 | Training duration |
| `REWARD_FN` | diff-waiting-time | Reward = reduction in cumulative delay |

## Technologies

- **SUMO** — microscopic traffic simulation
- **sumo-rl** — Gymnasium wrapper for SUMO traffic signal control
- **Stable-Baselines3** — DQN implementation with experience replay
- **Gymnasium** — RL environment interface
- **Matplotlib / Pandas** — result visualization and data processing
- **TensorBoard** — live training monitoring
