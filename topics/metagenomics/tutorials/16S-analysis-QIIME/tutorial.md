---
layout: tutorial_hands_on
topic_name: metagenomics
tutorial_name: 16S_analysis_QIIME
---

# Introduction
{:.no_toc}

In this tutorial you will learn how to perform a 16S Microbial Analysis using the QIIME toolsuite in Galaxy. We will work with Illumina sequencing data that is from [this study](https://www.nature.com/articles/ncomms9945) but that has been shortened to make this tutorial quicker to execute. We'll use the Greengenes reference OTUs, which is the default reference database used by QIIME. You may need to download and upload the reference database to your galaxy. 

In the study mentioned before, they performed high-throughput 16S rRNA gene sequencing of DNA extracted from faecal samples from day 3 of the evolution experiment (streptomycin-treated and colonized with E. coli) of **wild-type mice** and **immune-compromise mice lacking an adaptive immune system (Rag2−/−)**.

> ### Agenda
>
> In this tutorial, we will deal with:
>
> 1. TOC
> {:toc}
>
{: .agenda}

# Uploading the data
{:.no_toc}

This tutorial covers the case where you are starting with three specific input files: your sample metadata mapping file which contains all of the information about the samples necessary to perform the data analysis, a fastq file containing your amplicon sequence reads, and a corresponding fastq file containing the barcode reads for each amplicon sequence. 

> ### :pencil2: Hands-on: 
>
> 1. Create a new history
> 2. Upload to Galaxy the SRR1030347_1.fastq.interval.fq file provided. source:[1](http://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1004182), [2](https://www.ncbi.nlm.nih.gov/sra/?term=SRR1030347)

>>    > ### :bulb: Tip: Upload data to Galaxy
>>    >
>>    > * Click on the upload button in the upper left of the interface.

>>    >  ![](https://galaxyproject.github.io/training-material//topics/introduction/images/upload_button.png "source: https://galaxyproject.github.io/training-material//topics/introduction/tutorials/galaxy-intro-peaks2genes/tutorial.html")
>>    > * Press __Choose local file__ and search for your file.
>>    > * As __Type__ select 'fastq'.
>>    > * Press **Start** and wait for the upload to finish. Galaxy will automatically unpack the file.
>>    > * Rename the uploaded dataset to _First dataset_.     
>>    {: .tip}

> {: .hands_on}

# Validating the mapping file
{:.no_toc}

The QIIME mapping file contains all of the per-sample metadata, including technical information such as primers and barcodes that were used for each sample, and information about the samples, including what body site they were taken from. In this data set we're looking at human microbiome samples from four sites on the bodies of two individuals at mutliple time points. The metadata in this case therefore includes a subject identifier, a timepoint, and a body site for each sample. You can review the map.tsv file at the link in the previous cell to see an example of the data.
The contents of the mapping file are shown here - as you can see, a nucleotide barcode sequence is provided for each of the 9 samples, as well as metadata related to treatment group and date of birth, and general run descriptions about the project 
You should ensure that your mapping file is formatted correctly. This script will print a message indicating whether or not problems were found in the mapping file. An HTML file showing the location of errors and warnings will be generated in the output directory, and a plain text log file will also be created. Errors will cause fatal problems with subsequent scripts and must be corrected before moving forward. Warnings will not cause fatal problems, but it is encouraged that you fix these problems as they are often indicative of typos in your mapping file, invalid characters, or other unintended errors that will impact downstream analysis. A file ending with _corrected.txt will also be created in the output directory, which will have a copy of the mapping file with invalid characters replaced by underscores. 
>
> ### :nut_and_bolt: Comment
>
> Check the mapping file provided. What headers do we have?
> {: .comment}

> ### :pencil2: Hands-on: Validate the map file
>
> 1. **Validate mapping file**:wrench::Run the validate mapping file tool. **PUT SCRRENSHOT IMAGE HERE OF THE TOOL**
> 2. Select your map_bad.tsv file as the metadata mapping filepath. The rest of the parameters you can leave on default, since the mapping file that we have has the Barcodes and LinkerPrimerSequence headers.
> **Note**: If you scroll down on this page you will find the type of errors and warnings detected by the tool
>    >
>    > ### :question: Questions
>    >
>    > 1. Which type of files does the tool output to your history?
>    > 2. Check the HTML report file by clicking on the "eye" next to the name of the file. Can you identify any errors/warnings? What do you think they are?
>    > 3. Confirm your suspicions looking at the log file.
>    >    <details>
>    >    <summary>Click to view answers</summary>
>    >    <ol type="1">
>    >    <li> 
>    >     i. The tool produces 3 files: one that is an HTML report, a log file and a corrected map file.
>    > 
>    >     ii. If you open the HTML report you can see if there are any parts of the file in red (errors) or yellow (warnings). In this case we can see that the column "DaysSinceExperimentStart!" is yellow and some of the lines in the "BarcodeSequence" column are read. 
>    > 
>    >     iii. Now if you check the log file you can see the detailed information about the errors/warnings. We have a warning in the "DaysSinceExperimentStart!" column because there's an invalid character "!". We also have an error because our mapping file has the same column twice/because one of the barcodes is repeated for two different samples. </li>
>    >    </ol>
>    >    </details>
>    {: .question}
>
> 3. **Validate mapping file**:wrench:: Rerun the tool on the map.tsv which represents the new treated file and check if there are any errors. 
> {: .hands_on}

# Quality treatment and demultiplexing
{:.no_toc}
The next task is to assign the multiplexed reads to samples based on their nucleotide barcode (this is known as demultiplexing). This step also performs quality filtering based on the characteristics of each sequence, removing any low quality or ambiguous reads. To perform these steps we’ll use split_libraries.py
This will create three files in the new directory split_library_output/:

    split_library_log.txt : This file contains the summary of demultiplexing and quality filtering, including the number of reads detected for each sample and a brief summary of any reads that were removed due to quality considerations.
    histograms.txt : This tab-delimited file shows the number of reads at regular size intervals before and after splitting the library.
    seqs.fna : This is a fasta formatted file where each sequence is renamed according to the sample it came from. The header line also contains the name of the read in the input fasta file and information on any barcode errors that were corrected.
enerates sequences that begin at the BarcodeSequence, which is followed by the LinkerPrimerSequence, both of which are automatically removed during the demultiplexing step described below. The ReversePrimer (i.e., the primer at the end of the read) is not removed by default but can be using the -z option to split_libraries.py.
**NOTE:**THERES A TOOL IN GALAXY TO EXTRACT BARCODES IF YOU DONT HAVE AN EXTRA FILE WITH THE BARCODES.

> ### :question: Questions
>    >
>    > 1. Has the quality of the sequences been improved?
>    >    <details>
>    >    <summary>Click to view answers</summary>
>    >    <ol type="1">
>    >    <li> The per base quality score is better as it is the per base sequence content. The sequence length distribution is worse than before because sequences have different size after the trimming. </li>
>    >    </ol>
>    >    </details>
>    {: .question}
> ### :nut_and_bolt: Comment
>
> - If you send your sequencing to an external facility, they usually do these verifications and filtering for you, and you have “clean” sequences in the end, but it is always better to check. If you need to do this processing yourself, there are many small free programs to do this, such as [cutadapt](https://cutadapt.readthedocs.org/en/stable/). You'll also need to know the adaptors that were used in your library preparation (eg. Illumina TruSeq).
> {: .comment}

# Mapping the data
{:.no_toc}

After obtaining millions of short reads, we need to align them to a (sometimes large) reference genome. To achieve this, novel, more efficient, alignment methods had to be developed. One popular method is based on the burrows-wheeler transform and the use of efficient data structures, of which [bwa](http://bio-bwa.sourceforge.net/) and [bowtie](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml) are examples. They enable alignment of millions of reads in a few minutes, even in a laptop.

To store millions of alignments, researchers also had to develop new, more practical formats. The Sequence Alignment/Map (SAM) format is a tabular text file format, where each line contains information for one alignment. SAM files are most often compressed as BAM (Binary sAM) files, to reduce space and allow direct access to alignments in any arbitrary region of the genome. Several tools only work with BAM files.

> ### :pencil2: Hands-on: Alignment to a reference genome
>
> 1. **BWA-MEM**:wrench:: Search in the tool bar on the left the mapper ‘BWA-MEM’. 
> 2. Select **BWA-MEM** and use the uploaded dataset (First Dataset) as the fastqsanger file
> 3. Choose as reference genome the E.coli NC_000913.3_MG1655.fasta.([3](https://www.ncbi.nlm.nih.gov/nuccore/556503834))
>    **NOTE**: You may need to upload the file to your history. Please do it as already explained above.
{: .hands_on}

> ### :nut_and_bolt: Comment: Learning about SAM files 
>
> In the SAM format, each alignment line typically represents the linear alignment of a segment. Each line
has 11 mandatory fields.

> ![](https://galaxyproject.github.io/training-material//topics/usegalaxy/images/bam_structure.png) 
> To understand more about the information present in SAM files look briefly in [here](https://samtools.github.io/hts-specs/SAMv1.pdf) 
> {: .comment}

> ### :pencil2: Hands-on: Alignment to a reference genome

> 4. **BAM to SAM** :wrench:: Search in the tool bar on the left 'BAM to SAM'
> 5. Select the BAM file obtained with bwa-mem
>
>    > ### :question: Questions
>    >
>    > 1. After generating alignments and obtaining a SAM/BAM file, how do I know this step went well?
>    >
>    >    <details>
>    >    <summary>Click to view answers</summary>
>    >    <ol type="1">
>    >    <li> 
>    >    The same way as FastQC generates reports of fastq files to assess quality of raw data, there are programs that generate global reports on the quality of alignments.   
>    >
>    >    The way you check if the alignment step went well depends on your application. Usually, duplication levels higher than 20% are not a good sign (they're a sign of low input DNA and PCR artifacts) but again, depends on what you are sequencing and how much. Similarly, in the case of bacterial sequencing or targeted (eg. exonic) sequencing you expect >95% successful alignment, but if sequencing a full mamallian genome (with many duplicated areas) it may be normal to have as low as 70-80% alignment success. If you have to check the expected “quality” for your application. </li>
>    >    </ol>
>    >    </details>
>    {: .question}
{: .hands_on}

# Visualizing your results
{:.no_toc}

After finishing your analysis, even if you did all the quality checks, and obtained a list of variants, you may want to manually inspect your alignments (you should always manually inspect the regions that are most important for your analysis). For this, there is simple desktop software that you can use to visualize your data, such as [IGV](https://www.broadinstitute.org/igv/).


> ### :pencil2: Hands-on: Visualizing with the IGV browser
>
> 1. **IGV**:wrench::To display the result in IGV open the IGV browser locally on your computer. 
> 2. **IGV**:wrench::Open genome file ``NC_000913.3_MG1655.fasta``
> 2. **Galaxy**:wrench:: Click on the BWA-MEM item on the history panel .
> 3. **Galaxy**:wrench:: Choose in the history on the BWA-MEM results and click on ‘local’ at ‘display with IGV’.
> 4. **IGV**:wrench:: The BAM file should be opened in the IGV browser.
> 5. **IGV**:wrench:: Look at the following regions:
>> - NC_000913.3:3846244-3846290 
>> - NC_000913.3:1-1000 and NC_000913.3:4640500-4641652 
>> - NC_000913.3:3759212-3768438 

>> **NOTE:** Most genomes (particularly mamallian genomes) contain areas of low complexity, composed mostly of repetitive sequences. In the case of short reads, sometimes these align to multiple regions in the genome equally well, making it impossible to know where the fragment came from. Longer reads are needed to overcome these difficulties, or in the absence of these, paired-end data can also be used. **Paired end reads occur when a fragment of DNA is sequenced from two ends and therefore produces two reads.** Some aligners (such as bwa) can use information on paired reads to help disambiguate some alignments. Information on paired reads is also added to the SAM file when proper aligners are used. 

> ### :question: Questions
>    >
>    > 1. Look at the first region. What do you see?
>    > 2. Now look at the secong region suggested. You can see that some of the reads are coloured. What do you think may happen when assembling an area like this?
>    > 3. Finally, look at the last region. You can see that some reads are white. Try the option 'View as pairs' and analyse the results.
>    >    <details>
>    >    <summary>Click to view answers</summary>
>    >    <ol type="1">
>    >    <li> 
>    >    1.You can see that this region has an SNP (a C instead of an A in the genome).
>    >
>    >    2.The coloured reads that you see in this case represent reads in which the insert size is larger than expected. You can see it at the beggining and at the ending of the chromossome. This happens due to the chromossome of the E.coli being circular, therefore when sequencing from both ends of the fragment of DNA, the pair that corresponds to the the one in the beggining of the genome is at the end.
>    >
>    >    3.The reads are white because they have a mapping quality equal to zero. In this case, the read also maps to another location with equally good placement. When you view them as pairs, the program can gather information from a read that is sure in their location and that happens to be the pair for a white read. So, when viewing as pairs IGV uses that information and can be sure now that the white read belongs there or the opposite.</li>
>    >    </ol>
>    >    </details>
>    {: .question}
 
> {: .hands_on}



