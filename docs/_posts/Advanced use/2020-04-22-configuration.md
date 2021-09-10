---
layout: post
title: "Using the configuration file system"
category: advanced-use
author: jr
short-description: How to access to all parameters
---

-----

*** still a draft ***

This article is complementary to the [--> command line use <--]({{ site.baseurl }}{% link _posts/Using CAVIAR/2020-04-23-caviar-cmdline.md %}#configuration-file) of CAVIAR.  
We describe here all the parameters that can be used in a configuration file with the -custom_config option.  

### Custom configuration file for CAVIAR  

The general architecture of the configuration file should follow the standard set by [--> configparse <--](https://docs.python.org/3/library/configparser.html#supported-ini-file-structure).  
The section name in squared brackets does not matter, as it is not parsed for the custom configuration file. However, one section head has to be present, with any title, see [--> this article <--]({{ site.baseurl }}{% link _posts/Using CAVIAR/2020-04-23-caviar-cmdline.md %}#configuration-file) for more details. The parameters can be written in the configuration file in any order.  
For the sake of the clarity of this article, we sorted the parameters in seven categories, plus at the end the general default configuration file:
 - [Input](#input)
 - [Output](#output)
 - [Selection of objects](#selection-of-objects)
 - [Identification of cavities](#identification-of-cavities)
 - [Filtering of cavities](#filtering-of-cavities)
 - [Ligand validation routines](#ligand-validation-routines)
 - [Segmentation into subcavities](#segmentation-into-subcavities)
 - [General default configuration file](#default-configuration-file)


#### Input ####

  - sourcedir: /home/user/pdb/  
Source directory of the input PDB files, otherwise downloads from the RCSB PDB  
  - code: 1aaa  
PDB Code (no default)  
  - codeslist: file.list  
File (with path, if not in current directory), containing PDB codes to be computed. This can contain information about the PDB code, the protein chains of interest and of a ligand, in case a ligand validation routine is turned on (cf after). One PDB per line. Example:  
1aaa_A LIG    
1bbb_AB LGI    
for investigating PDBs 1aaa and 1bbb, chain A for the first one and chains A and B for the second one. The explicit protein chains comes after a "\_" and the ligand 3-letters residue identifier comes after a space. In other words, "pdbcode_chainIDs lig3letters".     
  - v: False  
Turn verbosity on (boolean)  
  - onlyxr: False  
Only work with X-ray crystallography structures (boolean)  
  - resolution_filter: False  
Activate a resolution filter (boolean)  
  - resolution: 3.0  
Value for the resolution filter   
  - pdbversion_filter: False  
Define a filter on the PDB version (boolean)  
  - pdbversion: 3.30  
Minimal PDB version  
  - caveat: False  
Exclude PDB tagged with CAVEATS (PDB header tag, boolean)  
  - obsolete: False  
Exclude PDB tagged with OBSOLETE (PDB header tag, boolean)  
  - deposition_date_filter: False  
Exclude PDB deposited/reviewed before a date (boolean)  
  - date: 2010  
Minimal deposition/revision date (year, 4 digits)  

#### Output ####

  - out: ./caviar_out/  
Path to output folder, containing the PDB files of cavities  
  - export_cavities: True  
Export PDB files with cavities in "out" (boolean)  
  - withprot: True  
Export said PDB with the protein in addition to the cavities (boolean)  
  - write_pml_cavs: True  
Write a PyMOL session file \*.pml (boolean)   
  - color_cavs_by: bychain   
If a PyMOL session file is written, how to color the cavities: by chain, buriedness, or pharmacophore (values: bychain, buriedness, pharmacophore)  
  - print_cav_info: True  
Print out the table report on cavity identification (boolean)  
  - export_descriptors: False  
Turns on the export of Python pickled descriptors (global descriptors as pd.dataframe, networkx graph and 4-channels 3D-image) (boolean)  
  - asph: False  
Investigates the local asphericity around a grid point at 'radius_asph' cubic radius /!\ Time consuming! Placeholder as the 4th channel in the 3D image.  


#### Selection of objects ####

  - metal: True  
Keep metals and consider them part of the protein (boolean)  
  - water: True  
Keep waters molecules that make at least 3 HB with protein atoms (only with a distance criterion, boolean)    
  - structural_ligand: False  
Keep a structural ligand, such as a cofactor occupying part of the pocket ("False" or ligand 3-letters code)    
  - threshold_nres: 30   
Minimum number of residues in a protein chain to keep it. Peptide chains with less than this threshold will be ignored.    
  - what: allproteins    
Selection of protein chains to keep for cavity detection:  
All protein chains (value "allproteins"), just the longest protein chain (value "longestchain") or the longest chain plus chains in contact at 5A (value "longestandcontacting").    
  - min_contacts: 75  
In case of choosing "longestandcontacting" for the option "what", this controls what is the minimum number of contacts between the protein longest chain and the putative chains in contact to keep said contacting chains. These are interatomic contacts at 5A, so one be loose with the number. The aim is to keep only chains of the PDB that are functionally in contact rather than non productive crystal contacts.  
  - chain_id:  
Explicit user-specified chain ID to investigate (Nothing or a letter, or several letters, e.g. A or ABC). Overseeds "what", unless "what" is "longestandcontacting". 
  - chainid_in_pdblist: False  
Same as "chain_id" but implemented in the input list specified in "codeslist". The chain identifier (e.g. A, B...) should be given after an underscore to the PDB code. For example 1AAA_A for chain A of PDB 1AAA. 1AAA_ABC for chains A, B, and C.  
  - size_limit: 10000000  
Size in grid points of the grid box surrounding the protein. That is a kill switch: if the protein is too big and necessitates more than 10 million grid points, it is likely to crash on a normal computer. For example, the cryoEM structure 6zme (Nsp1 of covid + Ribosome) takes 24 million grid points and run out of memory on a 16gb RAM computer. 10 million grid points already amounts to about 12 gb of RAM.  


#### Identification of cavities ####

All distance values in Angstroms.  

  - boxmargin: 2.0  
The first step of the algorithm is to put a cubic box around the protein. This keyword controls the box margin around the protein.    
  - max_distance: 6.0  
Maximum distance between a solvent grid point and any protein atom. This is useful for computational efficiency. Since the box is cubic, in some directions, there are many spurious grid points that are far away from the protein. Prunes out box grid points that are further than the threshold.    
  - gridspace: 1.0  
Grid spacing for grid points.    
  - size_probe: 1.0  
Size of the probe for calculating the solvent accessible surface of the protein. Any grid point within this surface is then set as protein point. This size is added to the vdW radius from vdw_size_atoms.dat (not specified explicitly as a parameter for now but can be edited in the package source code directory).  
  - radius_cube: 4  
For solvent grid points, the buriedness is investigated in 14 cubic directions. This gives a buriedness score for said solvent grid point between 0 and 14. In each of the 14 directions, the presence of a protein grid point is investigated, within "radius_cube" cubic distances. For example, in the x direction, if "gridspace" is 1 and radius_cube 4, checks if there is a protein grid point within 4A in the x axis. The distance here is not in A but in number of grid points (1A is radiuscube = 1, 0.5A is radiuscube = 0.5).  
  - min_burial: 8  
Minimum number of grid-protein contacts for a grid point (within -radius_cube) to consider it as a potential cavity point. Between 0 and 14, since we scan in the  14 cubic directions.  
  - radius_cube_enc: 3  
Same as radius_cube, but for the second pass. This second pass aims to find 'middle' cavity points that are not in direct contact with the protein [within radius_cube] but surrounded by grid cavity points (middle of a large pocket). Does not investigate for contacts with protein grid points but with pre existing putative cavity grid points.  
  - min_burial_enc: 8  
Equivalent to min_burial but for the second pass (cf help of radius_cube_enc).  
  - min_points: 40  
Minimum number of putative cavity points in a cluster to consider it further. This is modified by the gridspace argument (real value = min_points * 1 / gridspace). If gridspace = 0.5, the real value used for min_points is thus doubled.  
  - trim_score: 500  
Score for excluding potential cavity points as probably overspanning. Points within 2 grid spacing (in cubic directions, 125 points max) are detected. The score equals the number of points times 10\*\*(average_buriedness/10). Buriedness ranges from 8 to 14, len(points) from 0 to 12. The maximum value of this score is 3139 (125\*(10\*\*(14/10)). The default is 500. This corresponds roughly to an environement of 50 neighbors (out of 125 maximum, half) and an avg buriedness of 10.  
  - min_degree: 3  
In the graph representation, each node is a grid point and edges are created between nodes in direct contact (1 grid spacing in one of the 14 cubic directions). Cavity grid points with a node degree below this threshold are excluded, again to avoid overspanning.  

#### Filtering of cavities ####

  - min_burial_q: 10  
Minimum buriedness value of grid points at the x-th quantile (strictly greater than) (again between 8 and 14).  
  - quantile: 0.8  
Quantile related to min_burial_q. 0.8 means the 8th quantile. If min_burial_q is 10 and quantile is 0.8, that means that a cavity needs to have at least 20% of its grid points with a buriedness of 11-12-13-14 to be considered. Avoids large surface exposed cavities without any buried anchoring point.   
  - max_hydrophobicity: 1.0  
Maximum percentage of hydrophobic points in the cavity. It could be interesting to filter out cavities that are entirely hydrophobic without any polar specific interaction possible. (value between 0 and 1).  
  - exclude_interchain: False   
Exclude all cavities that are in between different protein chains (boolean).  
  - exclude_missing: False  
Exclude cavities that have missing atoms/residues (boolean). 
  - exclude_altlocs: False  
Exclude cavities that have alternative conformation of residues (boolean).  


#### Ligand validation routines ####

  - check_if_lig: False  
Activates ligand check routines.  
  - lig_id:  
Ligand 3 letters ID code in the PDB file, to check for presence in cavities (Nothing or 3 letters code, in capital letters, ie LIG, not lig).  
  - liglist_in_pdblist: False    
Ligand 3 letters ID code is in the second in the second column of the input list file "codeslist", eg, "1dwc_H MIT" for checking the presence of the ligand MIT in any cavity of chain H of PDB 1dwc (boolean).
  - iflig_print: False  
Print what was found if -check_if_lig was activated. It is quite verbose but if it is not activated, the ligand validation routine will be quite useless (boolean).    
  - excl_ligs: True  
Activates an explicit the tabu list for the ligand to exclude ions, cosolvent... (boolean)    
  - lig_tabu_list: tabulist_ligand_maximal  
Explicit the tabu list for the ligand. Options are:  
"tabulist_ligand_minimal" for the minimal exclusion list of ligands (eg water, ions)
"tabulist_ligand_min_sugars", "tabulist_ligand_min_peptides", "tabulist_ligand_min_nucleic", respectively for minimal exclusion + sugars or peptids or nucleic  
"tabulist_ligand_min_peptides_sugars" "tabulist_ligand_min_peptides_nucleic", "tabulist_ligand_min_nucleic_sugars"  
"tabulist_ligand_min_peptides_nucleic_sugars" and "tabulist_ligand_maximal"    
  - ligsizeflag: False  
Flag to define a minimal size for the ligand (boolean).   
  - ligminsize: 8  
Minimal size for the ligand if ligsizeflag is activated (in heavy atoms).  
  - lig_tocenter: False
Check if any ligand atom is within 4A of the geometric center of the pocket rather than within 1A of any cavity point (boolean).   

#### Segmentation into subcavities ####

  - subcavs_decomp: True  
Activates the subcavities decomposition (boolean).   
  - subcavs_lig_only: False  
Find subcavities only for liganded cavities (boolean).   
  - export_subcavs: True
Export subcavities in a pdb file.   
  - write_pml_subcavs: True
Export a pml PyMOL session file to open output with predefined visualization.  
  - seeds_mindist: 3
Minimum distance between seed points in the watershed algorithm (in A).  
  - merge_subcavs: True
Merge small subcavities enclosed in between other subcavities to prevent oversegmentation (boolean).  
  - print_pphores_subcavs: True
prints pharmacophore data of the subcavities as a table.  


#### Default configuration file ####
<blockquote><p>
[input] <br>
 &nbsp;&nbsp;sourcedir: /db/pdb/ # Source directory, otherwise downloads file <br>
 &nbsp;&nbsp;code:            # PDB Code (no default) <br>
 &nbsp;&nbsp;codeslist:       # List of PDB codes to be computed (no default) <br>
 &nbsp;&nbsp;v: False         # Turn verbosity on <br>
 &nbsp;&nbsp;onlyxr: False    # Only work with XR structures  <br>
 &nbsp;&nbsp;resolution_filter: False # Defines a resolution filter  <br>
 &nbsp;&nbsp;resolution: 3.0  # Value for the resolution filter  <br>
 &nbsp;&nbsp;pdbversion_filter: False # Define a filter on the PDB version <br>
 &nbsp;&nbsp;pdbversion: 3.30 # Minimal PDB version <br>
 &nbsp;&nbsp;caveat: False    # Exclude PDB tagged with CAVEATS <br>
 &nbsp;&nbsp;obsolete: False  # Exclude PDB tagged with OBSOLETE <br>
 &nbsp;&nbsp;deposition_date_filter: False # Exclude PDB deposited/reviewed before a date <br>
 &nbsp;&nbsp;date: 2010       # Minimal deposition/revision date <br> <br>

[output] <br>
 &nbsp;&nbsp;out: ./caviar_out/ # Path/to/outfolder. <br>
 &nbsp;&nbsp;export_cavities: True # Export PDB files with cavities  <br>
 &nbsp;&nbsp;withprot: True   # Export it with the protein  <br>
 &nbsp;&nbsp;write_pml_cavs: True  # Write a pymol session file \*.pml  <br>
 &nbsp;&nbsp;color_cavs_by: bychain # and color cavities by chain/buriedness/pharmacophore ("bychain", "buriedness", "pharmacophore") <br>
 &nbsp;&nbsp;print_cav_info: True # print a report on cavity identification  <br> <br>

[selection] <br>
 &nbsp;&nbsp;metal: True      # Keep metals <br>
 &nbsp;&nbsp;water: True      # Keep waters molecules that make at least 3 HB with protein atoms <br>
 &nbsp;&nbsp;structural_ligand: False # Keep a structural ligand (False or ligand 3 letters code) <br>
 &nbsp;&nbsp;threshold_nres: 30 # Minimum number of residues in a protein chain to keep it  <br>
 &nbsp;&nbsp;what: allproteins # What protein chains to keep for cavity detection: <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# all protein chains (above threshold_nres), just the longest chain 'longestchain' <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# or the longest chain plus contacting chains at 5A 'longestandcontacting'  <br>
 &nbsp;&nbsp;min_contacts: 75 # In case you of longestandcontacting, controls how many contacts between the main chain  <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# and the 'contacting' chain should be at minimum present <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# These are interatomic  contacts at 5A, so be loose with the number <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# The aim is to keep only chains of the PDB that are functionally in contact <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# and not simply symmetric chains of the same domain in the crystal unit <br>
 &nbsp;&nbsp;chain_id:         # User-specified chain ID to investigate (e.g., A). Overseeds -what, unless it's "longestandcontacting" <br>
 &nbsp;&nbsp;chainid_in_pdblist: False # Same chain_id but implemented in input list  <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# rather than passed as explicit argument. Same warning as -chain_id. <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# The chain identifier (e.g. A, B...) should be given after an underscore to the <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# PDB code. For example 1AAA_A for chain A of PDB 1AAA. <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# In case you want to specify more than one chain, just put all of them, e.g., <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# 1AAA_ABC for chains A, B, and C. <br> <br>

 &nbsp;&nbsp;size_limit: 10000000      # Size in grid points of the grid box surrounding the protein. That is a kill switch: <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;#if the protein is too big and necessitates more than 10 million grid points, it is <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;#likely to crash on a normal computer. For example, the cryoEM structure 6zme (Nsp1 <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# of covid + Ribosome) takes 24 million grid points and run out of memory on a 16gb RAM <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;#computer. 10 million grid points already amounts to about 12 gb of RAM. <br> <br>

[cavity_identification] <br>
 &nbsp;&nbsp;boxmargin: 2.0   # Margin around the protein  <br>
 &nbsp;&nbsp;max_distance: 6.0 # Maximum distance for a solvent grid point to the protein  <br>
 &nbsp;&nbsp;gridspace: 1.0   # Grid spacing  <br>
 &nbsp;&nbsp;size_probe: 1.0  # Size of the probe for defining protein points. This size is added to the vdW radius from vdw_size_atoms.dat <br>
 &nbsp;&nbsp;radius_cube: 4   # Size of the cubic solvation shell to investigate burial of cavity points (in number of grid points) <br>
 &nbsp;&nbsp;min_burial: 8    # Minimum number of grid-protein contacts for a grid point (within -radius_cube) to consider it as potential cavity point <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# This number is between 0 and 14 because we scan in the  14 cubic directions <br>
 &nbsp;&nbsp;radius_cube_enc: 3 # Same as radius_cube, but for the second passto identify buried cavity points <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# This second pass aims to find 'middle' cavity pointsthat are not in direct contact with the protein <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# [within radius_cube] but surrounded by grid cavity points (middle of a large pocket) <br>
 &nbsp;&nbsp;min_burial_enc: 8 # Equivalent to min_burial but for the second pass (cf help of radius_cube_enc)  <br>
 &nbsp;&nbsp;min_points: 40   # Minimum number of points to consider a group of cavity points as an actual cavity <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; # Is modified by gridspace argument (real value = min_points * 1 / gridspace) <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; # If gridspace = 0.5, the real value used for min_points is doubled <br>
 &nbsp;&nbsp;trim_score: 500  # Scoring value for excluding potential cavity points <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# Points within 2 grid spacing (in cubic directions, 125 points max) are detected <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# The score equals the number of points times 10\*\*(average_buriedness/10) <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# Buriedness ranges from 8 to 14, len(points) from 0 to 12  <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# The maximum value of this score is 3139 (125\*(10\*\*(14/10)). The default is 500  <br>
 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# This corresponds roughly to an environement of 50 neighbors (out of 125 maximum, half) and an avg buriedness of 10 <br>
 &nbsp;&nbsp;min_degree: 3    # Minimum node degree to keep it, ie, minimum number of  connections with other nodes <br> <br>

[cavity_filtering] <br>
 &nbsp;&nbsp;min_burial_q: 10 # Minimum buriedness value of grid points at the xth quantile (strictly greater than) [parameter -quantile] <br>
 &nbsp;&nbsp;quantile: 0.8    # Quantile related to min_burial_q  <br>
 &nbsp;&nbsp;max_hydrophobicity: 1.0 # Maximum percentage of hydrophobic points in the cavity.  <br>
 &nbsp;&nbsp;exclude_interchain: False # Exclude all cavities that are in between different protein chains.  <br>
 &nbsp;&nbsp;exclude_missing: False # Exclude cavities that have missing atoms/residues.  <br>
 &nbsp;&nbsp;exclude_altlocs: False # Exclude cavities that have alternative conformation of residues.  <br> <br>

[ligand_check] <br>
  &nbsp;&nbsp;check_if_lig: False # Activates ligand check routines. 
  &nbsp;&nbsp;iflig_print: False # Print what was found if -check_if_lig was activated.  <br>
  &nbsp;&nbsp;excl_ligs: True  # Activate an explicit the tabu list for the ligand.  <br>
  &nbsp;&nbsp;lig_tabu_list: tabulist_ligand_maximal # Explicit the tabu list for the ligand <br>
  &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# Other options include: tabulist_ligand_minimal for the minimal exclusion list of ligands (eg water, ions) <br>
  &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# tabulist_ligand_min_sugars, tabulist_ligand_min_peptides, tabulist_ligand_min_nucleic, respectively for minimal exclusion + sugars or peptids or nucleic <br>
  &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# tabulist_ligand_min_peptides_sugars tabulist_ligand_min_peptides_nucleic, tabulist_ligand_min_nucleic_sugars <br>
  &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# tabulist_ligand_min_peptides_nucleic_sugars and tabulist_ligand_maximal <br>
  &nbsp;&nbsp;ligsizeflag: False # Flag to define a minimal size for the ligand.  <br>
  &nbsp;&nbsp;ligminsize: 8    # Minimal size for the ligand if ligsizeflag is activated.  <br>
  &nbsp;&nbsp;lig_id:          # Ligand 3 letters ID code in the PDB file, to check for presence in cavities (no default) <br>
  &nbsp;&nbsp;liglist_in_pdblist: False # Specify ligand 3 letters ID code in the second in the second column <br>
  &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# of the PDB list file (to check for presence in cavities), eg, "1dwc_H MIT" "PDB_chainID LIG" <br>
  &nbsp;&nbsp;lig_tocenter: False # Check if a ligand atom is within 4A of the geometric center of the pocket  <br>
  &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;# rather than within 1A of any cavity point  <br> <br>

[subcavity_routines] <br>
 &nbsp;&nbsp;subcavs_decomp: True # Activates the subcavities decomposition  <br>
 &nbsp;&nbsp;subcavs_lig_only: False # Find subcavities only for liganded cavities  <br>
 &nbsp;&nbsp;export_subcavs: True # Export subcavities in pdb file  <br>
 &nbsp;&nbsp;write_pml_subcavs: True # Export a pml pymol session file to open output <br>
 &nbsp;&nbsp;seeds_mindist: 3 # Minimum distance between seed points in the watershed algorithm  <br>
 &nbsp;&nbsp;merge_subcavs: True # Merge small subcavities enclosed in between other subcavities (prevent oversegmentation)  <br>
 &nbsp;&nbsp;print_pphores_subcavs: True # prints pharmacophore data of the subcavities. <br> <br>

</p>
</blockquote>
