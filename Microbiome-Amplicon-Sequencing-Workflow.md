The following detailed protocol is for the generation of paired-end sequencing reads of 16S or 18S PCR amplicons with multiple barcodes (i.e.: “indices”) on the _Illumina MiSeq_ machine of length ≈450 bp (300+300 bp with ~150 bp overlap) using v3 chemistry. It assumes an input of 380 individual samples (+4 controls = 384) in four 96-well plates (384-well plates are discouraged). Catalog numbers are given when items from specific vendors (vs. generic choices) are required.  

_Note that this workflow is continually being updated. If you want a specific version for printing (including the current one), select one of the PDFs._
    
_Last updated: 06 Dec 2016 (see "revisions" above for earlier versions)_

***

## Required Equipment

* Single-channel pipettes (2-1000 µL)
* Multi-channel (8) pipette(s) (MCP; 2-100 µL)
* Required equipment for the DNA extraction kit(s) of your choice
* PCR machine with 96-well block
* Invitrogen Mother E-Base (#EB-M03) for E-Gels 96
* Documentation system for DNA gels (SYBR Safe filter)
* Microvolume DNA fluorescence reader (such as Invitrogen Qubit)
* Illumina Experiment Manager (iEM) software
* Illumina MiSeq sequencer with RTA v1.17.28 / MCS v2.2 or later

## Optional/Case-Dependent Equipment
* Bead-mill / TissueLyser / Homogenizer (if using bead-based DNA extractions)
* Microvolume DNA spectrophotometer (such as NanoDrop; for quantifying extracts)
* Centrifuge with rotor for 96-well plates (useful for spinning down condensation)
* Standard gel electrophoresis system (for analyzing recalcitrant samples)

## Required Reagents & Consumables
* Pipette filter tips (p2 to p1000)
* Reagent reservoirs for MCP
* 1.5 mL Eppendorf tubes
* DNA extraction kit(s)
* 96-well thin-wall 0.2 mL PCR plates (such as Bio-Rad #HSP9601)
* PCR plate films (such as Bio-Rad #MSB1001)
* Thermo Phusion High-Fidelity DNA Polymerase (#F-530L) or similar
* dNTP mix (at 40 mM = 10 mM of each base)
* Illumina fusion primers (see below and Excel template)
* PCR-grade water
* Invitrogen E-Gels 96 2% with SYBR Safe (#G7208-02)
* Invitrogen E-Gel Low Range Ladder (#12373-031; diluted 1:1 with PCR-grade water)
* Invitrogen SequalPrep Normalization Kit (#A10510-01)
* Invitrogen Qubit dsDNA HS reagent and assay tubes (clear 0.2 mL)
* 2 N NaOH
* 200 mM Tris-HCl, pH 7
* Illumina PhiX Control Kit v3 (#FC-110-3001)
* Illumina MiSeq Reagent Kit v3 (600 cycle) (#MS-102-3003)

## Optional/Case-Dependent Reagents & Consumables
* Ethanol (usually a requirement for extraction/purification kits)
* Mammalian blocking primer (see viii below; if substantial host contamination in 18S) 
* Thin-wall 0.2 mL PCR tubes or strips with caps (for re-amplifying recalcitrant samples)
* Agarose, loading buffer/stain and 100 bp ladder (for analyzing recalcitrant samples)
* PCR product purification kit (if need to concentrate final library)

## Advance Preparation – Order and Aliquot Primers (1 h)

i. Use our Excel template to copy existing 16S/18S/ITS primers or to design your own custom gene primers with the proper Illumina indices and Nextera adaptor orientations. We order IDT “Ultramers” (www.idtdna.com) for such long primers (~80-90 nt) as their coupling efficiency is one of the highest available (critical for obtaining high proportions of full-length oligos in the mix you obtain). Order the fusion primers at 4 nmole scale in deep-well plates; one set per 96-well plate, arranged as follows, leaving blank rows in between sets:

        mkdir fastqc_out
        fastqc -t 4 raw_data/* -o fastqc_out/

ii. Stich paired end reads together (summary of stitching results are written to "pear_summary_log.txt")

        run_pear.pl -p 4 -o stitched_reads raw_data/* 

iii. Filter stitched reads by quality score (at least Q30 over at least 90% of the read), length (at least 200 bp) and ensure forward and reverse primers match 100% each read (summary written to "read_filter_log.txt" by default). If you do not wish to force primer matching, then you must remove the -f/-r/-c options below. 

        read_filter.pl -f CYGCGGTAATTCCAGCTC -r CRAAGAYGATYAGATACCRT -c both --thread 4 -q 30 -p 90 -l 200 stitched_reads/*.assembled.*
									
iv. Convert FASTQ stitched files to FASTA AND remove any sequences that have an 'N' in them.

        run_fastq_to_fasta.pl -p -o fasta_files filtered_reads/*fastq

v. Remove chimeric sequences with VSEARCH (summary written to "chimera_filter_log.txt" by default).

        chimera_filter.pl -thread 4 -type 1 -db /home/shared/rRNA_db/Eukaryota_SILVA_123_SSURef_Nr99_tax_silva_U-replaced.fasta fasta_files/*	

vi. Create a QIIME "map.txt" file with the first column containing the sample names and another column called "FileInput" containing the filenames. This is a tab-delimited file and there must be columns named "BarcodeSequence" and "LinkerPrimerSequence" that are empy. This file can then contain other columns to group samples which will be used when figures are created later.

        create_qiime_map.pl non_chimeras/* > map.txt
		
vii. Combine files into single QIIME "seqs.fna" file (~5 minutes).

        add_qiime_labels.py -i non_chimeras/ -m map.txt -c FileInput -o combined_fasta
		
viii. Create OTU picking parameter file.

        echo "pick_otus:threads 4" >> clustering_params.txt
        echo "pick_otus:sortmerna_coverage 0.8" >> clustering_params.txt
        echo "pick_otus:similarity 0.98" >> clustering_params.txt
        echo "align_seqs:template_fp /home/shared/rRNA_db/90_Silva_111_rep_set_euk_aligned.filter.fasta" >> clustering_params.txt 
        echo "assign_taxonomy:id_to_taxonomy_fp /home/shared/rRNA_db/gb203_pr2_all_10_28_99p_tax_Xs-fixed_poly-fixed.txt" >> clustering_params.txt
        echo "assign_taxonomy:reference_seqs_fp /home/shared/rRNA_db/gb203_pr2_all_10_28_99p_clean.fasta" >> clustering_params.txt
        
