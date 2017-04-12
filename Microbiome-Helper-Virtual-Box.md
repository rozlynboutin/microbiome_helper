Our Linux (Ubuntu v16.04 LTS Desktop) Virtual Box image that can be used to run all Microbiome Helper workflows and tutorials. This Virtual Box image can be run on any operating system (Windows, Mac, Linux) and saves the time consuming task of installing and configuring multiple bioinformatic packages. If you use any tools packaged with this image be sure to cite the tool's authors. 

## Includes

The Microbiome Helper Virtual Box includes several [bioinformatic packages and databases](https://github.com/mlangill/microbiome_helper/wiki/requirements) used in our standard operating procedures and tutorials. The image also includes other tools for microbiome analyses such as for [sequencing error correction](https://github.com/mlangill/microbiome_helper/wiki/Sequence-Error-correction) or [predicting phenotypes](https://github.com/mlangill/microbiome_helper/wiki/Predict-phenotypes). Please cite all of the tools that you use for your analyses. 
  
## Requirements
* Any desktop or laptop purchased within the last 3-5 years should work.
* Must be 64-bit
* More than 4 GB RAM
* At least 40 GB available hard drive space: the OVA file is 11 GB and the imported image is over 20 GB (note that you can delete the OVA file once the image is imported).

## Installation
1. Download and install VirtualBox for the operating system you are using from here: https://www.virtualbox.org/wiki/Downloads
2. Download the latest version of [Microbiome Helper Vbox (v1.1.0)](http://kronos.pharmacology.dal.ca/public_files/Microbiome_Helper_Vbox/MicrobiomeHelper_v1.1.0.ova) (11 GB - OVA file). If that doesn't work, try the [alternate link](https://www.dropbox.com/s/elhz61bundti5cb/MicrobiomeHelper_v1.1.0.ova?dl=1) (check out the [CHANGELOG](http://kronos.pharmacology.dal.ca/public_files/Microbiome_Helper_Vbox/CHANGELOG) for details).
3. "Import" the Microbiome Helper Vbox into VirtualBox by opening the OVA file or within VirtualBox `File->Import Appliance`. More detailed and graphical instructions for importing the image are [available here](https://www.maketecheasier.com/import-export-ova-files-in-virtualbox). You can delete the OVA file once it is finished importing to save space!

## Optional Configuration
* During the import step (or later in settings) you can change the default amount of RAM and number of CPU available for use by the VirtualBox.
    * If your computer has 8GB of RAM or more, we would recommend configuring the virtual image with RAM: 4096MB. 
    * If your computer has 2 or more cores you can configure the virtual image with CPU == 2 (or more).

## Moving files to the VBox image
* To move files in and out of the VBox image you can create a "shared folder":
    * Settings->Shared Folders (Note this is in the VirtualBox GUI on your local PC) 
    * Click on the little "+" sign on the folder on the right.
    * Choose a "Folder Path" and "Folder Name"
    * Check the "Auto-Mount" and "Make Permanent" boxes
    * Click OK (twice) and restart the virtual image if it was already running. 
* The shared folder will be at `/media/sf_<folder name>`

## Other Notes
* The default username and password are both "mh_user".
* If you'd like to use USEARCH v6.1 for chimera removal (with chimera_filter_usearch61.pl), rather than VSEARCH, you will need to get a license to use this program and put the binary called "usearch61" in your PATH. 

# Known issues
* If you get a "VT-x/AMD-V hardware acceleration" error when booting for the first time, you might be able to fix it by [following the steps here](http://www.itworld.com/article/2981515/virtualization/virtualbox-diagnose-and-fix-vt-xamd-v-hardware-acceleration-errors.html).