# Photoluminescence Analysis

## Table of Contents
- [Overview](#overview)
- [Analyses](#analyses)
    - [Photoluminescence Statistical Analysis (PLSA)](#PLSA)
        - [Usage](#PLSA-usage)
        - [Techniques](#PLSA-techniques)
        - [Example Statistical Analysis](#example-statistical-analysis)
        - [Example Parameter Correlations](#example-parameter-correlations)
    - [Intensity Depenent Photoluminescence (IDPL)](#IDPL)
        - [Usage](#IDPL-usage)
        - [Techniques](#IDPL-techniques)
        - [Example Outputs](#example-outputs)
    - [Low Temperature Photoluminescence (LTPL)](#LTPL)
    - [Time Resolved Photoluminescence (TRPL)](#TRPL)
- [Requirements](#requirements)
- [Sample Data](#sample-data)
  
<a id="overview"></a>
## Overview
This set of Jupyter Notebooks introduces new techniques for batch processing and analysing photoluminescence (PL) data using the Pandas Python library. Although originally written for the study of hyprid perovskite thin films, **this code can be applied to almost any set of PL data.** Refer to the [Experimental Configuration](#experimental-configuration) and the two "Usage" sections ([PL statistical analysis](#PLSA_Usage) and [intensity-dependent PL](#IDPL_Usage)) for more information on transferrability to different data sets and experimental setups.

### Photoluminescence Statistical Analysis (PLSA)
A novel method of analysis, PLSA, is developed in the first notebook. PL signals collected from 15 to 20 random points among 1 to 3 films of the same sample type are numerically averaged. The standard error in the PL emission across all the wavelengths is calculated in addition to the uncertainty in the peak PL position. The uncertainty reveals information about thin film crystalline quality, homogeneity (chemical and physical), and processing reproducibility. The first graphic below summarizes the data handling in the notebook and the second graphic summarizes the process for this type of experiment using PLSA code output examples.

![PLSA_graphic1](https://raw.githubusercontent.com/vvz-cu/PL_analysis/main/graphics/PLSA_graphic1.png) 

![PLSA_graphic2](https://raw.githubusercontent.com/vvz-cu/PL_analysis/main/graphics/PLSA_graphic2.png) 

The other two critical capabilities of this notebook are the parameter averaging and parameter correlation analyses. Sample types can be categorized by different properties and the parameters are numerically averaged within the category to provide a higher level view. For example, samples A, B, and C (category 1) were processed at 100 $^\circ$C while samples D, E, and F (category 2) were processed at 120 $^\circ$C. If we want to test the hypothesis that a higher processing temperature generally increases parameters X, Y, and Z, this notebook provides the capability of averaging parameters X, Y, and Z within each category to quickly assess the changes. If parameters X, Y, and Z are obtained from averaged measurements, this categorization assessment provides a higher degree of statistical significance to the results.

Furthermore, this notebook generates a parameter correlation heatmap. Each element in the heatmap is the calculated linear correlation coefficient between two parameters. Additional datasets can be loaded into the notebook from other characterizations to introduce parameters beyond those extracted from PL and PLSA. The heatmap can reveal connections between parameters that would otherwise be hidden or difficult to see. For example, in data from Sn-based perovskite thin films, a moderate positive correlation between the uncertainty in the PL intensity and the degree of crystalline order (a parameter extracted from X-ray characterization) in the film was revealed. Thus, the standard error in PL intensity extracted using PLSA can predict the crystalline ordering of the film. The parameter correlation heatmap is intended to serve as a guide for further experiments.

This notebook also includes supplemental UV/vis absorption spectroscopy data analysis (Tauc plots) and Urbach energy calculation.

### Intensity Dependent Photoluminescence (IDPL)
The IDPL analysis in this notebook is used to characterize thin films with hot carrier generation. To assess the extent of this property, PL is measured at several different excitation laser power outputs and the peak position and the blue-side broadening are tracked. The carrier temperature as a function of excitation intensity is also extracted by fitting the blue-side of the PL curves to a Maxwell-Boltzmann distribution.

### Low Temperature Photoluminescence (LTPL)
Update coming soon.

### Time Resolved Photoluminescence (TRPL)
Update coming soon.

<a id="analyses"></a>
## Analyses  
<a id="PLSA"></a>
## Photoluminescence Statistical Anslysis (PLSA)
Given the photoluminescence data this notebook can perform a statistical analysis on the PL emission and extract the Urbach energies from mean PL emissions. Inputed UV/vis absorption spectra will allow for the calculation of bandgap energy. With parameters extracted from the statistical analysis (and with additional sample parameters if inputed), the notebook will perform a linear correlation analysis between the parameters.

**Outputs**:
- numerical average of PL signal of a sample type
- uncertainty in PL signal intensity
- uncertainty in PL peak position
- Urbach energy
- bandgap energy (and Tauc plots)
- Pearson correlation coefficient heatmap

<a id="PLSA-usage"></a>
### Usage
PL data is inputed as a .txt file with two columns: emission wavelength (nm) and PL counts.

UV/vis data is inputed as a .csv file with two columns: absorption wavelength (nm) and optical density (OD). The data includes a baseline background correction file.

For the notebook to perform the batch analysis, the file structure must be in the following structure:
```
├── Experiment I
│   ├── Sample A
│   │   ├── A_plsa_1.txt
│   │   ├── A_plsa_2.txt
│   │   ├── A_plsa_3.txt
│   │   ├── ...
│   ├── Sample B
│   │   ├── B_plsa_1.txt
│   │   ├── ...
│   ├── Sample C
│   │   ├── C_plsa_1.txt
│   │   ├── ...
│   ├── UVvis
│   │   ├── 100Tbaseline.csv
│   │   ├── A.Sample.Raw.csv
│   │   ├── B.Sample.Raw.csv
│   │   ├── C.Sample.Raw.csv
│   ├── extra
```

Properly formatted and structured data should output the following DataFrame with the mean PL signal ("mean") and the PL signal uncertainty ("\_se").

![DataFrame PLSA](https://raw.githubusercontent.com/vvz-cu/PL_analysis/main/graphics/PLSA_dataframe.png)

<a id="PLSA-techniques"></a>
 
### Techniques

**Corrections**:

1. Optical Density Correction
> UV/vis absorption spectroscopy is used to correct for thickness variations that could affect the collected PL emission. Each experiment's samples are normalized to the control film's optical density at the excitation wavelength. For the experiments for which this analysis was developed, this wavelength is far enough away from the emission wavelength and we can assume high light absorption. 

2. Laser Output Correction 
> This correction is useful for experiments that are performed over the course of several days. The laser signal at a given power is reflected off a clean glass slide and collected by the spectrometer with a set exposure time. The measurement should be taken at the start of every data collection sequence after the laser has been given time to warm up. An arbitrary (but constant over the course of the experiment) signal count (e.g. 2e4 counts) is chosen to normalize the laser output every instance data is collected.

3. Smoothing 
> Data is smoothed with a rolling average with a 20 point window, which does not distort the data.

**Calculations**:

1. Uncertainty
> The uncertainty in the PL signal intensity and the PL peak position is calculated as the standard error using the __[pandas.DataFrame.sem](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.sem.html)__ method. The standard error is then normalized for each sample to the sample's PL peak intensity.

2. Urbach Energy
> The Urbach energies are calculated from the red-side of the PL curves using the van Roosbroeck–Shockley relation at room temperature $$ I_{PL}(E,T) \propto E^2\exp{(\frac{\sigma (T) - 1}{k_bT}E)} $$ where $E$ is the PL emission energy in eV, $k_b$ is the Boltzmann constant, and $\sigma (T)$ is the steepness parameter defined as $k_bT / E_U(T)$ where $E_U$ is the extracted Urbach energy. Another example of this method in use for perovskite thin films can be found in the publication __[Handa et al. *Phys. Rev. Materials* **2** 075402 (2018)](https://journals.aps.org/prmaterials/abstract/10.1103/PhysRevMaterials.2.075402)__ and the relation is detailed in the publication __[W. van Roosbroeck and W. Shockley *Phys. Rev.* **94** 1558 (1954)](https://journals.aps.org/pr/abstract/10.1103/PhysRev.94.1558)__.

3. Tauc Plot (Bandgap Energy)
> For these calculations, the thickness ($z$) of the thin film must be known to calculated the absorption coefficient. To perform an accurate fit of the Tauc plot to determine the bandgap energy, the inflection point of the absorption onset must be found. The bounds around the target maximum of the first derivative of the $(\alpha h \nu)^{1/2}$ are defined by the user (as there are many local maxima) and the code determines the local maximum ($A$) within those bounds. The __[numpy.polyfit](https://numpy.org/doc/stable/reference/generated/numpy.polyfit.html)__ method is used to perform a linear fit in the region $A-0.02$ eV to $A+0.02$ eV. **This is the only section of the analysis that is not yet set up to do batch processing.** Thus, each sample's bandgap must be extracted and loaded into a bandgap DataFrame (Egdf) individually.

4. Linear Correlations
> Linear Correlations between extracted parameters are calculated using the __[pandas.DataFrame.corr](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.corr.html)__ method. The calculated Pearson correlation coefficients are then used in the heatmaps generated using __[seaborn.heatmap](https://seaborn.pydata.org/generated/seaborn.heatmap.html)__.

<a id="example-statistical-analysis"></a>
### Example Statistical Analysis

Once the DataFrame containing the mean PL signals and PL intensity uncertainty is compiled, the notebook can generate a plot of the new, statistically analyzed PL.

<img src="https://raw.githubusercontent.com/vvz-cu/PL_analysis/main/graphics/PLSA_plot.png" width="500"/>

The parameter averaging can generate plots such as these for any desired parameter.

<img src="https://raw.githubusercontent.com/vvz-cu/PL_analysis/main/graphics/PLSA_parameteravg.png" width="200"/>

<a id="example-parameter-correlations"></a>
### Example Parameter Correlations    

With the Pearson correlation matrix outputed, this analysis can generate a heatmap to visually examine correlations between parameters.

<img src="https://raw.githubusercontent.com/vvz-cu/PL_analysis/main/graphics/PLSA_heatmap.png" width="500"/>

<a id="IDPL"></a>
## Intensity Dependent Photoluminescence (IDPL)

Given multiple series of intensity-dependent photoluminescence data this notebook can plot the data and extract parameters related to the material hot carrier generation.

**Outputs**:
* PL peak position as a function of excitation intensity
* PL blue-side broadening as a function of excitation intensity
* carrier temperature ($T_C$) as a function of excitation intensity

<a id="IDPL-usage"></a>
### Usage

PL data is inputed as a .txt file with two columns: emission wavelength (nm) and PL counts.

For the notebook to perform the batch analysis, the file structure must be in the following structure. The PLSA data files can be stored in the same folders as indicated; the code will only handle files with "idpl" in the file name.

```
├── Experiment I
│   ├── Sample A
│   │   ├── A_idpl_[power 1 in mW]_.txt
│   │   ├── A_idpl_[power 2 in mW]_.txt
│   │   ├── A_idpl_[power 3 in mW]_.txt
│   │   ├── ...
│   │   ├── A_plsa_1.txt
│   │   ├── ...
│   ├── Sample B
│   │   ├── B_idpl_[power 1 in mW]_.txt
│   │   ├── ...
│   │   ├── B_plsa_1.txt
│   │   ├── ...
│   ├── Sample C
│   │   ├── C_idpl_[power 1 in mW]_.txt
│   │   ├── ...
│   │   ├── C_plsa_1.txt
│   │   ├── ...
```

Properly formatted and structured data should output the following DataFrames. The first houses the list of files that will be accessed for each sample and excitation intensity. The second contains the analysis of PL peak position ("\_pk") and blue-side inflection point ("\_infl"). 

<img src="https://raw.githubusercontent.com/vvz-cu/PL_analysis/main/graphics/IDPL_df1.png" width="500"/>

<img src="https://raw.githubusercontent.com/vvz-cu/PL_analysis/main/graphics/IDPL_df2.png" width="750"/> 

As indicated in the images above, **the code can handle missing data**.

<a id="IDPL-techniques"></a>
### Techniques

**Corrections**:

1. Smoothing:
> Data is smoothed with a rolling average with a 20 point window, which does not distort the data.

**Calculations**:

1. Inflection Point:
> To quantify the blue-side broadening of the PL curves, the inflection point on the blue-side is determined as a function of excitation intensity. The inflection point method has been used as a spectral analysis tool in the publication __[M. Sendova and C. Laughrey, *Phys. Chem. Chem. Phys.* **24** 14055-14063 (2022)](https://pubs.rsc.org/en/content/articlelanding/2022/CP/D2CP01366E)__, but **has never before been used for hot carrier analysis**.

<a id="example-outputs"></a>
### Example Outputs

Each sample's intensity-dependent PL data can be plotted as:
<img src="https://raw.githubusercontent.com/vvz-cu/PL_analysis/main/graphics/IDPL_plot.png" width="500"/>

The extracted change in peak position and blue-side broadening are tracked visually.
<img src="https://raw.githubusercontent.com/vvz-cu/PL_analysis/main/graphics/IDPL_tracking.png" width="500"/>

Carrier temperature extracted from Maxwell-Boltzmann fits are plotted and recorded in a new DataFrame.
<img src="https://raw.githubusercontent.com/vvz-cu/PL_analysis/main/graphics/IDPL_fit.png" width="500"/>
<img src="https://raw.githubusercontent.com/vvz-cu/PL_analysis/main/graphics/IDPL_results.png" width="300"/>

<a id="LTPL"></a>
## Low Temperature Photoluminescence (LTPL)
Updates coming soon.

<a id="TRPL"></a>
## Time Resolved Photoluminescence (TRPL)
Updates coming soon.

<a id="requirements"></a>
## Requirements
Python 3.8.8

The notebooks use the following libraries:
* __[matplotlib](https://matplotlib.org/3.5.3/index.html)__  3.5.3
    * __[matplotlib.pyplot](https://matplotlib.org/3.5.3/api/_as_gen/matplotlib.pyplot.html)__
* __[numpy](https://numpy.org/devdocs/index.html)__  1.23.3
* __[pandas](https://pandas.pydata.org/pandas-docs/version/1.4/index.html)__ 1.4.4
* __[scipy](https://scipy.org/)__ 1.9.1
    * __[scipy.signal](https://docs.scipy.org/doc/scipy/reference/signal.html)__ 
    * __[scipy.optimize](https://docs.scipy.org/doc/scipy/reference/optimize.html)__
* __[seaborn](https://seaborn.pydata.org/)__ 0.11.1


<a id="sample-data"></a>
## Sample Data
Sample data for both the PLSA and IDPL analyses with the correct file structure and format can be downloaded __[here](https://raw.githubusercontent.com/vvz-cu/PL_analysis/main/sampledata/SampleData.zip)__.
