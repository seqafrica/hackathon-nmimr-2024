# Kleborate Installation

## 1. Create Conda Environment
```conda create -n kleborate python=3.6```

## 2. Activate the Environment
```conda activate kleborate```

## 3. Install Dependencies
# Biopython v1.75 or later
```conda install conda-forge::biopython```

# BLAST+ command line tools (v2.7.1 or later)
```conda install bioconda::blast```

# Mash v2.0 or later
```conda install bioconda::mash```

## 4. Verify Versions of Dependencies
```
python3 --version
python3 -c "import Bio; print(Bio.__version__)"
blastn -version
mash --version
```

## 5. Install Kleborate
```
git clone --recursive https://github.com/klebgenomics/Kleborate.git
cd Kleborate
python3 setup.py install
kleborate -h
```

## 6. Update MLST Database
```
cd scripts
python3 getmlst.py --species "Klebsiella pneumoniae"
mv Klebsiella_pneumoniae.fasta ../kleborate/data
mv profiles_csv ../kleborate/data/kpneumoniae.txt
rm -f ../kleborate/data/Klebsiella_pneumoniae.fasta.n*
```

## 7. Make the PATH to kleborate-runner.py Global
```
cd ~
echo "export PATH=\"$PATH:/home/chrisow/Kleborate\"" >> .bashrc
```

##Exit and re-login
# OR
```sudo source .bashrc```

## 8. Rename Script with Version Attached
```
cd Kleborate
mv kleborate-runner.py kleboratev2.4.1
```
## 9. Test Run
```kleboratev2.4.1 -o results.txt -a test/test_genomes/GCF_002248955.1.fna.gz test/test_genomes/GCF_003095495.1.fna.gz test/test_genomes/GCF_000009885.1.fna.gz test/test_genomes/GCF_900501255.1.fna.gz test/test_genomes/GCF_000019565.1.fna.gz test/test_genomes/GCF_000492415.1.fna.gz test/test_genomes/GCF_000492795.1.fna.gz```

## Output

| strain          | species                                            | species_match   |   contig_count |     N50 |   largest_contig |   total_size | ambiguous_bases   | QC_warnings     | ST     |   virulence_score | Yersiniabactin   |   YbST | Colibactin   |   CbST | Aerobactin   |   AbST | Salmochelin   | SmST   | RmpADC                                  | RmST   | rmpA2        | wzi    | K_locus    | Chr_ST   |   gapA |   infB |   mdh |   pgi |   phoE |   rpoB |   tonB | ybtS   | ybtX   | ybtQ   | ybtP   | ybtA   | irp2   | irp1   | ybtU   | ybtT   | ybtE   | fyuA   | clbA   | clbB   | clbC   | clbD   | clbE   | clbF   | clbG   | clbH   | clbI   | clbL   | clbM   | clbN   | clbO   | clbP   | clbQ   | iucA   | iucB   | iucC   | iucD   | iutA   | iroB   | iroC   | iroD   | iroN   | rmpA      | rmpD     | rmpC   | spurious_virulence_hits   |
|:----------------|:---------------------------------------------------|:----------------|---------------:|--------:|-----------------:|-------------:|:------------------|:----------------|:-------|------------------:|:-----------------|-------:|:-------------|-------:|:-------------|-------:|:--------------|:-------|:----------------------------------------|:-------|:-------------|:-------|:-----------|:---------|-------:|-------:|------:|------:|-------:|-------:|-------:|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:----------|:---------|:-------|:--------------------------|
| GCF_002248955.1 | Klebsiella pneumoniae                              | strong          |             73 |  194261 |           362142 |      5388659 | no                | -               | ST15   |                 0 | -                |      0 | -            |      0 | -            |      0 | -             | 0      | -                                       | 0      | -            | wzi29  | KL106      | ST15     |      1 |      1 |     1 |     1 |      1 |      1 |      1 | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -         | -        | -      | -                         |
| GCF_003095495.1 | Klebsiella pneumoniae                              | strong          |            676 |   16918 |            71716 |      5800539 | no                | -               | ST258  |                 0 | -                |      0 | -            |      0 | -            |      0 | -             | 0      | -                                       | 0      | -            | wzi154 | KL107      | ST258    |      3 |      3 |     1 |     1 |      1 |      1 |     79 | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -         | -        | -      | -                         |
| GCF_000009885.1 | Klebsiella pneumoniae                              | strong          |              2 | 5248520 |          5248520 |      5472672 | no                | -               | ST23   |                 4 | ybt 2; ICEKp1    |    326 | -            |      0 | iuc 1        |      1 | iro 1,iro 3   | 19,18  | rmp 3; ICEKp1 (truncated),rmp 1; KpVP-1 | 119,26 | rmpA2_3-47%  | wzi1   | KL1        | ST23     |      2 |      1 |     1 |     1 |      9 |      4 |     12 | 9      | 7      | 9      | 6      | 5      | 1      | 1      | 6      | 7      | 7      | 6      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | 1      | 1      | 1      | 1      | 1      | 1,21   | 2,39   | 1,19   | 1,5    | 11,2      | 38-86%,2 | 6,2    | -                         |
| GCF_900501255.1 | Klebsiella pneumoniae                              | strong          |            134 |  303226 |           623663 |      5449387 | no                | -               | ST86   |                 3 | -                |      0 | -            |      0 | iuc 1        |      1 | iro 1         | 1      | rmp 1; KpVP-1,-                         | 26,0   | rmpA2_9*-50% | wzi2   | KL2 (KL30) | ST86     |      9 |      4 |     2 |     1 |      1 |      1 |     27 | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | 1      | 1      | 1      | 1      | 1      | 1      | 1      | 1      | 1      | 2,40*-51% | 2,-      | 2,-    | -                         |
| GCF_000019565.1 | Klebsiella variicola subsp. variicola              | strong          |              3 | 5641239 |          5641239 |      5920257 | no                | -               | ST146  |                 0 | -                |      0 | -            |      0 | -            |      0 | -             | 0      | -                                       | 0      | -            | wzi159 | KL30       | ST146    |     16 |     24 |    30 |    27 |     36 |     22 |     55 | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -         | -        | -      | -                         |
| GCF_000492415.1 | Klebsiella quasipneumoniae subsp. quasipneumoniae  | strong          |             10 | 5263297 |          5263297 |      5539132 | yes (21956)       | ambiguous_bases | ST1437 |                 0 | -                |      0 | -            |      0 | -            |      0 | -             | 0      | -                                       | 0      | -            | wzi185 | KL46       | ST1437   |     17 |     19 |    69 |    39 |    185 |     21 |    238 | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -         | -        | -      | -                         |
| GCF_000492795.1 | Klebsiella quasipneumoniae subsp. similipneumoniae | strong          |              2 | 5142035 |          5142035 |      5241816 | yes (1275)        | ambiguous_bases | ST1435 |                 0 | -                |      0 | -            |      0 | -            |      0 | -             | 0      | -                                       | 0      | -            | wzi183 | KL21       | ST1435   |     18 |     88 |   128 |   116 |     11 |     99 |    237 | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -      | -         | -        | -      | -                         |
