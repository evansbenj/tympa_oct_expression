# Align trimmed reads

I am using the trinity assembly from the GBE paper that was proocessed with cdhit to keep only unique reads as a reference for the cooresponding data from Tymp and Oct.

# Tymp data
On info, the data within /net/infofile4-inside/volume1/scratch/ben/2017_Tymp_Oct_expression_analysis/Tympa

Here are the files:
```
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
```

# I previously prepare the assemblies for mapping 
```
/usr/local/bin/bwa index -a bwtsw /home/ben/2014_Tympanoctomys_transcriptomes/Tympano/Tympano_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir/Tympa_all_transcriptomes_assembled_together_unique.fasta
/usr/local/bin/samtools faidx /home/ben/2014_Tympanoctomys_transcriptomes/Tympano/Tympano_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir/Tympa_all_transcriptomes_assembled_together_unique.fasta
```
not sure if I need this:
```
~/jre1.8.0_111/bin/java -Xmx2g -jar ~/picard-tools-1.131/picard.jar CreateSequenceDictionary REFERENCE=reference.fa OUTPUT=reference.dict
```


Define an bash array with all of the name prefixes
```
samples[1]=AO245_Heart
samples[2]=AO245_Liver
samples[3]=AO245_Muscle
samples[4]=AO245_Kidney
samples[5]=AO245_Lung
samples[6]=AO245_Testis
```
Define the ref seq
```
octreference=/home/ben/2014_Tympanoctomys_transcriptomes/Octomys/Octomys_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir/Octomys_all_transcriptomes_assembled_together_unique.fasta
echo $octreference

tympreference=/home/ben/2014_Tympanoctomys_transcriptomes/Tympano/Tympano_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir/Tympa_all_transcriptomes_assembled_together_unique.fasta
echo $tympreference
```

# gff3 files
oct
```
/home/ben/2014_Tympanoctomys_transcriptomes/Octomys/Octomys_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir/Octomys_all_transcriptomes_assembled_together_unique.fasta.transdecoder_dir/longest_orfs.gff3
```
tymp
```
/home/ben/2014_Tympanoctomys_transcriptomes/Tympano/Tympano_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir/Tympa_all_transcriptomes_assembled_together_unique.fasta.transdecoder_dir/longest_orfs.gff3
```

Now do the alignment with a bash script
```
#!/bin/bash                                                                                                                               

tympreference="/home/ben/2014_Tympanoctomys_transcriptomes/Tympano/Tympano_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir\
/Tympa_all_transcriptomes_assembled_together_unique.fasta"

annotation="/home/ben/2014_Tympanoctomys_transcriptomes/Tympano/Tympano_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir/Ty\
mpa_all_transcriptomes_assembled_together_unique.fasta.transdecoder_dir/longest_orfs.gff3"

samples="AO245_Heart                                                                                                                      
AO245_Liver                                                                                                                               
AO245_Muscle                                                                                                                              
AO245_Kidney                                                                                                                              
AO245_Lung                                                                                                                                
AO245_Testis"

for sample in $samples
do
    echo ${sample}
    #Map the reads                                                                                                                        
    # make a sam file                                                                                                                     
    /usr/local/bin/bwa mem -M -t 16 ${tympreference} ${sample}_R1_trim_paired.fastq.gz ${sample}_R2_trim_paired.fastq.gz > ${sample}.sam
    # make a bam file                                                                                                                     
    /usr/local/bin/samtools view -bt ${tympreference} -o ${sample}.bam ${sample}.sam                                                     
    # delete the sam file                                                                                                                 
    rm -f ${sample}.sam                                                                                                                  
    # Sort by read name                                                                                                                   
    samtools sort -n ${sample}.bam -o ${sample}_sorted.bam                                                                               
    #Count the number of reads mapping to each feature using HTSeq                                                                        
    htseq-count --format=bam --stranded=no --order=name --type=gene --idattr=ID ${sample}_sorted.bam ${annotation} > ${sample}_htseq_counts.txt
done
```
