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


# 0. Building of directory structure

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
  RawDatDir=/data/scratch/nanopore_tmp_data/20171203_FON139/20171203_FON139/fast5/pass
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


    cat $OutDir/albacore_v2.2.7/workspace/pass/*.fastq | gzip -cf > ${Strain}_${Date}_albacore_v2.2.7.fastq.gz

    tar -cz -f ${Strain}_${Date}_albacore_v2.2.7.tar.gz $OutDir

    FinalDir=/data/scratch/nanopore_tmp_data/FON/albacore_v2.2.7
    mkdir -p $FinalDir
    mv *_albacore_v2.2.7.* $FinalDir/.
    chmod +rw -R $FinalDir
