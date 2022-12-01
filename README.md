# Flujo de trabajo Analisis de ATACseq

Partiendo de las secuencias en crudo

#Control de calidad con fastqc
`conda activate Calidad
	Donde;
	Calidad: ambiente donde se alojan los programas 
`fastqc path/to/*.fastq.gz` -t n (Aplica el comando a todos los archivos con las terminacion fastq.gz)

	Donde;
 n es el numero de threads a usarse

#Trimming de adaptadores
#Utilizando trim galore, con secuencias de adaptadores especificas

`trim_galore --fastqc --adapter secuencia --adapter2 secuencia --gzip --paired archivo1.fq.gz archivo2.fq.gz

	Donde;

#Utilizando trim galore, sin secuencias de adaptadores especificas

`trim_galore --fastqc --gzip --paired -q 20 --stringency 1 archivo_pareado1.fq.gz archivo_pareado2.fq.gz

#Trimming de secuencias en los extremos
`trimmomatic PE -threads archivo_pareado_1.fq.gz archivo_pareado_2.fq.gz \ trim_ATAC_piloto_1.fastq  un_ATAC_piloto_1.fastq \ trim_ATAC_piloto_2.fastq un_ATAC_piloto_2.fastq CROP: n HEADCROP: n MINLEN: n
	Donde 
	HEADCROP: numero de bases a cortar a partir del inicio
	CROP: numero de bases a retener a partir del punto donde termina HEADCROP
MINLEN: minimo de longitud de la secuencia 

#Alineamiento
`conda activate Alineadores
`bowtie2 --very-sensitive -k 10 -x /home/aguilar-admin/Mau_ATAC/genomes/mm10/mm10 -1 secuencia_pareada_1.fastq -2 secuencia_pareada2.fastq -S ATAC_WAT_piloto.sam --threads n

#Conversión de .sam a .bam
`conda activate BED_BAM
`samtools view -S -b archivo1.sam > archivo1.bam --threads n

## Sorting e indexado de archivo .bam para obtener arvhivo narrowPeak (input para peak calling con Genrich)
`samtools sort -n archivo1.bam -o archivo1_sorted.bam --threads n

### Llamado de picos con Genrich
`conda activate CallPeakers
`Genrich -t archivo1_sorted.bam -o archivo1_peaks.narrowPeak -j -r -e chrM.chrY -v


## Sorting e indexado de archivo .bam para obtener archivo bigwig
`samtools sort arvhivo1.bam -o arvhivo1_sorted.bam --threads n
`samtools index ATAC_wat_sorted.bam -@ n

	Donde
	-@: numero de threads

# Obtención de archivo bigwig para visualización + Normalizacion 
`bamCoverage -b ATAC_wat_position.bam -o ATAC_wat.bw -of bigwig --ignoreDuplicates --minFragmentLength 150 --binSize 10 --extendRead`

bamCoverage -b archivo1_sortedbigwig.bam -o archivo1.bw --binSize 100 --normalizeUsing RPKM -e --ignoreDuplicates -p n

	Donde
	--binSize
	RPKM:
	-e


##Documentacion HeatMaps

`computeMatrix reference-point --referencePoint center -b 1500 -a 1500 -R Erythroblast_Chicken_SMC1_Rep1_peaks.narrowPeak -S Erythroblast_merged.bw eRBC_merged.bw aRBC_merged.bw --skipZeros -p 9 -o SMC1_peaks_lost.gz --outFileSortedRegions SMC1_peaks_lost.bed

# en este otro ejemplo, usé como punto de referencia los TSS y para ello le di un archivo GTF como referencia. 

`computeMatrix reference-point --referencePoint TSS  -b 2000 -a 2000 -R genoma_de_referencia.gtf -S archivo1.bw --skipZeros -p n -o archivo1.gz 
`plotProfile -m SMC1_peaks_lost.gz --plotFileFormat eps --perGroup -out SMC1_peaks_lost_profile.eps --plotTitle "tituloplot"
`plotHeatmap -m SMC1_peaks_lost.gz --plotFileFormat png -out SMC1_peaks_lost_Heatmap.png --yMin n --yMax n



##PCA
##utilizando deeptools, partiendo de los archivos bam
multiBamSummary BED-file --BED *.narrowPeak -b *sorted.bam -o WAT_HF_summary.npz -p n -e --ignoreDuplicates --outRawCounts conteos.tab

##utilizando deeptools, partiendo de los archivos bigwig
multiBigwigSummary BED-file --BED *.narrowPeak -b *.bw -o WAT_HF_sumary_bigwig.npz --outRawCounts conteos.tab
 
##Visualizacion
plotPCA -in archivo1.npz -o archivo1_PCA.png

bamPEFragmentSize -b archivo_1sorted.bam -o histogramafragmentos1.bam -p n  --maxFrragmentLength 1000

