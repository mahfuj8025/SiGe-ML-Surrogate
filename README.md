<div align="center">
  <h1>SiGe-ML-Surrogate</h1>
  <p><b>An ML surrogate that mimics Density Functional Theory to instantly predict SiGe alloy properties.</b></p>
</div>

<br>

<div align="center">
  <!-- Replace the link below with the actual path to your flowchart or plot once you have it -->
  <img src="https://via.placeholder.com/800x300?text=[Insert+Graphical+Abstract+or+Workflow+Diagram+Here]" alt="Graphical Abstract">
</div>

## 🌍 What We Are Trying to Do & Why It Matters

Silicon (Si) and Germanium (Ge) are the backbone of modern electronics. By mixing them into a SiGe alloy, we can tune their physical properties—like the bandgap—to create faster computer chips, better optoelectronics, and highly efficient solar cells. 

Traditionally, calculating the exact properties of these alloys requires Density Functional Theory (DFT) simulations using software like Quantum Espresso. These calculations are incredibly slow and require massive supercomputing power. 

**Our Goal:** We are building a machine learning "surrogate" model using XGBoost. Instead of waiting hours for a supercomputer to calculate the properties of a single $Si_{x}Ge_{1-x}$ crystal structure, our model learns how the physics software behaves and predicts the answers in milliseconds. This allows researchers to screen thousands of new material combinations instantly, accelerating the discovery of next-generation semiconductors.

---

## 🔬 Calculation Methodology

To generate accurate training data, the pipeline follows a strict computational workflow for every alloy configuration:

1. **Supercell Generation:** `pymatgen` constructs initial random solid solution supercells of $Si_{x}Ge_{1-x}$ across varying concentrations ($x$).
2. **Structural Relaxation (`vc-relax`):** Before any properties are extracted, the structures undergo variable-cell relaxation in Quantum Espresso to minimize atomic forces and stress. This ensures the atoms find their true ground-state lattice positions, rather than forcing them into an unrelaxed, artificial grid.
3. **Static & Band Calculations (`scf` & `bands`):** Once the relaxed lattice parameter is found, a self-consistent field (`scf`) calculation computes the converged electron density, followed by a non-self-consistent (`bands`) calculation to map the electronic band structure.
4. **Feature Extraction:** `matminer` parses these outputs to translate the relaxed structures into ML-readable descriptors.

---

## 🎯 Target Physical Properties

The surrogate model is specifically being trained to predict the following outputs derived from the DFT calculations:
* **Total Energy & Formation Enthalpy:** To evaluate the thermodynamic stability of the specific alloy composition.
* **Electronic Bandgap:** Predicting both the magnitude of the gap and identifying whether the transition is direct or indirect.
* **Relaxed Lattice Constants:** Predicting the structural expansion/contraction based on the Ge concentration.

---

## ⚠️ Model Operating Bounds & Limitations

If you are reviewing this data or utilizing the surrogate model, it is critical to understand the physical constraints of the training environment:

* **Thermodynamic Conditions:** The model is trained on standard DFT calculations, meaning all predictions represent the system at **0 K and 0 atm**. It does not account for finite temperature effects, thermal expansion, or phonon contributions.
* **Compositional Range:** The model is valid strictly for the $Si_{x}Ge_{1-x}$ binary alloy system where the composition $x$ ranges from 0.0 to 1.0. 
* **The DFT Bandgap Error:** The training data is generated using standard DFT approximations (e.g., LDA/GGA). It is a well-known limitation that these functionals systematically underestimate semiconductor bandgaps (often incorrectly predicting pure Ge to be metallic). 
* **The Engineering Value:** Because the ML model trains on DFT data, it acts as a functional clone and inherits these exact same bandgap errors. It is **not** designed to predict true experimental ground-truth values. However, it successfully captures the *physical trends* (e.g., composition dependence and strain effects), making it a highly valuable, low-cost screening tool before moving to highly expensive GW calculations or physical lab experiments.

---

## 🛠️ Tech Stack & Tools

**Core Physics & Materials Science:**
* **Quantum Espresso (`pw.x`):** The engine running the Density Functional Theory (DFT) calculations.
* **Pymatgen:** Used to programmatically generate SiGe supercells and parse QE output files.
* **VESTA:** Used for manual 3D visual verification of the relaxed crystal lattice configurations before running high-throughput batch calculations.

**Machine Learning & Data Processing:**
* **Matminer:** Extracts physical and chemical features from the generated crystal structures to feed into the machine learning model.
* **XGBoost & Scikit-learn:** The core machine learning algorithms used for the surrogate model.
* **NumPy & Pandas:** Handles the multi-dimensional arrays, data wrangling, and dataset structuring.

**Visualization:**
* **Matplotlib & Seaborn:** Used for plotting model accuracy and physical trends (e.g., Bandgap vs. Composition).
* **Pymatgen BSPlotter:** Used to visualize the raw electronic band structures directly from the DFT outputs.

---

## 📂 Repository Structure

```text
SiGe-ML-Surrogate/
├── .gitignore                # Blocks heavy files and Python caches
├── README.md                 # Project overview
├── data/                     
│   ├── raw/                  # Initial unrelaxed crystal structures
│   └── processed/            # Relaxed structures and ML features
├── qe_workspace/             
│   ├── inputs/               # Auto-generated .in files (vc-relax, scf, bands)
│   └── outputs/              # Raw Quantum Espresso .out files
├── notebooks/                # Jupyter notebooks for data visualization
├── src/                      
│   ├── 01_qe_runner.py       # Automates QE calculations via pymatgen
│   ├── 02_extract_feats.py   # Parses outputs for ML features using matminer
│   └── 03_train_model.py     # Trains the XGBoost surrogate
└── models/                   # Saved XGBoost models
```

---

## 💻 Installation & Setup

This project is built to run inside a Linux environment (specifically WSL Ubuntu on Windows).

**Prerequisites:**
Ensure you have the following installed in your WSL environment:
* Python 3.9+
* Quantum Espresso (`pw.x`)
* `pip install pymatgen matminer xgboost scikit-learn numpy pandas matplotlib seaborn`

**To clone this repository:**
```bash
git clone [https://github.com/mahfuj8025/SiGe-ML-Surrogate.git](https://github.com/mahfuj8025/SiGe-ML-Surrogate.git)
cd SiGe-ML-Surrogate
```

---

## 🚀 Project Roadmap

- [x] Set up WSL environment and Git version control
- [x] Establish secure folder structure and `.gitignore`
- [ ] Write Python script (`01_qe_runner.py`) using `pymatgen` to generate $Si_{x}Ge_{1-x}$ supercells
- [ ] Run high-throughput QE calculations (`vc-relax`, `scf`, `bands`)
- [ ] Use `matminer` to parse outputs and extract total energy, bandgaps, and descriptors
- [ ] Train XGBoost surrogate model and evaluate accuracy
- [ ] Generate feature importance plots and update visual abstract

---

## 📚 References

1. *Properties of 2D Electron or Hole Gases at Tailored s-Si/SiGe Interfaces* (Discusses DFT bandgap underestimation in SiGe). [arXiv:2606.28776](https://arxiv.org/pdf/2606.28776)
2. *Direct Bandgap Emission from Hexagonal Ge and SiGe Alloys* (Details composition control and bandgap tuning). [ResearchGate](https://www.researchgate.net/publication/337019882_Direct_Bandgap_Emission_from_Hexagonal_Ge_and_SiGe_Alloys)
3. *Band structure calculation of Si-Ge-Sn binary and ternary alloys, nanostructures and devices* (Mathematical expressions for bandgaps in strained configurations). [White Rose eTheses](https://etheses.whiterose.ac.uk/id/eprint/5850/1/uk_bl_ethos_507078.pdf)

---

## 👨‍🔬 Author

**Mahfujur Rahman**  
Undergraduate Researcher, Materials & Metallurgical Engineering  
Bangladesh University of Engineering and Technology (BUET)  
 