============
Installation
============

Downloads
=========

.. code-block:: bash

    # Clone the submodules
    git clone --recursive -b dev https://github.com/Joy1112/MARL.git
    

Running Environments Configuration
==================================

`Anaconda3 <https://www.anaconda.com/>`_ is recommended to manage your environment.

First, install the dependencies:

.. code-block:: bash

    conda env create -f environment.yaml
    while read requirement; do conda install --yes $requirement || pip install $requirement; done < requirements.txt


Next, install the **Pytorch** (version==1.6.0 is recommended) which is appropriate for your **cuda** version.


Experiments 'envs' Installation
===============================


StarCraft II Installation
+++++++++++++++++++++++++

.. code-block:: bash

    sh src/envs/starcraft_II/install_sc2.sh
    pip install third_party/smac/
    ln -s -F third_party/StarCraftII ~/
