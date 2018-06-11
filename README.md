# Fusarium_ex_narcissus assemblies
Commands used in the analysis of Fusarium oxysporum isolates ex. narcissus
==========

This document details the commands used to assemble and annotate the genomes.

Note - all this work was performed in the directory:
/home/groups/harrisonlab/project_files/fusarium_ex_narcissus

The following is a summary of the work presented in this Readme.

The following processes were applied to Fusarium genomes prior to analysis:
Data qc
Genome assembly
Repeatmasking
Gene prediction
Functional annotation


# 0. Recalling data and building of directory structure

### Minion Data

```bash
  screen -a
  ssh nanopore@nanopore
  # Untar datastores
  # 19-09-17
  mkdir -p /data/scratch/nanopore_tmp_data/20170920_1225_FON77-FON139
  tar -zxv -C /data/scratch/nanopore_tmp_data/20170920_1225_FON77-FON139 -f /data/seq_data/minion/2017/20170920_1225_FON77-FON139.tar.gz
  # 30-11-17
  mkdir -p /data/scratch/nanopore_tmp_data/20171203_FON139
  tar -zxv -C /data/scratch/nanopore_tmp_data/20171203_FON139 -f /data/seq_data/minion/2017/20171203_FON139.tar.gz
  # 12-12-17
  mkdir -p /data/scratch/nanopore_tmp_data/20171203_FON77
  tar -zxv -C /data/scratch/nanopore_tmp_data/20171203_FON77 -f /data/seq_data/minion/2017/20171221_FON77.tar.gz
  # 12-12-17
  mkdir -p /data/scratch/nanopore_tmp_data/20171203_FON89
  tar -zxv -C /data/scratch/nanopore_tmp_data/20171203_FON89 -f /data/seq_data/minion/2017/20171221_FON89.tar.gz
  # 26-01-18
  ls -d /data/seq_data/minion/2018/20180127_FON81/FON81/FON81
  # 22-02-18
  ls -d /data/seq_data/minion/2018/20180222_FON129  


  ProjectDir=/home/groups/harrisonlab/project_files/fusarium_ex_narcissus
  # FON139
  # 19-09-17
  # 30-11-17
	mkdir -p $ProjectDir/raw_dna/minion/F.oxysporum_fsp_narcissi/FON139
  # FON77
  # 19-09-17
  # 12-12-17
	mkdir -p $ProjectDir/raw_dna/minion/F.oxysporum_fsp_narcissi/FON77
  # FON89
  # 12-12-17
	mkdir -p $ProjectDir/raw_dna/minion/F.oxysporum_fsp_narcissi/FON89
  # FON81
  # 26-01-18
	mkdir -p $ProjectDir/raw_dna/minion/F.oxysporum_fsp_narcissi/FON81
  # FON129
  # 22-02-18
  mkdir -p $ProjectDir/raw_dna/minion/F.oxysporum_fsp_narcissi/FON129
```

### Basecalling

Data was basecalled again using Albacore 2.02 on the minion server:

```bash
screen -a
ssh nanopore@nanopore

# upgrade albacore
wget https://mirror.oxfordnanoportal.com/software/analysis/ont_albacore-2.2.7-cp34-cp34m-manylinux1_x86_64.whl
~/.local/bin/read_fast5_basecaller.py --version
pip3 install --user ont_albacore-2.2.7-cp34-cp34m-manylinux1_x86_64.whl --upgrade
~/.local/bin/read_fast5_basecaller.py --version

mkdir FoNarcissi_2018-05-04
cd FoNarcissi_2018-05-04

# Oxford nanopore FON139 FON77 Run 1
Organism=F.oxysporum_fsp_narcissi
Strain=FON139_FON77
Date=2017-09-20
FlowCell="FLO-MIN106"
Kit="SQK-LSK108"
RawDatDir=/data/scratch/nanopore_tmp_data/20170920_1225_FON77-FON139/20170920_1225_FON77-FON139/fast5/pass
OutDir=~/FoNarcissi_2018-05-04/$Organism/$Strain/$Date

mkdir -p $OutDir
cd $OutDir
~/.local/bin/read_fast5_basecaller.py \
  --flowcell $FlowCell \
  --kit $Kit \
  --input $RawDatDir \
  --recursive \
  --worker_threads 23 \
  --save_path albacore_v2.2.7 \
  --output_format fastq,fast5 \
  --reads_per_fastq_batch 4000 \
  --barcoding

Strain='FON77'
cat $OutDir/albacore_v2.2.7/workspace/pass/barcode01/*.fastq | gzip -cf > ${Strain}_${Date}_albacore_v2.2.7.fastq.gz
Strain='FON139'
cat $OutDir/albacore_v2.2.7/workspace/pass/barcode03/*.fastq | gzip -cf > ${Strain}_${Date}_albacore_v2.2.7.fastq.gz

Strain='FON139_FON77'
tar -cz -f ${Strain}_${Date}_albacore_v2.2.7.tar.gz $OutDir

FinalDir=/data/scratch/nanopore_tmp_data/FON/albacore_v2.2.7
mkdir -p $FinalDir
mv *_albacore_v2.2.7.* $FinalDir/.
chmod +rw -R $FinalDir


# Oxford nanopore FON77 Run 2
Organism=F.oxysporum_fsp_narcissi
Strain=FON77
Date=2017-12-03
FlowCell="FLO-MIN106"
Kit="SQK-LSK108"
RawDatDir=/data/scratch/nanopore_tmp_data/20171203_FON139/20171203_FON139/FON139/GA40000
OutDir=~/FoNarcissi_2018-05-04/$Organism/$Strain/$Date

mkdir -p $OutDir
cd $OutDir
~/.local/bin/read_fast5_basecaller.py \
  --flowcell $FlowCell \
  --kit $Kit \
  --input $RawDatDir \
  --recursive \
  --worker_threads 23 \
  --save_path albacore_v2.2.7 \
  --output_format fastq,fast5 \
  --reads_per_fastq_batch 4000

  cd ~/FoNarcissi_2018-05-04
  cat $OutDir/albacore_v2.2.7/workspace/pass/*.fastq | gzip -cf > ${Strain}_${Date}_albacore_v2.2.7.fastq.gz

  tar -cz -f ${Strain}_${Date}_albacore_v2.2.7.tar.gz $OutDir

  FinalDir=/data/scratch/nanopore_tmp_data/FON/albacore_v2.2.7
  mkdir -p $FinalDir
  mv *_albacore_v2.2.7.* $FinalDir/.
  chmod +rw -R $FinalDir

  # Oxford nanopore FON139 Run 2
  Organism=F.oxysporum_fsp_narcissi
  Strain=FON139
  Date=2017-12-03
  FlowCell="FLO-MIN106"
  Kit="SQK-LSK108"
  RawDatDir=/data/scratch/nanopore_tmp_data/20171203_FON139/20171203_FON139/FON139/GA40000/reads/
  OutDir=~/FoNarcissi_2018-05-04/$Organism/$Strain/$Date

  mkdir -p $OutDir
  cd $OutDir
  ~/.local/bin/read_fast5_basecaller.py \
    --flowcell $FlowCell \
    --kit $Kit \
    --input $RawDatDir \
    --recursive \
    --worker_threads 23 \
    --save_path albacore_v2.2.7 \
    --output_format fastq,fast5 \
    --reads_per_fastq_batch 4000

    cd ~/FoNarcissi_2018-05-04
    cat $OutDir/albacore_v2.2.7/workspace/pass/*.fastq | gzip -cf > ${Strain}_${Date}_albacore_v2.2.7.fastq.gz
    tar -cz -f ${Strain}_${Date}_albacore_v2.2.7.tar.gz $OutDir

    FinalDir=/data/scratch/nanopore_tmp_data/FON/albacore_v2.2.7
    mkdir -p $FinalDir
    mv *_albacore_v2.2.7.* $FinalDir/.
    chmod +rw -R $FinalDir

    # Oxford nanopore FON89
    Organism=F.oxysporum_fsp_narcissi
    Strain=FON89
    Date=2017-12-21
    FlowCell="FLO-MIN106"
    Kit="SQK-LSK108"
    RawDatDir=/data/scratch/nanopore_tmp_data/20171203_FON77/20171221_FON77/FON77/GA40000/reads/
    OutDir=~/FoNarcissi_2018-05-04/$Organism/$Strain/$Date

    mkdir -p $OutDir
    cd $OutDir
    ~/.local/bin/read_fast5_basecaller.py \
      --flowcell $FlowCell \
      --kit $Kit \
      --input $RawDatDir \
      --recursive \
      --worker_threads 23 \
      --save_path albacore_v2.2.7 \
      --output_format fastq,fast5 \
      --reads_per_fastq_batch 4000

      cd ~/FoNarcissi_2018-05-04
      cat $OutDir/albacore_v2.2.7/workspace/pass/*.fastq | gzip -cf > ${Strain}_${Date}_albacore_v2.2.7.fastq.gz

      tar -cz -f ${Strain}_${Date}_albacore_v2.2.7.tar.gz $OutDir

      FinalDir=/data/scratch/nanopore_tmp_data/FON/albacore_v2.2.7
      mkdir -p $FinalDir
      mv *_albacore_v2.2.7.* $FinalDir/.
      chmod +rw -R $FinalDir


      # Oxford nanopore FON81
      Organism=F.oxysporum_fsp_narcissi
      Strain=FON81
      Date=2017-12-21
      FlowCell="FLO-MIN106"
      Kit="SQK-LSK108"
      RawDatDir=/data/seq_data/minion/2018/20180127_FON81/FON81/FON81
      # RawDatDir=/data/seq_data/minion/2018/20180127_FON81/FON81/FON81/GA10000/reads
      # RawDatDir=/data/seq_data/minion/2018/20180127_FON81/FON81/FON81/GA20000/reads
      OutDir=~/FoNarcissi_2018-05-04/$Organism/$Strain/$Date

      mkdir -p $OutDir
      cd $OutDir
      ~/.local/bin/read_fast5_basecaller.py \
        --flowcell $FlowCell \
        --kit $Kit \
        --input $RawDatDir \
        --recursive \
        --worker_threads 23 \
        --save_path albacore_v2.2.7 \
        --output_format fastq,fast5 \
        --reads_per_fastq_batch 4000

        cd ~/FoNarcissi_2018-05-04
        cat $OutDir/albacore_v2.2.7/workspace/pass/*.fastq | gzip -cf > ${Strain}_${Date}_albacore_v2.2.7.fastq.gz

        tar -cz -f ${Strain}_${Date}_albacore_v2.2.7.tar.gz $OutDir

        FinalDir=/data/scratch/nanopore_tmp_data/FON/albacore_v2.2.7
        mkdir -p $FinalDir
        mv *_albacore_v2.2.7.* $FinalDir/.
        chmod +rw -R $FinalDir

        # Oxford nanopore FON81
        Organism=F.oxysporum_fsp_narcissi
        Strain=FON129
        Date=2017-12-21
        FlowCell="FLO-MIN106"
        Kit="SQK-LSK108"
        RawDatDir=/data/seq_data/minion/2018/20180222_FON129/FON129/
        OutDir=~/FoNarcissi_2018-05-04/$Organism/$Strain/$Date

        mkdir -p $OutDir
        cd $OutDir
        ~/.local/bin/read_fast5_basecaller.py \
          --flowcell $FlowCell \
          --kit $Kit \
          --input $RawDatDir \
          --recursive \
          --worker_threads 23 \
          --save_path albacore_v2.2.7 \
          --output_format fastq,fast5 \
          --reads_per_fastq_batch 4000

          cd ~/FoNarcissi_2018-05-04
          cat $OutDir/albacore_v2.2.7/workspace/pass/*.fastq | gzip -cf > ${Strain}_${Date}_albacore_v2.2.7.fastq.gz

          tar -cz -f ${Strain}_${Date}_albacore_v2.2.7.tar.gz $OutDir

          FinalDir=/data/scratch/nanopore_tmp_data/FON/albacore_v2.2.7
          mkdir -p $FinalDir
          mv *_albacore_v2.2.7.* $FinalDir/.
          chmod +rw -R $FinalDir
```

### Copying recalled reads to the project directory directory

```bash
DataDir=$(ls -d /data/scratch/nanopore_tmp_data/FON/albacore_v2.2.7)
ProjectDir=/home/groups/harrisonlab/project_files/fusarium_ex_narcissus
# FON139
# 19-09-17
# 30-11-17
OutDir=$ProjectDir/raw_dna/minion/F.oxysporum_fsp_narcissi/FON139
mkdir -p $OutDir
cp -s $DataDir/FON139_2017-09-20_albacore_v2.2.7.fastq.gz $OutDir/.
cp -s $DataDir/FON139_2017-12-03_albacore_v2.2.7.fastq.gz $OutDir/.
# FON77
# 19-09-17
# 12-12-17
OutDir=$ProjectDir/raw_dna/minion/F.oxysporum_fsp_narcissi/FON77
mkdir -p $OutDir
cp -s $DataDir/FON77_2017-09-20_albacore_v2.2.7.fastq.gz $OutDir/.
cp -s $DataDir/FON77_2017-12-03_albacore_v2.2.7.fastq.gz $OutDir/.
# FON89
# 12-12-17
OutDir=$ProjectDir/raw_dna/minion/F.oxysporum_fsp_narcissi/FON89
mkdir -p $OutDir
cp -s $DataDir/FON89_2017-12-21_albacore_v2.2.7.fastq.gz $OutDir/.
# FON81
# 26-01-18
OutDir=$ProjectDir/raw_dna/minion/F.oxysporum_fsp_narcissi/FON81
mkdir -p $OutDir
cp -s $DataDir/FON81_2017-12-21_albacore_v2.2.7.fastq.gz $OutDir/.
# FON129
# 22-02-18
OutDir=$ProjectDir/raw_dna/minion/F.oxysporum_fsp_narcissi/FON129
mkdir -p $OutDir
cp -s $DataDir/FON129_2017-12-21_albacore_v2.2.7.fastq.gz $OutDir/.
```



### Copying MiSeq reads to the project Directory

Trimmed Miseq reads were available in the fusarium project directory at:
/home/groups/harrisonlab/project_files/fusarium

<!--
```bash
  ProjectDir=/home/groups/harrisonlab/project_files/fusarium_ex_narcissus
  # HAPI_seq_3 FON139
  RawDatDir=/home/groups/harrisonlab/raw_data/raw_seq/fusarium/HAPI_seq_3
  OutDir=$ProjectDir/raw_dna/paired/F.oxysporum_fsp_narcissi/FON139
  mkdir -p $OutDir/F
  mkdir -p $OutDir/R
	cp $RawDatDir/FoxysporumN139_S2_L001_R1_001.fastq.gz $ProjectDir/$OutDir/F/.
	cp $RawDatDir/FoxysporumN139_S2_L001_R2_001.fastq.gz $ProjectDir/$OutDir/R/.
  # 171006 FON129
	# RawDatDir=/data/seq_data/miseq/2017/RAW/171006_M04465_0050_000000000-B49T7/Data/Intensities/BaseCalls
  RawDatDir=/home/groups/harrisonlab/project_files/fusarium/raw_dna/paired/F.oxysporum_fsp_narcissi/FON129
	OutDir=$ProjectDir/raw_dna/paired/F.oxysporum_fsp_narcissi/FON129
	mkdir -p $OutDir/F
	mkdir -p $OutDir/R
	cp -s $RawDatDir/F/FON129_S2_L001_R1_001.fastq.gz $ProjectDir/$OutDir/F/.
	cp -s $RawDatDir/R/FON129_S2_L001_R2_001.fastq.gz $ProjectDir/$OutDir/R/.
  # 170926 FON77
  RawDatDir=/data/seq_data/miseq/2017/RAW/170926_M04465_0049_000000000-B49VF/Data/Intensities/BaseCalls
	OutDir=$ProjectDir/raw_dna/paired/F.oxysporum_fsp_narcissi/FON77
	mkdir -p $OutDir/F
	mkdir -p $OutDir/R
  cd $OutDir/F
	cp -s $RawDatDir/FON77_S1_L001_R1_001.fastq.gz .
  cd $OutDir/R
	cp -s $RawDatDir/FON77_S1_L001_R2_001.fastq.gz .
  # FON81
  OutDir=$ProjectDir/raw_dna/paired/F.oxysporum_fsp_narcissi/FON81
  mkdir -p $OutDir/F
  mkdir -p $OutDir/R
  cd $OutDir/F
  cp -s $RawDatDir/FON81_S2_L001_R1_001.fastq.gz .
  cd $OutDir/R
  cp -s $RawDatDir/FON81_S2_L001_R2_001.fastq.gz .
  # FON89
  OutDir=$ProjectDir/raw_dna/paired/F.oxysporum_fsp_narcissi/FON89
  mkdir -p $OutDir/F
  mkdir -p $OutDir/R
  cd $OutDir/F
  cp -s $RawDatDir/FON89_S3_L001_R1_001.fastq.gz .
  cd $OutDir/R
  cp -s $RawDatDir/FON89_S3_L001_R2_001.fastq.gz .
``` -->



## Assembly

### Removal of adapters

Splitting reads and trimming adapters using porechop
```bash
  ProjectDir=/home/groups/harrisonlab/project_files/fusarium_ex_narcissus
  cd $ProjectDir
  for RawReads in $(ls raw_dna/minion/*/*/*.fastq.gz); do
    Organism=$(echo $RawReads| rev | cut -f3 -d '/' | rev)
    Strain=$(echo $RawReads | rev | cut -f2 -d '/' | rev)
    echo "$Organism - $Strain"
    OutDir=qc_dna/minion/$Organism/$Strain
    ProgDir=/home/armita/git_repos/emr_repos/tools/seq_tools/dna_qc
    qsub $ProgDir/sub_porechop.sh $RawReads $OutDir
  done
```


## Identify sequencing coverage

For Minion data:
```bash
for RawData in $(ls qc_dna/minion/*/*/*q.gz); do
echo $RawData;
ProgDir=/home/armita/git_repos/emr_repos/tools/seq_tools/dna_qc;
GenomeSz=58
OutDir=$(dirname $RawData)
# OutDir=$(echo $RawData | cut -f11,12,13,14 -d '/')
mkdir -p $OutDir
qsub $ProgDir/sub_count_nuc.sh $GenomeSz $RawData $OutDir
done
```

```bash
  for StrainDir in $(ls -d qc_dna/minion/*/*); do
    Strain=$(basename $StrainDir)
    printf "$Strain\t"
    for File in $(ls $StrainDir/*cov.txt); do
      echo $(basename $File);
      cat $File | tail -n1 | rev | cut -f2 -d ' ' | rev;
    done | grep -v '.txt' | awk '{ SUM += $1} END { print SUM }'
  done
```

MinION coverage

```
FON129	103.57
FON139	79.61
FON77	79.62
FON81	76.12
FON89	72.76
```


### Read correction using Canu

```bash
for TrimReads in $(ls qc_dna/minion/*/*/*q.gz); do
Organism=$(echo $TrimReads | rev | cut -f3 -d '/' | rev)
Strain=$(echo $TrimReads | rev | cut -f2 -d '/' | rev)
OutDir=assembly/canu-1.6/$Organism/"$Strain"
ProgDir=/home/armita/git_repos/emr_repos/tools/seq_tools/assemblers/canu
qsub $ProgDir/sub_canu_correction.sh $TrimReads 58m $Strain $OutDir
done
```


### Assembbly using SMARTdenovo

```bash
for CorrectedReads in $(ls assembly/canu-1.6/*/*/*.trimmedReads.fasta.gz); do
Organism=$(echo $CorrectedReads | rev | cut -f3 -d '/' | rev)
Strain=$(echo $CorrectedReads | rev | cut -f2 -d '/' | rev)
Prefix="$Strain"_smartdenovo
OutDir=assembly/SMARTdenovo/$Organism/"$Strain"
ProgDir=/home/armita/git_repos/emr_repos/tools/seq_tools/assemblers/SMARTdenovo
qsub $ProgDir/sub_SMARTdenovo.sh $CorrectedReads $Prefix $OutDir
done
```


Quast and busco were run to assess the effects of racon on assembly quality:

```bash
for Assembly in $(ls assembly/SMARTdenovo/*/*/*.dmo.lay.utg ); do
  Strain=$(echo $Assembly | rev | cut -f2 -d '/' | rev)
  Organism=$(echo $Assembly | rev | cut -f3 -d '/' | rev)  
	echo "$Organism - $Strain"
  OutDir=$(dirname $Assembly)
	ProgDir=/home/armita/git_repos/emr_repos/tools/seq_tools/assemblers/assembly_qc/quast
  qsub $ProgDir/sub_quast.sh $Assembly $OutDir
	OutDir=gene_pred/busco/$Organism/$Strain/assembly
	BuscoDB=$(ls -d /home/groups/harrisonlab/dbBusco/sordariomyceta_odb9)
	ProgDir=/home/armita/git_repos/emr_repos/tools/gene_prediction/busco
	qsub $ProgDir/sub_busco3.sh $Assembly $BuscoDB $OutDir
done
```


Error correction using racon:

```bash
for Assembly in $(ls assembly/SMARTdenovo/*/*/*.dmo.lay.utg | grep -e 'FON129' -e 'FON81'); do
Strain=$(echo $Assembly | rev | cut -f2 -d '/' | rev)
Organism=$(echo $Assembly | rev | cut -f3 -d '/' | rev)
echo "$Organism - $Strain"
ReadsFq=$(ls qc_dna/minion/*/$Strain/*q.gz)
Iterations=10
OutDir=$(dirname $Assembly)"/racon_$Iterations"
ProgDir=/home/armita/git_repos/emr_repos/tools/seq_tools/assemblers/racon
qsub $ProgDir/sub_racon.sh $Assembly $ReadsFq $Iterations $OutDir
done
```

```bash
for Assembly in $(ls assembly/SMARTdenovo/*/*/*.dmo.lay.utg | grep -v -e 'FON129' -e 'FON81'); do
Strain=$(echo $Assembly | rev | cut -f2 -d '/' | rev)
Organism=$(echo $Assembly | rev | cut -f3 -d '/' | rev)
echo "$Organism - $Strain"
ReadsFq=$(ls qc_dna/minion/*/$Strain/*q.gz)
Iterations=10
OutDir=$(dirname $Assembly)"/racon_$Iterations"
ProgDir=/home/armita/git_repos/emr_repos/tools/seq_tools/assemblers/racon
qsub $ProgDir/sub_racon_2libs.sh $Assembly $ReadsFq $Iterations $OutDir
done
```


```bash
ProgDir=/home/armita/git_repos/emr_repos/tools/seq_tools/assemblers/assembly_qc/quast
for Assembly in $(ls assembly/SMARTdenovo/*/*/racon_10/*.fasta | grep 'round_10'); do
OutDir=$(dirname $Assembly)
echo "" > tmp.txt
ProgDir=~/git_repos/emr_repos/tools/seq_tools/assemblers/assembly_qc/remove_contaminants
$ProgDir/remove_contaminants.py --keep_mitochondria --inp $Assembly --out $OutDir/racon_min_500bp_renamed.fasta --coord_file tmp.txt > $OutDir/log.txt
done
```

Quast and busco were run to assess the effects of racon on assembly quality:

```bash
ProgDir=/home/armita/git_repos/emr_repos/tools/seq_tools/assemblers/assembly_qc/quast
for Assembly in $(ls assembly/SMARTdenovo/*/*/racon_10/racon_min_500bp_renamed.fasta); do
Strain=$(echo $Assembly | rev | cut -f3 -d '/' | rev)
Organism=$(echo $Assembly | rev | cut -f4 -d '/' | rev)  
OutDir=$(dirname $Assembly)
qsub $ProgDir/sub_quast.sh $Assembly $OutDir
done
```


```bash
for Assembly in $(ls assembly/SMARTdenovo/*/*/racon_10/*.fasta); do
Strain=$(echo $Assembly | rev | cut -f3 -d '/' | rev)
Organism=$(echo $Assembly | rev | cut -f4 -d '/' | rev)
Jobs=$(qstat | grep 'sub_busco' | grep 'qw' | wc -l)
while [ $Jobs -gt 1 ]; do
sleep 1m
printf "."
Jobs=$(qstat | grep 'sub_busco' | grep 'qw' | wc -l)
done		
printf "\n"
echo "$Organism - $Strain"
ProgDir=/home/armita/git_repos/emr_repos/tools/gene_prediction/busco
BuscoDB=$(ls -d /home/groups/harrisonlab/dbBusco/ascomycota_odb9)
# OutDir=gene_pred/busco/$Organism/$Strain/assembly
OutDir=$(dirname $Assembly)
qsub $ProgDir/sub_busco3.sh $Assembly $BuscoDB $OutDir
done
```
```bash
printf "Filename\tComplete\tDuplicated\tFragmented\tMissing\tTotal\n"
for File in $(ls gene_pred/busco/A*/*/assembly/*/short_summary_*.txt); do
FileName=$(basename $File)
Complete=$(cat $File | grep "(C)" | cut -f2)
Duplicated=$(cat $File | grep "(D)" | cut -f2)
Fragmented=$(cat $File | grep "(F)" | cut -f2)
Missing=$(cat $File | grep "(M)" | cut -f2)
Total=$(cat $File | grep "Total" | cut -f2)
printf "$FileName\t$Complete\t$Duplicated\t$Fragmented\t$Missing\t$Total\n"
done
```

## Nanopolish

```bash
for Assembly in $(ls assembly/SMARTdenovo/*/*/racon_10/racon_min_500bp_renamed.fasta); do
Strain=$(echo $Assembly | rev | cut -f3 -d '/' | rev)
Organism=$(echo $Assembly | rev | cut -f4 -d '/' | rev)
echo "$Organism - $Strain"
ReadsFq=$(ls qc_dna/minion/*/$Strain/*q.gz)
OutDir=qc_dna/minion/$Organism/$Strain/appended
mkdir $OutDir
cat $ReadsFq > $OutDir/${Strain}_appended.fq.gz
done

```

```bash
for Assembly in $(ls assembly/SMARTdenovo/*/*/racon_10/racon_min_500bp_renamed.fasta | grep -e 'FON129' -e 'FON81'); do
Strain=$(echo $Assembly | rev | cut -f3 -d '/' | rev)
Organism=$(echo $Assembly | rev | cut -f4 -d '/' | rev)
echo "$Organism - $Strain"
# Step 1 extract reads as a .fq file which contain info on the location of the fast5 files
# Note - the full path from home must be used
ReadDir=raw_dna/nanopolish/$Organism/$Strain
mkdir -p $ReadDir
ReadsFq=$(ls raw_dna/minion/*/$Strain/*.fastq.gz)
raw_dna/minion/*/$Strain/*.fastq.gz)
ScratchDir=/data/scratch/armita/alternaria
Fast5Dir=$ScratchDir/albacore_v2.1.10/Alt_albacore_v2.10_demultiplexed/workspace/pass/barcode01
nanopolish index -d $Fast5Dir $ReadsFq
OutDir=$(dirname $Assembly)/nanopolish
mkdir -p $OutDir
ProgDir=/home/armita/git_repos/emr_repos/tools/seq_tools/assemblers/nanopolish
# submit alignments for nanoppolish
qsub $ProgDir/sub_minimap2_nanopolish.sh $Assembly $ReadsFq $OutDir/nanopolish
done
```

```bash
for Assembly in $(ls assembly/SMARTdenovo/*/650/racon2_10/racon_min_500bp_renamed.fasta); do
Strain=$(echo $Assembly | rev | cut -f3 -d '/' | rev)
Organism=$(echo $Assembly | rev | cut -f4 -d '/' | rev)
echo "$Organism - $Strain"
# Step 1 extract reads as a .fq file which contain info on the location of the fast5 files
# Note - the full path from home must be used
ReadDir=raw_dna/nanopolish/$Organism/$Strain
mkdir -p $ReadDir
ReadsFq=$(ls ../../../../../home/groups/harrisonlab/project_files/alternaria/raw_dna/minion/*/$Strain/*.fastq.gz)
ScratchDir=/data/scratch/armita/alternaria
Fast5Dir=$ScratchDir/albacore_v2.1.10/Alt_albacore_v2.10_demultiplexed/workspace/pass/barcode02
# nanopolish index -d $Fast5Dir $ReadsFq
OutDir=$(dirname $Assembly)
mkdir -p $OutDir
ProgDir=/home/armita/git_repos/emr_repos/tools/seq_tools/assemblers/nanopolish
# submit alignments for nanoppolish
qsub $ProgDir/sub_minimap2_nanopolish.sh $Assembly $ReadsFq $OutDir/nanopolish
done
```

 Split the assembly into 50Kb fragments an submit each to the cluster for
 nanopolish correction

```bash
for Assembly in $(ls assembly/SMARTdenovo/*/*/racon2_10/racon_min_500bp_renamed.fasta); do
Strain=$(echo $Assembly | rev | cut -f3 -d '/' | rev)
Organism=$(echo $Assembly | rev | cut -f4 -d '/' | rev)
echo "$Organism - $Strain"
OutDir=$(dirname $Assembly)/nanopolish
RawReads=$(ls ../../../../../home/groups/harrisonlab/project_files/alternaria/raw_dna/minion/*/$Strain/*.fastq.gz)
AlignedReads=$(ls $OutDir/reads.sorted.bam)

NanoPolishDir=/home/armita/prog/nanopolish/nanopolish/scripts
python $NanoPolishDir/nanopolish_makerange.py $Assembly --segment-length 50000 > $OutDir/nanopolish_range.txt

Ploidy=1
echo "nanopolish log:" > $OutDir/nanopolish_log.txt
for Region in $(cat $OutDir/nanopolish_range.txt); do
Jobs=$(qstat | grep 'sub_nanopo' | grep 'qw' | wc -l)
while [ $Jobs -gt 1 ]; do
sleep 1m
printf "."
Jobs=$(qstat | grep 'sub_nanopo' | grep 'qw' | wc -l)
done		
printf "\n"
echo $Region
echo $Region >> $OutDir/nanopolish_log.txt
ProgDir=/home/armita/git_repos/emr_repos/tools/seq_tools/assemblers/nanopolish
qsub $ProgDir/sub_nanopolish_variants.sh $Assembly $RawReads $AlignedReads $Ploidy $Region $OutDir/$Region
done
done
```

```bash
for Assembly in $(ls assembly/SMARTdenovo/*/*/racon2_10/racon_min_500bp_renamed.fasta | grep '1166'); do
Strain=$(echo $Assembly | rev | cut -f3 -d '/' | rev)
Organism=$(echo $Assembly | rev | cut -f4 -d '/' | rev)
OutDir=assembly/SMARTdenovo/$Organism/$Strain/nanopolish
mkdir -p $OutDir
# cat "" > $OutDir/"$Strain"_nanoplish.fa
NanoPolishDir=/home/armita/prog/nanopolish/nanopolish/scripts
InDir=$(dirname $Assembly)
python $NanoPolishDir/nanopolish_merge.py $InDir/nanopolish/*/*.fa > $OutDir/"$Strain"_nanoplish.fa

echo "" > tmp.txt
ProgDir=~/git_repos/emr_repos/tools/seq_tools/assemblers/assembly_qc/remove_contaminants
$ProgDir/remove_contaminants.py --keep_mitochondria --inp $OutDir/"$Strain"_nanoplish.fa --out $OutDir/"$Strain"_nanoplish_min_500bp_renamed.fasta --coord_file tmp.txt > $OutDir/log.txt
done
```

Quast and busco were run to assess the effects of nanopolish on assembly quality:

```bash

for Assembly in $(ls assembly/SMARTdenovo/*/*/nanopolish/*_nanoplish_min_500bp_renamed.fasta); do
Strain=$(echo $Assembly | rev | cut -f3 -d '/' | rev)
Organism=$(echo $Assembly | rev | cut -f4 -d '/' | rev)  
# Quast
OutDir=$(dirname $Assembly)
ProgDir=/home/armita/git_repos/emr_repos/tools/seq_tools/assemblers/assembly_qc/quast
qsub $ProgDir/sub_quast.sh $Assembly $OutDir
# Busco
BuscoDB=$(ls -d /home/groups/harrisonlab/dbBusco/ascomycota_odb9)
OutDir=gene_pred/busco/$Organism/$Strain/assembly
ProgDir=/home/armita/git_repos/emr_repos/tools/gene_prediction/busco
qsub $ProgDir/sub_busco3.sh $Assembly $BuscoDB $OutDir
done
```
