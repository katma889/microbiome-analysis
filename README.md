# Microbiome-analysis
This is a compilation of my scripts for 16S sequencing data analysis
## Quality check
We had paired read 16S sequencing data from Miseq. The first job is to check the quality of data before processing it. So, we used fastqc for this purpose.

```
#!/bin/bash -e

#SBATCH --job-name=fastqc
#SBATCH --account=uoo02772
#SBATCH --time=1:00:00
#SBATCH --mem=5G
#SBATCH --ntasks=10
#SBATCH --cpus-per-task=1
#SBATCH --output=%x-%j.out
#SBATCH --error=%x-%j.err
#SBATCH --mail-type=ALL
#SBATCH --mail-user=katma889@student.otago.ac.nz

module load FastQC/0.11.9
fastqc -o ./fastqc/ -t 10 *.fastq
```
