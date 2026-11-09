# Wi-Fi 6 vs. Wi-Fi 7: ns-3 Performance Evaluation

Source code and experimental data for the comparative evaluation of Wi-Fi 6 (IEEE 802.11ax) and Wi-Fi 7 (IEEE 802.11be) presented in the undergraduate thesis:

> **Análise do Wi-Fi 7 (802.11be): Tecnologias, Desafios e Performance em Futuras WLANs**  
> Nicholas Francis Lambert — Universidade Estadual de Londrina (UEL), 2026.

The experiments use **ns-3.47** and compare four configurations: Wi-Fi 6 with a single link, Wi-Fi 7 with a single link, Wi-Fi 7 using Multi-Link Operation (MLO) in STR mode, and Wi-Fi 7 using MLO in eMLSR mode.

## Repository contents

```text
simulacao_ns3/
├── wifi67-tcc.cc              # C++ ns-3 simulation program
├── run_experimentos.py        # experiment runner and CSV collection
├── analisar_resultados.py     # statistical analysis and figure/table generation
└── dados/
    └── resultados.csv         # per-run results used in the thesis
```

The committed `resultados.csv` contains **305 simulation observations** from experiments E1, E2 and E3, using five replications (runs 1–5) per experimental point. It is included so readers can inspect and reproduce the analysis without first rerunning the simulations.

## Requirements

- Linux or Windows with WSL2.
- [ns-3.47](https://www.nsnam.org/), built with the `wifi`, `applications`, `flow-monitor`, `internet`, `mobility`, `propagation` and `spectrum` modules.
- A C++ compiler and build tools required by ns-3.47.
- Python 3.9 or newer.
- Python packages listed in [`requirements.txt`](requirements.txt): NumPy, pandas and Matplotlib.

## Reproducing the experiments

### 1. Install Python dependencies

From the repository root:

```bash
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install -r requirements.txt
```

On Windows, run these commands from a WSL2 Linux distribution. The activation command may differ if you use a different shell.

### 2. Build ns-3.47

Download and extract ns-3.47 from the [official ns-3 website](https://www.nsnam.org/). Set `NS3_DIR` to the extracted `ns-3.47` directory (the directory containing the `ns3` script):

```bash
export NS3_DIR="$HOME/ns-3.47"
cd "$NS3_DIR"
./ns3 configure --build-profile=optimized --enable-modules="wifi;applications;flow-monitor;internet;mobility;propagation;spectrum"
./ns3 build
```

### 3. Add the simulation program to ns-3

```bash
cp /path/to/this/repository/simulacao_ns3/wifi67-tcc.cc "$NS3_DIR/scratch/"
cd "$NS3_DIR"
./ns3 build
```

Replace `/path/to/this/repository` with the local path where this repository was cloned or extracted.

### 4. Run a short validation test

From the repository root, with `NS3_DIR` set:

```bash
python3 simulacao_ns3/run_experimentos.py \
  --ns3-dir "$NS3_DIR" \
  --quick \
  --saida simulacao_ns3/dados/quick
```

The quick mode uses a reduced set of points, two replications and a shorter simulation window. It is intended only to check that the toolchain and pipeline work; it does **not** reproduce the final thesis dataset.

### 5. Run the complete experiment set

```bash
python3 simulacao_ns3/run_experimentos.py \
  --ns3-dir "$NS3_DIR" \
  --jobs 2 \
  --saida simulacao_ns3/dados/reproduzidos
```

The full run uses the default experiment set (E1, E2 and E3), five replications per point, a 0.5 s warm-up period and a 5 s measurement window. Depending on the machine, a complete run can take several hours.

The runner is resumable: rerunning the same command skips completed points already present in the output CSV. The newly generated data are written to `simulacao_ns3/dados/reproduzidos/resultados.csv`; the committed reference dataset at `simulacao_ns3/dados/resultados.csv` is left unchanged.

Useful options:

- `--only E1,E3` runs selected experiments.
- `--runs 5` controls replications per experimental point.
- `--jobs 2` controls the number of parallel simulations.
- `--dry-run` lists the simulation commands without executing them.

### 6. Recreate the figures and tables

After a complete run:

```bash
python3 simulacao_ns3/analisar_resultados.py \
  --csv simulacao_ns3/dados/reproduzidos/resultados.csv \
  --latex-dir reproducao_saida
```

The script writes generated figures to `reproducao_saida/figuras/` and LaTeX tables/macros to `reproducao_saida/resultados/`. These outputs are kept separate from the committed source dataset and from the thesis source files.

To regenerate the analysis using the already published dataset, omit the experiment-running step and use:

```bash
python3 simulacao_ns3/analisar_resultados.py \
  --csv simulacao_ns3/dados/resultados.csv \
  --latex-dir reproducao_saida
```

## Experimental scenarios

- **E1 — Saturated traffic:** aggregate throughput as the number of stations increases.
- **E2 — Channel width:** measured throughput and theoretical PHY rate as channel bandwidth changes.
- **E3 — Non-saturated traffic:** end-to-end delay, channel-access latency, delivered throughput and packet loss under CBR traffic.

The exact topology, channel model, traffic settings, fixed MCS values and other simulation parameters are documented in Chapter 4 of the thesis.

## Attribution

The C++ simulation program adapts the ns-3.47 example `examples/wireless/wifi-eht-network.cc`, originally authored by **Sébastien Deronne**. The original example and the adapted C++ program are distributed under GPL-2.0-only. The ns-3 simulator itself is a separate dependency and is not included in this repository.

## Citation

If you use this code or the supplied dataset in academic work, please cite:

> Lambert, N. F. (2026). *Análise do Wi-Fi 7 (802.11be): Tecnologias, Desafios e Performance em Futuras WLANs*. Trabalho de Conclusão de Curso, Bacharelado em Ciência da Computação, Universidade Estadual de Londrina, Londrina, Brasil.

A machine-readable citation is provided in [`CITATION.cff`](CITATION.cff).

## License

- **Software source code:** GNU General Public License v2.0 only (`GPL-2.0-only`). See [`LICENSE`](LICENSE).
- **Experimental dataset** (`simulacao_ns3/dados/resultados.csv`): Creative Commons Attribution 4.0 International (`CC BY 4.0`). See [`DATA-LICENSE.md`](DATA-LICENSE.md).
