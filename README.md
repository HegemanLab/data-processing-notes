# Data Processing Notes

These notes are for LC-MS data processing in the Cohen-Hegeman Labs.


## Resources

  - [Galaxy Training's Mass Spectrometry: LC-MS Analysis](https://galaxyproject.github.io/training-material/topics/metabolomics/tutorials/lcms/tutorial.html)
    - This guide closely follows W4M's GTN guide and is highly recommended reading.
  - [LCMS data preprocessing and analysis with XCMS](https://www.bioconductor.org/packages/devel/bioc/vignettes/XCMS/inst/doc/XCMS.html)
  - [W4M's HowTo Guides](https://workflow4metabolomics.org/howto)
  - W4M Publications
    - [Guitton et al. 2017](http://dx.doi.org/10.1016/j.biocel.2017.07.002)
    - [Giacomoni et al. 2014](http://bioinformatics.oxfordjournals.org/content/early/2014/12/18/bioinformatics.btu813)
  - XCMS Publications
    - [Smith et al. 2006](https://pubs.acs.org/doi/abs/10.1021/ac051437y)


## Experimental Design

### Conditioning, Blanks, and Pools

By including conditioning, blank, and pool injections an experiments design can improve consistency and create data for statistical quality control.

At the beginning of a batch injection, columns should be conditioned by loading them with a pool sample. Run conditioning injections until intensities between injections becomes consistent.

Running injections of blank solvent througout the run provides information about how the machine is running and can be used to remove contaiminate features. Blank data can be used to normalzing intensity drift within a batch.

> Note: Contaiminate peaks often come from dirty instruments and overloading the column.

Injecting pools provide information for subtracting peaks which are present in multiple samples which are not statistically important. Pool data can also be used to normalzing intensity drift within a batch.

See [Guidelines and considerations for the use of system suitability and quality control samples in mass spectrometry assays applied in untargeted clinical metabolomic studies](https://doi.org/10.1007/s11306-018-1367-3) by Broadhurst et al. for detailed experimental design recommendations.

### File Naming

Most of our data processing tools have specific file naming conventions.

Proper file naming conventions are required for by XCMS:
  - Filenames cannot begin with numbers or symbols
  - Filenames cannot include dashes `-`
    -  Underscores `_` are allowed, if possible, avoid symbols
  - Avoid spaces in filenames

We suggest the following format:
```
{initials}{experiment ID}_{injection type}_{sample type}_{sample type number}
```

e.g.:
```
meArab2019_conditioning_1
meArab2019_conditioning_2
meArab2019_conditioning_3
meArab2019_conditioning_4
meArab2019_conditioning_5
meArab2019_blank_1
meArab2019_blank_2
meArab2019_pool_1
meArab2019_sample_water_2
meArab2019_sample_etoh_3
meArab2019_sample_etoh_1
meArab2019_sample_water_3
meArab2019_pool_2
meArab2019_blank_3
meArab2019_sample_water_4
meArab2019_sample_etoh_2
meArab2019_sample_water_1
meArab2019_sample_etoh_4
meArab2019_pool_3
meArab2019_blank_4
```


## Preparing Data Processing

### Sample Metadata

XCMS relies on a tabular (Spreadsheet-like) file to map files to their exerpimental metadata. This file can be created in Excel, but it must be *saved as* a `.tsv` (aka tabular or tab seperated value file).

There are several required columns:
  - `sample_name`: defines the filename of the injection. File extension should not be included.
  - `sampleType`: set to either "blank", "pool", or "sample".
  - `injectionOrder`: order of injection
  - `batch`: if multiple runs are used, label batch names should differ. Otherwise, use the same name.
  - `class`: set to "blank", "pool" or name of biological sample

e.g.:

| sample\_name			| sampleType	| injectionOrder	| batch	| class		|
|-------------------------------|---------------|-----------------------|-------|---------------|
| meArab2019\_blank\_1		| blank		| 1			| c18	| blank		|
| meArab2019\_blank\_2		| blank		| 2			| c18	| blank		|
| meArab2019\_pool\_1		| pool		| 3			| c18	| pool		|
| meArab2019\_sample\_water\_2	| sample	| 4			| c18	| Water		|
| meArab2019\_sample\_etoh\_3	| sample	| 5			| c18	| Ethanol	|
| meArab2019\_sample\_etoh\_1	| sample	| 6			| c18	| Ethanol	|
| meArab2019\_sample\_water\_3	| sample	| 7			| c18	| Water		|
| meArab2019\_pool\_2		| pool		| 8			| c18	| pool		|
| meArab2019\_blank\_3		| blank		| 9			| c18	| blank		|
| meArab2019\_sample\_water\_4	| sample	| 10			| c18	| Water		|
| meArab2019\_sample\_etoh\_2	| sample	| 11			| c18	| Ethanol	|
| meArab2019\_sample\_water\_1	| sample	| 12			| c18	| Water		|
| meArab2019\_sample\_etoh\_4	| sample	| 13			| c18	| Ethanol	|
| meArab2019\_pool\_3		| sample	| 14			| c18	| pool		|
| meArab2019\_blank\_4		| sample	| 15			| c18	| blank		|

> Note: conditioning injections are not part of data processing

### Converting Thermo RAW to mzML

`msconvert` by [Proteowizard](http://proteowizard.sourceforge.net/tools.shtml) can convert vendor files to an open format, such as mzML. It is installed on Titan.

Generally, converting all of the RAW files in the present folder to mzML looks like:

```
msconvert Z:\eslerm\meArab2019\raw\meArab2019*.RAW --mzML --filter "peakPicking true 1" -o Z:\eslerm\meArab2019\mzML\
```

> Note: In the above command `*` is a wildcard symbol which finds all patterns (files) beginning with `Z:\eslerm\meArab2019\raw\meArab2019` and ending with `.RAW`.

#### Command Line

Ask for help if this is your first time using command line.

To open command line run `cmd` from the start menu. To change directories type: `cd Z:\folder`. To list what is in the directory enter: `dir`.

#### Voltage Polarity

Pre-processing steps can only be processed in a single polarity. If data was collected in with both polarities (switching) the modes must be seperated. Negative and positive data can be joined after spereately pre-processing both.

To filter for a polarity use `--filter "polarity negative"` or `--filter "polarity positive"`.  

#### Retention Time Subset

If only a portion of retention time will be analyzed, it is best to subset the data with msconvert. Subsetting the raw data with msconvert reduces the amount of storage space and processing time required by the rest of the dataprocessing..

> Note: Do not subset retention time with the `xcmsSet` tool. It will cause the `group` tool to fail.

### Galaxy Instances

There are several public Galaxy Instances which host W4M. Lab members are encouarged to use Leo which has access to Hazel.

Workflow4Metabolomics has two servers:
  - [Workflow4Metabolomics' UseGalaxy](https://workflow4metabolomics.usegalaxy.fr/login)
  - [Workflow4Metabolomics](https://galaxy.workflow4metabolomics.org/)

GalaxyP and UseGaalxy.eu accept our tool list suggestions:
  - [MSI GalaxyP](https://galaxyp.msi.umn.edu/)
  - [metabolomics.usegalaxy.eu](https://metabolomics.usegalaxy.eu/)


## Data Processing

See W4M's [Galaxy Training's Mass Spectrometry: LC-MS Analysis](https://galaxyproject.github.io/training-material/topics/metabolomics/tutorials/lcms/tutorial.html) for a walkthrough of W4M data processing steps. The following are companion notes to that guide.

> XCMS documentation is useful for understanding what parameters mean.

### Pre-processing Data

General parameters for the QE with a C18 column are saved in [a Galaxy Workflow](https://galaxy.workflow4metabolomics.org/u/mesler/w/pre-processing-extract). 

> Note: Make sure to upload mzML data as a collection. There is a [Galaxy Tutorial](https://galaxyproject.org/tutorials/collections/) on how collections work.

### Post-processing Data

A post-processing workflow which includes blank subtraction, CV subtraction, and normalization is [hosted here](https://galaxy.workflow4metabolomics.org/u/mesler/w/copy-of-post-processing-correction).
