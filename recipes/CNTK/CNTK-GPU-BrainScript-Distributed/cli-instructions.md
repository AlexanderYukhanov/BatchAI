Please follow [instructions](/recipes/Readme.md) to install Azure CLI 2.0, configure default location, create and configure default resource group and storage account.


### Data Deployment

- Download and extract preprocessed MNIST Database from this [location](https://batchaisamples.blob.core.windows.net/samples/mnist_dataset.zip?st=2017-09-29T18%3A29%3A00Z&se=2099-12-31T08%3A00%3A00Z&sp=rl&sv=2016-05-31&sr=c&sig=PmhL%2BYnYAyNTZr1DM2JySvrI12e%2F4wZNIwCtf7TRI%2BM%3D) into the current folder.

For GNU/Linux users:

```sh
wget "https://batchaisamples.blob.core.windows.net/samples/mnist_dataset.zip?st=2017-09-29T18%3A29%3A00Z&se=2099-12-31T08%3A00%3A00Z&sp=rl&sv=2016-05-31&sr=c&sig=PmhL%2BYnYAyNTZr1DM2JySvrI12e%2F4wZNIwCtf7TRI%2BM%3D" -O mnist_dataset.zip
unzip mnist_dataset.zip
```

- Download ConvNet_MNIST.cntk config file into the current folder:

For GNU/Linux users:

```sh
wget "https://raw.githubusercontent.com/Azure/BatchAI/master/recipes/CNTK/CNTK-GPU-BrainScript-Distributed/ConvNet_MNIST.cntk?token=AcZzrWPVqDDfb6ig-y98_6af-Fj3R9piks5Z4b7rwA%3D%3D" -O DistributedConvNet_MNIST.cntk
```

- Create an Azure File Share with `nmist_database` and `cntk_sample` folders and upload MNIST database and BrainScript DistibutedConvNet_MNIST.cntk config file:

```sh
az storage share create --name batchaisample
az storage directory create --share-name batchaisample --name mnist_database
az storage file upload --share-name batchaisample --source Train-28x28_cntk_text.txt --path mnist_database
az storage file upload --share-name batchaisample --source Test-28x28_cntk_text.txt --path mnist_database
az storage directory create --share-name batchaisample --name cntk_samples
az storage file upload --share-name batchaisample --source DistributedConvNet_MNIST.cntk --path cntk_samples
```

### Cluster

For this recipe we need two nodes GPU cluster (`min node = max node = 2`) of `Standard_NC6` size (one GPU) with standard Ubuntu LTS (`UbuntuLTS`) or Ubuntu DSVM (```UbuntuDSVM```) image and Azure File share `batchaisample` mounted at `$AZ_BATCHAI_MOUNT_ROOT/external`.

#### Cluster Creation Command

For GNU/Linux users:

```sh
az batchai cluster create -n nc6 -i UbuntuDSVM -s Standard_NC6 --min 2 --max 2 --afs-name batchaisample --afs-mount-path external -u $USER -k ~/.ssh/id_rsa.pub
```

For Windows users:

```sh
az batchai cluster create -n nc6 -i UbuntuDSVM -s Standard_NC6 --min 2 --max 2 --afs-name batchaisample --afs-mount-path external -u <user_name> -p <password>
```

### Job

The job creation parameters are in [job.json](./job.json):

- Two input directories with IDs `CONFIG` and `DATASET` to allow the job to find the sample config and MNIST Database via environment variables `$AZ_BATCHAI_INPUT_CONFIG` and `$AZ_BATCHAI_INPUT_DATASET`;
- stdOutErrPathPrefix specifies that the job should use file share for standard output and error streams;
- An output directory with ID `MODEL` to allow job to find the output directory for the model via `$AZ_BATCHAI_OUTPUT_MODEL` environment variable;
- node_count defining how many nodes will be used for the job execution;
- path and parameters for running DistributedConvNet_MNIST.cntk;
- ```microsoft/cntk:2.1-gpu-python3.5-cuda8.0-cudnn6.0``` docker image will be used for job execution.

Note, you can remove docker image information to run the job directly on DSVM.

#### Job Creation Command

```sh
az batchai job create -n distributed_cntk --cluster-name nc6 -c job.json
```

Note, the job will start running when the cluster finished allocation and initialization of the node.

### Get Help

The Azure CLI has built-in help documentation, which you can run from the command line:

```sh
az [command-group [command]] -h
```

For example, to get information about all Azure Batch AI categories, use:

```sh
az batchai -h
```

To get help with the command to create a cluster, use:

```sh
az batchai cluster create -h
```

You can use [CLI Quickstart](https://docs.microsoft.com/en-us/azure/batch-ai/quickstart-cli) as end-to-end example of CLI usage.
