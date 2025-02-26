**Run an Experiment on Azure Machine Learning**
===================================================

NNI supports running an experiment on `AML <https://azure.microsoft.com/en-us/services/machine-learning/>`__ , called aml mode.

Setup environment
-----------------

Step 1. Install NNI, follow the install guide `here <../Tutorial/QuickStart.rst>`__.   

Step 2. Create an Azure account/subscription using this `link <https://azure.microsoft.com/en-us/free/services/machine-learning/>`__. If you already have an Azure account/subscription, skip this step.

Step 3. Install the Azure CLI on your machine, follow the install guide `here <https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest>`__.

Step 4. Authenticate to your Azure subscription from the CLI. To authenticate interactively, open a command line or terminal and use the following command:

.. code-block:: bash

   az login

Step 5. Log into your Azure account with a web browser and create a Machine Learning resource. You will need to choose a resource group and specific a workspace name. Then download ``config.json`` which will be used later.

.. image:: ../../img/aml_workspace.png
   :target: ../../img/aml_workspace.png
   :alt: 


Step 6. Create an AML cluster as the computeTarget.

.. image:: ../../img/aml_cluster.png
   :target: ../../img/aml_cluster.png
   :alt: 


Step 7. Open a command line and install AML package environment.

.. code-block:: bash

   python3 -m pip install azureml
   python3 -m pip install azureml-sdk

Run an experiment
-----------------

Use ``examples/trials/mnist-pytorch`` as an example. The NNI config YAML file's content is like:

.. code-block:: yaml

   searchSpaceFile: search_space.json
   trialCommand: python3 mnist.py
   trialConcurrency: 1
   maxTrialNumber: 10
   tuner:
     name: TPE
     classArgs:
       optimize_mode: maximize
   trainingService:
     platform: aml
     dockerImage: msranni/nni
     subscriptionId: ${your subscription ID}
     resourceGroup: ${your resource group}
     workspaceName: ${your workspace name}
     computeTarget: ${your compute target}

Note: You should set ``platform: aml`` in NNI config YAML file if you want to start experiment in aml mode.

Compared with `LocalMode <LocalMode.rst>`__ training service configuration in aml mode have these additional keys:


* dockerImage

  * required key. The docker image name used in job. NNI support image ``msranni/nni`` for running aml jobs.

.. Note:: This image is build based on cuda environment, may not be suitable for CPU clusters in AML.

amlConfig:


* subscriptionId

  * required key, the subscriptionId of your account

* resourceGroup

  * required key, the resourceGroup of your account

* workspaceName

  * required key, the workspaceName of your account

* computeTarget

  * required key, the compute cluster name you want to use in your AML workspace. `refer <https://docs.microsoft.com/en-us/azure/machine-learning/concept-compute-target>`__ See Step 6.

* maxTrialNumberPerGpu

  * optional key, default 1. Used to specify the max concurrency trial number on a GPU device.

* useActiveGpu

  * optional key, default false. Used to specify whether to use a GPU if there is another process. By default, NNI will use the GPU only if there is no other active process in the GPU.

The required information of amlConfig could be found in the downloaded ``config.json`` in Step 5.

Run the following commands to start the example experiment:

.. code-block:: bash

   git clone -b ${NNI_VERSION} https://github.com/microsoft/nni
   cd nni/examples/trials/mnist-pytorch

   # modify config_aml.yml ...

   nnictl create --config config_aml.yml

Replace ``${NNI_VERSION}`` with a released version name or branch name, e.g., ``v2.3``.

Monitor your code in the cloud by using the studio
--------------------------------------------------

To monitor your job's code, you need to visit your studio which you create at step 5. Once the job completes, go to the Outputs + logs tab. There you can see a 70_driver_log.txt file, This file contains the standard output from a run and can be useful when you're debugging remote runs in the cloud. Learn more about aml from `here <https://docs.microsoft.com/en-us/azure/machine-learning/tutorial-1st-experiment-hello-world>`__.
