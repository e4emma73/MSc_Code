# MSc: Phenotypic and Genotypic Differences of Bovine- and Human-isolated _Staphylococcus aureus_ in New Zealand
All bioinformatic analysis was conducted on the New Zealand eScience Infrastructure NeSI. FastQC and Kraken2 was used for the QC of the samples. 

# FastQC
FastQC was used to check the QC of the samples sequence quality
```
#SBATCH --cpus-per-task=8 --mem 1Gb --time 0-6:00 -c 1 -J FASTQC

module load FastQC/0.11.9

FASTQC = /pathtorawreads
OUTPUT_DIR = /processedreadsdirectory/FASTQC/

mkdir -p $OUTPUT_DIR

for file in $FASTQC/*.fq.gz;
do
        echo "module load FastQC; fastqc $fastq -t 2 -o $OUTPUT_DIR" >> $TMPFILE
done

-e /pathtoerrorfile
-o  $OUTPUT_DIR
```

# Kraken2 
The default Kraken2 installed with Nullarbor was used for species identification. To ensure the samples were _S. aureus_, samples should have more than 70% of reads mapping to the species _S. aureus_ and do not contain more than 5% of reads mapping to another species.
```
sbatch -J KrakenIllumina --mem 75G --time 00:05:00 -c 12 --account <nesi_account> --wrap 'set -x; module load Kraken2; kraken2 --confidence 0.5 --db /pathtokrakendatabase/ --report NameOFReport.report --threads 24 --paired /pathtoforwardread.fastq.gz /pathtoreverseread.fastq.gz '
```
# Nullarbor 
Nullarbor is the genomic analysis pipeline chosen for the analysis of bovine- and human-isolated _S. aureus_. In the nullabor.pl file --mincov was specified as 60 and --minid as 90. The script was run on NESI.
```
#SBATCH --cpus-per-task=30 --mem 160Gb --time 166:00:00 -J nullarbor
mkdir=p$TMPDIR
module --force purge
module load NeSI
module load nullarbor/2.0.20191013.LIC
export KRAKEN2_DEFAULT_DB=/pathtodatabasefile
REF=/pathtoreferencegenomefile
nullarbor.pl --name StaphAureusIsolates --ref$Ref_Bovine \
--input Isolatestxtfile.txt \
--outdir outputfile --force \
--cpus $SLURM_CPUS_PER_TASK --run --mlst saureus --trim --taxoner-opt confidence 0.5 --treebuilder iqtree --treebuilder-opt "wbtl -st DNA -b 1000 --alrt 1000" --annotator-opt --"fullanno --compliant"
```
# Prokka
Prokka was used to annotate the bovine- and human-isolated _S. aureus_ genomes.
```
module load prokka/1.14.5-GCC-11.3.0
/opt/nesi/CS400_centos7_bdw/prokka/1.14.5-GCC-11.3.0/bin/prokka --kingdom Bacteria --genus Staphylococcus contigfile.fasta --outdir /pathtooutputdirectory/
```
# ABRicate
Used for the mass screening of contigs for antimicrobial resistance or virulence genes. It uses the databases: NCBI, CARD, ARG-ANNOT, Resfinder, MEGARES, EcOH, PlasmidFinder, Ecoli_VF and VFDB. 
```
module load ABRicate/1.0.0-GCC-11.3.0-Perl-5.34.1
abricate contigfastafile.fasta
```
