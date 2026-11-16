AVL optimizer, portable (12 threads)
====================================

Purpose
-------
Reference (AVL) optimization of the 12 original12 branches, one run per problem:
  run_avl_mtow.bat   minimise mtow_out        (H is a design variable, 500..10000 m)
  run_avl_qt.bat     minimise q_g_per_ton_km  (H fixed at 500 m)
The companion MLP repository uses the same source and problem definition with the
aerodynamic backend replaced by two trained MLPs. Both use the same initial populations (init_mtow, init_h500)
and the same default seed, so the first MLP run is a paired run with this one.

Requirements
------------
An installed 64-bit Python with numpy, pandas, scipy, joblib, openpyxl. Nothing is installed or downloaded.
SETUP_ENV.bat picks the first working Python (PYTHON_EXE, python, py); set PYTHON_EXE to force one.

Run
---
  run_avl_mtow.bat
  run_avl_qt.bat
12 worker processes by default (OPT_CORES=12). Output: gen_avl_<obj>_np140_s<seed>\<branch>\ with run_manifest.json
(seed, settings, SHA-256 of sources, avl.exe and initial populations).

To split the 12 branches over machines set ONLY_BRANCHES and OUTDIR before the call, e.g.
  set ONLY_BRANCHES=normal_1_1,normal_1_2,normal_1_3,duck_1_x
  set OUTDIR=gen_avl_mtow_part1
  run_avl_mtow.bat
(the seed of a branch does not depend on which branches run: seed = OPT_SEED + branch index).

Operation (workers, timing, resume)
-----------------------------------
  - At most 12 worker processes (OPT_CORES is capped at 12 and at the logical cores of the machine).
  - SETUP_ENV.bat only looks for an installed Python; AERO_SKIP_CHECK=1 skips even that check
    (PYTHON_EXE, or 'python' from PATH, is then used).
  - Timing: <OUTDIR>\<branch>\timing.csv (one row per generation: population, evaluations, seconds, elapsed,
    upper-bound ETA, feasible count, best fitness), <branch>\branch_summary.json, <OUTDIR>\timing_summary.json.
    The ETA is an upper bound: it assumes the whole evaluation budget; the spread-stop usually ends earlier.
  - Resume: a checkpoint (checkpoint.npz, previous copy checkpoint_prev.npz) is written after every generation.
    If the run is stopped, call the SAME .bat again with the SAME OUTDIR: finished branches are skipped
    (branch_done.json) and the interrupted one continues from the last finished generation. The checkpoint stores
    the random-number state, so a resumed run gives the same result as an uninterrupted one. It refuses to resume
    if bounds, settings or the initial population changed. OPT_RESUME=0 starts a branch again.
    resume_log.txt records every restart.

Problem definition (fixed in the .bat files)
--------------------------------------------
  mission 3000 km, payload 600 kg, SHADE NP=140 per branch, epsilon 0.0031
  cy <= 0.6, |mz| residual bias 0.001, pitch damping mz_omegaz <= -20,
  mx_beta <= 0, my_beta <= 0, A in [0.5, 1.2], alpha in [-10, 10], delta in [-5, 5]
  m0 feedback (relax 0.5), spread stop (feasible ratio 0.1), penalty R = 100
  sweep of both wings (f_sweep, a_sweep): 0..20 deg
  margin = (x_cg - x_np)/MAC: -0.30..-0.05 (static margin >= 5% MAC; negative = stable)
  H: 500..10000 m for the MTOW problem, fixed at 500 m for the q_T problem
  initial populations init_mtow / init_h500: seed 20260526, NP=140; np.random seed per branch = OPT_SEED (default 20261112) + branch index
