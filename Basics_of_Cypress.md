# Basics of Cypress
## TURF Workshop

By Steve Formel

Last updated: Aug. 9, 2019

**Description:**  This is a Tulane University R Fans (TURF) workshop to show how to locally install software, run a job, and move files between Cypress (the Tulane HPC), and cloud servers like box.com

***

#### Things to do ahead of time:

1. [Get a Cypress account](https://wiki.hpc.tulane.edu/trac/wiki/cypress/about#Gettinganaccount).
2. If you have a PC, [install putty](https://www.putty.org/).
3. a [box.com](https://account.box.com/login?redirect_url=/) account


#### What we will do:  

1. Login to cypress
2. Install rclone
3. Use rclone to move files between Cypress and box.com

*Note:  We will be using a lot of Unix commands.  If you're not familiar with them, just type whatever you're asked to type and remember that capital letters, punctuation, and spaces matter.  You can learn more about some [basic Unix commands here](https://wiki.hpc.tulane.edu/trac/wiki/Workshops/IntroToHpc2015/BasicLinuxComands), or by a quick google.*  

*As you continue to use Cypress, fluency in these basic commands will make your life a lot easier.*

***

##### About Cypress

The [Cypress wiki](https://wiki.hpc.tulane.edu/trac/wiki/cypress/about#AboutCypress) has a lot of information on it.  Here are some basics:

* **Cypress is a bunch of computers tied together**
	
	> Each computer is called a "node"
		
	> Two of these nodes are used for logging in
		
	> The rest are used for computing
		
	> Some nodes have more computing power than others

* **You run jobs on Cypress by submitting a set of instructions to a queue.**  

	> When your turn in the queue comes, Cypress loads up whatever software you have instructed it to load up and executes whatever code you have given it run.

Cypress has a [tutorial](https://wiki.hpc.tulane.edu/trac/wiki/Workshops/IntroToHpc2015%20) you can work through here that is much more detailed.

**Cypress is friendly!**

Cypress is a great team of people that maintain this awesome *free* resource for us.  They know a ton, and there is so much more that can be done with a supercomputer than what is in this tutorial. Feel free to email them with questions, they're very responsive and nice people.


Cypress Team             |  + Carl (should be in the photo on the left)
:-------------------------:|:-------------------------:
![](data:image/jpeg;base64,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) |  ![](https://crsc.tulane.edu/wp-content/uploads/sites/29/2014/09/IMG_3098.jpg)



Contact them with any questions at: hpcadmin (at) tulane.edu

***

#### Logging in to a Cypress login node

This will be slightly different for different operating systems.  Mac and Linux users will log in through their Terminal.  PC users will log in through putty. 

*Your username and password will be the same as that for your Tulane user account.*

You can walk your way through logging in by following [this Cypress page](https://wiki.hpc.tulane.edu/trac/wiki/Workshops/IntroToHpc2015/SshUsage).

**Short version:**

1. Windows:
	
	* Open putty
	* To log onto cypress, type:

			cypress1.tulane.edu into the Host Name field
	
	* make sure the port is set to 22. 
	* Click on Open and you will be given a login window to cypress.
	* follow the prompts
	* your password will be invisible when you type it

2. Mac/Linux

	* Open Terminal
	* type 
	
			ssh yourusername@cypress1.tulane.edu

	* It will ask for your password.  When you type the password, it will be invisible.
	* The first time you log into a new machine, you will be asked if you trust the server. Say yes.

	
#### Your storage space on Cypress

You have to places on Cypress that you can store data:

1. Home: 

		$HOME or ~/
2. Lustre: 

		/lustre/project/groupname/

###### Home

Your home directory on Cypress is intended to store customized source code, binaries, scripts, analyzed results, manuscripts, and other small but important files. This directory is limited to 10 GB (10,000 MB), and is backed up. 

*Please do not use your home directory to perform simulations with heavy I/O (file read/write) usage. Instead, please use your group's Lustre project directory.*

###### Lustre

Each group (e.g. your lab) is given a space allocation of 1 TB and is allowed to have up to 1 million files or directories on the Lustre filesystem. 

If you need additional disk space to run your computations, your PI may request a quota adjustment. 

The Lustre filesystem is not for bulk or archival storage of data. The filesystem is configured for redundancy and hardware fault tolerance, but is **not backed up**.

### Let's install rclone!
![](https://rclone.org/img/rclone-120x120.png)

**What:** 

[rclone](https://rclone.org/) is software to move files between commercial cloud services and another computer.  

**Why:** 

1. **No need to work through a browser**.  Rclone can be run as a job on cypress and doesn't need to be supervised.

2. The fewer times you copy data (downloading from box.com and then uploading to cypress via an intermediate computer and filezilla), **the less chance there is that your data will become corrupt**.

3. With large data, like DNA sequences, you often perform stepwise filtering and analyses that create copies of the entire data set.  For example, after trimming my sequences, I end up with the original raw sequences, and a slightly smaller set of trimmed sequences.  This quickly uses up my space on Cypress.  **By using rclone, I can shunt off files that aren't needed and keep my Cypress account relatively uncluttered.**

***

#### Step 1: Download rclone for linux into your lustre space

move into your lustre and make a folder:

	cd /lustre/project/groupname/username
	mkdir ./software
	
download rclone

	curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip
	
unzip it and move into the directory

	unzip rclone-current-linux-amd64.zip
	cd rclone-*-linux-amd64

#### Step 2: Make a module file for rclone in your home space

	mkdir $HOME/modulefiles
	mkdir $HOME/modulefiles/rclone
	
	nano /your/path/to/modulefiles/rclone/1.47
	
> note that the version I installed was 1.47, but may be different for you.
	
In the nano editor write:

	#%Module1.0 -*- tcl -*-
	##
	## modulefile
	##
	
	module-whatis    Rclone is a command line program to sync files and directories to and from cloud storage
	prepend-path     PATH /path/to/rclone/folder/rclone-v1.47.0-linux-amd64/
	
#### Step 3: configure the box.com repository in rclone

<img src=https://upload.wikimedia.org/wikipedia/commons/thumb/5/57/Box%2C_Inc._logo.svg/1200px-Box%2C_Inc._logo.svg.png width=100>

>because rclone uses your web browser to authenticate the connection and cypress doesn't have a web broswer, we need to get the authorization code via our laptops.

**For Windows**

1. Install rclone on a windows computer by downloading the appropriate zip file from https://rclone.org then unzip file. 
2. At the rclone unzipped folder press shift and mouse right click at same time then select Open command window here. This should open a command window in the unzipped rclone folder. 
3. Then follow the instructions at: [https://rclone.org/box/]()

	> Note: for some reason the rclone github tutorial skips one question:  the answer is we do not want to use advanced config
 
4. when you get to the authorization step copy the portion that looks like this, you will paste it into the cypress configuration in the next step

		{"access_token":"ZhfFlLOb0w7QstdewNpurIyw9yaGjjvv","token_type":"bearer","refresh_token":"EQFLNArhlXxIadSoRuOg8Ey6Rrps5QotxwfuU2VSVvM4eHIcu","expiry":"2013-04-14T22:32:48.844567-05:00"}
		
5. Go back to Cypress to configure rclone.
6. load the rclone module on cypress

		module load rclone/1.47
		
7. follow the configuration instructions at: https://rclone.org/box/
8. When you get this step:

		Use auto config?
 		* Say Y if not sure
 		* Say N if you are working on a remote or headless machine

 	say "N" and paste the authorization code from your laptop.
 	
**For mac/linux**

1. On your laptop, install rclone.  You may need to download a different version (e.g. mac instead of linux)

2. configure the box.com repository on your laptop:

	Instructions at: [https://rclone.org/box/]()

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
 	

#### Step 4: Test it out
 
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

### How to run a job on Cypress

<img src=https://crsc.tulane.edu/wp-content/uploads/sites/29/2014/09/shieldword_2color_web2.jpg width = 100>

If I have a bunch of files I want to copy, it's inappropriate do it in the login node of cypress.  By running it as a job I can walk away without worrying about the internet connection breaking or being unfair to other cypress users by hogging the login node.  Here is an example script:

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
	my_box:myproject/test_box \
	/lustre/project/groupname/username/test_box/ 	
	
	#Note that the backslashes at the end of each line are just a bash trick to make the code more readable, you do not need these for the rclone code to work properly.
	
#### Step 1: Make the script in nano

We used nano above, see if you can remember how to do it yourself!

Try to:

1. make the folder test_box
2. make the script in that folder
3. name the script "rclone_test.sh"

#### Step 2: Run the script you just made.

In the Cypress terminal type:

	sbatch rclone_test.sh
	
and hit enter.

*Remember, if you're not in the folder where the script is, then you'll also need to specify the path to the folder.*

##### What should you have?

1. You should get an email in your tulane account when the job begins and when it ends.

2. You should also now have a folder on your box.com account that is called "test\_box" that contains the file "rclone\_test.sh"

##### What did we just do?

We used a language called [SLURM](https://wiki.hpc.tulane.edu/trac/wiki/cypress/about#SLURMresourcemanager) (all the #SBATCH stuff) to tell Cypress what resources we need to run our job.  Then when our turn in the queue came, we told it to load the rclone software and execute the rclone command.


### SUMMARY

Here is what you should have gotten through:

1. Logging into Cypress through a secure shell (SSH)
2. Made a module file for rclone in our home space on Cypress
3. Downloaded and configured rclone for box.com in our lustre space on Cypress
4. made a SLURM script
5. Run a job that copied files from Cypress to box.com

***

### BONUS!

Investigate [the interactive mode of Cypress](https://wiki.hpc.tulane.edu/trac/wiki/cypress/using#SubmittingInteractiveJobs).  This allows you to snag a computing node for yourself and run heavy computing interactively.  Perfect for figuring out how to run those heavy simulation in R!