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

  