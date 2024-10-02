MSA3D
=====


About
-----

This software was developed for data reduction and cube design for the JWST slit-stepping survey GO-2136.
It is designed to be applicable for any future JWST slit-stepping surveys that employ a similar observing strategy.

The software consists of three main components:

1. ``jwst`` STScI calibration pipeline (v. 1.14.0 ) to (pre- and/or) process the data with a modified set of arguments and keywords
2. post-processing: includes ``jwst`` pathloss correction and L.A.Cosmic for outlier and cosmic ray treatment
3. original cube design software: developed for cube design in a slit-stepping strategy with NIRSpec MSA. This is currently an unsupported processing mode in the standard STScI pipeline (Oct 2024).  

See  `Barisic et al. 2024 <https://ui.adsabs.harvard.edu/abs/2024arXiv240808350B/abstract>`__ for
technical details and a case study analysis of an example target.


Citation
--------

If you use MSA3D for your data reduction and/or analysis, please cite the following publication 

    - Barisic et al. 2024 : `arXiv <https://ui.adsabs.harvard.edu/abs/2024arXiv240808350B/abstract>`__ (BibTeX entry `here <https://ui.adsabs.harvard.edu/abs/2024arXiv240808350B/exportcitation>`__ )
    - Barisic et al. 2024 : `Zenodo DOI`


Data access
-----------

The ``msa3d`` data reduction starts with slope images, which can be found on
`MAST Portal <https://mast.stsci.edu/portal/Mashup/Clients/Mast/Portal.html>`__.
GO-2136 program is publicly available. Search for the data set on the portal
using the Proposal ID: 2136 and download all the \*_rate.fits files.

After downloading the \*_rate.fits files, make sure all \*_rate.fits files are in the **same folder**. 
Make sure \_msa.fits file is also downloaded **into the same folder** as the \*_rate.fits files.


Installation
------------

Start by creating a new python environment using conda or venv, example:

.. code-block:: console

    conda create -n <env_name> python=3.11
    conda activate <env_name>


To install MSA3D, clone the git repository into a new folder:

.. code-block:: console

    cd <your destination folder>
    git clone https://github.com/barisiciv/msa3d.git


This pulls the files into a directory called 'msa3d'.  To install the package, run:

.. code-block:: console

    cd msa3d
    pip install -e .


Before running the pipeline, set the environment variables for the CRDS following `STScI Quickstart Guide 
<https://jwst-pipeline.readthedocs.io/en/latest/getting_started/quickstart.html>`__

.. code-block:: console

    export CRDS_PATH="$HOME/crds_cache"
    export CRDS_SERVER_URL="https://jwst-crds.stsci.edu"


Disk space
----------

Total disk space required for full reduction (excluding STScI/Spec1Pipeline) is ~80GB, of which approximately:

    - 11GB : \*_rate.fits files (available for download on MAST)

    - 50GB : products of custom JWST/STScI Spec2Pipeline + Spec3Pipeline reduction (2D spectra)

    - 12GB : products of cube design (data cubes and related products)


Running the software
---------------------

This repository includes a Jupyter notebook to assist users in running the pipeline. Below are detailed instructions for convenience.

Usage Instructions:

To run the pipeline, follow these steps:

1. Import all functions from the ``run_msa3d`` module
2. Define the following variables:

   - ``data_entries``: This variable should hold the path to a set of \*_rate.fits files.
   - ``msa_path``: This variable should be the path to the MSA file. 
Note: **MSA file needs to be located in the same directory as the \*_rate.fits files.**

3. Call the ``run`` function, passing following arguments: ``data_entries``, ``msa_path``, ``run_process``, ``run_postprocess`` and ``run_cubebuild``. The ``run`` function will perform data reduction, starting from the Spec2Pipeline and Spec3Pipeline reduction provided by the standard STScI reduction pipeline, followed by post-processing and cube design.


Arguments:

    - ``run_process=True`` enables ``jwst`` Spec2Pipeline and Spec3Pipeline reduction
    - ``run_postprocess=True`` enables postprocessing of 2D spectra, inluding pathloss correction and outlier/cosmic ray rejection
    - ``run_cubebuild=True`` enables cube design 

.. code-block:: console

    ### EXAMPLE CODE
    from run_msa3d import *

    ### example paths below 
    data_entries = np.sort(glob.glob('/home/user/GO-2136/JWST/jw*rate.fits'))
    msa_path = '/home/user/GO-2136/JWST/jw02136001001_01_msa.fits'

    run(data_entries, msa_path, run_process=True, run_postprocess=True, run_cubebuild=True)


Multiprocessing feature
-----------------------

This software includes a multiprocessing functionality to expedite the STScI Spec2Pipeline and Spec3Pipeline reduction steps. To enable this feature, use the additional argument ``N_gmembers`` and set it to your desired number of exposures per group. For example:

.. code-block:: console

    run(data_entries, msa_path, run_process=True, run_postprocess=True, run_cubebuild=True, N_gmembers=9)


In this example, ``N_gmembers=9`` specifies a number of exposures per group. For the GO-2136 program - having a total of 63 exposures, this will create 7 groups (each with 9 exposures). The multiprocessing feature will then utilize 7 workers to process the exposures in parallel.

**Note:** the value for ``N_gmember=9`` was chosen **for a system with 24GB RAM and 8 cores**. 


Expected output
---------------

Running the pipeline will automatically create the ``reduction`` folder within the parent directory specified from ``data_entries``.

For example, if the provided `data_entries` path is:

.. code-block:: python

    np.sort(glob.glob('/home/user/GO-2136/JWST/jw*rate.fits'))

Parent directory in this example is ``JWST``. The resulting folder structure would be: 

.. code-block::

    JWST               # Parent directory
    │
    ├── reduction/     # Subdirectory of JWST
    │   ├── cubes/     # Subdirectory of reduction containing output cubes 
    │   │   └── cube_[target_ID]/  # Directory for cube data of a individual targets
    │   │       └── cube_medians_[target_ID].fits	# File containing a [target_ID] cube 
    │   │       └── median_lam*_s[target_ID]_all.fits	# Files containing median 2D spectra for a given dispersion step 
    │   │       └── spec_lam*_s[target_ID]_all.fits	# Files containing stacked 2D spectra for a given dispersion step
    │   └── process/   # Subdirectory of reduction containing individual exposure folders
    │   │   └── exp_[exposure_ID]_nobar/  # Directory for 2D spectra of individual targets for a given exposure
    │   │       └── newoutput_s[target_ID]_s2d_pathcorr_astrocorr.fits	# Files (relevant) representing output 2D spectra for a [target_ID] incl. post-processing, to be used in ``cube_build`` step
    │   │       └── ...


Acknowledgements
-----------------

In development of ``MSA3D``, apart from original cube building software, we make use of following packages/tools:

1. STScI ``jwst`` package (v. 1.14.0) : for data processing in stages 2-3 (optional stage 1)

    - `Zenodo DOI <https://zenodo.org/badge/latestdoi/60551519>`__ , `JWST docs <https://jwst-docs.stsci.edu/jwst-science-calibration-pipeline#JWSTScienceCalibrationPipeline-Stage1pipeline>`__
    - `JWST Calibration Pipeline GitHub Repository <https://github.com/spacetelescope/jwst?tab=readme-ov-file>`__

2. NSClean (Benjamin Rauscher) : for residual correlated noise removal in \*_rate.fits files

    - Rauscher 2023 : `arXiv <10.48550/arXiv.2306.03250>`__ , algorithm `website <https://science.nasa.gov/mission/webb/for-scientists/#NSClean>`__

3. L.A.Cosmic (Pieter G. Van Dokkum): for its effective outlier/cosmic ray detection and removal capabilities 

    - van Dokkum 2001, PASP, 113, 789, 1420 : `arXiv <https://ui.adsabs.harvard.edu/abs/2001PASP..113.1420V/abstract>`__ , `website <http://www.astro.yale.edu/dokkum/lacosmic/>`__
    - Curtis McCully, Astro-SCRAPPY: `Zenodo DOI <https://zenodo.org/record/1482019>`__
    - `Astro-SCRAPPY GitHub Repository <https://github.com/astropy/astroscrappy?tab=readme-ov-file>`__



