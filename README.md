# QUANTA

QUANTA estimates species trees from weighted quartets. It keeps the
divide-and-conquer framework of wQFM and solves each bipartition step as a
QUBO on a quantum annealer (D-Wave).

## Installation

Tested on Ubuntu 24.04 with Python 3.12 and OpenJDK 21. Run these commands in order:

```bash
# System packages (Ubuntu/Debian)
sudo apt update
sudo apt install -y git python3.12 python3.12-venv openjdk-21-jdk ant time

# Get the code
git clone https://github.com/QuitttCat/QUANTA-Phylogenomics.git
cd QUANTA-Phylogenomics

# Python environment and packages (the run scripts expect it at QWQFM/.venv)
python3.12 -m venv QWQFM/.venv
QWQFM/.venv/bin/pip install --upgrade pip
QWQFM/.venv/bin/pip install -r requirements.txt

# Build QUANTA and wQFM
(cd QWQFM/WQFM && ant jar)

# Build QUANTA-Tree
(cd wqfm-tree/quanta-tree && javac -d build -sourcepath . src/Main.java)
```

### Baselines and wQFM-TREE

The ASTRAL (ASTER) and wQMC programs are included in `baselines/tools/`, so they need no
installation (see [baselines/commands.md](baselines/commands.md)).

The official wQFM-TREE repository is needed only to rerun wQFM-TREE or to redraw the 500-
and 1000-taxon figures:

```bash
git clone https://github.com/abdur-rafi/wQFM-TREE.git wqfm-tree/official
git -C wqfm-tree/official checkout 7d7c24b
```

## Input data

The weighted-quartet inputs are about 15 GB, too large for GitHub. They are
shared as one compressed archive per dataset (2.0 GB in total) here:

**[QUANTA input files (Google Drive)](https://drive.google.com/drive/folders/146aYkrE9gcgDiJst78e6Bp7wWcDZxZVR?usp=sharing)**

Download the archives you need and `SHA256SUMS.txt` into `QWQFM/WQFM/input_files/`, where
all run scripts look for their input. Then check and unpack them from the repository root:

```bash
cd QWQFM/WQFM/input_files
sha256sum -c --ignore-missing SHA256SUMS.txt
for f in *.tar.gz; do tar -xzf "$f"; done
cd ../../..
```

| Archive | Contents | Size (compressed) |
|---|---|---|
| `11-taxa-weighted-quartets` | 11-taxon simulated dataset: 200 files (estimated and true gene trees, 5–100 genes, strong ILS, 20 replicates) | < 1 MB |
| `15-taxa-weighted-quartets` | 15-taxon simulated dataset: 60 files (100 and 1000 genes; 100 bp, 1000 bp and true gene trees; 10 replicates) | < 1 MB |
| `37-taxa-weighted-quartets` | 37-taxon mammalian simulated dataset: 240 files (12 conditions × 20 replicates) | 146 MB |
| `48-taxa-weighted-quartets` | 48-taxon avian simulated dataset: the 114 files used in our experiments | 267 MB |
| `48-taxa-weighted-quartets_full_data` | all 140 files of the 48-taxon dataset (7 conditions × 20 replicates) | 327 MB |
| `101-taxa-weighted-quartets` | 101-taxon simulated dataset: 19 replicates, about 12.2 million quartets each | 845 MB |
| `101-taxa-true-species-tree` | true species trees of the 101-taxon replicates | < 1 MB |
| `150-taxa-s201-rep01-true-gene-trees` | 150-taxon true gene trees (s201, replicate 1) | < 1 MB |
| `150-taxa-s201-rep01-true-gene-trees-weighted-quartets` | weighted quartets of the 150-taxon gene trees | 192 MB |
| `200-taxa-s200-rep01-true-gene-tree` | 200-taxon true gene tree (s200, replicate 1) | < 1 MB |
| `200-taxa-s200-rep01-true-gene-tree-weighted-quartets` | weighted quartets of the 200-taxon gene tree | 170 MB |
| `biological_dataset_weigted_quartets` | biological datasets; the paper uses Amniota (AA, NT), avian (`avian-all`) and mammalian | 14 MB |
| `gtree_11tax_sim_5gR1`, `15_taxa`, `37_taxa` | one example input file each, for quick tests | < 1 MB |

The simulated and biological datasets come from earlier studies, collected in
the wQFM dataset release (see [QWQFM/README.md](QWQFM/README.md#datasets));
please cite their original publications when you use them.

### QUANTA-Tree: 500- and 1000-taxon datasets

The inputs of the QUANTA-Tree experiments on the ASTRAL-II 500- and 1000-taxon
datasets are shared separately, as one zip (28 MB):

**[QUANTA-Tree input files (Google Drive)](https://drive.google.com/file/d/13blEfFAYV6pPPoZEQbUKPa2jEgZp5mzU/view)**

Google Drive says it can't scan the file for viruses because of its size; choose
**Download anyway**. Or download it from the command line:

```bash
QWQFM/.venv/bin/pip install gdown
QWQFM/.venv/bin/gdown --fuzzy "https://drive.google.com/file/d/13blEfFAYV6pPPoZEQbUKPa2jEgZp5mzU/view" -O 500-1000-taxa-astral2-resolved-gene-trees.zip
```

Check it and unpack it into `wqfm-tree/`, where `astral2.py` looks for its input:

```bash
sha256sum 500-1000-taxa-astral2-resolved-gene-trees.zip
# 6660200354078113abf48d93ca0ce596c282a850c56ef00369625ffea9b8da7a
python3 -m zipfile -e 500-1000-taxa-astral2-resolved-gene-trees.zip wqfm-tree/
```

`README.txt` in the zip describes each file and which replicates the paper uses.

## Reproducing the results

Run every command from the repository root, after [installing QUANTA](#installation) and
downloading the [input data](#input-data). All paths are relative to the root.

QUANTA solves partitions of at most 20 taxa by exhaustive search, so only larger partitions
need a solver. Simulated annealing (`sa`) and tabu search (`tabu`) run on your own computer
and need no account, so you can reproduce every dataset below without D-Wave. Reruns are
written to `my_runs/`, which leaves our stored results untouched. To run N inputs at once, put
`JOBS=N` in front of a command.

### 11 and 15 taxa

Every partition here has at most 20 taxa, so QUANTA uses only exhaustive search. Simulated
annealing, tabu search and D-Wave are never called, and these commands need no token:

```bash
bash "result_analysis/11 Taxa/run_11_taxa.sh" my_results_11_taxa.csv
bash "result_analysis/15 Taxa/run_15_taxa.sh" my_results_15_taxa.csv
```

Each script also runs wQFM, then writes its CSV to `result/` and its figure to `figure/` inside
the dataset's folder in `result_analysis/`. They reserve a 6 GB Java heap; on a smaller machine,
put `JAVA_TOOL_OPTIONS="-Xmx2g"` in front of the command.

### 37 and 48 taxa

```bash
for solver in sa tabu; do
    WORK_DIR=my_runs/37_$solver RESULT_CSV=my_runs/37_$solver.csv \
        bash "result_analysis/37 Taxa/run_37_taxa_classical.sh" "$solver"
    WORK_DIR=my_runs/48_$solver RESULT_CSV=my_runs/48_$solver.csv \
        bash "result_analysis/48 Taxa/run_48_taxa_classical.sh" "$solver"
done
```

### 101 taxa

```bash
for solver in sa tabu; do
    JAVA_TOOL_OPTIONS="-Xmx4g -XX:+UseG1GC" WORK_DIR=my_runs/101_$solver RESULT_CSV=my_runs/101_$solver.csv \
        bash "result_analysis/classical solvers/run_classical.sh" 101 "$solver"
done
```

### 200 taxa

This input has about 64.7 million quartets, and the script sets a 26 GB Java heap, so it needs a
large-memory machine:

```bash
for solver in sa tabu; do
    WORK_DIR=my_runs/200_$solver RESULT_CSV=my_runs/200_$solver.csv \
        bash "result_analysis/classical solvers/run_classical.sh" 200 "$solver"
done
```

### Biological datasets

```bash
for solver in sa tabu; do
    WORK_DIR=my_runs/biological_$solver RESULT_CSV=my_runs/biological_$solver.csv \
        bash "result_analysis/classical solvers/run_classical.sh" biological "$solver"
done
```

These commands run the biological datasets in the paper: avian (`avian-all`), mammalian and
Amniota (AA and NT). In our runs, both solvers gave the avian tree a quartet score of
1,228,827,664, which is 108,551 above wQFM's 1,228,719,113.

### 500 and 1000 taxa (QUANTA-Tree)

Unpack the [QUANTA-Tree input files](#quanta-tree-500--and-1000-taxon-datasets) first. Our own
outputs are stored in `wqfm-tree/runs/a2r/`, and `astral2.py` reuses finished runs, so the first
command removes them. The paper's QUANTA-Tree results use simulated annealing:

```bash
cd wqfm-tree
rm -f runs/a2r/m*/*/g1000/quanta.tre runs/a2r/m*/*/g1000/quanta_solver.log

../QWQFM/.venv/bin/python astral2.py run --resolved --methods quanta --genes 1000 --taxa 500 \
    --reps 01 02 03 04 05 06 07 08 09 10 --java-opt=-Xmx5g --java-opt=-XX:+UseSerialGC
../QWQFM/.venv/bin/python astral2.py run --resolved --methods quanta --genes 1000 --taxa 1000 \
    --reps 01 02 03 05 07 09 10 12 13 14 --java-opt=-Xmx5g --java-opt=-XX:+UseSerialGC
cd ..
```



Results are written to `wqfm-tree/results/astral2_resolved_runs.csv`.

### D-Wave runs

If you can get an API token from [D-Wave Leap](https://cloud.dwavesys.com/leap/), you can run
QUANTA on D-Wave's quantum computers. Put `DWAVE_API_TOKEN=your-token` in a file named `.env` in
the repository root, then run the commands below. They use your D-Wave solver time.

```bash
bash "result_analysis/37 Taxa/run_37_taxa.sh" my_results_37_taxa.csv
bash "result_analysis/48 Taxa/run_48_taxa.sh" my_results_48_taxa.csv
OUTPUT_CSV=my_runs/101_dwave.csv WORK_DIR=my_runs/101_dwave bash QWQFM/benchmark/scripts/run_101_taxa_benchmark.sh
OUTPUT_CSV=my_runs/biological_dwave.csv WORK_DIR=my_runs/biological_dwave bash QWQFM/benchmark/scripts/run_biological_benchmark.sh

# 200 taxa: the root partition goes to D-Wave's hybrid solver, the others to the quantum computer
mkdir -p my_runs
source load-dwave-token.sh
(cd QWQFM/WQFM && JAVA_TOOL_OPTIONS="-Xms4g -Xmx26g -XX:+UseG1GC" bash run-wqfm.sh -o ../../my_runs/200_dwave.tre -algo 2 \
    -i input_files/200-taxa-s200-rep01-true-gene-tree-weighted-quartets/s200_rep01_true_gene_tree_001_200_taxa.weighted_quartets)
```

Without a token, you can read the results and logs of our runs instead:

| Dataset | D-Wave results | D-Wave logs |
|---|---|---|
| 37 taxa | `result_analysis/37 Taxa/result/results_37_taxa_qpu_plus_bruteforce_n>20.csv` |  |
| 48 taxa | `result_analysis/48 Taxa/result/results_48_taxa_z12_qpu_plus_bruteforce_n>20.csv` | `QWQFM/benchmark/work/48_taxa_z12_qpu_plus_bruteforce_n_gt_20/` |
| 101 taxa | `result_analysis/101 Taxa/result/results_101_taxa_qpu_plus_bruteforce.csv` | `QWQFM/benchmark/work/101_taxa_qpu_plus_bruteforce/` |
| 200 taxa | `QWQFM/benchmark/output_csv/results_200_taxa_qpu_hybrid_plus_bruteforce.csv` | `QWQFM/benchmark/work/200_taxa_full/` |
| Amniota and mammalian | `QWQFM/benchmark/output_csv/biological/results_biological_qpu_plus_bruteforce_n>20.csv` | `QWQFM/benchmark/work/biological_qpu_plus_bruteforce/` |
| Avian (`avian-all`) | Same CSV | Overwritten (see below) |

The D-Wave log of the `avian-all` result in the paper was mistakenly overwritten by a later run,
so the `avian-all` folder under `biological_qpu_plus_bruteforce/` does not match the tree in the
CSV. Simulated annealing and tabu search give exactly the same tree as the CSV, with a quartet score
of 1,228,827,664. To check this without a token (about 3 minutes in total):

```bash
for solver in sa tabu; do
    INPUT_NAMES="avian-all.weighted_quartets" WORK_DIR=my_runs/avian_$solver RESULT_CSV=my_runs/avian_$solver.csv \
        bash "result_analysis/classical solvers/run_classical.sh" biological "$solver"
    grep -qF "$(cat my_runs/avian_$solver/avian-all.weighted_quartets/quantum.tre)" \
        "QWQFM/benchmark/output_csv/biological/results_biological_qpu_plus_bruteforce_n>20.csv" \
        && echo "$solver: same tree" || echo "$solver: different tree"
done
```

[This figure](result_analysis/48%20Taxa/figure/avian-quantum.pdf) shows the estimated trees on the
avian dataset: (a) binned MP-EST (published reference), (b) QUANTA and (c) wQFM.

Results and logs of the other solvers:

| Dataset | Results | Logs |
|---|---|---|
| 11 and 15 taxa | `result_analysis/<11 or 15> Taxa/result/results_<11 or 15>_taxa.csv` | `QWQFM/benchmark/work/results_<11 or 15>_taxa/` |
| 37 taxa | `result_analysis/37 Taxa/result/results_37_taxa_<solver>_plus_bruteforce_n>20.csv` | `QWQFM/benchmark/work/37_taxa_<solver>_n_gt_20/` |
| 48 taxa | `result_analysis/48 Taxa/result/results_48_taxa_<solver>_plus_bruteforce_n>20.csv` | `QWQFM/benchmark/work/48_taxa_<solver>_n_gt_20/` |
| 101 taxa | `result_analysis/classical solvers/result/results_101_<solver>_plus_bruteforce_n>20.csv` | `QWQFM/benchmark/work/101_<solver>_n_gt_20/` |
| 200 taxa | `result_analysis/200 Taxa/result/results_200_<solver>_plus_bruteforce_n>20.csv` | `QWQFM/benchmark/work/200_<solver>_n_gt_20/` |
| Biological | `result_analysis/classical solvers/result/results_biological_<solver>_plus_bruteforce_n>20.csv` | `QWQFM/benchmark/work/biological_<solver>_n_gt_20/` |
| 500 and 1000 taxa (QUANTA-Tree) | `wqfm-tree/results/astral2_resolved_runs.csv` | `wqfm-tree/runs/a2r/m<500 or 1000>/<replicate>/g1000/` |

`<solver>` is `simulated_annealing` or `tabu_search`, except for QUANTA-Tree, which used only
simulated annealing.
