# Docking antifolates against wild-type and resistant P. falciparum DHFR

I'm a B.Sc. Biotechnology student at Jamia Millia Islamia, New Delhi. I did this project to learn structure-based docking with open-source tools, after working with commercial software during a research internship. I wanted to find out whether free tools could reproduce a known result, and what happens when you check them properly.

The full analysis is in the notebook in this repository. It can be run from top to bottom in Google Colab and downloads everything it needs.

## Question

Can AutoDock Vina reproduce the crystal pose of a known inhibitor in P. falciparum dihydrofolate reductase, and does it show the difference in drug binding between the drug-sensitive and drug-resistant forms of the enzyme?

## Background

PfDHFR is the target of the antimalarials pyrimethamine and cycloguanil. Resistance arises from point mutations in the enzyme. The quadruple mutant (N51I, C59R, S108N, I164L) is resistant to both drugs, while the experimental compound WR99210 remains active against it. Two crystal structures make this comparison possible:

- **1J3I** — wild-type PfDHFR-TS with WR99210, NADPH and dUMP (2.33 Å)
- **1J3K** — quadruple mutant with the same ligands (2.10 Å)

Both structures come from the same study (Yuvaniyama et al., 2003), so they were solved by the same group under comparable conditions. That makes them more directly comparable than structures taken from different sources.

## Methods

- Receptors: chain A of each structure, keeping NADPH because the inhibitor binds next to it. Water, dUMP and the other chains were removed. Converted to PDBQT with Open Babel (rigid receptor, Gasteiger charges).
- Ligands: WR99210, pyrimethamine and cycloguanil built from SMILES in RDKit, 3D coordinates generated and minimised with MMFF.
- Ligand identity was checked against the RCSB Chemical Component Dictionary (WRA, CP6, 1CY) by molecular formula and canonical SMILES. My first WR99210 SMILES had two chlorines instead of three; this check caught it.
- Docking: AutoDock Vina 1.2.5, 22 Å cubic box centred on the crystal ligand position, exhaustiveness 16, fixed random seed.

## Validation

Before comparing anything, I tested whether the method can recover a known answer. I removed WR99210 from each structure and asked Vina to find it again, then measured the RMSD against the crystal pose. The usual threshold for success is 2.0 Å.

| Structure | Top-ranked pose | Best of 9 poses |
|---|---|---|
| 1J3I (wild type) | **1.95 Å** | 1.95 Å |
| 1J3K (mutant) | **3.81 Å** | 2.50 Å |

The wild type passes, but only just. **The mutant fails.**

I tried to work out why:

- **Was the correct pose found but ranked too low?** No. I measured all nine poses, not just the top one. The best was 2.50 Å, still outside the threshold.
- **Was the search not thorough enough?** No. Repeating at exhaustiveness 64 gave an identical result — same top pose, same score of −8.626, same 3.81 Å. The best of those nine was 2.45 Å.
- **Was the receptor file broken?** No. Both PDBQT files contain NADPH (48 records each) and have nearly identical atom counts (1895 and 1885).

So across 18 poses from two independent runs, the correct pose was never sampled in the mutant structure. I treat this as a real limitation of the protocol rather than a technical artefact.

Reading the two numbers together, I think the honest interpretation is that this ligand sits near the limit of what rigid-receptor docking reproduces reliably in either structure. WR99210 has a long flexible propoxy linker with many rotatable bonds, which makes it a hard case. The wild-type result was a marginal pass rather than a convincing one.

## Results

Best Vina score in kcal/mol:

| Ligand | Wild type (1J3I) | Mutant (1J3K) | Difference |
|---|---|---|---|
| Cycloguanil | −9.32 | −7.99 | +1.33 |
| Pyrimethamine | −8.83 | −7.85 | +0.98 |
| WR99210 | −8.41 | −8.63 | −0.22 |

Across five random seeds, standard deviations were 0.01–0.06 kcal/mol, so the search converges reproducibly. That shows the numbers are repeatable, not that they are accurate — the scoring function's own error is much larger than this.

## What I can and cannot conclude

The direction is right. Pyrimethamine and cycloguanil are predicted to bind the resistant mutant more weakly, while WR99210 is predicted to be largely unaffected. That matches what is known about these drugs.

But I cannot present this as a validated result, because the protocol does not reproduce the crystal pose in the mutant structure, and the comparison depends on those numbers. The trend may be right for the wrong reasons.

Two further things the scores cannot support:

1. **The size of the effect is wrong.** About 1 kcal/mol corresponds to roughly a five-fold change in affinity, far smaller than the loss of potency these drugs actually show against the quadruple mutant. Docking scores are not binding free energies.
2. **Ligands cannot be ranked against each other.** Vina scores cycloguanil as the strongest binder to the wild type, ahead of WR99210, which the experimental data does not support. The method may track how one ligand changes between two structures, but it should not be used to rank different compounds.

## Limitations

- Redocking failed in the mutant structure, so results from 1J3K are not supported by a validated protocol.
- One crystal structure per variant, with no estimate of structural uncertainty.
- Rigid receptor, so no induced fit. The resistance mutations alter side chains in the binding site, which is exactly where flexibility would matter.
- Receptor preparation used Open Babel, which assigns protonation states automatically; I did not check these.
- The search box was centred on the known ligand position, so this is not a blind docking.
- Only the DHFR domain of chain A was used; chain B was not tested.

## What I learned

The validation step is the part that made this project worth doing. Without it I would have had a table of six numbers that matched expectations and looked convincing. Checking whether the method could reproduce a known answer showed that it could not, in the structure the comparison depends on most. Learning to run that check, and to report it when it fails, mattered more to me than the result itself.

## Tools

AutoDock Vina 1.2.5, RDKit, Open Babel, gemmi, NumPy, pandas, Google Colab.

## References

Yuvaniyama J., Chitnumsub P., Kamchonwongpaisan S., Vanichtanankul J., Sirawaraporn W., Taylor P., Walkinshaw M.D., Yuthavong Y. Insights into antifolate resistance from malarial DHFR-TS structures. *Nature Structural Biology* (2003) **10**(5): 357–365. DOI 10.1038/nsb921

Eberhardt J., Santos-Martins D., Tillack A.F., Forli S. AutoDock Vina 1.2.0: New Docking Methods, Expanded Force Field, and Python Bindings. *Journal of Chemical Information and Modeling* (2021). DOI 10.1021/acs.jcim.1c00203

Trott O., Olson A.J. AutoDock Vina: improving the speed and accuracy of docking with a new scoring function, efficient optimization and multithreading. *Journal of Computational Chemistry* (2010). DOI 10.1002/jcc.21334

Structures and ligand definitions from the RCSB Protein Data Bank: 1J3I, 1J3K, and chemical components WRA, CP6, 1CY.
