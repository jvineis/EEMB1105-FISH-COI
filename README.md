# EEMB1105-FISH-COI
Tutorial to analyze COI DNA sequences of fish tissue collected from commercial and restaurant sources.

## 1.  Open a terminal window on you laptop or PC
For all mac users you can find a terminal window by double clicking on the finder and navigating to "Applications" then to "Utilities" and double click on the "Terminal" icon  

*ALL THE STEPS BELOW WILL BE CARRIED OUT IN THIS TERMINAL WINDOW* - you will enter the text in the grey boxes below and hit enter to run the command.  We will also create bash scripts to run commands throught the cluster using emacs.  I will create a separate small tutorial for this.  

## 2. log into the discovery cluster

    ssh *user.name*@login.discovery.neu.edu 

instead of *user.name*, add your own credentials (usually, last name followed by a ".", then your first initial enter your password when prompted. Your password will not show up as you type - because someone might be reading it over your shoulder.

## 3. change to the course directory containing the COI fish sequences

    cd /work/jennifer.bowen/EEMB1105/Fish_COI_barcode_JL_30-359950134_ab1/

## 4. make a directory (folder) for your DNA sequences in your scratch directory

    mkdir /scratch/*user.name*/FISH-COI

## 5. copy the ab1 files from the Fish_COI_barcode_JL_30-359950134_ab1/ (where you should be) to your new directory in scratch

    cp *ab1 /scratch/*user.name*/FISH-COI

## 6. change to the directory where you just moved all the sequences to 

    cd /scratch/*user.name*/FISH-COI
    
## 7. Download the samples to your own computer and view the chromatogram files using 4peaks.  First you will need to download the 4Peaks software from here. https://nucleobytes.com/4peaks/index.html... If you have trouble with this, talk to Johannah.. she's the best!

#### 7a. open a new terminal window on your own computer by moving your mouse to the upper left of your screen and clicking on "Shell" the click on "New Window"

#### 7b. make a folder on your computer to copy the "ab1" files like this

     mkdir ~/Downloads/FISH-COI/

#### 7c. copy the files from the discovery cluster to your machine like this (make sure that you replace "user.name" with your own, e.g. vineis.j .  No need for the quotes, they are just there to remind you to use your own user name. 

    rsync -HalP "user.name"@login.discovery.neu.edu:/scratch/"user.name"/FISH-COI/*ab1 ~/Downloads/FISH-COI/

#### 7d. You should see the files download one at a time.  When its complete, you can open the ab1 files using 4Peaks.  Just open the software and click on the word "File" in the upper left of your computer display and then select "Open".  Then navigate in your Finder window to the ~/Downloads/FISH-COI/ and select the file that you would like to veiw. So beautiful!


## 8.  Load the module(settings) for all of the software that you will reuire for the analysis

    module load EEMB1105/01-24-2020

## 9.  Trim each forward and reverse sequence based on the presence of Ns (unknown characters). You will need to do this for all of the sequences in your directory.  This command will be run using sbatch   

    trimseq -sequence *sequence1F.ab1* -outseq *sequence1F-trimmed.fa* -window 20 -percent 5
    trimseq -sequence *sequence1R.ab1* -outseq *sequence1R-trimmed.fa* -window 20 -percent 5

## 10.  Reverse compliment the forward sequence. This command will be run using sbatch

    revseq -sequence sequence1F-trimmed.fa -outseq sequence1F-trimmed-rev.fa

## 11.  Merge the forward and reverse sequences.  This command requires the ouput from the previous sept as input.  This command will be run using sbatch

    merger -asequence sequence1F-trimmed-rev.fa -bsequence sequence1R-trimmed.fa -outfile sequence1-merged.aln -outseq sequence1-merged.fa

you should inspect the sequence1-merged.aln file. Enter this to see what it looks like

    more sequence1-merged.aln

## 12.  blast the merged sequences against the database of COI sequences.  This command will be run using sbatch
### first you need to unload the modules for the course and load the blast module
    module unload
    module load ncbi-blast+/2.9.0
### now run the blast command
    blastn -db FDA-RSSL.fa -query sequence1-merged.fa -out sequence1-merged-blastout
    
###  FOR JOE : working in this directory : /scratch/vineis.j/cap-test

I rec'd a set of fasta files with forward and rev sequences for some COI fish samples from Rosie Falco.  I placed these here /scratch/vineis.j/cap-test/RAW-sequences in the discovery server.  I tested quality filtering like this.  which seems to work well. Starts by removing Ns within a 20bp window from start and end of the sequences, then reverse compliment the forward sequence, then merge the two.. There was 100% consensus among the forward and reverse sequences.  Will be good to inspect the *.aln* file for each merger.

    trimseq -sequence Yuan_10-Fish-H-Fish-H.ab1 -outseq Yuan_10-Fish-H-Fish-H-trimmed.ab1 -window 20 -percent 5
    trimseq -sequence Yuan_10-Fish-L-Fish-L.ab1 -outseq Yuan_10-Fish-L-Fish-L-trimmed.ab1 -window 20 -percent 5
    revseq -sequence Yuan_10-Fish-H-Fish-H-trimmed.ab1 -outseq Yuan_10-Fish-H-Fish-H-trimmed-rev.ab1
    merger -asequence Yuan_10-Fish-H-Fish-H-trimmed-rev.ab1 -bsequence Yuan_10-Fish-L-Fish-L-trimmed.ab1 -outfile Yuan_10-merged.aln -outseq Yuan_10-merged.f

I rec'd an excel sheet with a list of sequences from Rosie Falco (Ocean Genome Legacy program) called something like FDA-RSSL-For-Website-Upload-Sept-2018.txt.  I converted this into a fasta file using the following script. Before running the script I manually edited the file so that there are three columns.  The first is the unique ID of the seq, scond column is the taxonomy of the sequence, and the third is the sequence.  Then I ran this to create a fasta file that I like.

    python ~/scripts/create-fasta-from-table.py FDA-RSSL-For-Website-Upload-Sept-2018.txt FDA-RSSL.fa
    
Then I build a blastdb like this

    makeblastdb -in FDA-RSSL.fa -parse_seqids -blastdb_version 5 -title "FDA-RSSL-db" -dbtype nucl
    
Then I tested it by unloading the class module and loading the blast module
    
    module unload
    module load ncbi-blast+/2.9.0
    
Then I tried and example blast and it worked great!

    blastn -db ../FDA-RSSL.fa -query Yuan_10-merged.fa -out x_test.txt
    
I also build an alignment using the batch script below

    #!/bin/bash

    #SBATCH --nodes=1
    #SBATCH --tasks-per-node=1
    #SBATCH --time=6:00:00
    #SBATCH --mem=10Gb
    #SBATCH --partition=short

    module load EEMB1105/01-24-2020
    clustalw2 FDA-RSSL.fa
    
Then I build a tree using the alignment.. I tried this on my own machine because of problems with the FastTree install on discovery.. Used clustall all the way.  Just wanted to see what the tree looks like and if its realistic for students.

    
    

    

