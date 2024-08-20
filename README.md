# This Pipeline is desinged for Human RNAseq at transcript level

## Trimmomatic
This step will filter the reads

```
java -jar $PATH/Trimmomatic-0.39/trimmomatic-0.39.jar SE \
-phred33 -threads 32 ${infile} ${base}_trimmed.fastq.gz \
ILLUMINACLIP:$PATH/Trimmomatic-0.39/adapters/TruSeq3-SE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
```

## Hisat2
Hisat2 alignment with multiple files per sample using human grch38 transcripts reference 
 ```
$PATH/hisat2/hisat2 --dta -q -p 8 -x $PATH/hisat2/indexes/grch38_tran/genome_tran -U ${base}_1_trimmed.fastq.gz,${base}_2_trimmed.fastq.gz -S ${base}.sam
```

## Samtools
Sorted .bam generation

Activate samtools
```
conda activate samtools
```

Run samtools

```
samtools view -Su ${infile} | samtools sort -o ${base}.sorted.bam
```

## StringTie

Transcript quantification

```
$PATH/stringtie/stringtie -e -B -G $PATH/hisat2/indexes/grch38.gtf -o ${base}_stringtie_output/${base}.gtf -l ${base}_stringtie_output/${base} ${infile}
```

## StringTie Merge

Merge all .gtf to consolidate quantifications for all samples

First we need to generate a mergelist.txt

```
for i in *output/*gtf;do  echo $i; done > mergelist.txt
```

Then, we can run the merge

```
$PATH/stringtie/stringtie --merge -p 8 -G $PATH/hisat2/indexes/grch38.gtf -o merged.gtf mergelist.txt
```

## StringTie Re-run

Now, we rerun Stringtie with the merged .gtf file

```
$PATH/stringtie/stringtie -e -B -G glunisertib_merged.gtf -o ${base}_stringtie_output_merged/${base}.gtf -l ${base}_stringtie_output_merged/${base} ${infile}
```

## Prepare for DESeq2

Finally, we generate a table with all counts for DESeq2 using StrinTie script

First, we generate sample_IDs.txt
```
for infile in *_output_merged/*.gtf; do base=$(basename -s .gtf $infile);echo $base $infile; done > sample_IDs.txt
```

Then, run the script

```
python $PATH/stringtie/prepDE.py3 -i sample_IDs.txt
```
