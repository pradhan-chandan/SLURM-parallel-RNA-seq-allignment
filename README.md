# SLURM-parallel-RNA-seq-allignment
SLURM batch script for parallel RNA-seq read alignment using STAR, designed for HPC environments with job array support for multi-sample processing


Features:

Parallel processing of multiple samples via SLURM job arrays
Supports paired-end FASTQ input (.fq.gz)
Configurable intron length thresholds (alignIntronMin / alignIntronMax)
Handles non-standard GFF/GTF annotations where exons are annotated as CDS
Outputs coordinate-sorted BAM files per sample
Automatically organizes output into per-sample directories


code:
(..... lines are comments)


#!/bin/bash

#SBATCH --profile=All
#SBATCH --job-name=B157_Alignment
#SBATCH --account=xxxx
#SBATCH --cpus-per-task=4
#SBATCH --mem=50G
#SBATCH --time=24:00:00
#SBATCH --array=0-9                        # Array index for 10 samples (adjust based on number of samples)
#SBATCH --output=Alignment_output_and_error/B157_Alignment%A_%a.out
#SBATCH --error=Alignment_output_and_error/B157_Alignment%A_%a.err


module load star/2.7.11b


# Set paths
FASTQ_DIR=/path_to_folder_fastq/
GENOME_DIR=/path_to_folder_genome_directory/
gtf_file=/path_to_folder_genome_annotation/B157.gff
OUTPUT_DIR=/path_to_folder_output/

THREADS=${SLURM_CPUS_PER_TASK}  # Threads per sample


FASTQ_FILES=($(ls ${FASTQ_DIR}/*_1.fq.gz))          ..... List of R1 files (paired-end reads)
R1_FILE=${FASTQ_FILES[$SLURM_ARRAY_TASK_ID]}          ..... Select the R1 file based on the array index
BASE_NAME=$(basename ${R1_FILE} _1.fq.gz)          ..... Extract the sample name
R2_FILE="${FASTQ_DIR}/${BASE_NAME}_2.fq.gz"          ..... Define the corresponding R2 file

    
# Create output directory for sample
SAMPLE_NAME=${BASE_NAME%%-*}
SAMPLE_OUTPUT_DIR="${OUTPUT_DIR}/${SAMPLE_NAME}_B157_500max"
mkdir -p "${SAMPLE_OUTPUT_DIR}"
    
    
# Run STAR alignment

STAR --runThreadN ${THREADS} \
 --genomeDir "${GENOME_DIR}" \
 --readFilesCommand gunzip -c \                     .....to decompress the files
 --readFilesIn ${R1_FILE} ${R2_FILE} \              .....read the fastq files
 --sjdbGTFfile ${gtf_file} \                        .....read gtf file
 --sjdbGTFfeatureExon CDS \                         .....read exons mentioned as cds in annotation file (only required if exons are mentioned something else)
 --outFileNamePrefix "${SAMPLE_OUTPUT_DIR}/" \
 --alignIntronMin 20 \                              .....minimun intron length specified
 --alignIntronMax 500 \                             .....maximum intron length specified
 --outSAMtype BAM SortedByCoordinate                .....output as .bam file
   
    
    echo "Alignment completed for ${SAMPLE_NAME}"
