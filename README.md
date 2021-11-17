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

`metadata.fasta`

```
>crw_c1_m_np_rk
^TATGCATC...ATCGTACG
>crw_c2_f_np_rk
^TATGCATC...TGCATCGC
>crw_c3_f_np_rk
^TATGCATC...ACGCTCGC
>crw_c4_m_np_rk
^TATGCATC...TGTCATAC
>crw_c5_m_np_rk
^TATGCATC...ACAGTCAG
>crw_c6_m_np_rk
^TATGCATC...TCTATCAG
>crw_c7_m_np_rk
^TATGCATC...AGATCGTC
>crw_c8_m_np_rk
^TATGCATC...TGACGCAG
>crw_c9_m_np_rk
ACAGCATC...TCTCTGAC
>crw_c10_m_p_rk
^ACAGCATC...TCGTCAGC
>crw_c11_f_p_rk
^ACAGCATC...ATCGTACG
>crw_c12_f_p_rk
^ACAGCATC...TGCATCGC
>crw_c13_1_parasite_rk
^ACAGCATC...ACGCTCGC
>crw_c16_1_parasite_rk
^ACAGCATC...TGTCATAC
>crw_c17_1_parasite_rk
^ACAGCATC...ACAGTCAG
>crw_c18_1_parasite_rk
^ACAGCATC...TCTATCAG
>crw_c19_1_parasite_rk
^ACAGCATC...AGATCGTC
>crw_c20_1_parasite_rk
^ACAGCATC...TGACGCAG
>crw_c21_4_parasite_rk
^AGTACATC...TCTCTGAC
>crw_c22_f_p_lin
^AGTACATC...TCGTCAGC
>crw_c23_f_p_lin
^AGTACATC...ATCGTACG
>crw_c24_f_p_lin
^AGTACATC...TGCATCGC
>crw_c25_m_p_lin
^AGTACATC...ACGCTCGC
>crw_c26_f_np_lin
^AGTACATC...TGTCATAC
>crw_c27_m_np_lin
^AGTACATC...ACAGTCAG
>crw_c28_f_np_lin
^AGTACATC...TCTATCAG
>crw_c29_f_np_lin
^AGTACATC...AGATCGTC
>crw_c30_f_np_lin
^AGTACATC...TGACGCAG
>crw_c31_f_p_lin
^AGTGCTGC...TCTCTGAC
>crw_c32_m_p_lin
^AGTGCTGC...TCGTCAGC
>crw_c33_2_parasite_lin
^AGTGCTGC...ATCGTACG
>crw_c34_m_np_lin
^AGTGCTGC...TGCATCGC
>crw_c35_m_p_lin
^AGTGCTGC...ACGCTCGC
>crw_c36_f_p_lin
^AGTGCTGC...TGTCATAC
>crw_c37_f_p_lin
^AGTGCTGC...ACAGTCAG
>crw_c38_m_p_lin
^AGTGCTGC...TCTATCAG
>crw_c39_m_p_lin
^AGTGCTGC...AGATCGTC
>crw_c40_f_p_lin
^AGTGCTGC...TGACGCAG
>pos_all_weevils
^TGCGATGC...TCTCTGAC
>neg_all_weevils
^TATGCATC...TCGTCAGC
>qpcr-neg_all_weevils
^TATGCATC...TCTCTGAC
>lw_l1_f_np_lin
^TGCGATGC...TCGTCAGC
>lw_l2_m_np_lin
^TGCGATGC...ATCGTACG
>lw_l3_f_np_lin
^TGCGATGC...TGCATCGC
>lw_l4_f_np_lin
^TGCGATGC...ACGCTCGC
>lw_l5_m_p_lin
^TGCGATGC...TGTCATAC
>lw_l6_m_p_lin
^TGCGATGC...ACAGTCAG
>lw_l7_f_np_lin
^TGCGATGC...TCTATCAG
>lw_l8_m_p_lin
^TGCGATGC...AGATCGTC
>lw_l9_m_np_lin
^TGCGATGC...TGACGCAG
>lw_l10_f_np_lin
^ACGCAGAG...TCTCTGAC
>lw_l11_f_np_lin
^ACGCAGAG...TCGTCAGC
>lw_l12_m_np_lin
^ACGCAGAG...ATCGTACG
>lw_l13_1_parasite_lin
^ACGCAGAG...TGCATCGC
>lw_l14_m_p_lin
^ACGCAGAG...ACGCTCGC
>lw_l15_m_p_lin
^ACGCAGAG...TGTCATAC
>lw_l16_m_p_lin
^ACGCAGAG...ACAGTCAG
>lw_l17_1_parasite_lin
^ACGCAGAG...TCTATCAG
>lw_l18_f_np_gr
^ACGCAGAG...AGATCGTC
>lw_l19_f_np_gr
^ACGCAGAG...TGACGCAG
>lw_l20_f_np_gr
^ACAGTCAG...TCTCTGAC
>lw_l21_f_np_gr
^ACAGTCAG...TCGTCAGC
>lw_l22_m_np_gr
^ACAGTCAG...ATCGTACG
>lw_l23_m_np_gr
^ACAGTCAG...TGCATCGC
>lw_l24_m_p_gr
^ACAGTCAG...ACGCTCGC
>asw_a1_m_np_lin
^ACAGTCAG...TGTCATAC
>asw_a2_m_np_lin
^ACAGTCAG...ACAGTCAG
>asw_a3_m_np_lin
^ACAGTCAG...TCTATCAG
>asw_a4_m_p_lin
^ACAGTCAG...AGATCGTC
>asw_a5_m_np_lin
^ACAGTCAG...TGACGCAG
>asw_a6_f_np_lin
^TCTATCAG...TCTCTGAC
>asw_a7_f_np_lin
^TCTATCAG...TCGTCAGC
>asw_a8_f_np_lin
^TCTATCAG...ATCGTACG
>asw_a9_f_p_lin
^TCTATCAG...TGCATCGC
>asw_a10_f_np_lin
^TCTATCAG...ACGCTCGC
>asw_a11_f_np_lin
^TCTATCAG...TGTCATAC
>asw_a12_m_p_lin
^TCTATCAG...ACAGTCAG
>asw_a13_f_np_lin
^TCTATCAG...TCTATCAG
>asw_a15_1_parasite_lin
^TCTATCAG...TGACGCAG
>asw_a16_f_p_lin
^TAGTGCAG...TCTCTGAC
>asw_a17_m_p_lin
^TAGTGCAG...TCGTCAGC
>asw_a18_m_p_lin
^TAGTGCAG...ATCGTACG
>asw_a19_1_parasite_lin
^TAGTGCAG...TGCATCGC
>asw_a20_2_parasite_lin
^TAGTGCAG...ACGCTCGC
>asw_21_m_np_lin
^TAGTGCAG...TGTCATAC
>asw_22_f_np_lin
^TAGTGCAG...ACAGTCAG
>asw_23_m_p_lin
^TAGTGCAG...TCTATCAG
>asw_24_m_p_lin
^TAGTGCAG...AGATCGTC
>asw_25_f_p_lin
^TAGTGCAG...TGACGCAG
>asw_a26_m_p_lin
^TGACGCAG...TCTCTGAC
>asw_a27_m_p_lin
^TGACGCAG...TCGTCAGC
>asw_a28_m_p_lin
^TGACGCAG...ATCGTACG
>asw_a29_3_parasites_lin
^TGACGCAG...TGCATCGC
>asw_a30_m_np_lin
^TGACGCAG...ACGCTCGC
>asw_a31_f_np_lin
^TGACGCAG...TGTCATAC
>asw_a32_m_p_lin
^TGACGCAG...ACAGTCAG
>asw_a33_na_p_lin
^TGACGCAG...TCTATCAG
>asw_a34_m_p_lin
^TGACGCAG...AGATCGTC
>asw_a35_3_parsite_lin
^TGACGCAG...TGACGCAG
```
script for cutadapt is:
```
#!/bin/bash -e

#SBATCH --job-name=demux
#SBATCH --account=uoo02772
#SBATCH --time=02:00:00
#SBATCH --mem=1G
#SBATCH --ntasks=6
#SBATCH --cpus-per-task=1
#SBATCH --output=%x-%j.out
#SBATCH --error=%x-%j.err
#SBATCH --mail-type=ALL
#SBATCH --mail-user=katma889@student.otago.ac.nz

module load cutadapt/3.3-gimkl-2020a-Python-3.8.2
cutadapt -g file:./metadata.fasta \
        -o ./demux_output/trimmed_{name}.fastq \
        ../16s_merged.fastq \
        --untrimmed-output ./demux_output/untrimmed.fastq \
        --no-indels -e 0
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


