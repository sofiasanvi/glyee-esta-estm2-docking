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
# Preparing GlyEE Docking Pose 5 for Amber

This document describes how docking pose 5 of GlyEE was extracted from the
AutoDock/Vina PDBQT output and converted into Amber-compatible ligand files.

## Input files

The workflow started from:

```text
results/EstA_GlyEE_seed1.pdbqt
```

This PDBQT file contains multiple docked GlyEE poses. Pose 5 was selected for
the molecular dynamics system.

The protein structure used later in the workflow is:

```text
EstA_pH7.pdb
```

## Output files

The main Amber-compatible ligand files are:

```text
GYE_gaff2.mol2
GYE_gaff2.frcmod
```

Additional files retained for reproducibility include:

```text
GlyEE_all_poses.sdf
GYE_pose5.sdf
sqm.in
sqm.out
sqm.pdb
```

## 1. Activate the Amber environment

```bash
conda activate AmberTools26
source "$CONDA_PREFIX/amber.sh"
```

Check that the required programs are available:

```bash
for program in mk_export.py python antechamber parmchk2; do
    printf "%-15s " "$program"
    command -v "$program" || echo "MISSING"
done
```

## 2. Create the ligand preparation directory

From the project root:

```bash
cd ~/glyee-esta-estm2-docking

mkdir -p amber_md/01_ligand
cd amber_md/01_ligand
```

## 3. Export the docking poses from PDBQT to SDF

Meeko was used to convert the PDBQT docking output into an SDF file containing
all poses:

```bash
mk_export.py \
  ../../results/EstA_GlyEE_seed1.pdbqt \
  -s GlyEE_all_poses.sdf
```

Check that the output file was created:

```bash
ls -lh GlyEE_all_poses.sdf
```

Count the number of molecules in the SDF file:

```bash
grep -c '^\$\$\$\$$' GlyEE_all_poses.sdf
```

This number should match the number of docking models in the PDBQT file:

```bash
grep -c '^MODEL' ../../results/EstA_GlyEE_seed1.pdbqt
```

## 4. Extract pose 5 with RDKit

The fifth molecule in the SDF file corresponds to docking pose 5.

```bash
python - <<'PY'
from pathlib import Path
from rdkit import Chem
from rdkit.Chem import rdMolDescriptors

input_file = Path("GlyEE_all_poses.sdf")
output_file = Path("GYE_pose5.sdf")

supplier = Chem.SDMolSupplier(
    str(input_file),
    removeHs=False,
    sanitize=True,
)

molecules = [mol for mol in supplier if mol is not None]

print(f"Readable poses: {len(molecules)}")

if len(molecules) < 5:
    raise RuntimeError("The SDF file contains fewer than five readable poses.")

pose5 = molecules[4]
pose5.SetProp("_Name", "GYE_pose5")

writer = Chem.SDWriter(str(output_file))
writer.write(pose5)
writer.close()

print(f"Saved: {output_file}")
print(f"Formula: {rdMolDescriptors.CalcMolFormula(pose5)}")
print(f"Formal charge: {Chem.GetFormalCharge(pose5):+d}")
print(f"Atoms: {pose5.GetNumAtoms()}")
print(f"Heavy atoms: {pose5.GetNumHeavyAtoms()}")
PY
```

For protonated glycine ethyl ester, the expected formal charge is:

```text
+1
```

The expected molecular formula is approximately:

```text
C4H10NO2+
```

Check that the file was created:

```bash
ls -lh GYE_pose5.sdf
```

## 5. Optional visual inspection

Pose 5 can be inspected together with the EstA structure in PyMOL:

```bash
pymol ../../EstA_pH7.pdb GYE_pose5.sdf
```

The ligand should remain in the same docked position in the active site.

PyMOL was used only for visual inspection. The ligand was not parameterized
from a PyMOL-generated MOL2 file because such files may contain generic Tripos
atom types and incomplete chemical information.

## 6. Generate GAFF2 atom types and AM1-BCC charges

The ligand was parameterized with Antechamber using:

- GAFF2 atom types
- AM1-BCC partial charges
- total molecular charge `+1`
- residue name `GYE`

```bash
antechamber \
  -i GYE_pose5.sdf \
  -fi sdf \
  -o GYE_gaff2.mol2 \
  -fo mol2 \
  -at gaff2 \
  -c bcc \
  -nc 1 \
  -rn GYE \
  -s 2
```

The main output from this step is:

```text
GYE_gaff2.mol2
```

This file contains:

- the three-dimensional pose 5 coordinates
- GAFF2 atom types
- AM1-BCC partial charges
- atom names
- bonds and bond orders
- residue name `GYE`

## 7. Verify the Antechamber calculation

Check that the SQM calculation completed successfully:

```bash
grep "Calculation Completed" sqm.out
```

Also inspect the end of the output:

```bash
tail -20 sqm.out
```

Check that the Amber MOL2 file is non-empty:

```bash
ls -lh GYE_gaff2.mol2
```

Verify that the partial charges sum to approximately `+1`:

```bash
awk '
/@<TRIPOS>ATOM/ {
    atoms = 1
    next
}
/@<TRIPOS>BOND/ {
    atoms = 0
}
atoms && NF >= 9 {
    charge += $9
}
END {
    printf "MOL2 net charge: %.6f\n", charge
}
' GYE_gaff2.mol2
```

Expected result:

```text
MOL2 net charge: 1.000000
```

## 8. Generate the Amber force-field parameter file

`parmchk2` was run on the GAFF2-typed MOL2 file:

```bash
parmchk2 \
  -i GYE_gaff2.mol2 \
  -f mol2 \
  -o GYE_gaff2.frcmod \
  -s 2
```

The resulting file is:

```text
GYE_gaff2.frcmod
```

This file contains any additional bond, angle, torsion, improper, or
non-bonded parameters required for GlyEE that are not already covered directly
by the standard GAFF2 parameter set.

Check that the file exists and is non-empty:

```bash
ls -lh GYE_gaff2.frcmod
```

Search for parameters that may require manual review:

```bash
grep -iE "ATTN|needs revision|missing|zero" \
  GYE_gaff2.frcmod || true
```

## 9. Final ligand files for tleap

The two files needed to load GlyEE into `tleap` are:

```text
GYE_gaff2.mol2
GYE_gaff2.frcmod
```

They can later be loaded with:

```text
source leaprc.gaff2

loadamberparams GYE_gaff2.frcmod
glyee = loadmol2 GYE_gaff2.mol2
```

The next stage is to combine the prepared GlyEE ligand with `EstA_pH7.pdb`,
add water and ions, and generate the Amber topology and coordinate files:

```text
EstA_GlyEE.prmtop
EstA_GlyEE.inpcrd
```

## 10. Files to keep

For reproducibility, keep at least:

```text
results/EstA_GlyEE_seed1.pdbqt
GlyEE_all_poses.sdf
GYE_pose5.sdf
GYE_gaff2.mol2
GYE_gaff2.frcmod
sqm.out
environment-amber.yml
```

The temporary files beginning with `ANTECHAMBER_` are useful for debugging but
are not required for the later `tleap` step.
