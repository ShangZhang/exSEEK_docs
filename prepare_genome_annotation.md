# Prepare Genome and Annotation

* Sequence
* Annotation
* Index files

## Genome hg38

### Sequence
#### download from gencode v27
- [ ]located in folder: 
#### located in folder: sequence/

```
mkdir sequence
cd sequence
wget wget ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_27/GRCh38.p10.genome.fa.gz
```
#### Decompression

```
gzip -d GRCh38.p10.genome.fa.gz 
```

#### parse genome

```
samtools faidx GRCh38.p10.genome.fa
cut -f1,2 GRCh38.p10.genome.fa.fai > hg38.chrom.sizes
```

### Index 
#### build by bowtie2 and STAR
- [ ]located in folder: 
#### located in folder: index/

* using bowtie2

```
mkdir bowtie2_hg38_index
cd bowtie2_hg38_index
bowtie2-build ../sequence/GRCh38.p10.genome.fa GRCh38.p10
```

* using STAR

```
mkdir ../STAR_hg38_index
cd ../STAR_hg38_index
STAR --runMode genomeGenerate --runThreadN 15 --genomeDir . --genomeFastaFiles ../sequence/GRCh38.p10.genome.fa
```


### Annotation files for genome hg38

provider: GENCODE and other projects

format: gtf and gff

#### statistics
| **RNA_type** | **gene_num** | **transcrips_num** | **source** | **file** | **Date** | **Download/Processed** | **Note** |
| :------- |:-------|:------|:-----|:---------|:-----|:-----|:------|
| allGenes | 58,288 | 200,401 | Gencode27 | gencode.v27.annotation.gtf / gencode.v27.annotation.gff | 2018.5.4 | [D/D](#download-gencode-v27-annotations) ||
| rRNA  | 544 | 544 | Gencode27 | rRNA.gencode27.gtf / rRNA.gencode27.gff | 2018.5.4 | [P/P](#parse-annotations) | |
| miRNA | 1,881 | 1,881 | Gencode27 | miRNA.gencode27.gtf / miRNA.gencode27.gff | 2018.5.4 | [P/P](#parse-annotations) | |
| piRNA | 812,347 | 812,347 | piRBase | piRNA.piRBase.hg38.gtf / piRNA.piRBase.hg38.gff | 2018.5.4 | [D/D](#download-piRNA-from-piRBase) ||
| snoRNA | 943 | 955 | Gencode27(misc_RNA) | snoRNA.gencode27.gtf / snoRNA.gencode27.gtf | 2018.5.4 | [P/P](#parse-annotations) ||
| snRNA | 1,900 | 1,900 | Gencode27 | snRNA.gencode27.gtf / snRNA.gencode27.gtf | 2018.5.4 | [P/P](#parse-annotations) ||
| srpRNA | 680 | 680 | Gencode27(misc_RNA) | srpRNA.gencode27.gtf / srpRNA.gencode27.gff | 2018.5.4 | [P/P](#parse-annotations) ||
| tRNA | 649 | 649 | Gencode27(predicted tRNA) | tRNA.gencode27.gtf / tRNA.gencode27.gff | 2018.5.4 | [D/D](#download-gencode-v27-annotations) ||
| lncRNA | 15,778 | 27,908 | Gencode27(lncRNA) | lncRNA.gencode27.gtf / lncRNA.gencode27.gff | 2018.5.4 | [D/D](#download-gencode-v27-annotations) ||
| lncRNA | 96,308 | 172,216 | NONCODEv5 | lncRNA.NONCODEv5.hg38.gtf / lncRNA.NONCODEv5.hg38.gff | 2018.5.4 | [P/P](#parse-and-convert) ||
| lncRNA | 63,427 | 174,657 | mitranscritome | lncRNA.mitranscriptome.v2.hg38.gtf / lncRNA.mitranscriptome.v2.hg38.gff | 2018.5.4 | [P/P](#parse-and-convert) ||
| tucp | 3,711 | 11,244 | mitranscritome | tucp.mitranscriptome.v2.hg38.gtf / tucp.mitranscriptome.v2.hg38.gff | 2018.5.4 | [P/P](#parse-and-convert) ||
| lncRNA | 131,683| 342,295 | Gencode27+NONCODEv5+ MiTranscriptome+NC2017 | merged_lncRNA.combined.gtf / merged_lncRNA.combined.gff | 2018.5.4 | [P/P](#parse-and-convert) ||
| mRNA | 19,836 | 80,930 | Gencode27(protein_coding) | mRNA.gencode27.gtf / mRNA.gencode27.gff | 2018.5.4 | [P/P](#parse-annotations) ||

#### pre-process annotaion
#### download gencode v27 annotations
```
wget ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_27/gencode.v27.annotation.gtf.gz
wget ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_27/gencode.v27.annotation.gff3.gz
wget ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_27/gencode.v27.long_noncoding_RNAs.gtf.gz
wget ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_27/gencode.v27.long_noncoding_RNAs.gff3.gz
wget ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_27/gencode.v27.tRNAs.gtf.gz
wget ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_27/gencode.v27.tRNAs.gff3.gz
```
#### Decompression
```
gzip -d *
```

#### parse annotations
```
gtf=gencode.v27.annotation.gtf
for i in rRNA snRNA snoRNA srpRNA miRNA vaultRNA Y_RNA; do
grep "gene_type \"$i\"" $gtf > $i.gencode27.gtf
gffread -E $i.gencode27.gtf -o- > $i.gencode27.gff
done;

grep "gene_type \"protein_coding\"" $gtf | grep -v Selenocysteine > mRNA.gencode27.gtf
grep "RN7SL" $gtf > srpRNA.gencode27.gtf
grep 'Y_RNA' $gtf > Y_RNA.gencode27.gtf 

for i in mRNA srpRNA Y_RNA; do
gffread -E $i.gencode27.gtf -o- > $i.gencode27.gff
done;

mv gencode.v27.tRNAs.gtf tRNA.gencode27.gtf
mv gencode.v27.tRNAs.gff3 tRNA.gencode27.gff

mv gencode.v27.long_noncoding_RNAs.gtf lncRNA.gencode27.gtf
mv gencode.v27.long_noncoding_RNAs.gff3 lncRNA.gencode27.gff

mv gencode.v27.annotation.gff3 gencode.v27.annotation.gff
```

#### download piRNA from piRBase

#### downlaod lncRNA from NONCODE(http://www.noncode.org/), mitranscriptome(http://mitranscriptome.org/), and Yang Yang's NC paper
```
wget http://mitranscriptome.org/download/mitranscriptome.gtf.tar.gz
wget http://www.noncode.org/datadownload/NONCODEv5_human_hg38_lncRNA.gtf.gz
wget https://media.nature.com/original/nature-assets/ncomms/2017/170213/ncomms14421/extref/ncomms14421-s3.txt
```
#### Decompression
```
tar zxvf mitranscriptome.gtf.tar.gz
gzip -d mitranscriptome.v2.gtf.gz
```
#### parse and convert 
```
wget http://hgdownload.soe.ucsc.edu/goldenPath/hg19/liftOver/hg19ToHg38.over.chain.gz
gzip -d hg19ToHg38.over.chain.gz 
cd mitranscriptome.gtf
gzip -d mitranscriptome.v2.gtf.gz
gtfToGenePred -genePredExt mitranscriptome.v2.gtf mitranscriptome.v2.gp
liftOver -genePred mitranscriptome.v2.gp ../hg19ToHg38.over.chain mitranscriptome.v2.hg38.gp unmapped.gtf
genePredToGtf -utr -source=mitranscriptome file mitranscriptome.v2.hg38.gp mitranscriptome.v2.hg38.gtf
gffread -E mitranscriptome.v2.hg38.gtf -o- > mitranscriptome.v2.hg38.gff
mv ../mitranscriptome.gtf ../mitranscriptome
cd ..

# filter out lncRNA
cd mitranscriptome.gtf
zcat mitranscriptome.v2.gtf.gz | grep -e 'tcat \"lncrna\"' > lncRNA.mitranscriptome.v2.gtf
# liftOver hg19 to hg38
gtfToGenePred -genePredExt lncRNA.mitranscriptome.v2.gtf lncRNA.mitranscriptome.v2.gp
liftOver -genePred lncRNA.mitranscriptome.v2.gp ../hg19ToHg38.over.chain lncRNA.mitranscriptome.v2.hg38.gp unmapped.gtf
genePredToGtf -utr -source=mitranscriptome file lncRNA.mitranscriptome.v2.hg38.gp lncRNA.mitranscriptome.v2.hg38.gtf
gffread -E lncRNA.mitranscriptome.v2.hg38.gtf -o- > lncRNA.mitranscriptome.v2.hg38.gff

# filter out tucp
zcat mitranscriptome.v2.gtf.gz | grep -e 'tcat \"tucp\"' | awk '$7!="."' > tucp.mitranscriptome.v2.gtf
# liftOver hg19 to hg38
gtfToGenePred -genePredExt tucp.mitranscriptome.v2.gtf tucp.mitranscriptome.v2.gp
liftOver -genePred tucp.mitranscriptome.v2.gp ../hg19ToHg38.over.chain tucp.mitranscriptome.v2.hg38.gp unmapped.gtf
genePredToGtf -utr -source=mitranscriptome file tucp.mitranscriptome.v2.hg38.gp tucp.mitranscriptome.v2.hg38.gtf
gffread -E tucp.mitranscriptome.v2.hg38.gtf -o- > tucp.mitranscriptome.v2.hg38.gff

cd ..
gtfToGenePred -genePredExt ncomms14421-s3.txt lncRNA.lulab_ncomms14421.gp
liftOver -genePred lncRNA.lulab_ncomms14421.gp hg19ToHg38.over.chain lncRNA.lulab_ncomms14421.hg38.gp unmapped.gtf
genePredToGtf -utr -source=lulab_ncomms14421 file lncRNA.lulab_ncomms14421.hg38.gp lncRNA.lulab_ncomms14421.hg38.gtf
gffread -E lncRNA.lulab_ncomms14421.hg38.gtf -o- > lncRNA.lulab_ncomms14421.hg38.gff

cd mitranscriptome.gtf
mv * ..
cd mitranscriptome
mv * ..
rm -rf mitranscriptome mitranscriptome.gtf mitranscriptome.gtf.tar.gz

rm -rf *.gp unmapped.gtf

mv NONCODEv5_human_hg38_lncRNA.gtf lncRNA.NONCODEv5.hg38.gtf
```

#### merge lncRNA annotations
```
gffcompare -o merged_lncRNA -s ../sequence/GRCh38.p10.genome.fa lncRNA.NONCODEv5.hg38.gtf  lncRNA.mitranscriptome.v2.hg38.gtf  lncRNA.gencode27.gtf lncRNA.lulab_ncomms14421.hg38.gtf
gffread -E merged_lncRNA.combined.gtf -o- > merged_lncRNA.combined.gff
```
#### mkdir gtf gff
```
cd ..
mkdir gtf gff
mv gencode/*.gtf gtf
mv gencode/*.gff gff
```

#### establish indexes
```
if [ "$1" = "0" ]; then
	for i in ${RNAs[@]} ; do
 		echo "start $i.gtf:"
		rsem-prepare-reference --gtf $gtf/$i.gtf --bowtie2 $hg38 RNA_index/$i
		echo "$i finished."
	done
```
#### cut lncRNA into bins
#### convert combined gtf to combined bed

```
gffread --bed merged_lncRNA.combined.gtf -o merged_lncRNA.combined.bed
```

#### grep the longest transciprt for each lncRNA gene and calculate its length
```
awk 'NR==FNR {split($10,m,"\"");split($12,n,"\"");D[m[2]]=n[2]} \
NR>FNR {split($11,ex,",");for (k in ex) len[$4]+=ex[k]; b[D[$4]] = len[$4] > a[D[$4]] ? $4 : b[D[$4]]; a[D[$4]] = len[$4] > a[D[$4]] ? len[$4] : a[D[$4]] } \
END { OFS = "\t"; for(x in a) print x, a[x],b[x]}' merged_lncRNA.combined.gtf merged_lncRNA.combined.bed >merged_lncRNA.combined.longest.tran
```

#### grep the exons of the longest transciprts
```
awk '{print $3}' merged_lncRNA.combined.longest.tran| grep -Ff - merged_lncRNA.combined.gtf|awk '$3=="exon"' >merged_lncRNA.combined.longest.exon.gtf
```

#### cut full length lncRNA transcripts into bins: bin_size_30, step_size_15
#### (1)deal with transcirpts longer than 30bp
```
awk 'BEGIN{ OFS="\t" } ($5-$4)>30 {split($10,a,"\""); \
for(i=0;i<=($5-$4-30)/15;i++) {print $1,$4+i*15, $4+i*15+30,a[2]"__"$4+i*15"__"$4+i*15+30,$6,$7} \
print $1, $4+(i+1)*15,$5,a[2]"__"$4+(i+1)*15"__"$5,$6,$7}' \
merged_lncRNA.combined.longest.exon.gtf |awk '$3>$2' >merged_lncRNA.combined.longest.exon.bin30.bed
```
#### (2)deal with transcirpts no longer than 30bp
```
awk 'BEGIN{ OFS="\t" } ($5-$4)<=30 {split($10,a,"\"");print $1,$4,$5,a[2]"__"$4"__"$5,$6,$7}' \
merged_lncRNA.combined.longest.exon.gtf | awk '$3>$2' > shorterthan30.bed
```

#### (3)concatenate two bed files generated abovely

```
cat merged_lncRNA.combined.longest.exon.bin30.bed shorterthan30.bed|sort -k1,1 -k2,2n -k3,3 >merged_lncRNA.combined.longest.exon.bin30.all.bed
awk 'BEGIN{FS=OFS="\t"}{print $1,$2,$3,$4,1,$6}' merged_lncRNA.combined.longest.exon.bin30.all.bed > foo 
bedToGenePred foo /dev/stdout | genePredToGtf -source=merged_lncRNA_bin file /dev/stdin merged_lncRNA.combined.longest.exon.bin30.all.gtf
```

```
for i in `ls ./gtf`;
do j=${i%.*};
echo $j
gffread --bed ./gtf/$j.gtf -o ./binned/$j.bed

awk 'NR==FNR {split($10,m,"\"");split($12,n,"\"");D[m[2]]=n[2]} \
NR>FNR {split($11,ex,",");for (k in ex) len[$4]+=ex[k]; b[D[$4]] = len[$4] > a[D[$4]] ? $4 : b[D[$4]]; a[D[$4]] = len[$4] > a[D[$4]] ? len[$4] : a[D[$4]] } \
END { OFS = "\t"; for(x in a) print x, a[x],b[x]}' ./gtf/$j.gtf ./binned/$j.bed > ./binned/$j.longest.tran

awk '{print $3}' ./binned/$j.longest.tran | grep -Ff - ./gtf/$j.gtf | awk '$3=="exon"' > ./binned/$j.longest.exon.gtf

awk 'BEGIN{ OFS="\t" } ($5-$4)>30 {split($10,a,"\""); \
for(i=0;i<=($5-$4-30)/15;i++) {print $1,$4+i*15, $4+i*15+30,a[2]"__"$4+i*15"__"$4+i*15+30,$6,$7} \
print $1, $4+(i+1)*15,$5,a[2]"__"$4+(i+1)*15"__"$5,$6,$7}' \
./binned/$j.longest.exon.gtf | awk '$3>$2' > ./binned/$j.longest.exon.bin30.bed

awk 'BEGIN{ OFS="\t" } ($5-$4)<=30 {split($10,a,"\"");print $1,$4,$5,a[2]"__"$4"__"$5,$6,$7}' \
./binned/$j.longest.exon.gtf | awk '$3>$2' > ./binned/$j.shorterthan30.bed

cat ./binned/$j.longest.exon.bin30.bed ./binned/$j.shorterthan30.bed | sort -k1,1 -k2,2n -k3,3 > ./binned/$j.longest.exon.bin30.all.bed
awk 'BEGIN{FS=OFS="\t"}{print $1,$2,$3,$4,1,$6}' ./binned/$j.longest.exon.bin30.all.bed > ./binned/$j.foo
bedToGenePred ./binned/$j.foo /dev/stdout | genePredToGtf -source=${j}_bin file /dev/stdin ./binned/$j.longest.exon.bin30.all.gtf

done
```






