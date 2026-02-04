# 🎯 Baseline Comparison Study Plan
**Date:** February 3, 2026  
**Goal:** Prove SAC-21D (with A* guidance) outperforms SAC-17D and all baseline methods  
**Expected Duration:** 6-8 hours (training) + 2 hours (testing) = ~10 hours total  
**Success Criteria:** SAC-21D achieves ≥60% overall success rate

---

## 📊 Experimental Design

### Methods to Compare (7 Total)

**Group A: Classical Shortest Path (No Flood Awareness)**
- [ ] 1. Pure Dijkstra
- [ ] 2. Pure A*

**Group B: Static Flood-Weighted Shortest Path**
- [ ] 3. Flood-Weighted Dijkstra
- [ ] 4. Flood-Weighted A*

**Group C: Dynamic Reactive Methods**
- [ ] 5. Hybrid A* (action-selection mode)

**Group D: Reinforcement Learning Agents**
- [ ] 6. SAC-17D (baseline RL, no A* guidance)
- [ ] 7. SAC-21D (enhanced RL, with A* guidance) ⭐

---

## 🔧 Implementation Checklist

### Phase 1: Environment Optimization (30 min)
- [ ] **Task 1.1:** Reduce flood sources from 51 to 20 (0.5% lowest elevation)
  - File: `backend/src/environment/flood_env.py`
  - Justification: Realistic flash flood scenario
  - Location: Line ~85-95 (flood source initialization)

- [ ] **Task 1.2:** Add seed control for reproducible tests
  - Add `random_seed` parameter to `FloodRoutingEnv.__init__`
  - Set numpy/random seeds in `reset()`

- [ ] **Task 1.3:** Verify current configuration
  - Propagation speed: 0.01 m/step ✅ (already set)
  - Death threshold: 0.8m ✅
  - Max steps: 1500 ✅

### Phase 2: Create 17-Dim Environment (1 hour)
- [ ] **Task 2.1:** Create `flood_env_17dim.py` (copy of flood_env.py)
  - Remove A* guidance from `_get_observation()` (lines 223-242)
  - Change observation_space to shape=(17,)
  - Remove `astar_guidance_cache` initialization
  - Keep all other features identical

- [ ] **Task 2.2:** Create `train_17dim.py`
  - Copy from `train_main.py`
  - Use FloodRoutingEnv17D
  - state_dim=17
  - Model name: `sac_17dim_baseline.pt`
  - Episodes: 6000 (1000 per stage)
  - Save metrics: `sac_17dim_metrics.csv`, `sac_17dim_losses.csv`

- [ ] **Task 2.3:** Create `run_training_17dim.py`
  - Wrapper script for 17-dim training
  - Prints start/end times, progress

### Phase 3: Update 21-Dim Training (30 min)
- [ ] **Task 3.1:** Create `train_21dim.py`
  - Copy from current `train_main.py`
  - Model name: `sac_21dim_enhanced.pt`
  - Episodes: 6000 (1000 per stage)
  - Save metrics: `sac_21dim_metrics.csv`, `sac_21dim_losses.csv`

- [ ] **Task 3.2:** Create `run_training_21dim.py`
  - Wrapper script for 21-dim training

### Phase 4: Implement Baseline Runners (2 hours)
- [ ] **Task 4.1:** Create `baseline_methods.py`
  - Class: `DijkstraRunner` (uses nx.dijkstra_path)
  - Class: `AStarRunner` (uses nx.astar_path)
  - Class: `FloodWeightedDijkstra` (edge weight = length + flood_penalty)
  - Class: `FloodWeightedAStar` (same with heuristic)
  - Class: `HybridAStarRunner` (picks action from A* path each step)
  - All return: (success, path_cost, steps_taken, death_cause)

- [ ] **Task 4.2:** Add flood weight calculation helper
  ```python
  def compute_flood_weight(edge_length, flood_depth, penalty=10.0):
      return edge_length + (flood_depth * penalty)
  ```

### Phase 5: Create Test Harness (2 hours)
- [ ] **Task 5.1:** Create `test_all_methods.py`
  - Load graph and environment
  - Generate 50 random origin-shelter pairs per stage (250 total)
  - For each method:
    * Run all 250 test cases
    * Track: success_rate, avg_path_cost, avg_steps, death_rate, timeout_rate
  - Save to `comparison_results.csv`

- [ ] **Task 5.2:** Add per-stage metrics tracking
  - Columns: method, stage, success_rate, avg_cost, avg_steps, deaths, timeouts

- [ ] **Task 5.3:** Add seed control for reproducibility
  - Same test pairs for all methods

### Phase 6: Training Execution (6-8 hours)
- [ ] **Task 6.1:** Start 17-dim training (Terminal 1)
  ```bash
  cd /Users/robbieespaldon/Code/pythontest/pyqgis/evacuafie_v2/sandbox
  conda run -n pt1 python run_training_17dim.py
  ```
  - ETA: ~3-4 hours
  - Monitor: Watch success rates per stage

- [ ] **Task 6.2:** Start 21-dim training (Terminal 2 - parallel)
  ```bash
  cd /Users/robbieespaldon/Code/pythontest/pyqgis/evacuafie_v2/sandbox
  conda run -n pt1 python run_training_21dim.py
  ```
  - ETA: ~3-4 hours
  - Monitor: Watch success rates per stage

- [ ] **Task 6.3:** Verify caffeinate still running
  ```bash
  ps aux | grep caffeinate
  ```

- [ ] **Task 6.4:** Check model saves periodically
  - `ls -lh backend/models/sac_17dim_baseline.pt`
  - `ls -lh backend/models/sac_21dim_enhanced.pt`

### Phase 7: Baseline Testing (2 hours)
- [ ] **Task 7.1:** Run comprehensive test
  ```bash
  conda run -n pt1 python test_all_methods.py
  ```
  - Tests all 7 methods
  - 250 test cases each = 1750 total episodes
  - ETA: ~1-2 hours

- [ ] **Task 7.2:** Verify results file
  - Check `comparison_results.csv` has 1750 rows (7 methods × 250 tests)

### Phase 8: Generate Visualizations (1 hour)
- [ ] **Task 8.1:** Create `generate_comparison_plots.py`
  - **Plot 1:** Success rate by method and stage (grouped bar chart)
  - **Plot 2:** Overall success rate ranking (horizontal bar chart)
  - **Plot 3:** Path cost comparison (box plots by method)
  - **Plot 4:** Steps-to-goal distribution (violin plots)
  - **Plot 5:** Death rate analysis (stacked bar: deaths, timeouts, success)

- [ ] **Task 8.2:** Create comparison summary table
  - Overall metrics per method
  - Statistical significance tests (SAC-21D vs SAC-17D)

---

## 📈 Expected Results (Hypothesis)

| Method | Type | Expected Success | Key Weakness |
|--------|------|------------------|--------------|
| Pure Dijkstra | Static shortest | 20-30% | Ignores floods entirely |
| Pure A* | Static shortest | 20-30% | Ignores floods entirely |
| Flood-Weighted Dijkstra | Static aware | 35-45% | Can't adapt to dynamic floods |
| Flood-Weighted A* | Static aware | 35-45% | Can't adapt to dynamic floods |
| Hybrid A* | Dynamic reactive | 40-55% | Greedy, no learning |
| **SAC-17D** | RL baseline | **45-55%** | No global path guidance |
| **SAC-21D** | RL enhanced | **60-70%** ⭐ | Has A* guidance + learning |

**Key Defense Points:**
1. SAC-21D should outperform SAC-17D by 10-15% → **proves A* guidance value**
2. Both SAC models should beat static methods → **proves learning value**
3. SAC-21D should beat Hybrid A* → **proves learning > greedy decisions**

---

## ⚠️ Risk Mitigation

### Risk 1: SAC-17D performs better than SAC-21D (Unlikely)
**Mitigation:** 
- Check if A* guidance causing confusion (too many features)
- Verify 21-dim network has enough capacity (512 hidden dims ✅)
- Check if A* cache working properly

### Risk 2: Both SAC models perform poorly (<40%)
**Mitigation:**
- Reduce flood sources to 15 (0.3% lowest)
- Slow propagation to 0.005 m/step
- Increase training to 12,000 episodes
- Check curriculum difficulty (Stage 1 should be >70% success)

### Risk 3: Training crashes overnight
**Mitigation:**
- Use `nohup` and redirect output to log files
- Save checkpoints every 500 episodes
- Monitor disk space

### Risk 4: Baselines perform unexpectedly well (>60%)
**Interpretation:**
- Good news! Problem is solvable
- Proves environment is fair
- SAC models still show learning capability

---

## 🚀 Execution Timeline

**Day 1 (Today):**
- [ ] Hour 1-2: Implement Phase 1-3 (environment + training scripts)
- [ ] Hour 3-4: Implement Phase 4-5 (baseline runners + test harness)
- [ ] Hour 5: Start both trainings (17D + 21D parallel)
- [ ] Hour 6-12: Overnight training (sleep, caffeinate handles it)

**Day 2 (Tomorrow):**
- [ ] Morning: Verify training completed successfully
- [ ] Hour 1-2: Run comprehensive testing (test_all_methods.py)
- [ ] Hour 3-4: Generate visualizations and tables
- [ ] Hour 5: Analysis and thesis writing prep

**Total Expected Time:** ~16 hours (mostly automated)

---

## 📝 Files to Create

1. `/backend/src/environment/flood_env_17dim.py` - 17-dim environment
2. `/sandbox/train_17dim.py` - 17-dim training script
3. `/sandbox/run_training_17dim.py` - 17-dim wrapper
4. `/sandbox/train_21dim.py` - 21-dim training script
5. `/sandbox/run_training_21dim.py` - 21-dim wrapper
6. `/sandbox/baseline_methods.py` - All baseline runners
7. `/sandbox/test_all_methods.py` - Comprehensive test harness
8. `/sandbox/generate_comparison_plots.py` - Visualization generator

**Output Files:**
- `backend/models/sac_17dim_baseline.pt`
- `backend/models/sac_21dim_enhanced.pt`
- `sandbox/sac_17dim_metrics.csv`
- `sandbox/sac_21dim_metrics.csv`
- `sandbox/comparison_results.csv`
- `sandbox/comparison_plots/` (directory with 5+ plots)

---

## ✅ Success Criteria

- [ ] Both models train to completion (6000 episodes each)
- [ ] All 7 methods tested on same 250 test cases
- [ ] SAC-21D achieves ≥60% overall success rate
- [ ] SAC-21D > SAC-17D by ≥10 percentage points
- [ ] SAC-21D > all baseline methods
- [ ] Statistical significance confirmed (p < 0.05)
- [ ] Visualizations clearly show performance hierarchy
- [ ] Results are reproducible (seed control working)

---

## 🎓 Thesis Defense Talking Points

**Research Question:** Does adding A* guidance to SAC improve flood evacuation routing?

**Methodology:** Ablation study comparing 7 methods in dynamic flood environment

**Key Finding 1:** "SAC-21D achieved X% success vs SAC-17D's Y%, demonstrating a Z% improvement from A* guidance integration."

**Key Finding 2:** "RL-based methods outperformed classical shortest path algorithms by [X]%, showing the value of adaptive learning in dynamic environments."

**Key Finding 3:** "Flood-aware static methods achieved [X]% vs dynamic methods' [Y]%, highlighting the importance of real-time adaptation."

**Justification for A* Guidance:** "Without global path information (17-dim), the agent struggled to navigate efficiently. Adding A* guidance (21-dim) provided directional hints while preserving the agent's ability to deviate for flood avoidance."

---

## 📊 Current Status

**Date:** Feb 3, 2026  
**Time:** Implementation Complete - Ready for Training

### ✅ Completed (Phases 1-5 + 8):
- [x] Plan created ✅
- [x] Research design finalized ✅
- [x] Environment optimization (flood sources 51→20, seed control) ✅
- [x] 17-dim environment (flood_env_17dim.py) ✅
- [x] 21-dim environment ready ✅
- [x] SAC agent architecture ready ✅
- [x] Training scripts (train_17dim.py, train_21dim.py) ✅
- [x] Wrapper scripts (run_training_17dim.py, run_training_21dim.py) ✅
- [x] Baseline runners (5 methods in baseline_methods.py) ✅
- [x] Test harness (test_all_methods.py) ✅
- [x] Visualization script (generate_comparison_plots.py) ✅

### 🚀 Ready to Execute:
- [ ] **Phase 6:** Training (17D + 21D parallel, ~6-8 hours)
- [ ] **Phase 7:** Testing (all 7 methods, ~1-2 hours)

---

**Next Action:** Start Phase 6 - Execute Training in 2 parallel terminals
