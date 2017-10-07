Please follow [instructions](/recipes/Readme.md) to install Azure CLI 2.0, configure default location, create and configure default resource group and storage account.


### Data Deployment

- Download train_mnist sample script into the current folder:

For GNU/Linux users:

```sh
wget "https://raw.githubusercontent.com/Azure/BatchAI/master/recipes/Chainer/Chainer-GPU-Distributed/train_mnist.py?token=AcZzrV-OFepRwpRSB1kyABIX-PLh2ZHqks5Z4eukwA%3D%3D" -O train_mnist.py
```

- Create an Azure File Share with `chainer_samples` folder and upload train_mnist.py into it:

```sh
az storage share create --name batchaisample
az storage directory create --share-name batchaisample --name chainer_samples
az storage file upload --share-name batchaisample --source train_mnist.py --path chainer_samples
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

- An input directory with ID `SCRIPT` to allow the job to find the sample script via environment variable `$AZ_BATCHAI_INPUT_SCRIPT`;
- stdOutErrPathPrefix specifies that the job should use file share for standard output and error streams;
- output directory with ID `MODEL` to allow the hob to find the output directory via environment variable `$AZ_BATCHAI_OUTPUT_MODEL`;
- nodeCount defining how many nodes will be used for the job execution;
- path and parameters for running train_mnist.py;
- ```batchaitraining/chainermn:openMPI``` docker image will be used for job execution.

#### Job Creation Command

```sh
az batchai job create -n distributed_chainer --cluster-name nc6 -c job.json
```

Note, the job will start running when the cluster finished allocation and initialization of the node.
