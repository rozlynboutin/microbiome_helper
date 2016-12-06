The following detailed protocol is for the generation of paired-end sequencing reads of 16S or 18S PCR amplicons with multiple barcodes (i.e.: “indices”) on the _Illumina MiSeq_ machine of length ≈450 bp (300+300 bp with ~150 bp overlap) using v3 chemistry. It assumes an input of 380 individual samples (+4 controls = 384) in four 96-well plates (384-well plates are discouraged). Catalog numbers are given when items from specific vendors (vs. generic choices) are required.  

_Note that this workflow is continually being updated. If you want a specific version for printing (including the current one), select one of the PDFs._
    
_Last updated: 06 Dec 2016 (see "revisions" above for earlier versions)_

***

## Required Equipment

* Single-channel pipettes (2-1000 µL)
* Multi-channel (8) pipette(s) (MCP; 2-100 µL)
* Required equipment for the DNA extraction kit(s) of your choice
* PCR machine with 96-well block
* _Invitrogen Mother E-Base_ (#EB-M03) for _E-Gels 96_
* Documentation system for DNA gels (SYBR Safe filter)
* Microvolume DNA fluorescence reader (such as _Invitrogen Qubit_)
* _Illumina Experiment Manager (iEM)_ software
* _Illumina MiSeq_ sequencer with _RTA v1.17.28 / MCS v2.2_ or later

## Optional/Case-Dependent Equipment
* Bead-mill / TissueLyser / Homogenizer (if using bead-based DNA extractions)
* Microvolume DNA spectrophotometer (such as _NanoDrop_; for quantifying extracts)
* Centrifuge with rotor for 96-well plates (useful for spinning down condensation)
* Standard gel electrophoresis system (for analyzing recalcitrant samples)

## Required Reagents & Consumables
* Pipette filter tips (p2 to p1000)
* Reagent reservoirs for MCP
* 1.5 mL Eppendorf tubes
* DNA extraction kit(s)
* 96-well thin-wall 0.2 mL PCR plates (such as _Bio-Rad_ #HSP9601)
* PCR plate films (such as _Bio-Rad_ #MSB1001)
* _Thermo Phusion_ High-Fidelity DNA Polymerase (#F-530L) or similar
* dNTP mix (at 40 mM = 10 mM of each base)
* _Illumina_ fusion primers (see below and Excel template)
* PCR-grade water
* _Invitrogen E-Gels 96 2% with SYBR Safe_ (#G7208-02)
* _Invitrogen E-Gel Low Range Ladder_ (#12373-031; diluted 1:1 with PCR-grade water)
* _Invitrogen SequalPrep Normalization Kit_ (#A10510-01)
* _Invitrogen Qubit dsDNA HS_ reagent and assay tubes (clear 0.2 mL)
* 2 N NaOH
* 200 mM Tris-HCl, pH 7
* _Illumina PhiX Control Kit v3_ (#FC-110-3001)
* _Illumina MiSeq Reagent Kit v3_ (600 cycle) (#MS-102-3003)

## Optional/Case-Dependent Reagents & Consumables
* Ethanol (usually a requirement for extraction/purification kits)
* Mammalian blocking primer (see **H.** below; if substantial host contamination in 18S) 
* Thin-wall 0.2 mL PCR tubes or strips with caps (for re-amplifying recalcitrant samples)
* Agarose, loading buffer/stain and 100 bp ladder (for analyzing recalcitrant samples)
* PCR product purification kit (if need to concentrate final library)

***

## Advance Preparation – Order and Aliquot Primers (1 h)

A. Use our Excel template to copy existing 16S/18S/ITS primers or to design your own custom gene primers with the proper _Illumina_ indices and _Nextera_ adapter orientations. We order _IDT “Ultramers”_ (www.idtdna.com) for such long primers (~80-90 nt) as their coupling efficiency is one of the highest available (critical for obtaining high proportions of full-length oligos in the mix you obtain). Order the fusion primers at 4 nmole scale in deep-well plates; one set per 96-well plate, arranged as follows, leaving blank rows in between sets:

[[images/PrimerLayout.jpg]]

B. Once arrived, add 400 µL of PCR-grade water to each well containing the primers in order to reconstitute them at a concentration of 10 µM (1/10th the typical 100 µM working stock concentration for primers). We have found that these usually need a significant incubation time for the lyophilized pellets to re-suspend – we typically leave them overnight at 4°C.

C. Prepare the 1 µM working stock **Forward Set 1 Primer Plate** by pipetting 63 µL of PCR-grade water into each well of the 96-well PCR plate from a sterile reservoir. Rotate the deep-well primer plate 90° clockwise and align it so that the 8 occupied wells (= 8 different indices) of **row 1** line up with the 8 rows of the new plate. Working by **column** and keeping the same set of tips, transfer 7 µL of reconstituted primer into each well of each column, mixing well by pipetting. Once complete, each column of the resulting plate will have enough primer for one complete 96-well plate PCR (leaving some extra for pipetting error; 12 columns × 5 µL = 60 µL required). Seal the plate with PCR film and store at -20°C. 

D. Prepare the 1 µM working stock **Forward Set 2 Primer Plate** by repeating step **C**, but using **row 3** of the reconstituted deep-well primer plate.

E. Prepare the 1 µM working stock **Reverse Set 1 Primer Plate** by pipetting 45 µL of PCR-grade water into each well of the 96-well PCR plate from a sterile reservoir. Align the deep-well primer plate horizontally (normal orientation) so that the 12 occupied wells (= 12 different indices) of **row 5** line up with the 12 columns of the new plate. Working by **row** and keeping the same set of tips, transfer 5 µL of reconstituted primer into each well of each row, mixing well by pipetting. Once complete, each row of the resulting plate will have enough primer for one complete 96-well plate PCR (leaving some extra; 8 rows × 5 µL = 40 µL required). Seal the plate with PCR film and store at -20°C.

F. Prepare the 1 µM working stock **Reverse Set 2 Primer Plate** by repeating step **E**, but using **row 7** of the reconstituted deep-well primer plate.

G. Once all aliquoting is complete, seal the deep-well plate with PCR film and archive at -20°C until new aliquots are required (minimized freeze-thaw cycles).

H. _Optional: For the generation of 18S V4 amplicons from microbiome samples containing substantial non-target host DNA (ex: human, mouse, etc.), order a custom PNA mammalian blocking primer (elongation arrest in the V4 region courtesy of Laura Parfrey and Matt Lemay [UBC]) with the sequence: 5’-TCTTAATCATGGCCTCAGTT-3’. Once arrived, prepare an archival stock of 100 µM and a working stock of 10 µM using PCR-grade water._

***

## 1.0 – DNA Extraction (1 day)

1. Extract DNAs from your samples using the method/kit appropriate to the specific samples (ex: generally stool [with bead-beating kits], but also urine, etc.). For example, we have had good success with the _MO-BIO PowerFecal Kit_ for mouse pellets and human stool. _Note: Do not overload the kit columns with excessive amounts of sample material as this reduces extraction efficiencies and can allow contaminants/inhibitors to co-purify with the DNAs._