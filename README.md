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

#### 7d. You should see the files download one at a time.  When its complete, you can open the ab1 files using 4Peaks.  Just open the software and click on the word "File" in the upper left of your computer display and then select "Open".  Then navigate in your Finder window to the ~/Downloads/FISH-COI/ and select the file that you would like to veiw. So beautiful! Once you are done with this, you can return to the terminal that is logged into discovery and pick up where you left off .  


## 8.  Load the module(settings) for all of the software that you will reuire for the analysis

    module load EEMB1105/01-24-2020
    
#### MUST READ TO UNDERSTAND THE REST OF THE TUTORIAL: Now that you are all set up with your data and the software that you need to be sequencing genious, you need to know how to run a program on the discovery cluster.  When you log into discovery, you are working from a computer (head node) that controls a huge number of other high performance computers (nodes).  Running commands that process your sequences from the head node is BAD!! The head node is already working hard for other people to distribute the commands that they are submitting, so slowing it down with heavier tasks is very inconsiderate!  To "submit a job" aka "run a script", aka "execute a command" we need to create a file that communicates details to the head node about a job we want to submit, such as how much memory we think it will take, and how long we expect the job to run etc.. We can communitcate all of these details in a "Slurm script" aka "bash script" aka "sbatch script". Here is an example of what one looks like with annotation of what each line means.


#### You will use "emacs" to create a "sbatch script" identical to the one above in your /scratch/*user.name*/FISH-COI/ directory according to the following steps.  

##### 1. Just to be sure that you are in the correct location, change to the directory you created that contains all of your COI sequences.

    cd /scratch/*user.name*/FISH-COI/
    
##### 2. open a new file called "x_sbatch-to-run-commands.shx".  The command below is using a program called emacs to create and open a new file called "x_sbatch-to-run-commands.shx".  When you type the command below, a blank screen will appear in your terminal.

    emacs x_sbatch-to-run-commands.shx
    
##### 3. Copy and paste the following text below (beginning with "#!/bin/bash) into the blank space that appeared when you ran the emacs command above.  Then you save the file and close it by followig the next steps exactly: 1. hold down the "control" key; 2. press the "x" key; 3. release the "x" key; 4. press the "s" key; 5. release the "s" key 6. release the "control" key.  7.  hold down the "control" key; 8. press the "x" key; 9. release the "x" key; 10. press the "c" key; 11. release the "c" key; 12. release the "control" key.  Your file will now be saved and closed.  You can reopen it using step 2 above and make the edits that you will need to run each of the scripts below. 

    #!/bin/bash  

    #SBATCH --nodes=1                
    #SBATCH --tasks-per-node=1       
    #SBATCH --time=00:30:00          
    #SBATCH --mem=50Gb               
    #SBATCH --partition=express      

    #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    ## You will never need to change any of the details above this line.  All the business is below the line above
    ## anything with a "#" in from of it will not be read by the head node except for the "#SBATCH" details above.  I use "##" before any notes I want to remember about this job submission and a "#" in front of commands that I don't want to run.
    
    ## Trim each forward and reverse sequences (step 9 of the tutorial)
    #trimseq -sequence *sequence1F.ab1* -outseq *sequence1F-trimmed.fa* -window 20 -percent 5
    #trimseq -sequence *sequence1R.ab1* -outseq *sequence1R-trimmed.fa* -window 20 -percent 5
    
    ## Reverse compliment the forward sequence (*step 10 of the tutorial)
    #revseq -sequence sequence1F-trimmed.fa -outseq sequence1F-trimmed-rev.fa
    
    ## Merge the forward and reverse sequences (step 11 of the tutorial)
    #merger -asequence sequence1F-trimmed-rev.fa -bsequence sequence1R-trimmed.fa -outfile sequence1-merged.aln -outseq sequence1-merged.fa
    
    ## To blast a sequence against the reference database of COI fish sequences provied by the Ocean Genome Legacy.
    #blastn -db /work/jennifer.bowen/EEMB1105/EXAMPLE-DATA/cap-test/FDA-RSSL.fa -query sequence1-merged.fa -out sequence1-merged-blastout
    
    ## If your top blast hit to the reference inicates that the match is "Strand=PLUS/MINUS" and not "Strand=PLUS/PLUS", you will need to reverse compliment the sequence prior to building a tree. Here is the command to reverse compliment your merged sequence.
    #revseq -sequence sequence1-merged.fa -outseq sequence1-merged-rev.fa

##### 4. You will make changes to the sbatch script anytime that you want to run a different command below.  The main change that you will make is adding and subtracting "#" at the beginning of specific lines in order to run each of the individual steps to process the sequences. So pick things up at step 9 below.  

## 9.  Trim each forward and reverse sequence based on the presence of Ns (unknown characters). You will need to do this for all of the COI sequences that you generated.  This command will be run using sbatch.  Delete the "#" in front of these lines, remove the * characters from the lines, and change all the names contained between these characters to the relevant names of your sequence "ab1" files.
   
    trimseq -sequence *sequence1F.ab1* -outseq *sequence1F-trimmed.fa* -window 20 -percent 5
    trimseq -sequence *sequence1R.ab1* -outseq *sequence1R-trimmed.fa* -window 20 -percent 5
    
##### once you make these changes in your x_sbatch-to-run-commands.shx file and you have saved your changes, run the command like this

    sbatch x_sbatch-to-run-commands.shx

## 10.  Reverse compliment the forward sequence. This command will be run using sbatch. Delete the "#" in front of these lines, remove the * characters from the lines, and change all the names contained between these characters to the relevant names of your sequence. 

    revseq -sequence *sequence1F-trimmed.fa* -outseq *sequence1F-trimmed-rev.fa*

##### once you make these changes, run the command below

    sbatch x_sbatch-to-run-commands.shx
    
## 11.  Merge the forward and reverse sequences.  This command requires the ouput from the previous sept as input.  This command will be run using sbatch. Comment out the last command that you ran in step 10 and delete the "#" in front of the line below. remove the * characters from the lines, and change all the names contained between these characters to the relevant names of your sample.  The "-outfile" in the command below is the alignmen of the forward and reverse sequences and the "-outseq" is a single fasta file that represents the consensus of your forward and reverse sequences   

    merger -asequence *sequence1F-trimmed-rev.fa* -bsequence *sequence1R-trimmed.fa* -outfile *sequence1-merged.aln* -outseq *sequence1-merged.fa*

##### you should inspect the sequence1-merged.aln file. Enter this command to see what it looks like

    more sequence1-merged.aln
    
##### it should look something like this.  Pay special attentio to the identity, similarity and the sequence alignment. 
![Image of alignment merger](https://github.com/jvineis/EEMB1105-FISH-COI/blob/master/Screen%20Shot%202020-04-03%20at%2011.11.05%20AM.png)

##### and the consensus sequence should look like this
![Image of merged sequence](https://github.com/jvineis/EEMB1105-FISH-COI/blob/master/Screen%20Shot%202020-04-03%20at%2011.24.30%20AM.png)

## 12.  blast the merged sequences against the database of COI sequences.  This command will be run using sbatch
### first you need to unload the modules for the course and load the blast module

    module unload
    module load ncbi-blast+/2.9.0

### now run the blast command using the sbatch script that you should be familiar with at this point. Remove the "#" in front of the line containing the blastn command shown below and make sure that the "-query" sequence is the same file name that you created in step 11 above.
   
    blastn -db /work/jennifer.bowen/EEMB1105/EXAMPLE-DATA/cap-test/FDA-RSSL.fa -query sequence1-merged.fa -out sequence1-merged-blastout
    
### run the sbatch and then inspect the results when its complete usning the more command as shown below
    
    sbatch x_sbatch-to-run-commands.shx
    more sequence1-merged-blastout

### Inspect the header information of your best blast alignment, there is a line that indicates whether the orientation of the matching sequence was in the forward or reverse orientation.  It looks like this. ![Image of blast header](https://github.com/jvineis/EEMB1105-FISH-COI/blob/master/Screen%20Shot%202020-04-08%20at%208.29.06%20AM.png) 

### See how the text indicates that the "Strand=Plus/Minus".  This incates that your sequence match is in the reverse compliment orientation.  Therefore, we need to reverse compliment your sequences prior to building a tree.  Comment out the lines below in the sbash script (you should be used to this now) to reverse compliment your sequences. The input "-sequence" should be the name of your merged sequence and the "-outseq" should reflect that the sequence is reversed. Run the x_sbatch-to-run-commands.shx in the usual way after making these changes. 

    revseq -sequence sequence1-merged.fa -outseq sequence1-merged-rev.fa

##  TREE Building: Here you will construct a phylogenetic tree that contains all the reference sequences that you ran a BLAST search against, along with your own sequences, in order to see the evolutionary relationship of your sequences with those in the reference.  

### 1. Add your three sequences to the reference multi-fasta file. This command pastes your merged, quality filtered single-fasta file onto the end of the multi-fasta reference that contains 1149 sequences to generate a new multi-fasta file called "fasta-for-treebuild.fa"

    cat /work/jennifer.bowen/EEMB1105/EXAMPLE-DATA/cap-test/FDA-RSSL.fa sequence1-merged.fa sequence2-merged.fa sequence3-merged.fa > fasta-for-treebuild.fa

### 2. Now you will create a new sbatch script (as you did in step three above) using emacs to that will include the commands you need to create an alignment and build a tree. You can copy and pase the following text into a file using emacs.  

##### so start with the emacs command
    
    emacs x_run-tree-build.shx

##### now copy and paste the following txt into the emacs window.

    #!/bin/bash

    #SBATCH --nodes=1
    #SBATCH --tasks-per-node=1
    #SBATCH --time=4:00:00
    #SBATCH --mem=100Gb
    #SBATCH --partition=short

    module load EEMB1105/01-24-2020
    clustalw2 fasta-for-treebuild.fa > /dev/null # build the alignment.. this will create a file called "fasta-for-treebuild.aln" which will be used in the next step
    clustalw2 -tree -infile=fasta-for-treebuild.aln > /dev/null # this will use the alignment from the script above to create a newick tree called "fasta-for-treebuild.ph"
    
###### now download the tree file to your computer... run this command in a new terminal on your machine NOT logged into discovery. don't forget to change the "user.name" to your username

     rsync -HalP "user.name"@login.discovery.neu.edu:/scratch/"user.name"/FISH-COI/fasta-for-treebuild.ph ~/Downloads/fasta-for-treebuild.ph
     
###### now use figtree http://tree.bio.ed.ac.uk/software/figtree/ to view your sequence in the overall tree. To download figtree, just click on the link for "FigTree.v1.4.4.dmg" and follow the prompts. You can view the tree in figtree by selecting "File" > "Open" and find the tree that you downloaded from discovery called "fasta-for-treebuild.ph" in your Downloads folder.  
       
## FOR JOE
We have a set of fasta files with forward and rev sequences for some COI fish samples from Rosie Falco from the Ocean Genome Legacy program at Northeastern.  I placed these here  on the discovery server.  I tested quality filtering like this.  which seems to work well. Starts by removing Ns within a 20bp window from start and end of the sequences, then reverse compliment the forward sequence, then merge the two.. There was 100% consensus among the forward and reverse sequences.  Will be good to inspect the *.aln* file for each merger.

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

    

