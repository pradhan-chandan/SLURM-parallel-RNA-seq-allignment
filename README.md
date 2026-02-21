# SLURM-parallel-RNA-seq-allignment
SLURM batch script for parallel RNA-seq read alignment using STAR, designed for HPC environments with job array support for multi-sample processing


Features:

Parallel processing of multiple samples via SLURM job arrays
Supports paired-end FASTQ input (.fq.gz)
Configurable intron length thresholds (alignIntronMin / alignIntronMax)
Handles non-standard GFF/GTF annotations where exons are annotated as CDS
Outputs coordinate-sorted BAM files per sample
Automatically organizes output into per-sample directories
