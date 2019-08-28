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
![](./images/DSC_0010-6.jpg) |  ![](./images/IMG_3098.jpg)



Contact them with any questions at: hpcadmin (at) tulane.edu

***

#### Logging in to a Cypress login node

This will be slightly different for different operating systems.  Mac and Linux users will log in through their Terminal.  PC users will log in through putty. 

*Your username and password will be the same as that for your Tulane user account.*

You can walk your way through logging in by following [this Cypress page](https://wiki.hpc.tulane.edu/trac/wiki/Workshops/IntroToHpc2015/SshUsage).

**Short version:**

1. Windows:
	
	* Open putty
	* In the Host Name field type:

			cypress1.tulane.edu 
	
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
	
	#This bit allows Cypress admins to track module usage and manage shared installations accordingly.
	
	set curMod [module-info name]
	if { [ module-info mode load ] } {
    	system "/bin/logger -p local0.info 'module_load: $curMod.'"
	}
	
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
