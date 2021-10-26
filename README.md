# ДЗ1
Создание ссылок на файлы
```
    ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
    ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
    ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
    ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```    
Выбираем случайные чтения(seed = 917)
```
    seqtk sample -s917 oil_R1.fastq 5000000 > sub_1.fq
    seqtk sample -s917 oil_R2.fastq 5000000 > sub_2.fq
    seqtk sample -s917 oilMP_S4_L001_R1_001.fastq 1500000 > sub_MP1.fq
    seqtk sample -s917 oilMP_S4_L001_R2_001.fastq 1500000 > sub_MP2.fq
```        
Считаем статистику с помощью fastqc и multiqc
```
    mkdir fastqc
    mkdir multiqc
    ls sub* | xargs -tI{} fastqc -o fastqc {}
    multiqc -o multiqc fastqc
```   
Подрезаем адаптеры
```
    platanus_trim sub_1.fq sub_2.fq
    platanus_internal_trim sub_MP1.fq sub_MP2.fq
```   
Удаляем исходные файлы
```
    rm sub_1.fq
    rm sub_2.fq
    rm sub_MP1.fq
    rm sub_MP2.fq
```    
Сбор анализа
```
    mkdir fastqc_trim
    mkdir multiqc_trim
    ls sub* | xargs -tI{} fastqc -o fastqc_trim {}
    multiqc -o multiqc_trim fastqc_trim
```     
Сбор контигов из подрезанных чтений
```
    platanus assemble -o Poil -t 1 -m 10 -f *.trimmed 2>assemble.log
```     
Сбор скаффолдов
```
    platanus scaffold -o Poil -t 1 -c Poil_contig.fa -IP1 *.trimmed -OP2 *.int_trimmed 2>scaffold.log
```   
Закрываем гэпы
```
    platanus gap_close -o Poil -t 1 -c Poil_scaffold.fa -IP1 *.trimmed -OP2 *.int_trimmed 2>gapclose.log
```   
Находим наидленнейшую последовательность
```
    echo scaffold1_len3834572_cov232 > tmp.txt
    seqtk subseq Poil_scaffold.fa tmp.txt >longest.fasta
```
