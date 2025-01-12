HOMS-TC: Accelerating Open Modification Spectral Library Searching on Tensor Core in Hyperdimensional Space
=======================================================

_HOMS-TC_ is an open modification spectral library search (OMS) tool for mass spectrometry-based proteomics. Our tool redesigns the MS/MS spectral matching algorithm based on hyperdimensional computing (HDC) and accelerates OMS on GPU in an end-to-end manner. The software is freely available under the Apache 2.0 license.

System Requirements
------------------------------------------------------

_HOMS-TC_ requires `Python 3.8+` with `CUDA v11+` environment. A NVIDIA GPU with Tensor Core is required for the best performance. _HOMS-TC_ has been tested on NVIDIA RTX 3090 and NVIDIA RTX 4090 with CUDA v11.8. 

Installation
------------------------------------------------------

Install via Docker
*********************

We recommend installing _HOMS-TC_ via docker using the following command:

```bash
git clone --recurse-submodules https://github.com/tycheyoung/homs-tc.git
cd homs-tc
docker build -f ./docker/Dockerfile -t homs_tc .
docker run --gpus all -it homs_tc /bin/bash  # Make sure to mount dataset folder
```

Install from Source
*********************
First, be sure to install all dependencies (Python and CUDA). In Ubuntu:

```bash
sudo apt-get update
sudo apt-get install python3 python3-dev python3-pip 
sudo apt-get install nvidia-cuda-toolkit  # This will install the latest version of CUDA. Read below before proceed
```
For CUDA installation, refer to the documentation [\[LINK\]](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html) to install a specific version.

Then, to install `homs-tc`:

```bash
git clone --recurse-submodules https://github.com/tycheyoung/homs-tc.git
cd homs-tc
./install.sh
```

Usage
------------------------------------------------------

    usage: python run.py [-h] [--ref REF] [--query QUERY] 
                              [--config CONFIG] [--output OUTPUT]
                
    Positional arguments:
    ref         The path of spectral library file (in `.splib` format) to be used to identify the experimental spectra (reference).
    query       The path of the spectral file (in `.mgf` format) to be searched (query).
    config      The path of config file (in `.ini` format). 
    output      The path of the `mztab` output file containing the search result.

Config file example
------------------------------------------------------

See `configs/iprg2012.ini` for the example config file. All parameters should be defined.
* preprocessing
  - min_spectra_ref: the minimum number of referenece spectra required for each charge. If it is below the threshold, the reference spectra with the corresponding charge are excluded from the search.
  - min_spectra_query: the minimum number of query spectra required for each charge. If it is below the threshold, the query spectra with the corresponding charge are excluded from the search.
  - resolution: the number of decimal places to round the `m/z`
  - bin_size: the bin width (in `Da`) for raw spectra to spectrum vector conversion
  - min_mz: the min m/z value to consider (inclusive) during preprocessing
  - max_mz: the max m/z value to consider (inclusive) during preprocessing
  - remove_precursor: a flag (boolean) to eliminate peaks near the precursor mass. Can be either `true` or `false`
  - remove_precursor_tolerance: a `m/z` window to eliminate peaks near the precursor mass
  - min_intensity: remove peaks with a lower intensity relative to the base peak intensity
  - min_peaks: a cutoff for discarding spectra with with fewer peaks
  - min_mz_range: a threshold to discard low-quality spectra with narrow mass range
  - max_peaks_used: the specified limit of the most intense peaks to retain for query spectra
  - max_peaks_used_library: the specified limit of the most intense peaks to retain for reference spectra
  - scaling: the method of scaling peak intensities, either square root ("sqrt") or rank-based ("rank")

* search
  - precursor_tolerance_mass_ppm: narrow window size (standard search - 1st stage of cascade search)
  - precursor_tolerance_mass_open_da: wide window size (open search - 2nd stage of cascade search)
  - max_ref_batch_size: batch size for reference hypervectors during the search stage
  - max_query_batch_size: batch size for query hypervectors during the search stage
  - hv_quantize_level: the quantization level of hypervector during the encoding stage
  - hv_dimensionality: the hypervector dimensionality
  - hv_precision: the hypervector precision. Can be either `fp32` or `fp16` or `int8`
  - use_precomputed_ref_hvs: a flag (boolean) to use dumped reference HVs. If there are no existing dump files, it will generate dump first. Can be either `true` or `false`. Usually, generating reference on-the-fly is faster than loading precomputed HVs.

* fdr
  - fdr_threshold: the FDR threshold for each stage of the cascade search
  - fdr_tolerance_mass: the bin width to group SSMs for subgroup FDR calculation during the second stage of the cascade search.
  - fdr_tolerance_mode: the unit of `fdr_tolerance_mass`. Can be either `Da` or `ppm`
  - fdr_min_group_size: the minimum group size to perform FDR control individually for that subgroup.

Troubleshooting
----
1. Make sure to add the proper [SM](https://arnon.dk/matching-sm-architectures-arch-and-gencode-for-various-nvidia-cards/) of your GPU in Makefile [\[LINK\]](https://github.com/tycheyoung/homs-tc/blob/main/Makefile#L236).
By default, `sm_89` and `sm_86` is enabled.


Benchmark Tools
----------------------

- [SpectraST](https://pubmed.ncbi.nlm.nih.gov/17295354/) running on CPU
- [ANN-SoLo](https://pubs.acs.org/doi/10.1021/acs.jproteome.8b00359) running on both CPU and GPU

Datasets
----------------------

Two real-world datasets were used for evaluation:

#### Small-Scale Dataset
- **Reference Libraries**: [Yeast spectral library](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4349991/) combined with human HCD spectral library (Total spectra: 1,162,392)
- **Query**: [iPRG2012 dataset](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3879627/) (Total spectra: 15,867)

#### Large-Scale Dataset
- **Reference Libraries**: [Human spectral library](https://www.sciencedirect.com/science/article/pii/S2405471218303193) (Total spectra: 2,992,672)
- **Query**: [HEK293 dataset (b1906∼b1938)](https://pubmed.ncbi.nlm.nih.gov/26076430/) (Total spectra per query: ~46,665)



Publication
------------------------------------------------------
Kang J, Xu W, Bittremieux W, Moshiri N, Rosing T. Accelerating open modification spectral library searching on tensor core in high-dimensional space. Bioinformatics. 2023 Jul 1;39(7):btad404. doi: 10.1093/bioinformatics/btad404. PMID: 37369033; PMCID: PMC10323168.


Contact
------------------------------------------------------

For more information, post an issue or send an email to <j5kang@ucsd.edu>.

Acknowledgements
------------------------------------------------------

This work was supported in part by Semiconductor Research Corporation (SRC).
