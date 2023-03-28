3G Metabarcoding
=================================


Introduction
^^^^^^^^^^^^
Here, we will perform metabarcoding analysis on 2 unknown Phytophthora samples. There is no published well evaluated pipeline for analyzing 3G metabarcdoing datasets. Here we will explore the potential of this method, although we are still testing it's ability to discriminate closely related species and establish the best analysis methods. The following pipeline was evaluated by Subodh Srivastava to evaluate the method.

Pipeline Picture
^^^^^^^^^^^^^^^^^^

    .. image:: _static/3gmetabarcode.png

Import Data
^^^^^^^^^^^
Lets import data from a shared history. These are raw reads, exactly how you would receive them from a sequencing company or off your own Nanopore sequencer with on-board basecalling enabled.

.. admonition:: Hands-On: Import 3G Metabarcoding Data

    1. At the top of the screen click on ``Shared Data`` and select ``Histories``

    2. In the search field, search for ``NPDN 2023``

    3. Find the history for ``NPDN 2023 3G Metabarcode Data`` and select it and click import.

Sequence QC
^^^^^^^^^^^^^
The first step in any sequencing analysis is quality check and trimming. These sequences have already been based called with on-board base calling and this is how you would receive them off of the sequencer. Let's first check the quality of the data we received.


.. admonition:: Hands-On: Quality Check

  1. In tools menu, search for 'Nanoplot' and click on it.

  2. Run Nanoplot tool with the following parameters

    * “files”: ``FCR8-NPDN.BC02.fastq`` and ``FCR8-NPDN.BC05.fastq``

    * Leave the rest as default.

  3. Click Execute.


Nanoplot should produce four output files for each read file. Let's take a look at the html output report.


-------------------------

.. container:: toggle

  .. container:: header

    **How many reads are in this dataset?**

  Each sample should have about 5,000 reads. This was only run for a few hours with a pool of 12 samples total.

----------------------------

Quality Filtering
^^^^^^^^^^^^^^^^^^^
 Let's filter the data to remove adapters and chimeric reads.

.. admonition:: Hands-On: Adapter Trimming

    1. In tools menu, search for 'porechop' and click on it.

    2. Run porechop tool with the following parameters

      * Input Fastq: ``FCR8-NPDN.BC02.fastq`` and ``FCR8-NPDN.BC05.fastq``

      * Output Format for the Reads: ``fastq.gz``

      * Leave the rest as default.

    3. Click Execute.

Porechop should produce a new fastq file with adapter and chimeric reads removed.


Read Mapping Against a Database
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
We will now map the reads to a well-curated Phytophthora gene database. This was generated over years of hard work by Gloria Abad using Ex-Type specimens and Sanger sequencing.

.. admonition:: Hands-On: Read Mapping

  1. In tools menu, search for 'bwa-mem2' and click on it.

  2. Run bwa-mem2 tool with the following parameters

    * Will you select a reference genome from your history or use a built-in index? ``Use a genome from history and build index``

    * Use the following dataset as the reference: ``ITSv6_YPT-COI_ALL_DB_Combined.fasta``

    * Single or Paired end reads: ``Single``

    * Select the two Porechop trimmed files

  3. Click Execute.

Count number of Reads Mapping to Each Database Entry
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. admonition:: Hands-On: Read Map Counting

  1. In tools menu, search for 'samtools idxstats' and click on it.

  2. Click icon to analyse multiple datasets and select both bwa-mem2 output Files

  3. Click Execute

Summarize Read Mapping
^^^^^^^^^^^^^^^^^^^^^^^

.. admonition:: Hands-On: Read Map Count Summary

  1. In tools menu search for multiqc and click on it.

  2. Run multiqc with the following parameters

    * Which tool was used to generate logs? ``samtools``

    * + Insert Samtools output

    * Type of Samtools output? ``idxstats``

    * Samtools idxstats output: Select the two samtools idxstats files

  3. Click Execute
