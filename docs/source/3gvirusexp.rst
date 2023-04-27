Experimental 3G Viral Metagenomics
===================================


Introduction
^^^^^^^^^^^^
Here, we will analyze the data you generated over the past two days. You should each have two samples, one from a tomato host and the other from an orange host. Lets see what viruses we have!


Import Data
^^^^^^^^^^^
Lets import data from a shared history. These are your raw reads. The only thing that has been done is the multiple read files have been concatenated into a single file for analysis. There is also a database of plant viruses.

.. admonition:: Hands-On: Import Viral Metagenomic Reads

    1. At the top of the screen click on ``Shared Data`` and select ``Histories``

    2. In the search field, search for ``NPDN HTS2 2023``

    3. Find the history for ``NPDN HTS2 2023 Data 3G Virus `` Select the green plus sign to import into your Galaxy environment.

    4. After uploading the shared history you can delete barcodes that are not yours and rename your sequences to identify what they are, i.e. 'meta_tomato.fastq.gz' and 'meta_orange.fastq.gz'.

    5. You should now have 2 seqeuncing files (fastq.gz) and a virus database (nt_vrl_plant_pathoscope_95.fasta) in your history.



Sequence QC
^^^^^^^^^^^^^
The first step in any sequencing analysis is quality check and trimming. These sequences have already been based called with on-board base calling and this is how you would receive them off of the sequencer. Let's first check the quality of the data we received.


.. admonition:: Hands-On: Quality Check

  1. In tools menu, search for 'Nanoplot' and click on it.

  2. Run Nanoplot tool with the following parameters

  * “files”: ``virus_3g.fastq.gz``

  * Leave the rest as default.

  3. Click Execute.


Nanoplot should produce four output files. Let's take a look at the html output report.


Quality Filtering
^^^^^^^^^^^^^^^^^^^
Let's filter the data to remove adapters, chimeric reads, and low quality bases. First we will filter to retain only high-quality long reads. Quality filtering is a balancing act to retain enough high-quality reads for analysis. Here, we will set a minimum length for reads to maintain. We will also only keep the top 90% of high quality reads.



.. admonition:: Hands-On: Adapter Trimming

    1. In tools menu, search for 'porechop' and click on it.

    2. Run porechop tool with the following parameters

      * Input Fastq: ``meta_X.fastq.gz``

      * Output Format for the Reads: ``fastq.gz``

      * Leave the rest as default.

    3. Click Execute.

Porechop should produce a new fastq file with adapter and chimeric reads removed. Let's now filter by quality.

.. admonition:: Hands-On: Quality Filtering

    1. In tools menu, search for 'Filtlong' and click on it.

    2. Run Filtlong tool with the following parameters

      * Input Fastq: ``porechop output``

      * Output Theshholds:

          - Keep Percentage: ``90``

          - Min Length: ``1000``

      * Leave the rest as default.

    3. Click Execute.





Non-Host Read Extraction
^^^^^^^^^^^^^^^^^^^^^^^^^^

We will now remove host reads from the dataset. We will use the arabidopsis genome because it is well studied, however you could also use a more specific genome (tomato and orange) from NCBI.

.. admonition:: Hands-On: Remove Host Reads

    1. Run minimap2 with the following parameters:

      * Will you select a reference genome from your history or use a built-in index? ``Use a built in genome index``

    * Using reference genome: ``Arabidopsis thaliana (TAIR10)

      * Select fastq datasets: ``filtlong output``

      * Leave rest as default press 'Execute'


    2. Run samtools fastx

      * “BAM or SAM file to convert”: ``Map with minimap2``

      * “Output format”: ``compressed FASTQ``

      * “Outputs”: ``others``

      * “Require that these flags are set”: ``Read is unmapped``

      * Leave rest as default press 'Execute'

    3. When job completes, rename the output files to something more useful.

      * Click on pencil icon next to ``data X converted to fastqsanger.gz`` and rename to ``meta_tomato_nonhost.fastq.gz``


Read Assignment with minimap2
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We will be mapping all reads to identify members in a mixed set of metagenomic reads.

.. admonition:: Hands-On: Viral Read Mapping with minimap2


    1. Run minimap2 with the following parameters:

      * Will you select a reference genome from your history or use a built-in index?: ``Use a genome from history and build index``

      * Use the following dataset as the reference sequence:  ``nt_vrl_plant_pathoscope_95.fasta``

      * Select fastq dataset: ``meta_X_nonhost.fastq.gz``

  2. Run tool.

.. admonition:: Hands-On: Count read Mapping

    1. Find tool ``samtools idxstats``

    2. Run samtools idxstats with the following parameters:

    * BAM file: ``Map with minimap...``

    3. Run tool.

    4. Download this file to your computer and open in excel to examine.

Metagenome Assembly
^^^^^^^^^^^^^^^^^^^^^

Next we will assemble all reads that did not map to host using an assembler for 3G data, Flye. There are multiple assemblers available for MinION data, but this assembler provides a nice balance of accuracy and speed.

.. admonition:: Hands-On: Assembly with Flye

    1. In the tools menu search for 'flye' tool and click on it.

    2. Run this tool with following parameters:

      * Input Reads: ``X_nonhost.fastq.gz``

      * estimated genome size: 10k

      * Perform metagenomic assembly: ``Yes``

      * Leave the rest as default

    3. Run tool.

When the assembly completes, take a look at the ``Flye assembly info`` output.




Blast Contigs
^^^^^^^^^^^^^^

Let's Blast the contigs we generated. First we will build a blast database

.. admonition:: Hands-On: Blast Contigs

  1. Find the tool ``NCBI BLAST+ makeblastdb``.

  2. Run this tool with the following parameters:

  * Molecule type of input: ``nucleotide``

  * Subject database/sequences: ``Blast database from your history``

  3. Run tool.



.. admonition:: Hands-On: Blast Contigs

  1. Find the tool ``NCBI BLAST+ blastn``.

  2. Run this tool with the following parameters:

  * Nucleotide query sequence(s): ``Flye Consensus``

  * Subject database/sequences: ``Blast database from your history``

  3. Run tool.

  4. Download results to computer and open in excel.
