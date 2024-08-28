# Kleborate Installation

## 1. Create Conda Environment
conda create -n kleborate python=3.6

## 2. Activate the Environment
conda activate kleborate

## 3. Install Dependencies
# Biopython v1.75 or later
conda install conda-forge::biopython

# BLAST+ command line tools (v2.7.1 or later)
conda install bioconda::blast

# Mash v2.0 or later
conda install bioconda::mash

## 4. Verify Versions of Dependencies
python3 --version
python3 -c "import Bio; print(Bio.__version__)"
blastn -version
mash --version

## 5. Install Kleborate
git clone --recursive https://github.com/klebgenomics/Kleborate.git
cd Kleborate
python3 setup.py install
kleborate -h

## 6. Update MLST Database
cd scripts
python3 getmlst.py --species "Klebsiella pneumoniae"
mv Klebsiella_pneumoniae.fasta ../kleborate/data
mv profiles_csv ../kleborate/data/kpneumoniae.txt
rm -f ../kleborate/data/Klebsiella_pneumoniae.fasta.n*

## 7. Make the PATH to kleborate-runner.py Global
cd ~
echo "export PATH=\"$PATH:/home/chrisow/Kleborate\"" >> .bashrc

# Exit and re-login
# OR
sudo source .bashrc

## 8. Rename Script with Version Attached
cd Kleborate
mv kleborate-runner.py kleboratev2.4.1

## 9. Test Run
kleboratev2.4.1 -o results.txt -a test/test_genomes/GCF_002248955.1.fna.gz test/test_genomes/GCF_003095495.1.fna.gz test/test_genomes/GCF_000009885.1.fna.gz test/test_genomes/GCF_900501255.1.fna.gz test/test_genomes/GCF_000019565.1.fna.gz test/test_genomes/GCF_000492415.1.fna.gz test/test_genomes/GCF_000492795.1.fna.gz

