We have constructed a Linux (Ubuntu v15.10 Desktop) Virtual Box image that can be used to run the Microbiome Helper pipeline. This Virtual Box image can be run on any operating system (Windows, Mac, Linux) and saves the time consuming task of installing and configuring multiple bioinformatic packages. 

## Requirements
* Any desktop or laptop purchased within the last 3-5 years should work.
* Must be 64-bit
* >4GB RAM
* >25GB available hard drive space

## Installation
1. Download and install VirtualBox for the operating system you are using. https://www.virtualbox.org/wiki/Downloads
1. Download the latest version of [Microbiome Helper Vbox (v0.2.4)](https://www.dropbox.com/s/szb7y418unqjeel/MicrobiomeHelper_v0.2.4.ova?dl=1) (12.5 GB - OVA file). 
1. "Import" the Microbiome Helper Vbox into VirtualBox by opening the OVA file or within VirtualBox `File->Import Appliance`. More detailed and graphical instructions for importing the image are [available here]. (https://www.maketecheasier.com/import-export-ova-files-in-virtualbox) 

## Notes
* During the import step (or later in settings) you can change the default amount of RAM and number of CPU available for use by the VirtualBox. This will make computation time faster.
* If you get a "VT-x/AMD-V hardware acceleration" error when booting for the first time, you might be able to fix it by [following the steps here](http://www.itworld.com/article/2981515/virtualization/virtualbox-diagnose-and-fix-vt-xamd-v-hardware-acceleration-errors.html).
* The default username and password are both "mh_user".
* If you'd like to use USEARCH v6.1 for chimera removal (with chimera_filter_usearch61.pl), rather than VSEARCH, you will need to get a license to use this program and put the binary called "usearch61" in your PATH. 
