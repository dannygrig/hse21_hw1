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
