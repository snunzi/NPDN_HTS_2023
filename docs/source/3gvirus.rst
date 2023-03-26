3G Viral Metagenomics
=================================


Introduction
^^^^^^^^^^^^
Here, we will perform virus discovery on a Tomato leaf sample. The data was generated using the SQK-PCS108 cDNA PCR kit (Oxford Nanopore Technologies) and sequenced in a MinION Mk1B device (MIN-101B). Detecting viruses from metagenomic MinION sequencing data follows a similar bioinformatic workflow as 2G data. However, different programs are used that account for the different read properties.


Import Data
^^^^^^^^^^^
Lets import data from a shared history. These are raw reads, exactly how you would receive them from a sequencing company or off your own Nanopore sequencer with on-board basecalling enabled.

.. admonition:: Hands-On: Import Viral Metagenomic Reads

    1. At the top of the screen click on ``Shared Data`` and select ``Pages``

    2. In the search field, search for ``NPDN 2023``

    3. Find the history for ``NPDN 2023 Data 3G Virus `` Select the green plus sign to import into your Galaxy environment.

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


-------------------------

.. container:: toggle

	.. container:: header

		**How many reads are in this dataset?**

	This sample should have 20,000 reads. (I sub-sampled the reads to this number so that programs would run in a reasonable amount of time. Originally, there were over 3 million reads!). If you would like to compare results from a full run versus the subsampled run on your own time the SRA accession number for the full dataset is SRR11794480.

----------------------------

Quality Filtering
^^^^^^^^^^^^^^^^^^^
Many of the reads appear to have  low quality bases. Let's filter the data to remove adapters, chimeric reads, and low quality bases. First we will filter to retain only high-quality long reads. Quality filtering is a balancing act to retain enough high-quality reads for analysis. Here, we will set a minimum length for reads to maintain. We will also only keep the top 20% of high quality reads. This will help our analysis run faster, you may also set a minimum quality threshold. Please play with filtering on your own time to see how this impacts analysis.



.. admonition:: Hands-On: Adapter Trimming

    1. In tools menu, search for 'porechop' and click on it.

    2. Run porechop tool with the following parameters

      * Input Fastq: ``virus_3g.fastq.gz``

      * Output Format for the Reads: ``fastq.gz``

      * Leave the rest as default.

    3. Click Execute.

Porechop should produce a new fastq file with adapter and chimeric reads removed. Let's now filter by quality.

.. admonition:: Hands-On: Quality Filtering

    1. In tools menu, search for 'Filtlong' and click on it.

    2. Run Filtlong tool with the following parameters

      * Input Fastq: ``prechop output``

      * Output Theshholds:

          - Keep Percentage: ``20``

          - Min Length: ``1000``

      * Leave the rest as default.

    3. Click Execute.





Non-Host Read Extraction
^^^^^^^^^^^^^^^^^^^^^^^^^^

Just like we did with 2G viral metagenomic data, we will now remove host reads from the dataset. The mapper is specialized to long read data, but all other steps and the process are the same.

.. admonition:: Hands-On: Remove Host Reads

    1. Run minimap2 with the following parameters:

      * Will you select a reference genome from your history or use a built-in index? ``Use genome from history and build index``

      * Use the following dataset as the reference sequence: ``tomato.fna.gz``

      * Select fastq dataset: ``Porechop on data x``

      * Leave rest as default press 'Execute'


    2. Run samtools fastx

      * “BAM or SAM file to convert”: ``Map with minimap2``

      * “Output format”: ``compressed FASTQ``

      * “Outputs”: ``Read1``

      * “Require that these flags are set”: ``Read is unmapped``

      * Leave rest as default press 'Execute'

    3. When job completes, rename the output files to something more useful.

      * Click on pencil icon next to ``data X converted to fastqsanger.gz`` and rename to ``virus3g_nonhost.fastq.gz``


Read Assignment with Kraken
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Just like with our 2g dataset, we will be using kraken to identify members in a mixed set of metagenomic reads.

.. admonition:: Hands-On: Taxonomic Read Assignment with Kraken


    1. Run kraken2 with the following parameters:

      * Single: ``single``

      * Input Sequences:  ``virus3g_nonhost.fastq.gz`` (file we just filtered).

      * Select a kraken database: ``Viruses``

      * Leave all others as default and click ``Execute``

    2. Run kraken-report with the following parameters:

      * Kraken output: ``Kraken on data x: Classification``

      * Select a Kraken database: ``Viruses_2020``

When this analysis finished running it should generate a file ``Kraken-report on x``. Click the eye icon next to the result file and view the results.

The columns in the output correspond to the following:

1. percentage of reads in the clade/taxon in Column 6

2. number of reads in the clade.

3. number of reads in the clade but not further classified.

4. code indicating the rank of the classification: (U)nclassified, (D)omain, (K)ingdom, (P)hylum, (C)lass, (O)rder, (F)amily, (G)enus, (S)pecies).

5. NCBI taxonomy ID.

6. Scientific name

.. container:: toggle

    .. container:: header

        **What viruses were classified in the sample?**

    You should see the majority of the sample was classified as Pepino mosaic virus and Tomato Brown Rugose Fruit virus.

Metagenome Assembly
^^^^^^^^^^^^^^^^^^^^^

Next we will assemble all reads that did not map to host using an assembler for 3G data, Flye. There are multiple assemblers available for MinION data, but this assembler provides a nice balance of accuracy and speed.

.. admonition:: Hands-On: Assembly with Flye

    1. In the tools menu search for 'flye' tool and click on it.

    2. Run this tool with following parameters:

      * Input Reads: ``virus3g_nonhost.fastq.gz``

      * estimated genome size: 10k

      * Perform metagenomic assembly: ``Yes``

      * Leave the rest as default

    3. Click Exceute.

When the assembly completes, take a look at the ``Flye assembly info`` output.

-------------------------

.. container:: toggle

	.. container:: header

		**How many contigs were assembled?**

	This sample should ~4 scaffolds assembled.

----------------------------



Blast Contigs
^^^^^^^^^^^^^^

Let's Blast the contigs we generated through NCBI server.

.. admonition:: Hands-On: Contig Filtering

	1. In the history panel, click on the eye icon to view your contigs ``Flye on X consensus``.

	2. Copy the entire content of this file. (Should be four contigs in fasta format)

	3. Open the NCBI Blastn website in another browser tab: https://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE_TYPE=BlastSearch

	4. Paste your contigs sequences	you copied into the box under ``Enter accession number(s), gi(s), or FASTA sequence(s)``

	5. Scroll down and hit Blast.


-------------------------

.. container:: toggle

	.. container:: header

		**What was your top Blast hit for each of your four contigs?**

	You should see your contigs are Pepino moasci virus (mixed infection) and Tomato Brown Rugose Fruit Virus.

----------------------------

Questions/Discussion
