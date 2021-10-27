# hse21_hw1
hw1
=====================
### Создаем ярлыки для наших файлов на сервере

    ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
    ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
    ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
    ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq

### Выбираем случайно 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs, создаем файлы fastq
    
    seqtk sample -s723 oil_R1.fastq 5000000 > R1sub.fq
    seqtk sample -s723 oil_R2.fastq 5000000 > R2sub.fq
    seqtk sample -s723 oilMP_S4_L001_R1_001.fastq 1500000 > MPR1sub.fq
    seqtk sample -s723 oilMP_S4_L001_R2_001.fastq 1500000 > MPR2sub.fq

### Используем программы fastqc и multiqc
   
    mkdir fastqc
    ls *.fq | xargs -P 4 -tI{} fastqc -o fastqc {}
    mkdir multifastqc 
    multiqc -o multiqc fastqc

### С помощью platanus подрезаем чтения по качеству и удаляем праймеры
    
    platanus_trim R1sub.fq R2sub.fq
    platanus_internal_trim MPR1sub.fq MPR2sub.fq

### Создаем директории и переименовываем подрезанные файлы
    
    mkdir trimmed_fastq
    mv -v *.fq.* *trimmed trimmed_fastq/

### Используем программы fastqc и multiqc для подрезанных файлов
    
    mkdir trimmed_fastqc
    ls trimmed_fastq/* | xargs -P 4 -tI{} fastqc -o trimmed_fastqc {}
    mkdir trimmed_multiqc
    multiqc -o trimmed_multiqc trimmed_fastqc

### Сравниваем получившиеся данные multiqc по графикам между начальными и подрезанными вариантами
#### До
![Before](https://github.com/dannygrig/hse21_hw1/blob/main/images/fastqc_sequence_counts_plot.png)
#### После
![After](https://github.com/dannygrig/hse21_hw1/blob/main/images/trim_fastqc_sequence_counts_plot.png)
#### До
![Before](https://github.com/dannygrig/hse21_hw1/blob/main/images/fastqc_per_base_sequence_quality_plot.png)
#### После
![After](https://github.com/dannygrig/hse21_hw1/blob/main/images/trim_fastqc_per_base_sequence_quality_plot.png)
#### До
![Before](https://github.com/dannygrig/hse21_hw1/blob/main/images/fastqc_adapter_content_plot.png)
#### После
![After](https://github.com/dannygrig/hse21_hw1/blob/main/images/trim_fastqc_adapter_content_plot.png)
##### Как видно из графиков, уменьшилась длина последовательностей, меньше дупликатов в процентах (из графиков Sequence counts), улучшилась оценка качества по каждой базовой позиции (на графиках Mean Quality scores): значения выше, теперь все в зеленом делении, кроме того на последнем графике видно, что были удалены адаптеры.

### С помощью программы platanus assemble собираем контиги из подрезанных чтений
    
    time platanus assemble -o Poil -f trimmed_fastq/R1sub.fq.trimmed trimmed_fastq/R2sub.fq.trimmed 2> assemble.log
Анализ контигов произведен в Jupiter ноутбуке

### С помощью platanus scaffold собираем скаффолды из контигов и из подрезанных чтений
    
    time platanus scaffold -o Poil -c Poil_contig.fa -IP1 trimmed_fastq/R1sub.fq.trimmed  trimmed_fastq/R2sub.fq.trimmed -OP2 trimmed_fastq/MPR1sub.fq.int_trimmed trimmed_fastq/MPR2sub.fq.int_trimmed 2> scaffold.log
Анализ скаффолдов произведен в Jupiter ноутбуке

### Применяем программу platanus gap_close для уменьшения количества гэпов

    time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 trimmed_fastq/R1sub.fq.trimmed  trimmed_fastq/R2sub.fq.trimmed -OP2 trimmed_fastq/MPR1sub.fq.int_trimmed trimmed_fastq/MPR2sub.fq.int_trimmed 2> gap_close.log

### Создаем файл с самым длинным скаффолдом

    echo scaffold1_len3834450_cov231 > longest_scaffold.txt
    seqtk subseq Poil_scaffold.fa longest_scaffold.txt > longest_scaffold.fa

### Создаем файл с самым длинным скаффолдом уже после использования platanus gap_close
     
    echo scaffold1_cov231 > longest_scaffold.txt
    seqtk subseq Poil_gapClosed.fa longest_scaffold.txt > gapclosed_longest_scaffold.fa
    
### Сравним полученные из скрипта данные о гэпах в самом большом скаффолде до и после использования platanus gap_close
#### До
    Количество гэпов =  61 Общая длина гэпов =  6429
#### После
    Количество гэпов =  11 Общая длина гэпов =  1653
##### Можно заметить, что количество и, соответственно, длина гэпов сильно уменьшились.

## Дополнительное задание

##### Для выполнения этого задания я изменил число чтений, уменьшив в 5 раз: теперь вместо 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs, 1 миллион и 300 тыс. соответственно.
##### Команды на сервере были использованы такие же как выше, были изменены только названия файлов. В итоге получаем нужные нам файлы, которые можно прогнать в python скрипте.
### Cравним получившиеся данные с исходным большим чтением.
#### Скаффолды и контиги 
##### Большее кол-во чтений
![Before](https://github.com/dannygrig/hse21_hw1/blob/main/images/cont_scaf1.png)
##### Меньшее кол-во чтений
![After](https://github.com/dannygrig/hse21_hw1/blob/main/images/cont_scaf2.png)
#### Гэпы в самом большом скаффолде
##### Большее кол-во чтений
![Before](https://github.com/dannygrig/hse21_hw1/blob/main/images/gaps1.png)
##### Меньшее кол-во чтений
![After](https://github.com/dannygrig/hse21_hw1/blob/main/images/gaps2.png)
##### Можно заметить, что во втором случае более чем в 2 раза больше скаффолдов, однако, их суммарная длина ниже. При этом количество и длина гэпов во втором случае заметно больше, что говорит о более низком качестве сборки.
