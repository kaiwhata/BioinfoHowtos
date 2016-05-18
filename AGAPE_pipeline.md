#AGAPE

AGAPE has terrible installation instructions for their pipeline at (AGAPE on Github)[https://github.com/yeastgenome/AGAPE] which is also described in this paper: doi:10.1371/journal.pone.0129184
Given that many using it wil be first timers to Unix and the Command Line I thought I would make better installation instructions.

##Overview

I would suggest completing the pipeline install in the following order (because many of the later software depends on the earlier software:

1. bamtools
2. sga
3. lastz
4. augustus
5. NCBI BLAST
6. Boost
7. Chain NET
8. AGAPE
9. BWA
10. SAMtools

##Operating system
This guide is specifically written for installation on Ubuntu server 14.04.3. I chose this specifically because many of the programs have apt-get repositories already which vastly simplify the installation process. Some (but not many) of the software has RedHet yum equivalents, so if you're using a non-debian linux distribution be prepared for a lot of building from source code and the associated pain.

If you are new to linux I would HIGHLY recommend starting with Ubuntu, getting the pipeline working and having a play and then getting your HPC techician to help with installation on the cluster itself (which may well not be on a debian system).
Remember that, once installation is complete and sucessful, the command line commands will be the same regardless of the flavour of linux you are using. So all your hard work, testing and practice on Ubuntu will not be in vain!

##Installing SGA

`sudo apt-get install sparsehash`

`sudo apt-get install zlib1g-dev`

`sudo pip install ruffus`

`sudo pip install pysam`

(optional) `sudo apt-get install libjemalloc-dev`


###Installing bamtools

Yes there is an `apt-get` package for bamtools but it is (as of May 2016) bamtools.2.3.0 which is insufficuently recent for this pipeline hence why we're building from source.

`git clone https://www.github.com/pezmaster31/bamtools`

`sudo apt-get install cmake`

`mkdir build`

`cd build`

`cmake ..`

`make`

###Updating the bamtools lib files for Augustus (and others)

Must now copy bamtools.so to /usr/lib/
`sudo cp <path_to_bamtools_directory>/bamtools/lib/libbamtools.so /usr/lib/libbamtools.so`

`sudo mkdir /usr/include/bamtools`

`sudo cp -R shared /usr/include/bamtools/shared`

`sudo cp -R api /usr/include/bamtools/api`


##Installing NCBI BLAST command line tool

'sudo apt-get install ncbi-blast+'

##Installing Boost

`sudo apt-get install libboost-all-dev`

##Getting Chain NET
'wget http://hgwdev.cse.ucsc.edu/~kent/exe/linux/axtChainNet.zip'

##Installing Augustus
###Augustus Dependancies

You will need the following packages before installing Augusts

`sudo apt-get install python-numpy`

`sudo apt-get install libsqlite3-dev`

`sudo apt-get install liblpsolve55-dev`


These should already be installed but it pays to check they are:

`sudo apt-get install libgsl0-dev`

`sudo apt-get install libboost-graph-dev`

`sudo apt-get install libboost-iostreams-dev` 

`sudo apt-get install zlib1g-dev` 

###Augustus Itself
'wget http://bioinf.uni-greifswald.de/augustus/binaries/augustus-3.2.2.tar.gz'

'tar -xzf augustus-3.2.2.tar.gz'

`make`

`sudo make install`

##Installing ABySS

`sudo apt-get install abyss`

##Installing AGAPE

`git clone https://github.com/yeastgenome/AGAPE`

##Installing LASTZ

A copy of LASTZ is included in the AGAPE git repo. Simply move into the AGAPE/src/lastz/src directory and run:

`make`

`make install`

`make test`

You may also wish to edit the `$installDir` variable to a relevant location before running the commands above. You can do this by setting the variable in `nano make‑include.mak`.
Otherwise, by default, lastz will be created in a folder called `lastz-distib` in your home directory.

###Other versions of LASTZ

`wget http://www.bx.psu.edu/~rsharris/lastz/newer/lastz-1.03.73.tar.gz`

`tar -xzf lastz-1.03.73.tar.gz`

`cd last-1.03.73/src`

##Installing BWA

`sudo apt-get install bwa`

##Installing SAMtools

`sudo apt-get install samtools`

##To Do
- [ ] Install HugeSeq and dependancies.
- [ ] Install BOWTIE as a possible replacement fot LASTZ.
- [ ] Install velvet as a possible replacement for ABySS.
- [ ] Install FastQC (for some reason).
- [ ] Install FASTX-toolkit to replace SGA.
