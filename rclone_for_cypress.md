# How to Install and use Rclone on Cypress

By Steve Formel

Last updated: August 6, 2019

***

Rclone is very similar to rsync, but it allows transfer of files between a cloud service, like google drive, dropbox, or box.com, and a computer.

**Your Tulane box.com account:**

> Note: As of writing this, there are box.com accounts that Tulane provides to PIs for backing up data.  If your PI has no idea that this exists, it's because they need to formally request it to be given one (although I'm not sure who they ask). Importantly, a single file can be no larger than 15Gb, which is problematic for files like large alignment files of nucleic acids.  So plan accordingly.  Otherwise, to the best of my knowledge, the space is unlimited, so back up your work early and often on box.com!

***

**Why?**

We can install rclone on Cypress and move files between your Cypress account and your box.com account with ease.  This is especially helpful when working with large amounts of data for multiple reasons:

1. **No need to work through a browser**.  Rclone can be run as a job on cypress and doesn't need to be supervised.

2. The fewer times you copy data (downloading from box.com and then uploading to cypress via an intermediate computer), **the less chance there is that your data will become corrupt**.

3. With large data, like DNA sequences, you often perform stepwise filtering and analyses that create copies of the entire data set.  For example, after trimming my sequences, I end up with the original raw sequences, and a slightly smaller set of trimmed sequences.  This quickly uses up my space on Cypress.  **By using rclone, I can shunt off files that aren't needed and keep my Cypress account relatively uncluttered.**

***

#### Install rclone on cypress

If you've never installed software on Cypress before, you can find a [tutorial in this same github repository](https://github.com/sformel/tulane_HPC_cypress/blob/master/How_to_make_a_module_on_Cypress.md).  More details to the process are given there.

##### Step 1: Download rclone for linux

	curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip
	
unzip it and move into the directory

	unzip rclone-current-linux-amd64.zip
	cd rclone-*-linux-amd64

##### Step 2: Make a module file for rclone

	nano /your/path/to/moduleFiles/rclone/1.47
	
> note that the version I installed was 1.47, but may be different for you.
	
In the nano editor write:

	#%Module1.0 -*- tcl -*-
	##
	## modulefile
	##
	
	module-whatis    Rclone is a command line program to sync files and directories to and from cloud storage
	prepend-path     PATH /path/to/rclone/folder/rclone-v1.47.0-linux-amd64/
	
##### Step 3: configure the box.com repository in rclone

>because rclone uses your web browser to authenticate the connection and cypress doesn't have a web broswer, we need to get the authorization code via our laptops.

1. On your laptop, install rclone.  You may need to download a different version (e.g. mac instead of linux)

2. configure the box.com repository on your laptop:

	Instructions at: https://rclone.org/box/

	> Note: for some reason the rclone github tutorial skips one question:  the answer is we do not want to use advanced config

3. when you get to the authorization step copy the portion that looks like this, you will paste it into the cypress configuration in the next step:

		{"access_token":"ZhfFlLOb0w7QstdewNpurIyw9yaGjjvv","token_type":"bearer","refresh_token":"EQFLNArhlXxIadSoRuOg8Ey6Rrps5QotxwfuU2VSVvM4eHIcu","expiry":"2013-04-14T22:32:48.844567-05:00"}
		
4. Go back to Cypress to configure rclone.
5. load the rclone module on cypress

		module load rclone/1.47
		
6. follow the configuration instructions at: https://rclone.org/box/
7. When you get this step:

		Use auto config?
 		* Say Y if not sure
 		* Say N if you are working on a remote or headless machine

 	say "N" and paste the authorization code from your laptop.
 	
 ##### Step 4: Test it out
 
Once rclone is configured you can transfer files back and forth.  Look at the rclone website to see more options.  Here are the basics:

1. List directories in top level of your Box

	> assuming you've named your repo "my_box"

		rclone lsd my_box:

2. List all the files in your Box

		rclone ls my_box:
		
3. list the files in a particular directory of box:

		rclone ls my_box:myproject/myfolder/
		
4. To copy a local directory on cypress to a Box directory:

		rclone copy /lustre/project/steve/test_box_ftp/ my_box:myproject/test_box_ftp
	
5. To copy from box to cypress:

		rclone copy my_box:myproject/test_box_ftp /lustre/project/steve/test_box_ftp/ 	

##### Example script

If I have a bunch of files I want to copy, I hsouldn't do it in the login node of cypress.  By running it as a job I can walk away without worrying about the internet connection breaking or being unfair to otther cypress users by hogging the login node.  Here is an example script:

	#!/bin/bash

	#SBATCH --qos=normal
	#SBATCH --job-name  box_sync
	#SBATCH --error  box_sync.error
	#SBATCH --output  box_sync.output
	#SBATCH --time=23:00:00
	#SBATCH --nodes=1
	#SBATCH --mem=64000
	#SBATCH --cpus-per-task=20
	#SBATCH --mail-type=ALL
	#SBATCH --mail-user=yourusername@tulane.edu

	module load rclone/1.47
	
	rclone copy \
	my_box:myproject/test_box_ftp \
	/lustre/project/steve/test_box_ftp/ 	
	
	#Note that the backslashes at the ned of each line are just a bash trick to make the code more readable, you do not need these for the rclone code to work properly.