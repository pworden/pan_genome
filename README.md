# Core/Pan genome analysis

---

## Roary

1. Set up a conda environment for Roary (use conda or mamba)

    <https://sanger-pathogens.github.io/Roary>

    ```bash
    conda create --name roary
    conda activate roary 
    # Install roary
    conda config --add channels r
    conda config --add channels defaults
    conda config --add channels conda-forge
    conda config --add channels bioconda
    conda install roary
    ```

1. Some simple commands to prepare inputs and run Roary.
    - Use *'roary --help'* for more information.

    ```bash
    # User inputs --------------------------------------------------------------------
    inDir="/home/tlaird/Paul_data/panGenome"
    outDir="/home/tlaird/Paul_data/panGenome/roary_out"
    iDsuffix=".gff"
    # --------------------------------------------------------------------------------
    # Create an output directory if it does not exist
    if [ -e $outDir ]; then echo "Folder exists!"; else mkdir $outDir; echo "Creating folder: $outDir"; fi
    # Look for '.gff' files and their paths to run with
    inPathAndFiles=${inDir}/"*"$iDsuffix
    roary -f $outDir -p 24 -e -n -v --group_limit 200000 $inPathAndFiles
    ```

## Panaroo

<https://github.com/gtonkinhill/panaroo>

See the Panaroo **install** github page for detalied information on a conda install
<https://gtonkinhill.github.io/panaroo/#/gettingstarted/installation>

1. First create a text file of '.gff' paths to serve as inputs for Panaroo

    ```bash
    # Input the path to the parent dir that contains the files or subdirectories with '.gff' files of interest
    inDir="/home/tlaird/Paul_data/panGenome"
    # This command looks for all ".gff" files in a parent directory and 2 levels of subdirectories
        # the "-type" should be "f" for files
    find $inDir -maxdepth 2 -type f -name *".gff" > $inDir"/P_larvae_GFF_List.txt"
    ```


1. Knowing the path of the saved text file from above, run the following on the command line.

    ```bash
    conda activate panaroo # Install as described on the github repo (above)

    # Input
    inputFileAndPath="/home/tlaird/Paul_data/panGenome/P_larvae_GFF_List.txt"
    outputDirectory="path/to/output/directory"
    # -----

    inputFileList=$(<"$inputFileAndPath")

    # Run panaroo using the flag to keep paralogs as needed for piggy [use the flag: --merge_paralogs]
    panaroo -i ${inputFileList[@]} -o $outputDirectory --clean-mode strict --remove-invalid-genes -t 30 -a core --aligner mafft --core_threshold 0.98
    ```

1. Run IQ-Tree on the alignment file from the Panaroo analysis

    <https://github.com/iqtree/iqtree2>

    - Set up in a separate conda environment
    - Then activate the IQ-Tree conda environment

    ```bash
    # Output directory for IQ-Tree
    outputDirectory=path/to/the/output/directory
    # Path to the Panaroo alignment file
    coregenePathAndFile=directory/of/panaroo/alignment/file/core_gene_alignment.aln
    # Run IQ-Tree
    iqtree -s $coregenePathAndFile -pre $outputDirectory/core_tree -nt 16 --fast -m GTR --redo
    ```

1. Run ClonalFrameML on the newick and alignment files output from Panaroo (and add a prefix for outputs)

    <https://github.com/xavierdidelot/clonalframeml/wiki>

    - Set up in a separate conda environment for clonalframeml
    - Then activate the clonalframeml conda environment

    ```bash
    # Install conalframeml using conda (this conda environment must be active)
    conda install -c conda-forge -c bioconda -c defaults clonalframeml
    newick_file="path/to/newick/file/from/panaroo"
    seq_file=directory/of/panaroo/alignment/file/core_gene_alignment.aln
    output_prefix="clonalFrameML" # To name/organise output files
    # Run ClonalframeML
    ClonalFrameML newick_file seq_file output_prefix
    ```

