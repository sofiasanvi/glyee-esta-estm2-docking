# glyee-esta-estm2-docking

1.
conda env create --file Docking-Tutotial-main/environments/autodock_env.yml
conda env create --file Docking-Tutotial-main/environments/meeko_babelmin_env.yml
conda env create --file Docking-Tutotial-main/environments/mgltoolsmin_env.yml
conda env create --file Docking-Tutotial-main/environments/rdkit.yml

2.
conda activate mko_obbl

obabel data/raw/GlyEE.sdf \
  -O data/prepared/GlyEE_pH7.sdf \
  -p 7.0 \
  --gen3d \
  --minimize \
  --ff MMFF94


3. Clean structures
grep -v "^ANISOU" data/raw/EstA.pdb | grep -v " HOH " > data/prepared/EstA_clean.pdb
grep -v "^ANISOU" data/raw/EstM2.pdb | grep -v " HOH " > data/prepared/EstM2_clean.pdb


#Preperation pH = 7.0
4. 
conda env create --file environments/protein_prep.yml
conda activate protein_prep

5. 
pdb2pqr --ff=AMBER --with-ph=7.0 --keep-chain \
  data/prepared/EstA_clean.pdb data/prepared/EstA_pH7.pqr

pdb2pqr --ff=AMBER --with-ph=7.0 --keep-chain \
  data/prepared/EstM2_clean.pdb data/prepared/EstM2_pH7.pqr

6.
obabel data/prepared/EstA_pH7.pqr -O data/prepared/EstA_pH7.pdb
obabel data/prepared/EstM2_pH7.pqr -O data/prepared/EstM2_pH7.pdb

7.
Open PyMoL
- load EstA_pH7.pdb
- select site, resi 59+D323+H322+A221+A325 centerofmass site

8. Save site in csv-file
config/docking_boxes.csv

9.  vina   --receptor data/prepared/EstA.pdbqt   --ligand data/prepared/GlyEE.pdbqt   --center_x 92.267   --center_y 79.626   --center_z 111.761   --size_x 20   --size_y 20   --size_z 20   --exhaustiveness 32   --num_modes 20   --seed 1   --out results/EstA_GlyEE_seed1.pdbqt   | tee logs/EstA_GlyEE_seed1.log

10.
vina   --receptor data/prepared/EstM2.pdbqt   --ligand data/prepa
red/GlyEE.pdbqt   --center_x 2.991   --center_y -0.846   --center_z 1.595   --size_x 20   --size_y 20   --size_z
 20   --exhaustiveness 32   --num_modes 20   --seed 1   --out results/EstM2_GlyEE_seed1.pdbqt   | tee logs/EstM2_GlyEE_seed1.log