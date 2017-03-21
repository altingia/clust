# Clust
Optimised consensus clustering of multiple heterogenous datasets
### Contents
* [What does *Clust* do?](#what-does-clust-do)
  * [In the language of bioinformatics, again, what does it do?](#in-the-language-of-bioinformatics-again-what-does-it-do)
* [How does *Clust* do it?](#how-does-clust-do-it)
* [Simplest usage](#simplest-usage)
  * [The results (output) directory](#the-results-output-directory)
  * [The tightness parameter](#the-tightness-parameter)
  * [Data files](#data-files)
* [Next level usage](#next-level-usage-replicates-normalisation-and-id-maps-eg-orthologues)
  * [Replicates file](#replicates-file)
  * [Normalisation file](#normalisation-file)
  * [Map file (e.g. for datasets from multiple species)](#map-file-eg-for-datasets-from-multiple-species)
* [Advanced usage](#advanced-usage)
  * [Filter out objects with low values](#filter-out-objects-with-low-values)
  * [Objects missed from some datasets](#objects-missed-from-some-datasets)
* [List of all parameters](#list-of-all-parameters)
* [Example datasets](#example-datasets)
  * [For simplest usage](#for-simplest-usage)
  * [For next level usage](#for-next-level-usage)
  * [For advanced usage](#for-advanced-usage)
* [Install *Clust*](#install-clust)

# What does Clust do?
*Clust* is a fully automated method for identification of clusters (groups) of objects that are well-correlated with each in one or more datasets. For example clustering gene expression data accross multiple time series or multiple species.

![Clusters](Images/Clusters.png)

*Figure 1: Clust processeses independent quantiative datasets (X0, X1, X2 ... Xn) to identifiy clusters of objects with coordinate behaviour in each of the iunput datasets (C0, C1, C2 ... Cn). The left-hand panel shows the profiles of all objects in each of the input datasets, while the right-hand panel shows the profiles of the objects within each cluster. Note that the number of conditions or time points are different for each dataset.

**Features!**

1. No need to pre-process your data.
2. No need to preset the number of clusters; the algorithm finds this number automatically.
3. The algorithm discards objects that do not fit into any cluster.
4. You can control the tightness of the clusters simply by varying a parameter, which has a default value if you wish not to set it!
5. You can include heterogeneous datasets (e.g. gene expression datasets from different technologies, different species, different numbers of conditions, etc).
5. The package calculates key statistics and provides them in the output.
6. A table of clusters' members is provided in an output TSV file.
7. A figure showing the profiles of the generated clusters is provided as an output PDF file.

---

### In the language of bioinformatics, again, what does it do?
If you analyse a set of gene expression datasets, Clust identifies the clusters (subsets) of genes that are
consistently co-expressed in each one of the datasets in an automatic and accurate manner. Pre-processing
(e.g. data normalisation) and post-processing (e.g. false-positive and false-negative error correction) are performed
automatically by *Clust* and require minimal intervention by the user.

*Clust* generates the following output files:

1. A table of clustering statistics (e.g. number of generated clusters, average, minimum, and maximum
cluster size, total number of genes included in the experiment, etc.)
2. A table listing genes included in each cluster
3. Pre-processed (normalised, summarised, and filtered) datasets' files
4. Plotted gene expression profiles of clusters (a PDF file)

> *It is okay if these datasets:*
>
>1. **Were generated by different technologies** (RNA-seq, one-colour microarrays, or two-colour microarrays)
>2. **Are from different species.** But in this case you need to provide a simple orthology mapping file, which can be
straightforwardly generated for any set of species by [OrthoFinder](https://github.com/davidemms/OrthoFinder) 
(See [Next level usage/Map file](#map-file-eg-for-datasets-from-multiple-species) below)
>3. **Have different numbers of conditions or time points**
>4. **Have multiple replicates for the same condition**
(See [Next level usage/Replicates file](#replicates-file) below)
>5. **Require different types of normalisation**
(See [Next level usage/Normalisation file](#normalisation-file) below)
>6. **Were generated in different years and laboratories**
>7. **Have some missing values**
>8. **Do not include every single gene in every single dataset**, i.e. some genes are totally not represented in some
datasets, for example because these datasets were generated by older microarray chips which did not have probes for
these particular genes (See [Advances usage/Objects missed from some datasets](#objects-missed-from-some-datasets))


# How does Clust do it?
![Clust workflow](Images/Workflow_PyPkg.png)

*Figure 2: Automatic Clust analysis pipeline*

# Simplest usage
- `clust data_path`
- `clust data_path -o output_directory [...]`
- `clust data_path -t tightness [...]`

This applies the *Clust* pipeline over the datasets with files included in the data_path
directory with default parameters.

### The results (output) directory
The `-o` argument specifies the path of the results (output)
directory. If it is not provided, *Clust* creates a directory Results_[Date] in the current working
directory.

### The tightness parameter
It defines how tight the clusters should be (tighter and smaller clusters versus less tight and larger clusters).
This is a real positive number with the default value of **1.0**. Values smaller than 1.0 (e.g. 0.5) produce less
tight clusters, while values larger than 1.0 (e.g. 2.0, 5.0, 10.0, ...) produce tighter clusters.

### Data files
Each dataset is represented in a single TAB delimited (TSV) file in which the first column represents the identifiers
(IDs) of the objects (e.g. gene names), the first row represents unique labels of the samples (e.g. conditions or time
points), and the rest of the file includes numerical values of these objects at those samples. Figure 3 shows a screen 
shot of the first few lines of 3 datasets' files.
![Data_simple](Images/Data_simple.png)

*Figure 3: Snapshots of three data files X0, X1, and X2.*

* When the same object ID appears in different datasets, it is considered to refer to the same object. For example, the
object ID O01 in the dataset X0 is considered to refer to the same object as O01 in the datasets X1 and X2.
* If more than one row in the same file had the same identifier, they are automatically summarised by summing up their values.

# Next level usage (replicates, normalisation, and ID maps (e.g. orthologues))
- `clust data_path  -r replicates_file -n normalisation_file -m map_file [...]`
- `clust data_path  -r replicates_file -n normalisation_file -m map_file -t tightness [...]`

Consider the gene expression datasets shown in Figure 4:

![Level2_Data](Images/Datasets_level2.png)

*Figure 4: Snapshots of three gene expression datasets from two yeast species, X0 and X1 from fission yeast
(Schizosaccharomyces pombe) and X2 from budding yeast (Saccharomyces cerevisiae).*

These are three datasets from two species, two from fission yeast and one from budding yeast. There are few
issues in these datasets:

1. **Replicates:** There are multiple replicates for the same conditions. For example, X0 has 18 columns (samples) of
data, but every three of them represent replicates for a single condition. One does not want *Clust* to consider 
these 18 columns as 18 independent conditions; rather one wants *Clust* to summarise the replicates of each
condition to form a single column. This is done by providing the Replicates file (file format is below).
2. **Normalisation:** Different datasets require different normalisation procedures to be suitable for cluster analysis.
X0 and X1 for example have log2 values while X2 does not. By providing a simple normalisation file, one can direct the
Bi-CoPaM towards proper pre-processing of data. (file format is below).
3. **Mapping object IDs:** Fission yeast genes are not the same as budding yeast genes and have different gene ID.
Therefore, it is important to provide the algorithm with a file that maps fission yeast gene IDs to their orthologous
budding yeast gene IDs. Such mapping is required whenever the datasets objects' IDs are not mutual. For example, if
one dataset uses Entrez gene IDs while the other dataset uses gene names, a mapping file is required to inform the
algorithm that this or that Entrez gene ID refers to this or that gene name. (file format is below).

### Replicates file
![ReplicatesFile](Images/ReplicatesFile.png)

*Figure 5: Replicates file defining the replicates. The right-hand file is the same as the left-hand file but while
indicating that the Z condition of X1.txt should be ignored in the analysis, as well as the time points 4 and 5 of
X2.txt. Starting a line with the hash character (#) indicates that it should be ignored.*

Each row of this file lists the replicates of a single condition or time point.

Each line includes these elements in order:

1. The name of the dataset file (e.g. X0.txt)
2. A name for the condition of time-point; this can be any label that the user chooses
3. One or more names of the replicates of this condition. These should match column names in the dataset file (e.g.
if the dataset file is X0.txt, these names must all be names of columns in X0.txt)

* Delimiters between these elements can be spaces, TABs, commas, or semicolons.

### Normalisation file
![NormalisationFile](Images/NormalisationFile.png)

*Figure 7: Normalisation file indicating the types of normalisation that should be applied to each of the datasets.*

If datasets need normalisation, this file indicates what techniques of normalisation should be applied to them.

Each line includes these elements in order:

1. The name of the dataset file (e.g. X0.txt)
2. One or more normalisation techniques' codes (see below). **The order** of these codes defines the order of the
application of their respective normalisation techniques.

* Delimiters between these elements can be spaces, TABs, commas, or semicolons.

#### Codes suggested for commonly used datasets

* RNA-seq gene expression data: **101 3 4**
* Log2 RNA-seq gene expression data: **101 4**
* One-colour microarray gene expression data: **101 3 4**
* Log2 one-colour microarray gene expression data: **101 4**
* Two-colour microarray gene expression data: **3 6**
* Log2 two-colour microarray gene expression data: **6**

#### All codes

Code | Definition
|:---:|:---|
0|No normalisation
1|Divide by the mean value of the row
2|Divide by the first value of the row
3|Log2
4|Subtract the mean of the row and then divide by its standard deviation
5|Divide by the total (sum) of the row
6|Subtract the mean value of the row
7|Divide by the maximum value of the row
8|2 to the power X
9|Subtract the minimum value of the row
10|Rank across rows (1 for the lowest, up to *N* for *N* columns; average ranks at ties)
11|Rank across rows (1 for the lowest, up to *N* for *N* columns; order arbitrarly at ties)
12|Linear transformation to the [0, 1] range across rows (0.0 for the lowest and 1.0 for the highest)
-|-
101|Quantile normalisation
102|Column-wise mean subtraction
103|Subtract the global mean of the entire dataset

### Map file (e.g. for datasets from multiple species)
![MapFile](Images/MapFile.png)

*Figure 7: TAB delimited file which maps fission and budding yeast genes, i.e. defines orthologues across the
two species.*

The first column shows the orthologue/object group (OG) IDs, the second column shows the budding
yeast gene IDs which belong to these OG IDs while the third column shows the fission yeast gene IDs which
belong to these IDs. If more than one gene within the same species are orthologues, i.e. belong to the same OG
ID, they are included next to that OG ID in the column which belongs to that species while being separated by
commas; for example, the fission yeast genes 2541313 and 2543377 are orthologues to each other and are both
orthologues to the budding yeast gene 855993; these gene IDs are all included in the OG ID 29; see the sixth
line in Figure 7. In this case, both fission yeast genes 2541313 and 2543377 in the datasets of fission yeast
are summarised by summing them up. In other words, the algorithm considers one value for the OG ID 29 at each
sample in each dataset, and whenever this OG is represented by multiple rows in the same dataset, they are
summed up to a single row.


# Advanced usage

To be written

### Filter out objects with low values

To be written

### Objects missed from some datasets

To be written

# List of all parameters
Parameter | Definition
--- | ---
data_directory | The path of the directory including all data files
-|-
-m map_file | Path of the map file
-r replicates_file | Path of the replicates file
-n normalisation_file | Path of the normalisation file
-o output_directory | Custom path of the output directory
-t tightness | (Cluster tightness) versus (cluster size) weight: a real positive number, where 1.0 means equal weights, values smaller than 1.0 means larger and less tight clusters, and values larger than 1.0 produce smaller and tighter clusters (default: 1.0).
-s standard_deviations | Number of standard deviations that define an outlier (default: 3.0)
-d #datasets | Minimum number of datasets in which an object has to be included for it to be considered in the *Clust* analysis. If an object is included only in fewer datasets than this, it will be excluded from the analysis (default: 1)
-fil-v FILV | Threshold of data values (e.g. gene expression). Any value lower than this will be set to 0.0. If an object never exceeds this value at least in FILC conditions in at least FILD datasets, it is excluded from the analysis (default: -inf)
-fil-c FILC | Minimum number of conditions in a dataset in which an object should exceed the data value FILV at least in FILD datasets to be included in the analysis (default: 0)
-fil-d FILD | Minimum number of datasets in which an object should exceed the data value FILV at least in FILC conditions to be included in the analysis (default: 0)
-cs CS | Smallest cluster size (default: 11)
-K k0 [k1 ...] | K values: refer to the publication for details (default: all values from 2 to 20 inclusively)
-h, --help | show the help message and exit

# Example datasets
### For simplest usage

Example datasets are available in [ExampleData/1_Simplest](ExampleData/1_Simplest),
or more specifically in the [Data](ExampleData/1_Simplest/Data) directory therein.

Simply run this command over the directory "Data" by:

* `clust Data/`

Find the results in the Results_[Date] directory that *Clust*
will have generated in your current working directory.

This runs *Clust* with the default tightness `-t` value of 1.0.
You may like to make the generated clusters tighter by increase `-t` or less tight
by decreasing `-t`. For example, try -t = 5.0 or -t = 0.2 by: 

* `clust Data/ -t 5`
* `clust Data/ -t 0.2`

You may also like to save results in an output directory of your choice by using `-o`:

* `clust Data/ -t 5 -o MyResultsDirectory/`

### For next level usage

Example datasets are available in [ExampleData/2_Next_Level](ExampleData/2_Next_level). These
are three datasets from two yeast species, two datasets from fission yeast, and one from budding
yeast.

That directory contains the datasets' files in a [Data](ExampleData/2_Next_level/Data) sub-directory,
and includes three other files specifying the [replicates](ExampleData/2_Next_level/Replicates.txt),
the required [normalisation](ExampleData/2_Next_level/Normalisation.txt), and the object
[mapping](ExampleData/2_Next_level/MapIDs.txt) across the datasets,
i.e. orthologous genes across the two yeast species.

Run *Clust* over this data by:

* `clust Data/ -r Replicates.txt -n Normalisation.txt -m MapIDs.txt`

You may like to specify a tightness level `-t` other than the default by adding:

* `... -t 5`

You may also specify an output directory other than the default by adding:

* `... -o MyResultsDirectory/`

### For advanced usage

To be written

# Install *Clust*

### Required Python packages

*Clust* is a Python package, which requires Python 2.7 or newer and depends on these Python packages:
* numpy
* scipy
* matplotlib
* sklearn
* sompy

### If you have privileges to install (sudo privileges)

If `matplotlib` is not already installed, run this:

* `sudo apt-get install python-matplotlib`

Then install *Clust* by:

* `sudo -H pip install Clust`

You can after than run *Clust* straightforwardly from any place:

* `clust ...`

### If you do not have privileges to install

This works if the five Python packages that *Clust* requires are already installed (listed above).

Download the source code (tar.gz) file from the
[release tab](https://github.com/BaselAbujamous/clust/releases)
and run the script `clust.py` that is in the top level directory of the source code by:

* `python clust.py ...`

