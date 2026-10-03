<div align="center">

# TMDET-Python

**Predict where the lipid membrane sits around a transmembrane protein, from its 3D structure alone.**

A Python reimplementation of the TMDET algorithm, with PyMOL visualization.

![Python](https://img.shields.io/badge/python-3.x-3776AB?logo=python&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-vectorized-013243?logo=numpy&logoColor=white)
![PyMOL](https://img.shields.io/badge/visualization-PyMOL-1f6feb)
![License](https://img.shields.io/badge/license-MIT-green)

<!-- Add a screenshot of the PyMOL result here, e.g. docs/images/1PRN_pymol.png -->
<img src="docs/images/1PRN_pymol.png" alt="Predicted membrane planes around the porin 1PRN in PyMOL" width="600">

*Predicted membrane boundaries around the porin 1PRN.*

</div>

---

## About

Transmembrane proteins make up roughly a quarter of all proteins, but their structures in the Protein Data Bank (PDB) don't say where the membrane was. **TMDET** ([Tusnády, Dosztányi & Simon, 2004](https://doi.org/10.1093/bioinformatics/bth340)) answers that question geometrically: it searches for the orientation and position of a membrane slab that best matches the protein's exposed hydrophobic surface.

This project reproduces the core idea of TMDET in Python:

1. **Load and center** the protein. Cα coordinates are read from the PDB file and solvent accessibility (SASA) is computed with NACCESS.
2. **Generate candidate axes**: 1,000 directions evenly spread over a sphere (Fibonacci sphere).
3. **Scan**: for each axis, exposed residues are projected onto it and a 30 Å membrane slab is slid along it in 1 Å steps. Each position is scored with the Kyte-Doolittle hydrophobicity scale.
4. **Report and visualize** the best axis, its score and the membrane center, then draw the two membrane planes in PyMOL.

> Developed as part of the M2 Bioinformatics programme (Université Paris Cité, *Advanced programming and project management* course). The full report (in French) is in [`doc/STEPHEN_rapport.pdf`](doc/STEPHEN_rapport.pdf).

## Scoring methods

Two scoring strategies are implemented, each in a pure-Python and a NumPy-vectorized version:

| Method | Score | Idea |
|---|---|---|
| **Method 1** | Mean hydrophobicity of a 30 Å window in the 1 Å hydrophobicity profile | Finds the most hydrophobic slice of the protein |
| **Method 2** *(default)* | Mean hydrophobicity **inside** the slab minus mean hydrophobicity **outside** | Rewards contrast between the membrane core and the solvent-exposed regions |

The vectorized versions give the same results and run noticeably faster, especially for Method 2.

## Results

Predictions were compared with reference membrane positions from the [OPM database](https://opm.phar.umich.edu/):

| Protein | PDB | Fold | Outcome |
|---|---|---|---|
| Porin | [1PRN](https://www.rcsb.org/structure/1PRN) | β-barrel | ✅ Method 2 matches the OPM orientation |
| Bacteriorhodopsin | [1QHJ](https://www.rcsb.org/structure/1QHJ) | α-helical bundle | ⚠️ Axis misoriented |

The hydrophobicity-only score works well for β-barrels, whose regular hydrophobic belt gives a strong signal. For compact helical bundles, several axes give similar contrast. The original TMDET also uses geometric criteria (such as straightness of the membrane-crossing segments), which this implementation doesn't include yet. That's the main lead for improvement.

## Installation

**Requirements**

- Python 3.x and NumPy
- [PyMOL](https://pymol.org/) for visualization (e.g. `conda install -c conda-forge pymol-open-source`)
- [NACCESS](http://www.bioinf.manchester.ac.uk/naccess/) to compute solvent accessibility (`naccess` must be on your `PATH`)

```bash
git clone https://github.com/Eve-Ast/TMDET_BI.git
cd TMDET_BI
pip install -r requirements.txt
```

## Usage

### Predict the membrane position

```bash
python src/main.py 1PRN
python src/main.py 1PRN.pdb --method method1_vec
```

| Argument | Description |
|---|---|
| `pdb_file` | PDB file name or path. The `.pdb` extension is optional; bare names are looked up in `data/`. |
| `--method` | `method1_vec`, `method2_vec` *(default)*, `method1_novec` or `method2_novec` |

**Output:** the best membrane axis, its hydrophobicity score and the position of the membrane center. A PyMOL window then opens with the predicted membrane planes.

### Benchmark the methods

```bash
python src/benchmark.py 1PRN.pdb 1PRN.rsa 1000
```

Arguments: PDB file, RSA file, number of axes to test. The script prints the run time and score of each of the 4 methods.

## Project structure

```
.
├── src/
│   ├── main.py        # Command-line entry point
│   ├── Protein.py     # PDB/SASA parsing, centering, PyMOL rendering
│   ├── Grid.py        # Fibonacci sphere + the 4 scanning methods
│   └── benchmark.py   # Speed and score comparison of the methods
├── data/              # Example PDB structures (1F88, 1G90, 1PRN, 1QHJ)
├── result/            # NACCESS output (.rsa / .asa)
└── doc/               # Project report and reference papers
```

## References

- Tusnády G.E., Dosztányi Z. & Simon I. (2004). *Transmembrane proteins in the Protein Data Bank: identification and classification.* Bioinformatics 20(17):2964–2972. [doi:10.1093/bioinformatics/bth340](https://doi.org/10.1093/bioinformatics/bth340)
- Kyte J. & Doolittle R.F. (1982). *A simple method for displaying the hydropathic character of a protein.* J. Mol. Biol. 157(1):105–132.
- Lomize M.A. *et al.* OPM database: Orientations of Proteins in Membranes. [opm.phar.umich.edu](https://opm.phar.umich.edu/)

## Author

**Eve-Angeline Stephen** · M2 Bioinformatics, Université Paris Cité

## License

Distributed under the MIT License. See [`LICENSE`](LICENSE).
