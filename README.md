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
##Vsearch
the second step after the quality check is to merge the paired end reads for which I tried Vsearch. As my amplicon size is around 400 bp so I kept minlegth 300 and maximum 600 as options.

```
#!/bin/bash -e

#SBATCH --job-name=vsearch
#SBATCH --account=uoo02772
#SBATCH --time=00:10:00
#SBATCH --mem=1G
#SBATCH --ntasks=10
#SBATCH --cpus-per-task=1
#SBATCH --output=%x-%j.out
#SBATCH --error=%x-%j.err
#SBATCH --mail-type=ALL
#SBATCH --mail-user=katma889@student.otago.ac.nz

module load VSEARCH/2.17.1-GCC-9.2.0

vsearch --fastq_mergepairs ../6888-P1-00-01_S1_L001_R1_001.fastq \
        --reverse ../6888-P1-00-01_S1_L001_R2_001.fastq \
        --fastqout 16s_merged.fastq.gz \
        --fastqout_notmerged_fwd 16s_notmerged_fwd.fastq.gz \
        --fastqout_notmerged_rev 16s_notmerged_rev.fastq.gz \
        --fastq_maxdiffs 10 \
        --fastq_maxdiffpct 100 \
        --fastq_minmergelen 300 \
        --fastq_maxmergelen 600 \
        --threads 10
```
