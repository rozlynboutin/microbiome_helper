Run pre-HUMAnN (DIAMOND search).

    run_pre_humann.pl -p 4 -o pre_humann/ screened_reads/*

Run HUMAnN (link files to HUMAnN "input" directory and then run HUMAnN with scons command). Note that you can run this in parallel with `-j` option (e.g. scons -j 4), but I have found this often causes HUMAnN to unexpectedly error.

    ln -s $PWD/pre_humann/* ~/programs/humann-0.99/input/
    cd ~/programs/humann-0.99/
    scons

Convert HUMAnN output to STAMP format

    humann_to_stamp.pl 04b-hit-keg-mpm-cop-nul-nve-nve.txt > hummann_modules.spf
    humann_to_stamp.pl 04b-hit-keg-mpt-cop-nul-nve-nve.txt > hummann_pathways.spf
    humann_to_stamp.pl 01b-hit-keg-cat.txt > hummann_kos.spf
