# ДЗ1
## Ссылка на Colab
https://colab.research.google.com/drive/12z5yYSHlOIYH_tYIJkE7VyYcNrck20M6?usp=sharing
## Используемые команды
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
## Multiqc screenshots
Начальные варианты
![image](https://user-images.githubusercontent.com/55449081/138959676-79aec7c4-7006-47c1-9c09-7cdac27466a2.png)
![image](https://user-images.githubusercontent.com/55449081/138959771-bf64d99e-fbcf-4af3-8026-ded254eb1315.png)
![image](https://user-images.githubusercontent.com/55449081/138959846-29ffc01a-d778-4084-8f33-dce5307ca695.png)

Обрезанные адаптеры
![image](https://user-images.githubusercontent.com/55449081/138960636-d64f6a87-20a5-4827-a85d-14d7da068dc4.png)
![image](https://user-images.githubusercontent.com/55449081/138960664-40728463-1491-49e3-a21b-83334a111255.png)
![image](https://user-images.githubusercontent.com/55449081/138960849-687e3efa-6b5b-449e-ad11-f4de0bd1b1b3.png)

