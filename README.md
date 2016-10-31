This repository provides a description of the data analysis tools used for a Flash X-ray Imaging (FXI) experiment which was performed at 
the Linac Coherent Light Source (LCLS) and is described in 

**Daurer B.J., Okamoto K., et al. TITLE. Manuscript in preparation.**

The data has been deposited in the Coherent X-ray Imaging Data Base (CXIDB) with ID XX and can be downloaded from here: 
http://cxidb.org/id-XX.html

### List of available files: ###
File name                                           | Name     | Description
--------------------------------------------------- | -------- | ----------------------------------
http://cxidb.org/data/XX/cxidb_XX_hits.tar.gz       | **HITS** | Diffraction hits saved as CXI files.
http://cxidb.org/data/XX/cxidb_XX_background.tar.gz | **BKGR** | Diffraction background saved as CXI files.
http://cxidb.org/data/XX/cxidb_XX_metadata.tar.gz   | **META** | Auxiliary files.

### Inspecting CXI files ###
The easiest way to inspect CXI files is to use the viewing tool Owl 
(http://github.com/FXIhub/owl), but any inspection tool for HDF5 files can be used.

### Requirements ###
In order to be able to run all the provided scripts and jupyter notebooks, the following has to be installed:

* python 2.7
* numpy
* scipy
* h5py
* matplotlib
* jupyter-notebook
* libspimage (http://github.com/FXIhub/libspimage)
* condor (http://github.com/FXIhub/condor)

### Contact information
For questions about this repository and the data analysis tools, feel free to contact the author: 
* Benedikt Daurer (benedikt@xray.bmc.uu.se)

________________________________________________

## Step-by-step instructions on data analysis ##
The following sections provide a detailed description of the individual data analysis steps performed for this work. The names **HITS**, **BKGR** and **META** are used as reference to the data files which are available for download using the links above. The entire processing pipeline for this experiment looks like this:

![Overview](overview.png?raw=true)

### 1. Dark calibration
All raw data frames from run 63 have been averaged (using *Cheetah*) and saved in `META/back/darkcal.h5` and `META/front/darkcal.h5` for the back and front detector respectively. 

### 2. Detector characterization
All dark calibrated and common-mode corrected data frames from the runs 163-214 have been histogrammed (per-pixel) with *Cheetah* and saved in `META/back/histograms/\*histogram.h5` and `META/front/histograms/\*histogram.h5`. All individual histograms are merged together using
```
scripts/merge_histograms.py META/back/histograms/r0163-histogram.h5 META/back/histograms/r0*-histogram.h5 --cm -o META/back/
scripts/merge_histograms.py META/front/histograms/r0163-histogram.h5 META/front/histograms/r0*-histogram.h5 --cm -o META/front/
```
and saved in `META/back/merged_histogram.h5` and `META/front/merged/merged_histogram.h5`.

Fitting Gaussians to each histogram by using
```
scripts/fit_histograms.py fit META/back/merged_histogram.h5 -o META/back/gain/
scripts/fit_histograms.py fit META/front/merged_histogram.h5 -o META/front/gain/
```
generates per-pixel fitting results which are saved in `META/back/fitting_results.h5` and `META/front/fitting_results.h5`. Based on these numbers, per-pixel estimates for gain and noise can be generated using
```
scripts/fit_histograms.py generate META/back/fitting_results.h5 -o META/back/
scripts/fit_histograms.py generate META/front/fitting_results.h5 -o META/front/
```
which saves gain/noise maps into `META/back/gainmap.h5`/`META/back/bg_sigmamap.h5` and `META/front/gainmap.h5`/`META/front/bg_sigmamap.h5` respectively. 

The outcome of this detector characterization is summarized in this notebook: [Detector charachterization (Figure 3)](./ipynb/fig03_detector.ipynb).

### 3. Background characterization
Two background runs, the first during buffer injection (run 175) and the second with no injection (run 199) have been processed with *Cheetah* (using dark and common-mode correction) as described in `BKGR/cheetah.ini` and saved in `BKGR/cxic9714-r0175.cxi` and `META/cxic9714-r0199.cxi`. Using 
```
scripts/background_buffer.py
scripts/background_beamline.py
```
a statistical analysis of the background frames is performed and results are saved in `META/background_buffer_stats.h5` and `META/background_beamline_stats.h5`. The outcome of this background characterization is summarized in this notebook: [Background characterization (Figure 9)](./ipynb/fig09_background.ipynb). 

### 4. Hit-finding 
For all sample runs between run 163 and 211, *Cheetah* was used to find diffraction hits based on a lit-pixel counter applied to dark and common-mode corrected back detector images (see `HITS/cheetah.ini` for configuration details). For all hit events, assembled images of both detectors (back and front) together with auxilliary data (injector positions, photon energies, pulse energies, ...) are saved in `HITS/cxic9714-r0XXX.cxi`. Using the visualisation tool [**Owl**](http://github.com/FXIhub/owl), it is simple to inspect the content of these CXI files:

![Owl inspect](owl_inspect.png?raw=true)

### 5. Classification based on size and intensity
Using the sphere-model option of [**Owl**](http://github.com/FXIhub/owl) (&#8984; + M), diffraction from a homogeneous sphere has been fitted to all low-resolution diffraction patterns (back detector) using the following recipe:

1. Specify model properties (5.5 keV photon energy, material density of poliovirus,, 2.4 m detector distance,  95 % detector quantum efficiency, 110e-6 m detector pixelsize)
2. Find center position using the 

![Owl sizing](owl_sizing.png?raw=true)

![Owl tagging](owl_tagging.png?raw=true)
