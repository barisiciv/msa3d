MSA3D
=====


About
-----

This software was developed for data reduction and cube design for the JWST slit-stepping survey GO-2136.
It is designed to be applicable for any future JWST slit-stepping surveys employing a similar observing strategy.

The software consists of two components:
a) the version 1.14.0 ``jwst`` STScI pipeline to process the data via a modified set of arguments and keywords, and 
b) the original software developed for cube design in a slit-stepping strategy with NIRSpec MSA 
- the latter being an unsupported data processing mode in the standard STScI pipeline.  

See  `Barisic et al. 2024 <https://ui.adsabs.harvard.edu/abs/2024arXiv240808350B/abstract>`__ for
technical details and a case study analysis of an example target.


Citation
--------

If you use MSA3D for your data reduction and/or analysis, please cite the following publication 

    - Barisic et al. 2024 : `arXiv <https://ui.adsabs.harvard.edu/abs/2024arXiv240808350B/abstract>`__ (BibTeX entry `here <https://ui.adsabs.harvard.edu/abs/2024arXiv240808350B/exportcitation>`__ )
    - Barisic et al. 2024 : `Zenodo`


Installation
------------

Start by creating a new python environment using conda or venv, example:

.. code-block:: console

    conda create -n ENVNAME python=3.11
    conda activate ENVNAME


To install MSA3D, clone the git repository into a new folder:

.. code-block:: console

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

    - 11GB : \*rate.fits files (available for download on MAST)

    - 50GB : products of custom JWST/STScI Spec2Pipeline + Spec3Pipeline reduction (2D spectra)

    - 12GB : products of cube design (data cubes and related products)


Data access
-----------

The ``msa3d`` data reduction starts with slope images, which can be found on
`MAST Portal <https://mast.stsci.edu/portal/Mashup/Clients/Mast/Portal.html>`__.
GO-2136 program is publicly available. Search for the data set on the portal
using the Proposal ID: 2136 and download all the \*_rate.fits files.

After downloading the \*_rate.fits files, make sure all \*_rate.fits files are in the **same folder**. 
Make sure \_msa.fits file is also downloaded **into the same folder** as the \*_rate.fits files.


Running the software
---------------------

This repository includes a Jupyter notebook to assist users in running the pipeline. Below are detailed instructions for convenience.

Usage Instructions:

To run the pipeline, follow these steps:

1. Import all functions from the ``run_msa3d`` module
2. Define the following variables:

   - ``data_entries``: This variable should hold the path to a set of `*_rate.fits` files.
   - ``msa_path``: This variable should be the path to the MSA file located in the **same** directory as the `*_rate.fits` files.

3. Call the ``run`` function, passing following arguments: ``data_entries``, ``msa_path``, ``run_process``, ``run_postprocess`` and ``run_cubebuild``. The ``run`` function will perform data reduction, starting from the Spec2Pipeline and Spec3Pipeline reduction provided by the standard STScI reduction pipeline, followed by post-processing and cube design.

.. code-block:: console
    ## EXAMPLE CODE

    from run_msa3d import *

    # paths below are examples
    data_entries = np.sort(glob.glob('/home/user/GO-2136/JWST/jw*rate.fits'))
    msa_path = '/home/user/GO-2136/JWST/jw02136001001_01_msa.fits'

    run(data_entries, msa_path, run_process=True, run_postprocess=True, run_cubebuild=True)


Arguments:

    - ``run_process=True`` enables ``jwst`` Spec2Pipeline and Spec3Pipeline reduction
    - ``run_postprocess=True`` enables postprocessing of 2D spectra, inluding pathloss correction and outlier/cosmic ray rejection
    - ``run_cubebuild=True`` enables cube design 


Multiprocessing feature
-----------------------

This software includes a multiprocessing functionality to expedite the STScI Spec2Pipeline and Spec3Pipeline reduction steps. To enable this feature, use the additional argument ``N_gmembers`` and set it to your desired number of exposures per group. For example:

.. code-block:: console

    run(data_entries, msa_path, run_process=True, run_postprocess=True, run_cubebuild=True, N_gmembers=9)


In this example, ``N_gmembers=9`` specifies a number of exposures per group. For the GO-2136 program - having a total of 63 exposures, this will create 7 groups (each with 9 exposures). The multiprocessing feature will then utilize 7 workers to process the exposures in parallel.

**Note:** the value for ``N_gmember=9`` was chosen **for a system with 24GB RAM and 8 cores**. 


Acknowledgements
-----------------







