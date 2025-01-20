OSC Data Extraction Pipeline
=============================

|osc-climate-project| |osc-climate-slack| |osc-climate-github| |pypi| |build-status| |pdm| |PyScaffold|

This README provides a step-by-step guide to set up and run the OSC data extraction pipeline. Follow the instructions below to create a virtual environment, install necessary packages, and execute the data extraction, curation, and training processes.

💬 **Important**

On June 26 2024, Linux Foundation announced the merger of its financial services umbrella, the Fintech Open Source Foundation (`FINOS <https://finos.org>`_), with OS-Climate, an open source community dedicated to building data technologies, modelling, and analytic tools that will drive global capital flows into climate change mitigation and resilience; OS-Climate projects are in the process of transitioning to the `FINOS governance framework <https://community.finos.org/docs/governance>`_; read more on `finos.org/press/finos-join-forces-os-open-source-climate-sustainability-esg <https://finos.org/press/finos-join-forces-os-open-source-climate-sustainability-esg>`_

Prerequisites
-------------

- Python 3.10
- Git

Setup
-----

### Create and Activate Virtual Environment

Create a virtual environment named `.venv`:

.. code-block:: shell

    python3 -m venv .venv

Activate the virtual environment:

.. code-block:: shell

    source .venv/bin/activate

### Install Required Packages

Install the `osc-transformer-presteps` package:

.. code-block:: shell

    pip install osc-transformer-presteps

Install the `osc-transformer-based-extractor` package:

.. code-block:: shell

    pip install osc-transformer-based-extractor

### Create Data Structure and Content

Create the necessary folders:

.. code-block:: shell

    mkdir -p inputs/pdfs_training inputs/pdfs_inference logs outputs/jsons_training outputs/jsons_inference outputs/curated_data_rel outputs/curated_data_kpi model outputs/inference_rel

Place your training PDFs into the `inputs/pdfs_training` folder and inference PDFs into the `inputs/pdfs_inference` folder.

Place the `kpi_mapping.csv` file into the `inputs` folder.

Place the annotations Excel file (with "annotation" in its name) into the `inputs` folder.

Run Extraction
--------------

Run the extraction process:

.. code-block:: shell

    osc-transformer-presteps extraction run-local-extraction inputs/pdfs_training --output-folder=outputs/jsons_training --logs-folder=logs

Run Curation for Relevance Detection
------------------------------------

Run the curation process for relevance detection:

.. code-block:: shell

    osc-transformer-presteps relevance-curation run-local-curation outputs/jsons_training inputs/annotations_training_large_20240416.xlsx inputs/kpi_mapping.csv outputs/curated_data_rel --logs-folder=logs --create_neg_samples

*Note: Change the name of the annotation file to your specific file name.*

Train Relevance Model
----------------------

Run the training of the relevance model:

.. code-block:: shell

    osc-transformer-based-extractor relevance-detector fine-tune "outputs/curated_data_rel/Curated_dataset_20112024_1347.csv" "bert-base-uncased" 2 128 10 16 0.0001 "models/" "dabetamo_best_model" 16

*Note:*

- Change the data path according to the name of your curated data file.
- If you want to use another base model or have a pre-trained one, change the name accordingly.
- For a pre-trained model, use `models/` instead of `bert-base-uncased`. If no local model path is provided, the tool will try to use the model via HuggingFace.
- If you want to change the parameters check the documentation of the CLI tool to get more information.

Copy the files from `outputs/jsons_training` to `outputs/jsons_inference`:

.. code-block:: shell

    cp outputs/jsons_training/* outputs/jsons_inference/

*Note: Ensure that the `jsons_inference` folder is empty before copying to avoid running inference on unnecessary files.*

Run Inference
--------------

Run the inference process:

.. code-block:: shell

    osc-transformer-based-extractor relevance-detector inference outputs/jsons_inference inputs/kpi_mapping.csv outputs/rel_inference 'models/dabetamo_best_model' 'models/dabetamo_best_model' 1 0.5

Run KPI Curation
----------------

Run the KPI curation process:

.. code-block:: shell

    osc-transformer-presteps kpi-curation run-local-kpi-curation inputs outputs/jsons_training outputs/curated_data_kpi inputs/kpi_mapping.csv outputs/rel_inference --agg-annotation " " --val-ratio 0.2

By following these steps, you will have successfully set up and run the OSC data extraction pipeline.

----

2FA KPI Extract
----------------

- Extract Relevant Para top 100
- Extract KPIs top 100
- Send all to LLM to evaluate
- Evaluated answer --> Product

Contributing
------------

We welcome contributions! Please fork the repository and submit a pull request.  
Ensure you sign off each commit with the **Developer Certificate of Origin (DCO)**.  
Read more: http://developercertificate.org/.

Governance Transition
---------------------

On June 26, 2024, the **Linux Foundation** announced the merger of **FINOS** with OS-Climate.  
Projects are now transitioning to the [FINOS governance framework](https://community.finos.org/docs/governance).

Shields
-------

|osc-climate-project| |osc-climate-slack| |osc-climate-github| |pypi| |build-status| |pdm| |PyScaffold|

.. |osc-climate-project| image:: https://img.shields.io/badge/OS-Climate-blue
   :alt: An OS-Climate Project
   :target: https://os-climate.org/

.. |osc-climate-slack| image:: https://img.shields.io/badge/slack-osclimate-brightgreen.svg?logo=slack
   :alt: Join OS-Climate on Slack
   :target: https://os-climate.slack.com

.. |osc-climate-github| image:: https://img.shields.io/badge/GitHub-100000?logo=github&logoColor=white
   :alt: Source code on GitHub
   :target: https://github.com/ModeSevenIndustrialSolutions/osc-data-extractor

.. |pypi| image:: https://img.shields.io/pypi/v/osc-data-extractor.svg
   :alt: PyPI package
   :target: https://pypi.org/project/osc-data-extractor/

.. |build-status| image:: https://api.cirrus-ci.com/github/os-climate/osc-data-extractor.svg?branch=main
   :alt: Build Status
   :target: https://cirrus-ci.com/github/os-climate/osc-data-extractor

.. |pdm| image:: https://img.shields.io/badge/PDM-Project-purple
   :alt: Built using PDM
   :target: https://pdm-project.org/latest/

.. |PyScaffold| image:: https://img.shields.io/badge/-PyScaffold-005CA0?logo=pyscaffold
   :alt: Project generated with PyScaffold
   :target: https://pyscaffold.org/
