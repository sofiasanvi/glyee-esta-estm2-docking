# glyee-esta-estm2-docking

Docking workflow for GlyEE against the esterases EstA and EstM2 using Open Babel, PDB2PQR, PyMOL and AutoDock Vina.

## Project structure

```text
glyee-esta-estm2-docking/
├── config/
│   └── docking_boxes.csv
├── data/
│   ├── raw/
│   │   ├── GlyEE.sdf
│   │   ├── EstA.pdb
│   │   └── EstM2.pdb
│   └── prepared/
├── environments/
│   └── protein_prep.yml
├── logs/
└── results/
```

## 1. Create conda environments

Create the docking-related environments from the tutorial files:

```bash
conda env create --file Docking-Tutotial-main/environments/autodock_env.yml
conda env create --file Docking-Tutotial-main/environments/meeko_babelmin_env.yml
conda env create --file Docking-Tutotial-main/environments/mgltoolsmin_env.yml
conda env create --file Docking-Tutotial-main/environments/rdkit.yml
```

## 2. Prepare the GlyEE ligand at pH 7.0

Activate the Open Babel environment:

```bash
conda activate mko_obbl
```

Generate a 3D structure, protonate the ligand at pH 7.0, and minimize it with MMFF94:

```bash
obabel data/raw/GlyEE.sdf \
  -O data/prepared/GlyEE_pH7.sdf \
  -p 7.0 \
  --gen3d \
  --minimize \
  --ff MMFF94
```
Convert to PDBQT:

```
conda activate mko_obbl
```
```
mk_prepare_ligand.py -i ligand.sdf -o ligand.pdbqt
```


## 3. Clean protein structures

Remove `ANISOU` records and crystallographic water molecules from the raw PDB files:

```bash
grep -v "^ANISOU" data/raw/EstA.pdb | grep -v " HOH " > data/prepared/EstA_clean.pdb
grep -v "^ANISOU" data/raw/EstM2.pdb | grep -v " HOH " > data/prepared/EstM2_clean.pdb
```

## 4. Prepare proteins at pH 7.0

Create and activate the protein preparation environment:

```bash
conda env create --file environments/protein_prep.yml
conda activate protein_prep
```

Run PDB2PQR with the AMBER force field at pH 7.0:

```bash
pdb2pqr --ff=AMBER --with-ph=7.0 --keep-chain \
  data/prepared/EstA_clean.pdb data/prepared/EstA_pH7.pqr

pdb2pqr --ff=AMBER --with-ph=7.0 --keep-chain \
  data/prepared/EstM2_clean.pdb data/prepared/EstM2_pH7.pqr
```

Convert the prepared `.pqr` files back to `.pdb` format:

```bash
obabel data/prepared/EstA_pH7.pqr -O data/prepared/EstA_pH7.pdb
obabel data/prepared/EstM2_pH7.pqr -O data/prepared/EstM2_pH7.pdb
```

 ````bash
conda activate mgltools

prepare_receptor4.py \
  -r prepared/EstA_pH7.pdb \
  -o prepared/EstA.pdbqt \
  -U waters

prepare_receptor4.py \
  -r prepared/EstM2_pH7.pdb \
  -o prepared/EstM2.pdbqt \
  -U waters
`````
## 5. Identify docking box centers in PyMOL

Open PyMOL and load the prepared protein structure:

```pymol
load data/prepared/EstA_pH7.pdb
```

Select the active-site residues for EstA:

```pymol
select site, resi 59+323+322+221+325
centerofmass site
```

Save the docking box coordinates in:

```text
config/docking_boxes.csv
```

Example columns:

```csv
protein,ligand,center_x,center_y,center_z,size_x,size_y,size_z
EstA,GlyEE,92.267,79.626,111.761,20,20,20
EstM2,GlyEE,2.991,-0.846,1.595,20,20,20
```

## 6. Run docking with AutoDock Vina

Before running Vina, make sure the receptor and ligand have been converted to `.pdbqt` format:

```text
data/prepared/EstA.pdbqt
data/prepared/EstM2.pdbqt
data/prepared/GlyEE.pdbqt
```

### EstA + GlyEE

```bash
vina \
  --receptor data/prepared/EstA.pdbqt \
  --ligand data/prepared/GlyEE.pdbqt \
  --center_x 92.267 \
  --center_y 79.626 \
  --center_z 111.761 \
  --size_x 20 \
  --size_y 20 \
  --size_z 20 \
  --exhaustiveness 32 \
  --num_modes 20 \
  --seed 1 \
  --out results/EstA_GlyEE_seed1.pdbqt \
  | tee logs/EstA_GlyEE_seed1.log
```

### EstM2 + GlyEE

```bash
vina \
  --receptor data/prepared/EstM2.pdbqt \
  --ligand data/prepared/GlyEE.pdbqt \
  --center_x 2.991 \
  --center_y -0.846 \
  --center_z 1.595 \
  --size_x 20 \
  --size_y 20 \
  --size_z 20 \
  --exhaustiveness 32 \
  --num_modes 20 \
  --seed 1 \
  --out results/EstM2_GlyEE_seed1.pdbqt \
  | tee logs/EstM2_GlyEE_seed1.log
```

## 7. Output files

Docking results are saved in:

```text
results/
```

Vina log files are saved in:

```text
logs/
```

Expected output examples:

```text
results/EstA_GlyEE_seed1.pdbqt
results/EstM2_GlyEE_seed1.pdbqt
logs/EstA_GlyEE_seed1.log
logs/EstM2_GlyEE_seed1.log
```

## Notes

- The docking box size is set to `20 Å × 20 Å × 20 Å`, which corresponds to `2 nm × 2 nm × 2 nm`.
- This is below the commonly recommended upper limit of approximately `4 nm`.
- `exhaustiveness 32` is used to increase the docking search effort.
- The seed is fixed to `1` to make the docking runs more reproducible.


#amber
Activate the environment:

```bash
conda activate AmberTools26
source "$CONDA_PREFIX/amber.sh"
```

`source "$CONDA_PREFIX/amber.sh"` sets the `AMBERHOME` environment variable, among others.

---

## 3. Verify the installation

```bash
echo "CONDA_PREFIX=$CONDA_PREFIX"
echo "AMBERHOME=$AMBERHOME"
```

Check the Amber tools:

```bash
for program in tleap sander cpptraj antechamber parmchk2 pdb4amber; do
    printf "%-15s " "$program"
    command -v "$program" || echo "MISSING"
done
```

Also check the tools used for docking files:

```bash
for program in mk_export.py obabel; do
    printf "%-15s " "$program"
    command -v "$program" || echo "MISSING"
done
```

All paths should point to executables inside the Conda environment, for example:

```text
/home/sofia/miniconda3/envs/AmberTools26/bin/tleap
```

---
