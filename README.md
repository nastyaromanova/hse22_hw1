# hse22_hw1

## Создание символических ссылок на каждый из файлов
``` 
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq 
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq 
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```

## Выбор случайных чтений через ```seqtk``` и random seed = 603
```
seqtk sample -s 603 oil_R1.fastq 5000000 > paired_end1.fastq
seqtk sample -s 603 oil_R2.fastq 5000000 > paired_end2.fastq
seqtk sample -s 603 oilMP_S4_L001_R1_001.fastq 1500000 > mate_pairs1.fastq
seqtk sample -s 603 oilMP_S4_L001_R2_001.fastq 1500000 > mate_pairs2.fastq
```

## Оценка качества исходных чтений и получение по ним общей статистики с помощью программ ```fastqc``` и ```multiqc```
```
mkdir fastqc
mkdir multiqc

fastqc -o fastqc paired_end1.fastq
fastqc -o fastqc paired_end2.fastq
fastqc -o fastqc mate_pairs1.fastq
fastqc -o fastqc mate_pairs2.fastq

multiqc -o multiqc fastqc
```

![multiqc_untrimmed1](Pictures/multiqc_untrimmed1.png)
![multiqc_untrimmed2](Pictures/multiqc_untrimmed2.png)

## Подрезаем чтения по качеству с помощью platanus
```
platanus_trim paired_end*
platanus_internal_trim mate_pairs*
```

## Удаляем исходные чтения – файлы типа ```.fastq```, так как они больше не нужны
```
rm paired_end1.fastq paired_end2.fastq
rm mate_pairs1.fastq mate_pairs2.fastq
```

## Оценка качества подрезанных чтений и получение по ним общей статистики с помощью программ ```fastqc``` и ```multiqc```
```
mkdir fastqc_trimmed
mkdir multiqc_trimmed

fastqc -o fastqc_trimmed paired_end1.fastq.trimmed
fastqc -o fastqc_trimmed paired_end2.fastq.trimmed
fastqc -o fastqc_trimmed mate_pairs1.fastq.int_trimmed
fastqc -o fastqc_trimmed mate_pairs2.fastq.int_trimmed

multiqc -o multiqc_trimmed fastqc_trimmed
```

![multiqc_trimmed1](Pictures/multiqc_trimmed1.png)
![multiqc_trimmed2](Pictures/multiqc_trimmed2.png)

## Собираем контиги из подрезанных чтений с помощью программы ```platanus assemble```
```
time platanus assemble -o Poil -f paired_end1.fastq.trimmed paired_end2.fastq.trimmed 2> assemble.log
```

## Собираем скаффолды из контигов и подрезанных чтений с помощью программы ```platanus scaffold```
```
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 paired_end1.fastq.trimmed paired_end2.fastq.trimmed -OP2 mate_pairs1.fastq.int_trimmed mate_pairs2.fastq.int_trimmed 2> scaffold.log
```

## Уменьшаем число гэпов с помощью подрезанных чтений с помощью программы ```platanus gap_close```
```
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 paired_end1.fastq.trimmed paired_end2.fastq.trimmed -OP2 mate_pairs1.fastq.int_trimmed mate_pairs2.fastq.int_trimmed 2> gapclose.log
```

## Удаляем обрезанные чтения – файлы типа ```.fastq.trimmed``` и ```.fastq.int_trimmed```, так как они больше не нужны
```
rm paired_end1.fastq.trimmed paired_end2.fastq.trimmed
rm mate_pairs1.fastq.int_trimmed mate_pairs2.fastq.int_trimmed
```

## Переносим на локальный компьютер необходимые файлы, чтобы выложить на github, с помощью ```scp```
```
scp -P 5222 aaromanova_10@92.242.58.92:~/multiqc/multiqc_report.html ~/multiqc_untrimmed.html
scp -P 5222 aaromanova_10@92.242.58.92:~/multiqc_trimmed/multiqc_report.html ~/multiqc_trimmed.html

scp -P 5222 aaromanova_10@92.242.58.92:~/assemble.log ~/hse22_hw1/logs/assemble.log
scp -P 5222 aaromanova_10@92.242.58.92:~/scaffold.log ~/hse22_hw1/logs/scaffold.log
scp -P 5222 aaromanova_10@92.242.58.92:~/gapclose.log ~/hse22_hw1/logs/gapclose.log

scp -P 5222 aaromanova_10@92.242.58.92:~/Poil_contig.fa ~/hse22_hw1/data/Poil_contig.fa
scp -P 5222 aaromanova_10@92.242.58.92:~/Poil_scaffold.fa ~/hse22_hw1/data/Poil_scaffold.fa
scp -P 5222 aaromanova_10@92.242.58.92:~/Poil_gapClosed.fa ~/hse22_hw1/data/Poil_gapClosed.fa

scp -P 5222 aaromanova_10@92.242.58.92:~/Poil_contigBubble.fa ~/hse22_hw1/other/Poil_contigBubble.fa
scp -P 5222 aaromanova_10@92.242.58.92:~/Poil_scaffoldBubble.fa ~/hse22_hw1/other/Poil_scaffoldBubble.fa
scp -P 5222 aaromanova_10@92.242.58.92:~/Poil_32merFrq.tsv ~/hse22_hw1/other/Poil_32merFrq.tsv
scp -P 5222 aaromanova_10@92.242.58.92:~/Poil_lib1_insFreq.tsv ~/hse22_hw1/other/Poil_lib1_insFreq.tsv
scp -P 5222 aaromanova_10@92.242.58.92:~/Poil_lib2_insFreq.tsv ~/hse22_hw1/other/Poil_lib2_insFreq.tsv
scp -P 5222 aaromanova_10@92.242.58.92:~/Poil_scaffoldComponent.tsv ~/hse22_hw1/other/Poil_scaffoldComponent.tsv
```

