# EEMB1105-FISH-COI
Tutorial to analyze COI DNA sequences of fish tissue collected from commercial and restaurant sources.

## 1.  Open a terminal window on you laptop or PC.: 
For all mac users you can find a terminal window by double clicking on the finder and navigating to "Applications" then to "Utilities" and double click on the "Terminal" icon  

*ALL THE STEPS BELOW WILL BE CARRIED OUT IN THIS TERMINAL WINDOW*

## 2. log into the discovery cluster:
type ssh *user.name*@login.discovery.neu.edu but instead of *user.name*, add your own credentials (usually, last name followed by a ".", then your first initial
enter your password when prompted 

## 3. change to the course directory:
cd /shared/rc/training/

## 4. make a directory (folder) for your DNA sequences which should be something like your last name, for example:
mkdir vineis
this will make a new directory in /shared/rc/training/ called vineis

## 5. change to this directory by typing "cd" followed by the text you provided after mkdir in step4
cd vineis

## 6. copy your sequences from the directory that contains everyones sequences to your working directory (where you should be now).
cp /shared/rc/training/*ALL-SEQ-DIRECTORY*/*my_sequences.ab1* /shared/rc/training/*user.name*/
you will need to do this for each of the sequences in your list.. can you think of an awesome way to do this so that you don't have to do the same thing more than once?

## 7. Copy the sequences to your computer to view their quality in (whatever we choose to look at chromatogram files)
keep your current terminal window open and hit the *command* followed by the *n* key (without letting go of the *command* key) to open a new terminal window.  Here we will use the rsync command to copy the files from the discovery cluster to our laptops like thus.

rsync -HalP *user.name*@login.discovery.neu.edu:/shared/rc/training/*user.name*/*.ab1* */a/directory/of-your-choice/*

## 8.  Use the 


