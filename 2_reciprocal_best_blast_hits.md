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


This is the script I wrote to identify reciprocal best blast hits (identifies_reciptocal_best_blast_hits.pl):

```
#!/usr/bin/perl
use warnings;
use strict;

# This program reads in two blast outputs.  One is unique tymp transcripts blasted 
# against the unique oct transcripts and the other is the reverse.  We need to find tymp and oct ids
# that are each others best hit.


# on my computer I made a blast db like this:
# /usr/local/ncbi/blast/bin/makeblastdb -in infile -dbtype nucl -out infile_blastable

# To get the blast output that is parsed by this program, I used these commands

#/usr/local/blast/2.6.0/bin/blastn -query /home/ben/2014_Tympanoctomys_transcriptomes/Tympano/Tympano_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir/Tympa_all_transcriptomes_assembled_together_unique.fasta -db /home/ben/2014_Tympanoctomys_transcriptomes/Octomys/Octomys_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir/Octomys_all_transcriptomes_assembled_together_unique.fasta_blastable -outfmt 6 -out /net/infofile4-inside/volume1/scratch/ben/2017_Tymp_Oct_expression_analysis/reciprocal_best_unique_transcriptome_blast/tymp_to_oct.out -max_target_seqs 1
#/usr/local/blast/2.6.0/bin/blastn -query /home/ben/2014_Tympanoctomys_transcriptomes/Octomys/Octomys_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir/Octomys_all_transcriptomes_assembled_together_unique.fasta -db /home/ben/2014_Tympanoctomys_transcriptomes/Tympano/Tympano_joint_trinity_assembly_with_concatenated_reads/trinity_out_dir/Tympa_all_transcriptomes_assembled_together_unique.fasta_blastable -outfmt 6 -out /net/infofile4-inside/volume1/scratch/ben/2017_Tymp_Oct_expression_analysis/reciprocal_best_unique_transcriptome_blast/oct_to_tymp.out -max_target_seqs 1

my $outputfile = "octomys_tympa_reciprocal_best_blast_hits.out";

unless (open(OUTFILE, ">$outputfile"))  {
	print "I can\'t write to $outputfile  $!\n\n";
	exit;
}
print "Creating output file: $outputfile\n";

my $outputfile2 = "tympa_IDs_with_no_reciprocal_best_blast_hit_and_multiple_oct_seqs_that_match_it_nonreciprocally.out";

unless (open(OUTFILE2, ">$outputfile2"))  {
	print "I can\'t write to $outputfile2  $!\n\n";
	exit;
}
print "Creating output file: $outputfile2\n";

my $outputfile3 = "oct_IDs_with_no_reciprocal_best_blast_hit_and_multiple_tymp_seqs_that_match_it_nonreciprocally.out";

unless (open(OUTFILE3, ">$outputfile3"))  {
	print "I can\'t write to $outputfile3  $!\n\n";
	exit;
}
print "Creating output file: $outputfile3\n";





my @temp;
my %tymp_blast_results;
my %oct_blast_results;
my %reciprocal_hash;
my %other_reciprocal_hash;
my @tymp_IDs_with_no_reciprocal_best_blast_hit;
my @oct_IDs_with_no_reciprocal_best_blast_hit;
my $key;

# first open tymp blast hits
open (DATA, "tymp_to_oct.out") or die "Failed to open tymp Blast results";
while ( my $line = <DATA>) {
	@temp = split("\t",$line);
	# assign the tymp hash irrespective of hit quality
	if (defined($tymp_blast_results{$temp[0]})){
		if($tymp_blast_results{$temp[0]} ne $temp[1]){
			print $temp[0]," has top matches to multiple different oct seqs (",$tymp_blast_results{$temp[0]}," and ",$temp[1],").\n";
		}
	}
	else{
		$tymp_blast_results{$temp[0]} = $temp[1];
	}
}	
close DATA;

# now open oct blast hits and find reciprocal best blast hit pairs
open (DATA2, "oct_to_tymp.out") or die "Failed to open oct Blast results";
	while ( my $line2 = <DATA2>) {
		@temp = split("\t",$line2);
		if (defined($oct_blast_results{$temp[0]})){
			if($oct_blast_results{$temp[0]} ne $temp[1]){
				print $temp[0]," has top matches to multiple different tymp seqs (",$oct_blast_results{$temp[0]}," and ",$temp[1],").\n";
			}
		}
		else{
			$oct_blast_results{$temp[0]} = $temp[1];
		}

		if ((defined($tymp_blast_results{$temp[1]}))&&($tymp_blast_results{$temp[1]} eq $temp[0])){
			# these may be reciprocal best blast hits
			# check if it is in a pair that has already been identified
			if(defined ($reciprocal_hash{$temp[0]}) ){
				# this is a check for duplicated entries:
				if($reciprocal_hash{$temp[0]} ne $temp[1]){
					print "This pair was previously defined but there appears to be a problem \n
					because the oct seq also matched another tymp seq: ",$reciprocal_hash{$temp[0]}," and",$temp[1],"\n";
				}
			}
			else{
				# thus the key is the octomys seq and the value is the tympa seq
				$reciprocal_hash{$temp[0]}=$temp[1];
				# the other one has the key as tympa seq and the value is the oct seq
				$other_reciprocal_hash{$temp[1]}=$temp[0];
			}
		}
		elsif ((defined($tymp_blast_results{$temp[1]}))&&($tymp_blast_results{$temp[1]} ne $temp[0])){
			push(@oct_IDs_with_no_reciprocal_best_blast_hit,$temp[0]);
			push(@tymp_IDs_with_no_reciprocal_best_blast_hit,$temp[1]);
		}
		else{
			print "This tymp was matched by octomys but didn't match octomys back: ",$temp[1],"\n";
			#push(@tymp_IDs_with_no_reciprocal_best_blast_hit,$temp[1]);
		}	
	}
close DATA2;

# now open the tymp blast hits again and confirm that the top hits were 1:1 only
open (DATA, "tymp_to_oct.out") or die "Failed to open tymp Blast results";
	while ( my $line2 = <DATA>) {
		@temp = split("\t",$line2);
		if(defined ($reciprocal_hash{$temp[1]}) ){
			# this is a check for duplicated entries:
			if($reciprocal_hash{$temp[1]} ne $temp[0]){
				# This tymp seq matches an oct seqs that is not its reciprocal best 
				print OUTFILE2 "tympseq ",$temp[0],"\toctseq ",$temp[1],"\ttymp_has_nonreciprocalmatch_to_oct\n";
			}
		}
		else{
				print OUTFILE2 "tympseq ",$temp[0],"\toctseq ",$temp[1],"\tnonreciprocal\n";			
		}
	}
close DATA;

# now do the same for oct
open (DATA2, "oct_to_tymp.out") or die "Failed to open oct Blast results";
	while ( my $line2 = <DATA2>) {
		@temp = split("\t",$line2);
		if(defined ($other_reciprocal_hash{$temp[1]}) ){
			# this is a check for duplicated entries:
			if($other_reciprocal_hash{$temp[1]} ne $temp[0]){
				# This oct seq matches a tymp seqs that is not its reciprocal best
				print OUTFILE3 "octseq ",$temp[0],"\ttympseq ",$temp[1],"\toct_has_nonreciprocalmatch_to_tymp\n";
			}
		}
		else{
				print OUTFILE3 "octseq ",$temp[0],"\ttympseq ",$temp[1],"\tnonreciprocal\n";			
		}
	}
close DATA2;


foreach $key (sort(keys %reciprocal_hash)) {
	print OUTFILE $key,"\t",$reciprocal_hash{$key},"\n";
}

#print "Weird ones ",$#tymp_IDs_with_no_reciprocal_best_blast_hit+1,"\n";

close OUTFILE;
```
