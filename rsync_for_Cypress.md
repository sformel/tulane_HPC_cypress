## Using rsync to maintain and update files on Cypress (the Tulane HPC)

by Steve Formel
last updated May 31, 2019

*Note: I got this from Erik Enbody, via Sarah Khalil.  Please contact me via github if you find any errors so I can fix them.*

### Description

rsync is a program for mac and linux computers that allows you to keep a folder in "sync" with a remote folder on another computer (like Cypress).  This way you can edit you scripts on your local computer and "push" them to cypress without having to use Filezilla.  Or you can download results, without having to use Filezilla.  

I think it's value is that it will keep you from having different copies of the same file floating around, ensuring that you won't accidentally publish the wrong results/code with your work.

### Installation

rsync may already be installed on your mac or linux computer.  check by typing "rsync" in terminal.  If not, start googling to install it.  If you're not familiar with rsync, [this is a good place to start](https://explainshell.com/explain/1/rsync).

### In "terminal" or some kind of shell on your local computer

##### Set variables in terminal:

	cypress_fp='/path/to/folder/on/cypress/'
	local_fp='/path/to/folder/on/your/computer'

##### if you want to see that you set the variable correctly:

	echo $cypress_fp
	echo $local_fp

### to upload to Cypress:

	rsync -avzPhe ssh $local_fp yourusername@cypress1.tulane.edu:$cypress_fp


### to download from Cypress:

	rsync -avzPhe ssh yourusername@cypress1.tulane.edu:$cypress_fp $local_fp


### Make it better...

If you're curious what the flags (-avzPhe) mean, [go here](https://explainshell.com/explain/1/rsync).  There are many more things you can do with rsync, which are also explained on that website.

