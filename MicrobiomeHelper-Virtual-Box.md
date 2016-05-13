We have constructed a Linux (Ubuntu v15.10 Desktop) Virtual Box image that can be used to run the Microbiome Helper pipeline. This Virtual Box image can be run on any operating system (Windows, Mac, Linux) and saves the time consuming task of installing and configuring multiple bioinformatic packages. 

## Requirements
* Any desktop or laptop purchased within the last 3-5 years should work.
* Must be 64-bit
* >4GB RAM
* >25GB available hard drive space

## Installation
1. Download and install VirtualBox for the operating system you are using. https://www.virtualbox.org/wiki/Downloads
1. Download the latest version of [Microbiome Helper Vbox (v0.2.3)](https://www.dropbox.com/s/b3pozsczxjn356l/MicrobiomeHelper_v0.2.3.ova?dl=1) (11.7 GB - OVA file). 
1. "Import" the Microbiome Helper Vbox into VirtualBox by opening the OVA file or within VirtualBox `File->Import Appliance`. More detailed and graphical instructions for importing the image are [here] (https://www.maketecheasier.com/import-export-ova-files-in-virtualbox) 

## Notes
* You can change the amount of RAM and # of processors used by the VirtualBox if you have a more powerful computer.
* If you get a "VT-x/AMD-V hardware acceleration" error when booting for the first time, you might be able to fix it by [following the steps here](http://www.itworld.com/article/2981515/virtualization/virtualbox-diagnose-and-fix-vt-xamd-v-hardware-acceleration-errors.html).


Please email gavin[dot]douglas[at]dal[dot]ca if you have any questions or issues!