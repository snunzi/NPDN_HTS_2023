Sequence QC/Intro to Galaxy
===========================

Lecture
^^^^^^^

.. slide:: https://docs.google.com/presentation/d/e/2PACX-1vTQHb8pGMTlHNy_9iM5Y9cdMprYYswDbOlx3x5ZM4jV_dmaRZMj7FQy48diov2Ffw

Introduction
^^^^^^^^^^^^

In this practical you will learn to import and assess the quality of raw high throughput sequencing sequencing data in Galaxy.

The first dataset you will be working with is from an Illumina MiSeq dataset. The sequenced organism is the unculturable phloem-limited alphaproteobacterium “Candidatus Liberibacter
asiaticus”, which causes Huanglongbing (HLB) disease on citrus. The sequenced bacterium was obtained directly from the root of an infected pummelo sample in California. The sequencing was done as paired-end 2x300bp.

.. image:: _static/CitrusGreening1.jpg


Login to Galaxy
^^^^^^^^^^^^^^^
We will be using Galaxy for performing many of our bioinformatic analyses. This is a great free tool for performing bioinformatics, no coding necessary! There are lots of tutorials available online, both on how to use Galaxy and perform specific analysis. See training materials at link below on your own time.

https://training.galaxyproject.org/training-material/

.. admonition:: Hands-On: Login

    1. Open your favorite web browser (Chrome, Safari, or Firefox--not Internet Explorer) and login to Galaxy Europe. If not already registered for Galaxy Europe, use this link to register and login:

    https://usegalaxy.eu/

     .. image:: _static/galaxylogin.png

    2. Once logged in use the following link to access the training queue for this training. This link will only be available during the training, however your data and analysis results will still be available at usegalaxy.eu once training is over.

    https://usegalaxy.eu/join-training/npdn-hts1/


The Galaxy homepage is divided into three panels:

- Tools on the left
- Viewing panel in the middle
- History of analysis and files on the right

.. image:: _static/galaxy_interface.png


Name History
^^^^^^^^^^^^

.. admonition:: Hands-On: Name History

    1. Go to history panel (on the right)

    2. Click on the history name (which by default is “Unnamed history”)

    .. image:: _static/rename_history.png

    3. Type in a new name,  “NPDN Day 1”

    4. Press Enter on your keyboard to save it.


Import Sample Data
^^^^^^^^^^^^^^^^^^^
We will be working with reads as you would receive them off of a sequencer, or from a sequencing company. Let's import them into our Galaxy environment.

.. admonition:: Hands-On: Upload Data

    1. Copy the link locations below:

        https://zenodo.org/record/6110829/files/CA-Root_R1.fastq.gz
        https://zenodo.org/record/6110829/files/CA-Root_R2.fastq.gz

    2. Open the Galaxy Upload Manager ('Upload Data' on top-right of the tool panel)

    3. Select Paste/Fetch Data.

    4. Paste the links into the text field.

    5. Press start and close the window.

If you have any issues importing the files (slow network speed, etc.), please see the help page.

-------------------------

.. container:: toggle

    .. container:: header

        **We only sequenced this genome once, why do we have two files?**

    We performed paired-end sequencing, one set of reads is the forward read and the other is the reverse.

    .. image:: _static/pairedend.png

----------------------------

Assess Dataset Quality
^^^^^^^^^^^^^^^^^^^^^^

.. admonition:: Hands-On: Assess Data Quality

    1. At the top of the Tools panel (on the left), search for 'fastqc' and click on it.

    2. Run fastqc with the following parameters:

      * Short read data from your current history: Click on the icon for 'multiple datasets' and highlight both ``CA-Root_R1.fastq.gz`` and ``CA-Root_R2.fastq.gz`` (Ctrl-click until they turn blue)

      * Leave everything else as defaults.

    .. image:: _static/fastqc_upload.png

    3. Scroll down and click 'Execute'


You will see four new files generated in your history, while the analysis is running you will see a spinning wheel next to these files. When analysis completes, those files turn green. You should have two history items ``FastQC on 1[2]: Webpage``, one for forward reads and one for reverse). Click on the eye icon next each of these files to examine the results.

FastQC provides various output statistics. Scroll through and examine them.

At what point in the read do quality scores start declining?

-------------------------

.. container:: toggle

    .. container:: header

        **Look at the GC content plot, there may be two peaks, why is this?**

    In metagenomic datasets, like this, you may get multiple GC peaks representing different GC content for the different taxa in the sample (i.e. one peak for host DNA and one for pathogen)

----------------------------

Improve Dataset Quality
^^^^^^^^^^^^^^^^^^^^^^^

Illumina sequencing technology requires us to ligate adapters to both ends of genomic material to facilitate binding and sequencing on the flowcell. Adapter sequences should be removed because they can interfere with genome assembly. We will use Trimmomatic for adapter trimming and quality filtering.

Read more about Trimmomatic here: http://www.usadellab.org/cms/?page=trimmomatic

.. admonition:: Hands-On: Improve Data Quality

    1. At the top of the Tools panel (on the left), search for 'trimmomatic' and click on it.

    2. Run trimmomatic with the following parameters:

        * Single-end or paired-end reads? ``Select 'Paired-end' (two separate input files)``

        * Input FASTQ file (R1\first pair of reads): Click on the down arrow and select ``CA_Root_R1.fastq.gz``

        * Input FASTQ file (R2\second pair of reads): Click on the down arrow and select ``CA_Root_R2.fastq.gz``

        * Perform initial ILLUMINACLIP step? ``Yes``


        * Leave all other parameters as default.

    .. image:: _static/trim.png


    3. Click 'Execute'

    4. Repeat fastqc analysis on the paired trimmed files (``Trimmomatic on CA-Root_R1.fastq.gz  (R1 paired)`` and ``Trimmomatic on CA-Root_R2.fastq.gz  (R2) paired``).


Summarize Quality Metrics
^^^^^^^^^^^^^^^^^^^^^^^^^

In order to visualize and evaluate how trimming and filtering impacted our quality metrics, we will use the program MultiQC to summarize the results of multiple analysis tools.

.. admonition:: Hands-On: Summarize Quality Metrics

    1. At the top of the Tools panel (on the left), search for 'multiQC' and click on it.

    2. Run multiQC with the following parameters:

        * Which tool was used to generate logs? ``fastqc``

        * In “FastQC output”:

            * Type of fastQC output: ``Raw data``

            * FastQC output: Select raw data output files from FastQC (4 total files)


        * Leave all other parameters as default.


    3. Click 'Execute'

-------------------------

.. container:: toggle

    .. container:: header

        **Compare the seqeunce quality before and after trimming, is it good enough?**

    It looks like most quality flags have been resolved. You can proceed with analysis, however if downstream analyses fail trimming will have to be re-evaluated.

----------------------------

Convert Analysis into a Workflow
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When you look at your history, you can see that it contains all the steps of our analysis, from the beginning (at the bottom) to the end (on top). The history in Galaxy records details of every tool you run and preserves all parameter settings applied at each step. But when you need to analyze new data, it would be tedious to do each step one-by-one again. Wouldn’t it be nice to just convert this history into a workflow that we will be able to execute again and again?

Galaxy makes this very easy with the Extract workflow option. This means any time you want to build a workflow, you can just perform the steps once manually, and then convert it to a workflow, so that next time it will be a lot less work to do the same analysis.


.. admonition:: Hands-On: Create a Seq QC Workflow

    1. Clean up your history: remove any failed (red) jobs from your history. This will make the creation of the workflow easier.

    2. Click on galaxy-history-options (History options) at the top of your history panel and select Extract workflow.

    .. image:: _static/extractworkflow.png

    The central panel will show the content of the history in reverse order (oldest on top), and you will be able to choose which steps to include in the workflow.

    .. image:: _static/extractworkflow2.png

    3. Replace the Workflow name to something more descriptive, for example: ``Illumina PE QC``

    4. Rename the workflow input in the box at the top of second column to: ``Read1`` and ``Read2``

    5. Click on the Create Workflow button near the top.

Create a New History
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Let’s create a new history so that we can test out our new workflow and run some QC on another dataset we will be analyzing during this workshop.

.. admonition:: Hands-On: Create a New History

    1. Create a new history

    .. image:: _static/createnewhis.png

    2. Rename your history to ``NPDN 2023 2G Virus``

Upload Data from SRA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Here we will import Ilumina reads from NCBIs SRA database.

.. admonition:: Hands-On: Import Data from SRA

    1. In the tools panel search for ``Faster Download and Extract Reads in FASTQ`` and click on it

    2. Enter this Accession: SRR11794481

    3. Click ``Run tool``

    4. Several collections are created in your history panel when you submit this job:

        * Paired-end data (fasterq-dump); Contains Paired-end datasets (if available)

        * Single-end data (fasterq-dump); Contains Single-end datasets (if available)

        * Other data (fasterq-dump); Contains Unpaired datasets (if available)

        * fasterq-dump log; Contains information about the tool execution

Once fasterq finishes transferring the data explore the collections created by clicking on the collection name in the history panel. You should see in the paired-end data collection there is a pair of reads. This is what we will be analyzing.

Subset Data
^^^^^^^^^^^^

Because the dataset we just downloaded is very large analysis on the full dataset may take an extended period of time. To reduce the time spent running analysis lets subset the reads to only the first 1,000,000 reads.

.. admonition:: Hands-On: Import Data from SRA

    1. In the tools panel search for ``Select first lines from a dataset`` and click on it

    2. Set the following parameters:

        * Select first *: ``4,000,000``

		* from: ``SRR11794481 forward uncompressed`` and ``SRR11794481 reverse uncompressed``

	.. image:: _static/subsample.png

    3. Click ``Run tool``

	4. After the files are generated lets rename them to, ``Raw Read 1`` and ``Raw Read 2``



Run a Workflow
^^^^^^^^^^^^^^^
Lets run our quality control pipeline on our newly downloaded and subsetted dataset.

.. admonition:: Hands-On: Run A Workflow

    1. Click on Workflow in the top menu bar of Galaxy. Here you have a list of all your workflows. Your newly created workflow should be listed at the top:

    .. image:: _static/selectworkflow.png

    2. Click on the Run workflow button next to your workflow. The central panel will change to allow you to configure and launch the workflow.

    .. image:: _static/selectworkflow2.png

    3. Click on the Browse datasets icon on the right of each input box. For Read1 input select the ``Raw Read 1`` file, and for Read2 input choose ``Raw Read 2``.

    4. Select Run Workflow.

Examine the output from the workflow as it finishes.
