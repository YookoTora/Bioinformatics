# hse21_hw1
###### Махирева Алёна БМТ191

## Задание 1

#### 1. Создадим ссылки в папке
```bash
 ls /usr/share/data-minor-bioinf/assembly/* | xargs -tI{} ln -s {}
```
#### 2. Выберем случайные чтения
```bash
SEED=108
seqtk sample -s$SEED oil_R1.fastq 5000000 > sub1.fastq
seqtk sample -s$SEED oil_R2.fastq 5000000 > sub2.fastq
seqtk sample -s$SEED oilMP_S4_L001_R1_001.fastq 1500000 > matep1.fastq
seqtk sample -s$SEED oilMP_S4_L001_R2_001.fastq 1500000 > matep2.fastq
```
#### 3. Оценим чтения используя FastQC
```bash
mkdir fastqc
ls sub* matep* | xargs -tI{} fastqc -o fastqc {}
```
#### 4. Создадим отчет через MultiQC
```bash
mkdir multiqc
multiqc -o multiqc fastqc
```
#### 5. Обрежим чтения
```bash
platanus_trim sub*
platanus_internal_trim matep*
```
#### 6. Удалим изначальные файлы
```bash
rm sub*.fastq matep*.fastq
```
#### 7. Оценим качество обрезанных чтений используя FastQC
```bash
mkdir fastqc_trimmed
ls sub* matep*| xargs -tI{} fastqc -o fastqc_trimmed {}
```
#### 8. Создадим отчет для обрезанных чтений через MultiQC
```bash
mkdir multiqc_trimmed
multiqc -o multiqc_trimmed fastqc_trimmed
```
#### 9. Соберем контиг используя “platanus assemble”
```bash
time platanus assemble -o Poil -f sub1.fastq.trimmed sub2.fastq.trimmed 2> assemble.log
```
#### 10. Соберем скаффолды используя
```bash
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matep1.fastq.int_trimmed matep2.fastq.int_trimmed 2> scaffold.log
```
#### 11. Уменьшим число промежутков
```bash
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matep1.fastq.int_trimmed matep2.fastq.int_trimmed 2> gapclose.log
```

#### 12. Удалим подрезанные чтения
```bash
rm sub*.trimmed matep*.int_trimmed
```

## Отчёты multiQC
#### Для исходных чтений
![](https://github.com/YookoTora/hse21_hw1/blob/main/main1.png)
![](https://github.com/YookoTora/hse21_hw1/blob/main/main2.png)

#### Для подрезанных чтений
![](https://github.com/YookoTora/hse21_hw1/blob/main/trimmed1.png)
![](https://github.com/YookoTora/hse21_hw1/blob/main/trimmed2.png)

