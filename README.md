# Northern White-Cheeked Gibbon (*Nomascus leucogenys*) LD-Based Recombination Map

**Status**: Unpublished dataset. Shared here with permission from Jeff Wall, as part of the data/analysis accompanying the Gibbon Genome Consortium, led by Lucia Carbone. This was collaborative with the Hammer Lab, led by Dr. August Woerner originally as a goal to be part of his PhD dissertation. 

## 1. Overview

This dataset is a genome-wide, fine-scale, linkage-disequilibrium (LD)-based recombination map for the northern white-cheeked gibbon (*Nomascus leucogenys*, NLE), built from 8 whole-genome-sequenced individuals. It was never formally published as a
standalone paper; the methodology directly follows the pipeline used in Stevison, Woerner, Kidd, Kelley, Veeramah, McManus, Great Ape Genome Project, Bustamante, Hammer & Wall (2016), *"The Time Scale of Recombination Rate Evolution in Great Apes,"*
*Molecular Biology and Evolution* 33(4):928–945 (large overlap in lab/collaborator groups, same analytical pipeline, applied here to gibbon rather than great-ape samples).

**Why this exists / why it was never finished**: the gibbon map was built as part of a collaboration to investigate X-chromosome-vs-autosome (X/A) recombination and diversity broadly across apes. Because the X vs. A comparison was the whole point of the exercise, both an **autosomal** map (completed in 2016) and a **chrX** map (attempted, never completed) were needed. The collaboration ultimately did not produce a finished X/A analysis for gibbon: a genotyping-QC problem in the autosomal data was discovered too late to correct, and  multiple trainees had career-stage transitions, before the comparison was completed, leading to an incomplete product. 

This repository contains a per-chromosome, fine-scale population recombination rate (ρ/kb) map across the 25 gibbon autosomes. These files were used for a synthetic review of primate recombination rate in 2026 to calculate genome-wide summary statistics (weighted mean ρ/kb, Gini coefficient of rate unevenness, and — via an externally sourced effective population size — cM/Mb). See #5 and #6 below.

## 2. Sample information

Genotypes are called for **8 *N. leucogenys* individuals**, identified in the VCF as:

| VCF sample ID | Individual name | Sex | WGS coverage | Source study (SRA) | Notes |
|---|---|---|---|---|---|
| D16_NLE_Asia | Asia | Female | 19.4× (Okhovat et al. 2020) / 16.7× (Wang et al. 2026 realignment) | Okhovat et al. 2020 (SRS5353014) | Reference genome individual for the Nleu1.0/Nleu3.0 assembly (Carbone et al. 2014) — studbook no. 0098, ISIS no. NLL605, Virginia Zoo, Norfolk, VA. Population-scale short-read WGS (this sample) was generated later, separately from the original reference-assembly sequencing effort. |
| D3_NLE_Vok | Vok | Male | 15.2× (Okhovat 2020) / 15.4× (Wang 2026) | **Carbone et al. 2014** (SRS496874) | One of the two non-reference NLE individuals in Carbone et al. (2014)'s original cross-genus comparative panel (§8.1); also the subject of that paper's 116× high-coverage exome validation (Table ST8.1).  |
| D4_NLE_Asteriks | Asteriks (spelled "Asterisks" in Okhovat 2020's own supplementary tables) | Female | 20.4× (Okhovat 2020) / 13.0× (Wang 2026) | **Carbone et al. 2014** (SRS634434) | The *other* of the two comparative-panel individuals from Carbone et al. (2014)  |
| D17_NLE_Johannes | Johannes | Male | 19.7× (Okhovat 2020) / 12.0× (Wang 2026) | Okhovat et al. 2020 (SRS5353029) | Not part of Carbone et al. (2014)'s original comparative panel — additional, project-specific resequencing. |
| D15_NLE_Khao | Khao | Male | 20.2× (Okhovat 2020) / 19.5× (Wang 2026) | Okhovat et al. 2020 (SRS5353017) |  |
| D18_NLE_Nancy | Nancy | Female | 14.2× (Okhovat 2020) / 15.0× (Wang 2026) | Okhovat et al. 2020 (SRS5353019) | Additional, project-specific resequencing (as Johannes, above). |
| D19_NLE_Phyllis | Phyllis | Female | 11.1× (Okhovat 2020) / 11.2× (Wang 2026) | Okhovat et al. 2020 (SRS5353020) | Additional, project-specific resequencing. Lowest coverage of the 8. |
| D20_NLE_China | China | Female | 14.3× (Okhovat 2020) / 9.7× (Wang 2026) | Okhovat et al. 2020 (SRS5353016) | Additional, project-specific resequencing. |

All 8 NLE samples were provided by Lucia Carbone and re-mapped to Nleu3.0 specifically for this project. 

## 3. Data files in this directory

| File / folder | Description |
|---|---|
| `LDhat_maps/` | **Final deliverables**: 25 gzipped per-chromosome files (`chrN-map.txt.gz`), each a fine-scale ρ/kb map with columns `chr, start_Mb, end_Mb, midpoint_Mb, rho_per_kb, L95, U95` (95% confidence bounds). Chromosome names (`chr1a, chr2–chr6, chr7b, chr8–chr21, chr22a, chr23–chr25`) reflect *N. leucogenys*-specific karyotype fissions relative to the human reference (2n=52 vs. human 2n=46) — see Carbone et al. 2014 for the synteny-breakpoint mapping. 

## 4. Methodology

The pipeline follows Stevison et al. (2016) directly (see that paper's Methods for the full published description; summarized here as reconstructed from the old scripts):

1. **Variant calling and genotype QC** (encoded in the VCF filename): joint genotyping across the 8 individuals in GATK's "emit all sites" mode (originally run 2013 by Hammer Lab, U. Arizona: `ProjectNLE.emitAll.vqsr.vcf.gz`), filtered by VQSR (Variant Quality Score Recalibration) to `justPassedSites`, followed by removal of anomalous/outlier calls (`noWackies`) targeting a genome-wide density of roughly 4.5 million SNPs.

2. **Haplotype phasing** (`scripts/Gibbon.phase.sh`, `scripts/phase.qsub_body.sh`, `scripts/chrX.phase_blocks.sh` see [Stevison 2016 GitHub](https://github.com/lstevison/great-ape-recombination)): each chromosome's variants are split into overlapping blocks (~400 SNPs per block based on the comments in `Gibbon.phase.sh`), converted to PHASE input format (`vcf2PHASE.pl` / `vcf2PHASE_X.pl`, see Stevison 2016 GitHub for scripts), and phased with **PHASE v2.1.1** (Stephens, Smith & Donnelly 2001; Stephens & Scheet 2005) run as an SGE array job per block.
   
3. **Rejoining phased blocks** (`scripts/cleanup_phase.sh`): per-chromosome reassembly of the block-wise PHASE output (`join_phase_blocks.pl`, see Stevison 2016 GitHub for scripts).
   
4. **Re-phased VCF + LDhat input construction** (`scripts/post_phase_qsub_body.sh`): phased haplotypes are written back to VCF (`PHASE2VCF.pl`), split into "synteny blocks" (`CreateChrBedFromVCF.pl`), and converted to **LDhat**
   (McVean, Awadalla & Fearnhead 2002; Auton & McVean 2007) input format via `vcftools --ldhat`.
    
5. **LDhat interval analysis**: this is the step that actually produces the ρ/kb estimates in `LDhat_maps/`.

## 5. Calculations

- **No effective population size (Ne) estimate accompanies this dataset directly.**
  Converting ρ/kb to cM/Mb (ρ = 4·Ne·r) required borrowing an Ne from elsewhere. Now using **Ne = 57,000**, the present-day, NLE-specific estimate from Veeramah, Woerner, Johnstone et al. (2015, *Genetics* 200:295–308 — an ABC analysis), 95% CI 35,667–91,000.

## 6. Use in this project

Computed here for the primate recombination-rate comparison review:
- Genome-wide physical-length-weighted mean **ρ/kb = 3.5335**
- Raw/unbinned **Gini coefficient = 0.8441** (directly comparable to the Stevison et al. 2016 great-ape Gini values, 0.81–0.82, computed with the identical method)
- **cM/Mb = 1.5498**, using Ne = 57,000, the present-day NLE-specific estimate from Veeramah et al. (2015)
  
## 7. Citation

No standalone publication exists for this dataset. Cite as:

> Stevison, L.S. unpublished. Northern white-cheeked gibbon (*Nomascus leucogenys*) LD-based recombination map. DOI: 10.5281/zenodo.22131713. Methodology as described in Stevison et al. (2016), *Mol. Biol. Evol.* 33(4):928–945.

Alternatively, this will be included as part of an upcoming book chapter on non-human primate recombination rate. Cite review as:

> Stevison, L.S. 2026. Recombination Rate Heterogeneity Across Non-Human Primates. In _Evolutionary Genomics of Non-Human Primates_ Ed. S. P. Pfiefer and J. D. Jensen. Elsevier.

Sample/assembly background (not recombination-data sources):

> Carbone, L. et al. (2014). Gibbon genome and the fast karyotype evolution of small apes. *Nature* 513, 195–201. https://doi.org/10.1038/nature13679

> Okhovat, M., Nevonen, K.A., Davis, B.A., Michener, P., Ward, S., Milhaven, M., Harshman, L., Sohota, A., Fernandes, J.D., Salama, S.R., O'Neill, R.J., Ahituv, N., Veeramah, K.R. & Carbone, L. (2020). Co-option of the lineage-specific LAVA
> retrotransposon in the gibbon genome. *Proc. Natl. Acad. Sci. USA* 117(32), 19328–19338. https://doi.org/10.1073/pnas.2006038117 (source of WGS sample sex/coverage/SRA data for 6 of the 8 individuals in this VCF — Dataset S1).

> Wang, S., Chen, Z., Luo, A. et al., Fan, P., Fu, Q. & Wu, D.-D. (2026). Genome sequences of extant and extinct gibbons reveal their phylogeny, demographic history, and conservation status. *Cell* 189, 34–51.
> https://doi.org/10.1016/j.cell.2025.10.016 (independent cross-check of sample sex/coverage/SRA data — Table S3).

> Veeramah, K.R., Woerner, A.E., Johnstone, L., Gut, I., Gut, M., Marques-Bonet, T., Carbone, L., Wall, J.D. & Hammer, M.F. (2015). Examining phylogenetic relationships among gibbon genera using whole genome sequence data using an
> approximate Bayesian computation approach. *Genetics* 200(1), 295–308. https://doi.org/10.1534/genetics.115.174425 (source of the present-day, NLE-specific Ne = 57,000 used for the cM/Mb conversion above).
