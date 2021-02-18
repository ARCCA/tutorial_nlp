---
title: "Example job"
teaching: 20
exercises: 0
questions:
- "How do I approach running code?"
- "What software should I user?"
objectives:
- "Run a job that is relevant for NLP work"
keypoints:
- "Once you have run one job, the others are very similar."
---

> ## Training available!
>
> The training course [An Introduction to Machine Learning Applications]({{site.url}}/An-Introduction-to-Machine-Learning-Applications)
> may be suitable to learn further.
{: .callout}

## Example job


The example job is from [Kaggle](https://www.kaggle.com), specifically [AG News Classification
Dataset](https://www.kaggle.com/amananandrai/ag-news-classification-dataset).

The specific task will be:

```
This dataset contains news articles with 4 different genres namely "world news" , "sports news" , "business news" and "science and technology". These categories are represented by 1,2,3 and 4 in the class id respectively. The task requires to classify the articles accurately in the different categories (genres).
Also, preprocess the data and do EDA for the data set.
```
{: .source}

## Setting up an environment

Lets first upload some data.  This can be accomplished with Filezilla or
via the command line:

```
$ scp train.csv.zip c.username@hawklogin.cf.ac.uk:
$ scp test.csv.zip c.username@hawklogin.cf.ac.uk:
```
{: .language-bash}

The files can then be unzipped where required.

One method would be to use the
[Transformers](https://huggingface.co/transformers) library we could perform the following.

~~~
$ module load anaconda/2020.02
$ . activate
$ conda create -n NPL jupyter
$ conda activate NPL
$ conda install pytorch torchvision torchaudio cudatoolkit=11.0 -c pytorch -c conda-forge
$ conda install pandas
$ pip install transformers simpletransformers
~~~
{: .language-bash}

Notice the mix of Anaconda and Pip.  Not ideal but seems to work in this place.

If you know which Transformers model you will be using it will need to be downloaded.  The compute nodes do not have
Internet Access by default (certain addresses can be opened up).  One way to make sure Transformers downloads the
correct file is to run a small portion of the code on the login node.  E.g.

```
from simpletransformers.classification import ClassificationModel, ClassificationArgs

model = ClassificationModel(
    "bert", "bert-base-cased", num_labels=4, use_cuda=False, args={"reprocess_input_data": True, "overwrite_output_dir": True}
)
```
{: .language-python}

This will cache the model for us before running the actual code on the compute/GPU nodes.

## Traditional approach

We would now run this within a Slurm job or interactive session.  For example an interacive session (would have to wait
for resource to be free) can be started with:

```
$ srun -p c_gpu_diri1 --gres=gpu:2 -A scwXXXX --pty bash --login
```
{: .language-bash}

The run `python` to get a prompt.

Alternatively if we know the code to run we can copy the following to `npl.py`

```
import pandas as pd
from simpletransformers.classification import ClassificationModel, ClassificationArgs
import logging

logging.basicConfig(level=logging.INFO)
transformers_logger = logging.getLogger("transformers")
transformers_logger.setLevel(logging.WARNING)

train_data = pd.read_csv('/home/c.username/npl_tut/train.csv')
test_data = pd.read_csv('/home/c.username/npl_tut/test.csv')

train_data['text'] = train_data['Title'] + ' ' + train_data['Description']
test_data['text'] = test_data['Title'] + ' ' + test_data['Description']

train_data['labels'] = train_data['Class Index']-1
test_data['labels'] = test_data['Class Index']-1

train_data = train_data.drop(columns=['Title', 'Description', 'Class Index'])
test_data = test_data.drop(columns=['Title', 'Description', 'Class Index'])


model = ClassificationModel(
    "bert", "bert-base-cased", num_labels=4, args={"reprocess_input_data": True, "overwrite_output_dir": True}
)


model.train_model(train_data)
```
{: .language-python}

And then write a job script, for example called `npl.sh` with the following:

```
#!/bin/bash --login
#SBATCH -p c_gpu_diri1
#SBATCH -A scw1001
#SBATCH --gres=gpu:2
#SBATCH -t 1:00:00

module load anaconda/2020.02
. activate

conda activate NPL

python npl.py
```
{: .language-bash}

And submit with:

```
$ sbatch npl.sh
```
{: .language-bash}

This should provide you with output in the location you submitted the job, e.g. `slurm-20880.out` and `slurm-20880.err`.
However we have opportunities to use other technologies.

## Jupyter

This seems to be a good opportunity to use a Jupyter Notebook.  We can run on Slurm within the environment setup above with:

~~~
$ srun -n 1 -p c_comsc_diri1 --gres=gpu:1 --account=scwXXXX -t 1:00:00 jupyter-lab --ip=0.0.0.0
~~~
{: .language-bash}

~~~
http://ccs9202:8888/lab?token=627a2a159dd6c1d81f4d52238c6e09e952ab84f4c85a0c58
     or http://127.0.0.1:8888/lab?token=627a2a159dd6c1d81f4d52238c6e09e952ab84f4c85a0c58
~~~
{: .output}

Making sure the `srun` has the correct partition and possible GPU requirements.

This can then be accessed either using ssh port forwarding or VNC. 

> ## Port forwarding
>
> Port forwarding is achieved by specifying to ssh to redirect traffic, e.g.
>
> ```
> $ ssh -L9009:ccs9202:8888 c.username@hawklogin.cf.ac.uk
> ```
> {: .language-bash}
> 
> Then the URL to connect to Jupyter will be
> `http://localhost:9009/lab?token=627a2a159dd6c1d81f4d52238c6e09e952ab84f4c85a0c58`
> {: .language-bash}


Port forwarding can be tricky for some users, VNC is
possible by connecting to `clvnc1.cf.ac.uk:5901` with a VNC viewer such as [TigerVNC](https://tigervnc.org/).  This will get you a GNOME login prompt and
eventually a Linux GNOME desktop that has a browser available to access `ccs9202` URL above.

You should now be able to connect to Jupyter. Using the code above the model has been trained on the GPU and the model
can be used to evaluate with the `test.csv` case in `test_data`.

## Early preview

We have been looking at improving access to Web interfaces such as Jupyter and VNC and discovered
[OnDemand](https://osc.github.io/ood-documentation/master/).  This provides a web-based approach to deliver HPC
services.  In Cardiff we have been testing out an instance.  For example it can:

1. Provide VNC through website.
2. Provide access to Jupyter and Rstudio directly without user having to do anything.
3. Provide SSH access via web browser.
4. Provide easy method to roll out other ways to deliver applications.

Demonstration provided during tutorial.

{% include links.md %}

