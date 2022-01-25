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

## Jupiter Notebook

#### 1. Импорт библиотек
```bash
import re
```

#### 2. Функция для получения данных
```bash
def get_info(f, text, output = True):
    lengths = []
    total_len = 0
    num = 0
    max_len = 0
    length = 0
    score = 0
    max_sequence = ''
    curr_sequence = ''
    for line in f:
        if (line[0] == '>'):
            if num != 0:
                lengths.append(length)
            num += 1
            if length >= max_len:
                max_len = length
                max_sequence = curr_sequence
            curr_sequence = ''
            length = 0
        else:
            curr_sequence += line.strip()
            length += len(line.strip())
            total_len += len(line.strip())
     
    lengths.sort(reverse = True) 
    for i in lengths:
        score += i
        if score >= total_len / 2:
            if output == True:
                print(f'Анализ {text}\n\
Общее количество: {num},\n\
Общая длина: {total_len},\n\
Длина самого длинного: {max_len},\n\
N50: {i}\n')
            break
    return max_sequence
```

#### 3. Контиги
```bash
max_cont = get_info(open('Poil_contig.fa', 'r'), 'Контигов')
```

Анализ Контигов\n\
Общее количество: 620,\n\
Общая длина: 3925993,\n\
Длина самого длинного: 179304,\n\
N50: 53942

#### 4. Скаффолды
```bash
max_scaf = get_info(open('Poil_scaffold.fa', 'r'), 'Скаффолдов')
```

Анализ Скаффолдов\n\
Общее количество: 74,\n\
Общая длина: 3872350,\n\
Длина самого длинного: 3831701,\n\
N50: 3831701


#### 5. Подсчет гэпов для необрезанных чтений
```bash
print(f'Общая длина гэпов: {max_scaf.count("N")}')
max_scaf = re.sub(r'N{2,}', 'N', max_scaf)
print(f'Число гэпов: {max_scaf.count("N")}')
```

Общая длина гэпов: 6182\n\
Число гэпов: 62


#### 6.
```bash
max_scaf = get_info(open('Poil_gapClosed.fa', 'r'), 'Скаффолдов', False)
```


#### 7. Подсчет гэпов для обрезанных чтений
```bash
print(f'Общая длина гэпов для обрезанных чтений: {max_scaf.count("N")}')
max_scaf = re.sub(r'N{2,}', 'N', max_scaf)
print(f'Число гэпов для обрезанных чтений: {max_scaf.count("N")}')
```

Общая длина гэпов для обрезанных чтений: 1378\n\
Число гэпов для обрезанных чтений: 7
