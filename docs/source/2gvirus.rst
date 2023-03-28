2G Viral Metagenomics
======================

Lecture
^^^^^^^



Introduction
^^^^^^^^^^^^

We will be diagnosing an unknown virus from a hibiscus dataset. This sample was sequenced on an Illumina NextSeq 550 with PE 2x75bp sequencing.

Detecting viruses from metagenomic sequencing data can be done with two main techniques:

* Taxonomic assignment of reads
* Contig assembly followed by taxonomic assignment of contigs

First, we will focus on the taxonomic assignment of reads.

Pipeline Picture
^^^^^^^^^^^^^^^^^^

  .. image:: _static/2gviruspipeline.png

Open History
^^^^^^^^^^^^^

On the first day of training we imported the reads from SRA and performed quality evaluation and filtering.

.. admonition:: Hands-On: Switch History

  1. At the top of the screen select ``Shared Data`` and select ``Histories`` from the drop down menu.

  2. In the Published Histories Panel that opens search for ``NPDN 2023`` and select, ``NPDN 2023 2G Virus Data``

  3. At the top of the page click ``Import History``

  4. If the history does not load automatically, click on the home icon on top then in the History panel click ``Switch Histories`` icon and select the newly imported history


Host Removal
^^^^^^^^^^^^^
The first step in any sequencing analysis is quality check (FastQC) and trimming (Trimmomatic or alternative). These sequences have already been trimmed since you practiced those steps on day 1.

First, we need to remove host contamination since this is a metagenomic sample. Most of the sample will be reads from the plant host. Let's map all the reads to the host genome so that we can remove them prior to further analysis.

.. admonition:: Hands-On: Map to Host Genome

  1. In Tools Panel, upload the Hibiscus Genome available from NCBI: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/006/381/635/GCF_006381635.1_ASM638163v2/GCF_006381635.1_ASM638163v2_genomic.fna.gz

  * Click ``Upload Data`` --> ``Paste/fetch Data`` --> Paste link above --> Start --> Close

  2. In tools menu, search for 'bowtie2' and click on it.

  3. Run bowtie2 tool with the following parameters

  * “Is this single or paired library”: ``Paired-end``

  * “FASTA/Q File #1”: Click on the down arrow and select ``Trimmomatic on Raw Read 1 (R1 paired)``

  * “FASTA/Q File #2”: Click on the down arrow and select ``Trimmomatic on Raw Read 2 (R2 paired)``

  * “Will you select a reference genome from your history or use a built-in index?”: ``Use a genome from the history and build index``

  * “Select reference genome”: ``GCF_006381635.1_ASM638163v2_genomic.fna.gz``

  * Write unaligned reads (in fastq format) to separate file(s): ``Yes``

  * “Save the bowtie2 mapping statistics to the history”: ``Yes``

  * Leave the rest as default.

  4. Click Execute.

  5. When the tool finishes rename the ``unaligned reads (L)`` to ``Read 1 Non-host`` and the ``unaligned read (R)`` to ``Read 2 Non-host``


Bowtie2 should produce four output datasets, one with mapping information (bam file), Reads 1 and reads 2 that were NOT aligned to the host genome (i.e. non-host reads) and the other with the corresponding mapping statistics. Let's take a look at the mapping statistics.

Inspect the mapping stats by clicking on eye icon next to ``Bowtie2 on X:mapping stats`` output in your history panel.

-------------------------

.. container:: toggle

  .. container:: header

  **What percentage of reads mapped to the plant genome?**

  This sample should have ~8-9% of the reads mapped to the host, hibiscus, genome.

----------------------------


Read Assignment with Kraken
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In this tutorial we will be using kraken to identify members in a mixed set of metagenomic reads. Kraken breaks reads into k-mers (substrings of length k) and queries a database with those k-mers

.. admonition:: Hands-On: Taxonomic Read Assignment with Kraken

    1. In the tools menu search for 'kraken2' tool and click on it.

    2. Run kraken2 with the following parameters:

  * Single or paired end reads: ``paired``

  * Forward strand:  ``Read 1 Non-host`` (file we just filtered).

  * Reverse strand: ``Read 2 Non-host`` (file we just filtered).

  .. image:: _static/kraken_input.png

  * Select a kraken database: ``Viral genomes (2019)``

  * Leave all others as default and click ``Execute``


Examine Kraken Output
^^^^^^^^^^^^^^^^^^^^^^

You should see a new output file at the top of your history panel called ``Kraken on data x: Classification``. Lets take a look at it.

When the file turns green (analysis done running) click on the eye icon next to the file to view it.

The columns correspond to the following:

1. "C"/"U": one letter code indicating that the sequence was either classified or unclassified.

2. The sequence ID, obtained from the FASTA/FASTQ header.

3. The taxonomy ID Kraken used to label the sequence; this is 0 if the sequence is unclassified.

4. The length of the sequence in bp.

5. A space-delimited list indicating the LCA mapping of each k-mer in the sequence. For example, "562:13 561:4 A:31 0:1 562:3" would indicate that:

  * the first 13 k-mers mapped to taxonomy ID #562

  * the next 4 k-mers mapped to taxonomy ID #561

  * the next 31 k-mers contained an ambiguous nucleotide

  * the next k-mer was not in the database

  * the last 3 k-mers mapped to taxonomy ID #562

.. container:: toggle

    .. container:: header

        **After looking at the first few sections of the results, in general are more reads classified or unclassified?**

    You should see the first column contains a lot of "U's", therefore most of the reads appear to be unclassified. Remember, we are just screening these against the virus database, so these reads could be host, bacteria, etc.

Kraken Report
^^^^^^^^^^^^^^
While the raw kraken output contains a lot of information, it is impossible to make sense of without summarizing it. Here, we will generate a kraken report to summarize the results.

.. admonition:: Hands-On: Generate a Kraken Report

  1. In the tools menu search for 'kraken-report' tool and click on it.

  2. Run kraken-report with the following parameters:

  * Kraken output: ``Kraken on data x: Classification``

  * Select a Kraken database: ``viral_2020``

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

        **What is the predominant classified species in the sample?**

    You should see the majority of the sample was unclassified (probably host, bacteria, etc.), and the predominant viruses in the sample are Citrus leprosis virus C2 and Hibiscus chlorotic ringspot virus.


Kraken allowed us to identify what virus(es) were present in out sample, but gave us no information on whether this is a new strain, percent identity, etc. We will perform assembly of our reads to get more information.



Genome Assembly with Metaspades
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Next we will assemble all reads that did not map to host using a specialized version of Spades designed for metagenomic samples, metaSpades.

.. admonition:: Hands-On: Assembly with metaSpades

  1. In the tools menu search for 'metaspades' tool and click on it.

  2. Run this tool with following parameters:

  * Forward Reads: ``Read 1 Non-host``

  * Reverse Reads: ``Read 2 Non-host``

  * Leave the rest as default

  3. Click Exceute.

When the assembly completes, take a look at the ``metaSPades scaffolds`` output.

-------------------------

.. container:: toggle

  .. container:: header

  **How many scaffolds were assembled?**

  This sample should ~5,000-6,000 scaffolds assembled.

----------------------------

Contig Length Filtering
^^^^^^^^^^^^^^^^^^^^^^^^

Because it would take us a long time to blast search over 5,000 contigs, we will filter by length and only look at the longest contigs here. Normally we would pick a much lower threshold (~200 nt) in order not to miss anything, especially viroids.

.. admonition:: Hands-On: Contig Filtering

  1. At the top of the Tools panel (on the left), search for 'filter sequences by length' and click on it.

  2. Run this tool with following parameters:

  * Fasta file: ``metaSPades scaffolds``

  * Minimal length: ``3000``

  * Maximum length: ``0``

-------------------------

.. container:: toggle

  .. container:: header

  **How many contigs are left after filtering?**

  This sample should have ~5 contigs left after filtering.

--------------------------

Blast Contigs
^^^^^^^^^^^^^^

While Galaxy does have a built in Blast tool, I found it very slow. With the small number of contigs we have left, we can use Blast through NCBI.

.. admonition:: Hands-On: Contig Filtering

  1. In the history panel, click on the eye icon to view your newly filtered contigs ``Filter sequences by length on X``.

  2. Copy the entire content of this file. (Should be ~5 contigs in fasta format)

  3. Open the NCBI Blastn website in another browser tab: https://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE_TYPE=BlastSearch

  4. Paste your contigs sequences	you copied into the box under ``Enter accession number(s), gi(s), or FASTA sequence(s)``

  5. Scroll down and hit Blast.


-------------------------

.. container:: toggle

  .. container:: header

  **What was your top Blast hit for each of your contigs?**

  You should see similar to your kraken analysis we recover  Citrus leprosis virus C2 and Hibiscus chlorotic ringspot virus, and also some host RNA and possible fungi.

----------------------------

Questions/Discussion
