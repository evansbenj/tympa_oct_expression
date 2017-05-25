# Reciprocal best blasting

We need to identify orthologs in Tymp and Oct so we can figure out what to compare to what.

Blast unique seqs from each db against the other and save tophit:

Tymp to Oct
```
/usr/local/blast/2.6.0/bin/blastn -query /home/ben/2014_Tympanoctomys_transcriptomes/Tympano/Tympano_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir/Tympa_all_transcriptomes_assembled_together_unique.fasta -db /home/ben/2014_Tympanoctomys_transcriptomes/Octomys/Octomys_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir/Octomys_all_transcriptomes_assembled_together_unique.fasta_blastable -outfmt 6 -out /net/infofile4-inside/volume1/scratch/ben/2017_Tymp_Oct_expression_analysis/reciprocal_best_unique_transcriptome_blast/tymp_to_oct.out -max_target_seqs 1
```
Oct to Tymp
```
/usr/local/blast/2.6.0/bin/blastn -query /home/ben/2014_Tympanoctomys_transcriptomes/Octomys/Octomys_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir/Octomys_all_transcriptomes_assembled_together_unique.fasta -db /home/ben/2014_Tympanoctomys_transcriptomes/Tympano/Tympano_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir/Tympa_all_transcriptomes_assembled_together_unique.fasta_blastable -outfmt 6 -out /net/infofile4-inside/volume1/scratch/ben/2017_Tymp_Oct_expression_analysis/reciprocal_best_unique_transcriptome_blast/oct_to_tymp.out -max_target_seqs 1
```
