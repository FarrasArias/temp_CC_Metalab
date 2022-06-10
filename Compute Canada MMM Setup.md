[![N|Solid](https://drive.google.com/uc?export=view&id=1u4xiWN3s0PAii8zn3-qxJ7wn35tBOypY)](https://metacreation.net/category/projects/)

# Compute Canada MMM Setup
## mmm_api Installation

If you're unfamiliar with Compute Canada, make sure to check the introductory .md [here](https://github.com/FarrasArias/temp_CC_Metalab/blob/main/Compute%20Canada%20Guide.md).

1. First, make sure to clone the MMM_API into a folder in your CC machine.
```sh
https://<TOKENNUMBERS>@github.com/jeffreyjohnens/MMM_API.git
```
2. First we need to load python 3.6.10.
```sh
module load python/3.6.10
```
3. Then we must create an environment.
```sh
virtualenv --no-download ./ENV                # ENV is the name of the environment
source ./ENV/bin/activate
pip install --no-index --upgrade pip          
pip install --no-index --upgrade setuptools   # probably not necessary

```
4. Then we must install torch
```sh
pip install torch==1.6.0
```
> **Note:** if possible, when installing other dependencies you might need, make sure to use the --no-index flag to install from the Compute Canada Wheels

> **Note:** Maybe the 2 following steps can now be ignored, since we're always working with StdEnv/2020 now.

5. Deactivate the environment once you installed all packages
```sh
deactivate
```
6. Now we must purge the loaded modules.
```sh
module --force purge
```
7. After this, we can load the protobuf module, first loading the correct environment.
```sh
module load StdEnv/2020 protobuf python/3.6.10
```
8. Once we do this, we can activate the python3.6.3 environment. This will allow us to have a python 3.6.3 env and use the correct version of protobuf.
```sh
./ENV/bin/activate
```
9. Now, navigate into the MMM_API python folder and download pybind.
```sh
cd MMM_API/python
rm -r pybind11
git clone https://github.com/pybind/pybind11.git
```
10. That's it. Go back to the main MMM_API folder and run the following:
```sh
bash python_build.sh
```
Everything should get installed correctly in your python environment! If you log out and back in to CC make sure to activate the environment in which you installed the mmm_api.

## MMM Training

First, download the MMM_TRAINING project [here](https://gitlab.com/jeffreyjohnens/MMM_TRAINING/-/tree/master/). You can transfer it through Globus to CC or download it directly there with git.

### Dataset Building

In order to train a new model, you must first build a dataset. You can upload the files you need using Globus (check the CC [guide]()). To check an example of the structure of the folders in your dataset, you can check the DDD_split.zip file in the shared folder (projects/def-pasquier). 
> **Note**: Remember that to copy from the shared folder to your own folders you must use absolute paths.

Once you have the folder with the data, run the following command
```sh
python3 build_dataset.py --data_dir /scratch/USERNAME/DATASET_NAME --output /scratch/USERNAME/data.arr --nthreads 40 
```

### Training a Model

To train a model, run the train.py file. Different lab members have managed to set the paths differently. What works for me is to use global paths. An example would be:
```sh
python train.py --arch gpt2 --config /home/raa60/scratch/MMM_TRAINING-master/config/gpt2_tiny.json --encoding EL_VELOCITY_DURATION_POLYPHONY_YELLOW_FIXED_ENCODER --ngpu 4 --dataset /home/raa60/scratch/farrastest_NUM_BARS=4_OPZ_False.arr --batch_size 32 --label DELETE_ME
```
Important things to look out for:
1. The choice of encoder matters. Right now, Jeff recommedns using EL_VELOCITY_DURATION_POLYPHONY_YELLOW_FIXED_ENCODER.

There are currently 2 main models implemented: GPT2 and TransformerXL. In order to use one or the other, change the argument arch to `gpt2` or `xl` and change to the correspinding config file (you can find them in the `MMM_TRAINING-master/config/` directory).

### Training with different HuggingFace models

The arguments **arch** and **config** are the ones that need to be changed in order to use the pipeline to train another model.

For example, let's say you want to use the [reformer](https://huggingface.co/docs/transformers/model_doc/reformer) to train a model. Check the HuggingFace documentation for the [Config](https://huggingface.co/docs/transformers/model_doc/reformer#transformers.ReformerConfig) and [Head](https://huggingface.co/docs/transformers/model_doc/reformer#transformers.ReformerModelWithLMHead) objects. 

Create a new reformer.json file in the `MMM_TRAINING-master/config/` directory and add the parameters (based on the Config information from the HuggingFace Website) you want/must change for the model to work with the pipeline (this usually means to at least change the input dimensions to 512).

Then, we must add the reformer argument to train.py:

```sh
if args.arch == "gpt2":
    config = GPT2Config().from_json_file(args.config)
    model_cls = GPT2LMHeadModel
elif args.arch == "xl":
    config = TransfoXLConfig().from_json_file(args.config)
    model_cls = TransfoXLLMHeadModel
#----------------- NEW CODE ---------------------
elif args.arch == "reformer":
    config = ReformerConfig().from_json_file(args.config)
    model_cls = ReformerModelWithLMHead
#-------------- NEW CODE ENDS ---------------------
```

Finally, run the command shown above with the new config file and model argument:
```sh
python train.py --arch reformer --config /home/raa60/scratch/MMM_TRAINING-master/config/reformer.json --encoding EL_VELOCITY_DURATION_POLYPHONY_YELLOW_FIXED_ENCODER --ngpu 4 --dataset /home/raa60/scratch/DATA_NUM_BARS=4_OPZ_False.arr --batch_size 32 --label DELETE_ME
```

### Running Jobs

To read the CC documentation, cick [here](https://docs.alliancecan.ca/wiki/Running_jobs). You can run small snippets of code to test things out without allocating any resources. However, to train a model or perform any time/resource consuming task, you must schedule a job. A list of different types of job scheduling will be added here.

#### Interactive Jobs
You can start an interactive session on a compute node with salloc.
```sh
salloc --time=3:0:0 --nodes 1 --cpus-per-task 32 --mem=128000 --account=def-pasquier
```

#### Scheduled jobs (use this for training)
For time-expensive tasks it is better to create a bash file and submit a job with sbatch:
```sh
sbatch simple_job.sh
```

Here is an example of the contents of a bash file to submit a mmm training job:
```sh
#!/bin/bash
#SBATCH --gres=gpu:v100l:4
#SBATCH --cpus-per-task=32
#SBATCH --exclusive
#SBATCH --mem=0
#SBATCH --time=2-23:00
#SBATCH --account=def-pasquier
#SBATCH --mail-user USERNAME@sfu.ca  <---- MAKE SURE TO PUT YOUR EMAIL
#SBATCH --mail-type ALL
#SBATCH --output=CCLOG/FILENAME.out  <---- MAKE SURE TO CHANGE THE NAME OF THE FILE

source $SCRATCH/PY_3610/bin/activate   <---- THIS IS THE DIRECTORY TO THE ENV WHERE YOU HAVE THE mmm_api INSTALLED
cd $SCRATCH/MMM_TRAINING-master
module load StdEnv/2020 protobuf python/3.6.10
source $SCRATCH/PY_3610/bin/activate  <---- SAME HERE, MAKE SURE THE DIRECTORY IS PLACED CORRECTLY
python train.py --arch reformer --config /home/raa60/scratch/MMM_TRAINING-master/config/reformer.json --encoding EL_VELOCITY_DURATION_POLYPHONY_YELLOW_FIXED_ENCODER --ngpu 4 --dataset /home/raa60/scratch/dataset_NUM_BARS=4_OPZ_False.arr --batch_size 32 --label DELETE_ME
```

In this case we are using 4 v1001 GPUs (**gres** argument) and we're asking for 2 days and 23 hours of time to run the job (**time** argument).
