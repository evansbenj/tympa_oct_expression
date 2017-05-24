# Align trimmed reads

I amo using the trinity assembly from the GBE paper that was proocessed with cdhit to keep only unique reads as a reference for the cooresponding data from Tymp and Oct.

# Tymp data
On info, the data within /net/infofile4-inside/volume1/scratch/ben/2017_Tymp_Oct_expression_analysis/Tympa

Here are the files:
AO245_Heart_R1_trim_paired.fastq.gz   
AO245_Heart_R2_trim_paired.fastq.gz   
AO245_Liver_R1_trim_paired.fastq.gz  
AO245_Liver_R2_trim_paired.fastq.gz  
AO245_Muscle_R1_trim_paired.fastq.gz
AO245_Muscle_R2_trim_paired.fastq.gz
AO245_Kidney_R1_trim_paired.fastq.gz  
AO245_Kidney_R2_trim_paired.fastq.gz  
AO245_Lung_R1_trim_paired.fastq.gz   
AO245_Lung_R2_trim_paired.fastq.gz   
AO245_Testis_R1_trim_paired.fastq.gz
AO245_Testis_R2_trim_paired.fastq.gz

# I previously prepare the assemblies for mapping 

/usr/local/bin/bwa index -a bwtsw reference.fa
/usr/local/bin/samtools faidx reference.fa
~/jre1.8.0_111/bin/java -Xmx2g -jar ~/picard-tools-1.131/picard.jar CreateSequenceDictionary REFERENCE=reference.fa OUTPUT=reference.dict



Define an bash array with all of the name prefixes
```
samples[1]=AO245_Heart
samples[2]=AO245_Liver
samples[3]=AO245_Muscle
samples[4]=AO245_Kidney
samples[5]=AO245_Lung
samples[6]=AO245_Testis
```
Now do the alignment with a bash script
```
for i in {1..6}
do
    sample=${samples[${i}]}
    echo ${sample}
    #Map the reads
    /usr/local/tophat-2.0.8/tophat -p 4 -G ${annotation} -o ${sample} ${reference} \
     ${sample}_R1_trim_paired.fastq.gz \
     ${sample}_R2_trim_paired.fastq.gz \

    # make a sam file
    /usr/local/bin/bwa mem -M -t 16 ${reference} ${sample}_R1_trim_paired.fastq.gz ${sample}_R2_trim_paired.fastq.gz > ${sample}.sam
    # make a bam file
    /usr/local/bin/samtools view -bt ${reference} -o ${sample}.bam ${sample}.sam
    # delete the sam file
    rm -f ${sample}.sam
    # sort the file
    samtools sort -on file.bam | samtools view - | htseq-count (options) - GTF > counts.txt
    
    #Count the number of reads mapping to each feature using HTSeq
    htseq-count --format=bam --stranded=no --order=pos ${sample}.bam ${annotation} > ${sample}_htseq_counts.txt
done
```
