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

###Actually installing sga

Then:
`cd src`

`./autogen.sh` 

`./configure --with-bamtools=/home/fogbank/Bioinfotools/bamtools`  

`make`

`make install`


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

add abyss subcommands to PATH

`nano ~/.bashrc`

`export PATH = /usr/lib/abyss:$PATH`

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

#Usage

must add header to SGA and ABySS files
`bin/edit_fq_head inputfile1.fastq 1 > inputfile1.sra.fastq` 
`sga preprocess --pe-mode 1 inputfile1.sra.fastq inputfile2.sra.fastq > output_paired_end.pp.fa`

possibly need to gunzip files before preprocessing??
`gunzip filename`

`sga preprocess --pe-mode 1 inputfile1.fastq.gz inputfile2.fastq.gz > output_paired_end.fastq`
takes about 128 seconds, 190 with unzipped files

`sga index -a ropebwt -t 8 --no-reverse output_paired_end.fastq`
takes about 553 seconds with 2 cores (-t 2)
takes about 286 seconds with 8 cores (-t 8)

`sga correct -k 41 -t 1 --discard --learn output_paired_end.pp.fa -o output_paired_end.pp.ec.fa`
takes about  8870 seconds with 1 core (-t 1)
takes about seconds with 8 cores (-t 8)

`abyss-pe aligner=map k=41 name=gNDA_paired_end in=gDNA_paired_end.pp.ec.fa`
Tag `np=8` can be used to create multiple processes. Can be distributed using openmpi
takes about __ seconds

`sga-align --name gDNA_paired_end.frag gDNA_paired_end-contigs.fa gDNA_paired_end.pp.ec.fa`
takes about 456 seconds

`sga-bam2de.pl -n 5 -m 100 -mina 95 -- prefix gDNA_paired_end.frag gDNA_paired_end.frag.bam`
takes a couple of minutes

`sga-astat.py -m 100 gDNA_paired_end.frag.refsort.bam > gDNA_paired_end.ctg.astat`
takes < 1 minute

`sga scaffold -m 100 --pe gDNA_paired_end.frag.de -a gDNA_paired_emd.ctg.astat -o gDNA_paired_emn.scaf` gDNA_paired_end-contigs.fa`
takes < 1 minute

###BWA commands

`bwa index ref_gen.fsa`
takes < 1 minute

At this point we decided to use the non SGA-error corrected reads as inputs. This should be checked or justified.
We also changed k from 2.
`bwm aln -q 15 -l 35 -k 2 -n 0.04 -o 2 -e 6 -t 8 ref_gen.fa gDNA_fromhERstrain_1.fastq > bwm_aligned_1.sai`
`bwm aln -q 15 -l 35 -k 2 -n 0.04 -o 2 -e 6 -t 8 ref_gen.fa gDNA_fromhERstrain_2.fastq > bwm_aligned_2.sai`

`bwa aln -q 15 -l 35 -k 2 -n 0.04 -o 2 -e 6 -t 8 ref_gen.fa  gDNA_paired_end.pp.ec.fa > bwm_aligned.sai`
takes about 1000 seconds across 8 cores

`bwa sampe ref_gen.fa bwn_aligned.sai > aln-pe.sam`
takes about  seconds 

##To Do
- [ ] Install HugeSeq and dependancies.
- [ ] Install BOWTIE as a possible replacement fot LASTZ.
- [ ] Install velvet as a possible replacement for ABySS.
- [ ] Install FastQC (for some reason).
- [ ] Install FASTX-toolkit to replace SGA.
- [ ] Maker install
