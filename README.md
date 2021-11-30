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
## Vsearch
the second step after the quality check is to merge the paired end reads for which I tried Vsearch. As my amplicon size is around 400 bp so I kept minlegth 300 and maximum 600 as options. I used default options 10 and 100 for maxdiffs and maxdiffpct respectively.

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
## Cutadapt
Then we used Cutadapt to demultiplex  our data. first we created a metadata file including indiviadual sample name and their indexes.

`metadata.all.fasta`

```
>N1
^TATGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTCAGAGA
>N2
^TATGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCTGACGA
>C1
^TATGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCGTACGAT
>C2
^TATGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGATGCA
>C3
^TATGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGAGCGT
>C4
^TATGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTATGACA
>C5
^TATGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGACTGT
>C6
^TATGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGATAGA
>C7
^TATGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGACGATCT
>C8
^TATGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGCGTCA
>C9
^ACAGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTCAGAGA
>C10
^ACAGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCTGACGA
>C11
^ACAGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCGTACGAT
>C12
^ACAGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGATGCA
>C13
^ACAGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGAGCGT
>C16
^ACAGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTATGACA
>C17
^ACAGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGACTGT
>C18
^ACAGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGATAGA
>C19
^ACAGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGACGATCT
>C20
^ACAGCATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGCGTCA
>C21
^AGTACATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTCAGAGA
>C22
^AGTACATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCTGACGA
>C23
^AGTACATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCGTACGAT
>C24
^AGTACATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGATGCA
>C25
^AGTACATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGAGCGT
>C26
^AGTACATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTATGACA
>C27
^AGTACATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGACTGT
>C28
^AGTACATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGATAGA
>C29
^AGTACATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGACGATCT
>C30
^AGTACATCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGCGTCA
>C31
^AGTGCTGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTCAGAGA
>C32
^AGTGCTGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCTGACGA
>C33
^AGTGCTGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCGTACGAT
>C34
^AGTGCTGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGATGCA
>C35
^AGTGCTGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGAGCGT
>C36
^AGTGCTGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTATGACA
>C37
^AGTGCTGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGACTGT
>C38
^AGTGCTGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGATAGA
>C39
^AGTGCTGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGACGATCT
>C40
^AGTGCTGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGCGTCA
>P1
^TGCGATGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTCAGAGA
>L1
^TGCGATGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCTGACGA
>L2
^TGCGATGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCGTACGAT
>L3
^TGCGATGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGATGCA
>L4
^TGCGATGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGAGCGT
>L5
^TGCGATGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTATGACA
>L6
^TGCGATGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGACTGT
>L7
^TGCGATGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGATAGA
>L8
^TGCGATGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGACGATCT
>L9
^TGCGATGCGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGCGTCA
>L10
^ACGCAGAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTCAGAGA
>L11
^ACGCAGAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCTGACGA
>L12
^ACGCAGAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCGTACGAT
>L13
^ACGCAGAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGATGCA
>L14
^ACGCAGAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGAGCGT
>L15
^ACGCAGAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTATGACA
>L17
^ACGCAGAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGATAGA
>L18
^ACGCAGAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGACGATCT
>L19
^ACGCAGAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGCGTCA
>L20
^ACAGTCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTCAGAGA
>L21
^ACAGTCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCTGACGA
>L22
^ACAGTCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCGTACGAT
>L23
^ACAGTCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGATGCA
>L24
^ACAGTCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGAGCGT
>A1
^ACAGTCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTATGACA
>A2
^ACAGTCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGACTGT
>A3
^ACAGTCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGATAGA
>A4
^ACAGTCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGACGATCT
>A5
^ACAGTCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGCGTCA
>A6
^TCTATCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTCAGAGA
>A7
^TCTATCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCTGACGA
>A8
^TCTATCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCGTACGAT
>A9
^TCTATCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGATGCA
>A10
^TCTATCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGAGCGT
>A11
^TCTATCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTATGACA
>A12
^TCTATCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGACTGT
>A13
^TCTATCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGATAGA
>A15
^TCTATCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGCGTCA
>A16
^TAGTGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTCAGAGA
>A17
^AGTGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCTGACGA
>A18
^AGTGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCGTACGAT
>A19
^AGTGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGATGCA
>A20
^AGTGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGAGCGT
>A21
^AGTGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTATGACA
>A22
^AGTGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGACTGT
>A23
^AGTGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGATAGA
>A24
^AGTGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGACGATCT
>A25
^AGTGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGCGTCA
>A26
^TGACGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTCAGAGA
>A27
^TGACGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCTGACGA
>A28
^TGACGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCGTACGAT
>A29
^TGACGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGATGCA
>A30
^TGACGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGCGAGCGT
>A31
^TGACGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGTATGACA
>A32
^TGACGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGACTGT
>A33
^TGACGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGATAGA
>A34
^TGACGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCGACGATCT
>A35
^TGACGCAGGTGYCAGCMGCCGCGGTAA...ATTAGAWACCCBNGTAGTCCCTGCGTCA


```
script for cutadapt is:
```
#!/bin/bash -e

#SBATCH --job-name=demuxall
#SBATCH --account=uoo02772
#SBATCH --time=02:00:00
#SBATCH --mem=1G
#SBATCH --ntasks=16
#SBATCH --cpus-per-task=1
#SBATCH --output=%x-%j.out
#SBATCH --error=%x-%j.err
#SBATCH --mail-type=ALL
#SBATCH --mail-user=katma889@student.otago.ac.nz

mkdir All
module load cutadapt/3.3-gimkl-2020a-Python-3.8.2
cutadapt -g file:./metadata.all.fasta \
        -o ./All/{name}.fastq \
        ../3.merge/16s_merged.fastq \
        --untrimmed-output ./All/untrimmed.fastq \
        --no-indels -e 0.15
```
After doing the cutadapt script with our metadata, we were able to generate a single fastq file of each sample from the metadata file. At this step, we also removed the barcodes and primer sequences only keeping the amplicon sequences.
## Preparing sequencing files for quality filtering
After cutadapt we further did renaming of every sequences so that the information of the correspponding samples is well incorporated. Then we combined all relable files into a single file before we did quality filtering. This will be a great aid to generate our OTU table.
script for relabeling is:
```
for fq in trimmed*; do vsearch --fastq_filter $fq --relabel $fq. --fastqout relabel_$fq; done
Then we moved all relabeled file to output folder, Then we used the below command to merge all the relabel trimmed files into single relabel.fastq
cat relabel_trimmed* > relabel.fastq
Then we used Vsearch to dicard all the sequences that do no match specific set of min and maximum length based on the amplicon size. As our amplicon size is approximated 390 bp therefore we kept minimum 37o and maximum 430. The command is below:

vsearch --fastq_filter relabel.fastq --fastq_maxee 1.0 --fastq_maxlen 430 --fastq_minlen 370 --fastq_maxns 0 --fastaout ./filtered.fasta --fastqout ./filtered.fastq
But running the above command we got following result "Reading input file 100%  
2265 sequences kept (of which 0 truncated), 4498561 sequences discarded." 
##checking the number of reads in fastq file
grep -c '^@' untrimmed.fastq 
