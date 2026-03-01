# 🧬 Molecular Docking Analysis: A Complete Protocol

[![AutoDock](https://img.shields.io/badge/AutoDock-Tools-blue.svg)](http://autodock.scripps.edu/)
[![AutoDock Vina](https://img.shields.io/badge/Vina-v1.2.3-brightgreen.svg)](http://vina.scripps.edu/)
[![PyMOL](https://img.shields.io/badge/PyMOL-Visualization-orange.svg)](https://pymol.org/2/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This repository provides a complete, step-by-step protocol for performing molecular docking analysis, from data acquisition to final visualization. It is designed to be a robust and reproducible guide for investigating the interaction between small molecule ligands and protein targets.

This protocol was developed and validated for a study investigating the effects of **Kojic Acid** and **D-Limonene** on six key enzymes: Chitinase, Tyrosinase, Lipase, Catalase, Peroxidase, and NF-κB.

---

## 📑 Table of Contents
1.  [Prerequisites](#prerequisites)
2.  [Workflow Overview](#workflow-overview)
3.  [Step-by-Step Protocol](#step-by-step-protocol)
    *   [Phase 1: Data Acquisition & Preparation](#phase-1-data-acquisition--preparation)
    *   [Phase 2: Docking Execution](#phase-2-docking-execution)
    *   [Phase 3: Analysis & Visualization](#phase-3-analysis--visualization)
4.  [File Formats Explained](#file-formats-explained)
5.  [Interpreting the Results](#interpreting-the-results)
6.  [Troubleshooting & FAQ](#troubleshooting--faq)
7.  [Citation](#citation)

---

## 🚀 Prerequisites

Before you begin, ensure you have the following software installed:

| Tool | Purpose | Download Link |
| :--- | :--- | :--- |
| **AutoDockTools (ADT)** | Protein & Ligand Preparation, Grid Box Definition | [MGL Tools](http://mgltools.scripps.edu/downloads) |
| **AutoDock Vina** | Core Docking Engine (Command-Line) | [AutoDock Vina](http://vina.scripps.edu/download.html) |
| **PyMOL** | 3D Visualization & Figure Generation | [PyMOL](https://pymol.org/2/) |
| **Open Babel** (Optional but recommended) | File Format Conversion | [Open Babel](https://openbabel.org/wiki/Get_OpenBabel) |

---

## 🗺️ Workflow Overview

The entire process can be summarized in the following flowchart. This provides a high-level view of the pipeline from start to finish.

```mermaid
graph TD
    A[Start] --> B[Acquire Protein Structure (.pdb)];
    B --> C[Acquire Ligand Structure (.sdf)];
    C --> D[Protein Preparation (ADT)];
    D --> E[Ligand Preparation (ADT)];
    E --> F[Define Grid Box (ADT)];
    F --> G[Create Config File (.txt)];
    G --> H[Run AutoDock Vina];
    H --> I[Analyze Log File (Affinity Scores)];
    I --> J[Visualize in PyMOL];
    J --> K[Generate Figures & Interpret];
    K --> L[End];

    subgraph Preparation
        D; E; F;
    end
    subgraph Execution
        G; H;
    end
    subgraph Analysis
        I; J; K;
    end

    style Preparation fill:#e6f3ff,stroke:#0066cc
    style Execution fill:#e6ffe6,stroke:#00cc66
    style Analysis fill:#fff0e6,stroke:#ff9933
```

---

## 📋 Step-by-Step Protocol

### Phase 1: Data Acquisition & Preparation

#### 1.1. Acquire Protein Structures (The "Locks")
- **Source:** RCSB Protein Data Bank ([https://www.rcsb.org/](https://www.rcsb.org/)).
- **Action:** Search for your target enzyme (e.g., "insect chitinase").
- **Selection Criteria (in order of importance):**
    1.  **Organism:** Choose a structure from your target organism or the closest evolutionary relative.
    2.  **Resolution:** Aim for **≤ 2.5 Å**. Lower is better.
    3.  **Bound Ligand:** Prioritize structures that already have an inhibitor or substrate bound in the active site. This is a **goldmine** for defining your grid box.
- **Output:** A `.pdb` file (e.g., `3W4U.pdb`).

#### 1.2. Acquire Ligand Structures (The "Keys")
- **Source:** PubChem ([https://pubchem.ncbi.nlm.nih.gov/](https://pubchem.ncbi.nlm.nih.gov/)).
- **Action:** Search for your ligand (e.g., "Kojic Acid").
- **Download:** Download the **3D Conformer** in **SDF** format.
- **Output:** An `.sdf` file (e.g., `kojic_acid.sdf`).

#### 1.3. Convert Ligand to PDB (if needed)
AutoDockTools cannot read `.sdf` files directly.
- **Easiest Method:** Re-download the ligand from PubChem, but select **PDB** as the file format instead of SDF.
- **Alternative Method:** Use Open Babel: `obabel -isdf ligand.sdf -opdb -O ligand.pdb -h`

#### 1.4. Protein Preparation (Using AutoDockTools)
1.  **Open ADT** and `File > Read Molecule...` your protein `.pdb` file.
2.  **Remove Unnecessary Components:**
    -   `Edit > Delete > Water`.
    -   Delete any non-essential ligands/ions. **<span style="color:red;">CRITICAL:</span>** Keep essential cofactors (e.g., Copper in Tyrosinase, Zinc in NF-κB).
3.  **Add Hydrogens:** `Edit > Hydrogens > Add > All`.
4.  **Add Charges:** `Edit > Charges > Gasteiger`.
5.  **Save:** `File > Save > Write PDBQT...` (e.g., `chitinase_prepared.pdbqt`).

#### 1.5. Ligand Preparation (Using AutoDockTools)
1.  `File > Read Molecule...` your ligand `.pdb` file.
2.  **Define Torsions:** `Ligand > Torsion Tree > Detect Root`. ADT will automatically identify rotatable bonds.
3.  **Add Hydrogens & Charges:** `Edit > Hydrogens > Add > All`, then `Edit > Charges > Gasteiger`.
4.  **Save:** `File > Save > Write PDBQT...` (e.g., `kojic_acid_prepared.pdbqt`).

#### 1.6. Define the Grid Box (Using AutoDockTools)
1.  With your prepared protein loaded, go to `Grid > Grid Box...`.
2.  **Center the Box:**
    -   **If you have a bound ligand:** Click **"Center on Macromolecule's Ligand"**. This is the best method.
    -   **If not:** Manually enter the coordinates of the known active site residues.
3.  **Set the Size:** Adjust `size_x`, `size_y`, and `size_z` to be large enough to encompass the binding site (e.g., 22-25 Å each).
4.  **Record** the `center` and `size` values. You will need them for the configuration file.

---

### Phase 2: Docking Execution

#### 2.1. Create the Vina Configuration File
- Create a plain text file named `config.txt`.
- Populate it with your parameters. **<span style="color:red;">Note for Vina v1.2.3+:</span>** The `log = ...` line is not used.

```ini
# --- Configuration File for AutoDock Vina ---

# Input files
receptor = chitinase_prepared.pdbqt
ligand = kojic_acid_prepared.pdbqt

# Search space (center and size in Angstroms)
center_x = 15.832  # <-- REPLACE WITH YOUR VALUES
center_y = 35.411  # <-- REPLACE WITH YOUR VALUES
center_z = -1.254  # <-- REPLACE WITH YOUR VALUES

size_x = 22
size_y = 22
size_z = 22

# Output file
out = chitinase_kojic_acid_out.pdbqt

# --- Key Parameters ---
exhaustiveness = 12
num_modes = 9
energy_range = 3
```

#### 2.2. Run AutoDock Vina
- Open your command prompt or terminal.
- Navigate to the directory containing your files.
- Execute the following command. This uses output redirection to create the log file, which is required for Vina v1.2.3+.

```bash
vina --config config.txt > log.txt
```
- The program will run and display a progress bar. Upon completion, it will generate your output file (`chitinase_kojic_acid_out.pdbqt`) and the log file (`log.txt`).

---

### Phase 3: Analysis & Visualization

#### 3.1. Analyze the Log File
- Open `log.txt` and scroll to the bottom.
- The results table shows the predicted binding affinity for each mode.

```
mode |   affinity | dist from best mode
     | (kcal/mol) | rmsd l.b.| rmsd u.b.
-----+------------+----------+----------
   1       -7.1          0          0
   2       -6.8      1.234      2.456
   ...
```
- **The key metric is the `affinity` of `mode 1`**. The more negative this value, the stronger the predicted binding.

#### 3.2. Visualize in PyMOL
1.  **Open PyMOL.**
2.  `File > Open...` your original protein `.pdb` file.
3.  `File > Open...` your Vina output `.pdbqt` file. Choose to load all states.
4.  **Isolate the Best Pose:** In the object panel, set the ligand's `State` to `1`.
5.  **Create Your Figure:**
    -   `Hide > Everything`, then `Show > Cartoon` for the protein.
    -   `Show > Sticks` for the ligand.
    -   Use the `dist` command to find hydrogen bonds: `dist hbonds, (protein and name N or O), (ligand and name N or O), mode=2`
    -   Color, label, and take a high-resolution screenshot (`File > Save Image...`).

---

## 📁 File Formats Explained

| Extension | Full Name | Used For | Key Feature |
| :--- | :--- | :--- | :--- |
| **.pdb** | Protein Data Bank | Standard protein/ligand structure | Atomic coordinates, widely supported. |
| **.sdf** | Structure Data File | Ligand structure | Rich chemical information, connectivity. |
| **.pdbqt** | PDB + Charges + Types | **AutoDock input/output** | PDB format + Gasteiger charges + AD4 atom types. |
| **.txt** | Text File | **Vina configuration** | Contains all parameters for the docking run. |

---

## 📊 Interpreting the Results

- **<span style="color:green;">Strong Binding:</span>** Affinity score ≤ -7.0 kcal/mol. This suggests a stable and potentially potent interaction.
- **<span style="color:orange;">Moderate Binding:</span>** Affinity score between -5.0 and -7.0 kcal/mol. A plausible interaction that warrants further investigation.
- **<span style="color:red;">Weak Binding:</span>** Affinity score > -5.0 kcal/mol. The interaction is likely not significant.

**Visualization is Key:** A good score must be supported by a sensible binding pose. Does the ligand fit well in the pocket? Are there key hydrogen bonds or hydrophobic interactions with important residues? **A picture is worth a thousand docking scores.**

---

## ❓ Troubleshooting & FAQ

<details>
<summary><b>Q1: I get an error: "unrecognised option '--log'"</b></summary>
<br>
This is because you are using AutoDock Vina v1.2.0 or newer. The `--log` command-line option has been deprecated.
<br><br>
<b>Solution:</b> Remove `--log log.txt` from your command. Use output redirection instead: <br>
<code>vina --config config.txt > log.txt</code>
</details>

<details>
<summary><b>Q2: I get an error: "Configuration file parse error: unrecognised option 'log'"</b></summary>
<br>
This is the companion error to the one above. The `log = ...` line is also no longer supported inside the `config.txt` file for Vina v1.2.0+.
<br><br>
<b>Solution:</b> Delete the `log = ...` line from your `config.txt` file and use the output redirection command shown in the answer above.
</details>

<details>
<summary><b>Q3: My visualization software (e.g., Discovery Studio) can't open the .pdbqt file.</b></summary>
<br>
This is normal. The `.pdbqt` format is specific to AutoDock.
<br><br>
<b>Solution:</b> Convert the `.pdbqt` file to a standard `.pdb` file. The easiest way is to simply rename the file extension from `.pdbqt` to `.pdb`. For a more robust conversion, use Open Babel: <br>
<code>obabel -ipdbqt output.pdbqt -opdb -O output.pdb</code>
</details>

---

## ✍️ Citation

If you use this protocol or the results from it in your work, please cite the relevant software papers:

- For AutoDock Vina 1.2.x: Eberhardt, J. et al. (2021). *AutoDock Vina 1.2.0: New Docking Methods, Expanded Force Field, and Python Bindings*. J. Chem. Inf. Model. DOI: [10.1021/acs.jcim.1c00203](https://doi.org/10.1021/acs.jcim.1c00203)
- For the original AutoDock Vina: Trott, O. & Olson, A. J. (2010). *AutoDock Vina: improving the speed and accuracy of docking with a new scoring function, efficient optimization and multithreading*. J. Comp. Chem. DOI: [10.1002/jcc.21334](https://doi.org/10.1002/jcc.21334)
