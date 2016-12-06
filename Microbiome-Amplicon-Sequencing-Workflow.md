The following detailed protocol is for the generation of paired-end sequencing reads of 16S or 18S PCR amplicons with multiple barcodes (i.e.: “indices”) on the _Illumina MiSeq_ machine of length ≈450 bp (300+300 bp with ~150 bp overlap) using v3 chemistry. It assumes an input of 380 individual samples (+4 controls = 384) in four 96-well plates (384-well plates are discouraged). Catalog numbers are given when items from specific vendors (vs. generic choices) are required.  

_Note that this workflow is continually being updated. If you want a specific version for printing (including the current one), select one of the PDFs below._
    
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

2. Quantify and quality-check your final DNAs via _NanoDrop_ or _Qubit_ to verify success. The A280/260 ratios should be 1.8 or better and our experience with community DNAs has shown that a concentration of at least 1 ng/µL is required to get consistent PCR results. Avoid costly secondary extractions/clean-ups for inhibitors (unless obvious) until PCRs below have truly shown inhibition – many sub-optimal DNAs will still perform adequately in PCR. _Optional: A gel can be run to verify integrity (generally unnecessary for PCR-only studies)._

3. Aliquot 5 µL of each of the 380 DNA samples into 4 × 96-well PCR plates (**DNA Plates 1-4**) in the order desired (we prefer by column), allowing for one PCR negative control well (position H12) on each plate; the layout of the primers (for ex: showing F1+R2 sets are used for Plate 2) are also indicated on this figure:
[[images/PlateLayout.jpg]]

## 2.0 – 16S or 18S PCR (1 day)

1. Prepare the following PCR master-mix for Plate 1 in a 1.5 mL Eppendorf tube (adjust if not using _Phusion_):[[images/PCRmastermix.jpg]]

2. Dispense 78.5 µL of the master-mix into the 16 wells of 2 columns (or 105 µL in 12 wells of one row) of a 96-well plate (remaining wells to be used in subsequent PCR preps) – this plate now becomes the **Master-Mix Plate** and is used to transfer the master-mix into the PCR plate using an MCP.

3. Dispense 13 µL of master-mix into each well of the **PCR Plate 1 (2 µL)**, one column (or row) at a time with the MCP.

4. Remove the protective film (“uncover”) from one **column** of the **Forward Set 1 Primer Plate**, align it horizontally on the bench to the left of **PCR Plate 1 (2 µL)** and dispense 5 µL into each well, one **column** at a time using the MCP. _Note: You can use the same set of 8 tips for all._

5. Uncover one **row** of the **Reverse Set 1 Primer Plate**, align it vertically on the bench along the top of **PCR Plate 1 (2 µL)** and dispense 5 µL into each well, one **row** at a time using the MCP. _Note: You must now **change tips** after every row to avoid cross-contamination (since different F primers/indices are now in each row)._

6. Uncover the **DNA Plate 1**, align it along the top of **PCR Plate 1 (2 µL)** and dispense 2 µL into each well, one **column** at a time using the MCP. _Note: Remember to **change tips** after every column._

7. Once complete, seal the plate with PCR film, place in a thermocycler and run the following program (~1.5 h; adjust if not using _Phusion_):[[images/PCRthermalprofile.jpg]]

8. While the first PCR is running, prepare the 1/10th dilution of **DNA Plate 1** – add 27 µL of PCR-grade water to each well of the remaining 3 µL of template in **DNA Plate 1**, for a total of 30 µL final volume per well, using a reservoir (~3 mL required) and MCP. _Note: Remember to **change tips** after every column._

9. Once the first PCR is complete (or nearly so), repeat **steps 1-7** to prepare **PCR Plate 1 (0.2 µL)** using the newly diluted templates. _Optional: Here, and for the subsequent plates below, once the two PCRs of the same plate are complete you can proceed to **steps 3.1-3.4** and verify the PCR products before continuing with the next plate each time._

10. Once the two PCRs for **Plate 1** are complete, repeat **steps 1-9** to prepare **PCR Plates 2 (2 µL) & (0.2 µL)** from **DNA Plate 2** using **Forward Set 1 Primer Plate** and **Reverse Set 2 Primer Plate** (_note change to F1+R2 here_).

11. Once the two PCRs for **Plate 2** are complete, repeat **steps 1-9** to prepare **PCR Plates 3 (2 µL) & (0.2 µL)** from **DNA Plate 3** using **Forward Set 2 Primer Plate** and **Reverse Set 1 Primer Plate** (_note change to F2+R1 here_).

12. Once the two PCRs for **Plate 3** are complete, repeat **steps 1-9** to prepare **PCR Plates 4 (2 µL) & (0.2 µL)** from **DNA Plate 4** using **Forward Set 2 Primer Plate** and **Reverse Set 2 Primer Plate** (_note change to F2+R2 here_).

***

## 3.0 - Gel Verification (2 h)

1. Plug in the _Mother E-Base_, unwrap a fresh _E-Gel 96_ and insert it into the base.

2. The duplicate PCR reactions of **Plate 1** are aggregated then loaded onto the gel in the same action: using the MCP and working by **rows** (the gel cannot be loaded by columns as they are staggered), pipette 20 µL out of the **PCR Plate 1 (0.2 µL)** into the corresponding wells of **PCR Plate 1 (2 µL)** and mix by pipetting up and down, then take 20 µL of this aggregate and load into the appropriate wells of the gel. _Note: Remember to **change tips** after every row. Discard the empty **PCR Plate 1 (0.2 µL)** when finished and relabel the **PCR Plate 1 (2 µL)** the **Aggregated PCR Plate 1**._

3. Once all rows are complete, load 20 µL of the _E-Gel Low Range Ladder_ into some of the marker (“M”) wells, then run the gel for the pre-set 12 min.

4. Visualize the gel and photograph on a UV/blue transilluminator with a SYBR filter. Example gels of 16S and 18S amplicons from this protocol are included in the PDF version of this document for reference.

5. Repeat **steps 1-4** for **PCR Plates 2 (2 µL) & (0.2 µL)**.

6. Repeat **steps 1-4** for **PCR Plates 3 (2 µL) & (0.2 µL)**.

7. Repeat **steps 1-4** for **PCR Plates 4 (2 µL) & (0.2 µL)**.

8. Any samples with failed PCRs (or spurious bands) are re-amplified by optimizing the PCR (further template dilution to 1:100 or using BSA/other additives) to produce correct bands in order to complete the amplicon plate. Unless this represents the majority of a plate (in which case continue with plates and E-gels), PCRs are done in standard tubes/strips and visualized using a traditional gel box. Once correct bands have been obtained, amalgamate those few tubes into the appropriate wells of the respective **Aggregated PCR Plates** before continuing.

***

## 4.0 - PCR Clean-up + Normalization & Final Library Pool (2 h)

1. Use the remaining 20 µL of each well in the **Aggregated PCR Plate 1** to cleaned-up and normalize the amplicons using the high-throughput _Invitrogen SequalPrep 96-well Plate Kit_. Label this final plate **SequalPrep Plate 1**. _Note: We have found it more convenient to inverse steps 1 & 2 of the SequalPrep protocol (i.e.: buffer first with one set of tips, then amplicons with another set). Only one set of tips (per step) is need in this kit as carry-over is minimal, amplicons are barcoded (therefore can’t “contaminate” each other’s reads) and amplicons will shortly be pooled anyways._

2. Once the _SequalPrep_ protocol is complete, pool the 95 samples from **SequalPrep Plate 1** by using the MCP to transfer 5 µL of each column into one column of a new 96-well plate named the **Library Pool Plate** (remaining columns to be used in subsequent pooling). _Note: You can use the same set of 8 tips for all._ Once complete, pipette 50 µL of each of the 8 wells into one 1.5 mL Eppendorf tube and label **Plate 1 Library Pool**.

3. Repeat **steps 1-2** for **Aggregated PCR Plate 2**.

4. Repeat **steps 1-2** for **Aggregated PCR Plate 3**.

5. Repeat **steps 1-2** for **Aggregated PCR Plate 4**.

6. Once all four pools are complete, pipette 100 µL of each of the four tubes into one 1.5 mL Eppendorf tube and label Final Library Pool (add the run name to the tube or some other identifier to keep your various pools separate).

7. Quantify the Final Library Pool using the Invitrogen Qubit dsDNA HS assay (or similar fluorescence-based alternative; 5 µL of pool to be assayed) and calculate the molar concentration using the following formula, knowing that 1 ng/µL of a 500 bp amplicon = 3.29 nM:
[[images/Equation.jpg]]
For the 16S amplicon generated using our V6-V8 primers (= 574 bp, including target region + adaptors + indices) at a concentration of 3.0 ng/µL, for example:
[[images/EquationExample.jpg]]
_Note: We have found that the anticipated 1-2 ng/µL output from the SequalPrep plate is near impossible to achieve; we typically see concentrations in the range of 0.3-0.9 ng/µL._

## 5.0 - _Illumina MiSeq_ Sequencing (~3 days [only ~2 h hands-on at the start and end])

This section is based upon the following Illumina documents, with some small procedural changes (including using the NextSeq variant for sample denaturation), and the inclusion of instructions to be able to load >96 samples (i.e.: 384 combinations of indices) which are not written out by Illumina – familiarize yourself with these documents / have them on-hand:
*_MiSeq Reagent Kit v3 – Reagent Preparation Guide_
*_Preparing Libraries for Sequencing on the MiSeq_
*_Denaturing and Diluting Libraries for the NextSeq 500_
*_MiSeq System User Guide_

1. 