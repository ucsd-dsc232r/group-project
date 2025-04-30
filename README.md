# Group Project Information

## Login to SDSC Expanse
Go to [portal.expanse.sdsc.edu](https://portal.expanse.sdsc.edu/) and login with your "ucsd.edu" credentials, unless instructed otherwise, in which case, you may use your "access-ci.org" credentials (if you have one).

<br>
<br>

## Portal Navigation
Once you are logged in, you will see the following SDSC Expanse Portal, along with several Pinned Apps. 

![Portal Navigation](images/portal-navigation.png "Portal Navigation")

We will be working with the following apps:
1. **expanse Shell Acess**: To access Terminal.
2. **Jupyter**: To setup Jupyter notebook and run JupyterLab.
3. **Active Jobs**: To check jobs' status.

<br>
<br>

## 1. expanse Shell Acess and First Time Login
Click on the "expanse Shell Access" app in the SDSC Expanse Portal. Once you are in the Terminal, please run the following commands: 
```shell
# Create a new folder with your username and add symbolic link
ln -sf /expanse/lustre/projects/uci150/$USER

# Add symbolic link to `esolares` folder where the singularity
# images are stored
ln -sf /expanse/lustre/projects/uci150/esolares

# (Optional)
# To see your group members folders, do the following for each 
# group member's usernames
ln -sf /expanse/lustre/projects/uci150/GROUPMEMBERUSERNAME
```
>Note that you need to run the above instructions only once when accessing the Portal for the first time. 

<br>
<br>

## 2. Jupyter
Click on the "Jupyter" app in the SDSC Expanse Portal. 

![Jupyter Portal App](images/jupyter-icon.png "Jupyter Portal App")

It will allow you to request a new Jupyter Session as shown below:

![Jupyter Session](images/jupyter-config.png "Jupyter Session")

You will need to fill out the following fields: 
- **Account**: `TG-CIS240277`

- **Partition**: `shared`

- **Time limit (min)**: Enter an integer value denoting the number of minutes you want to work on your notebook environment.

- **Number of cores**: Enter an integer value denoting the number of cores your pyspark session with need. Enter a value between `2` and `128`.

- **Memory required per node (GB)**: Enter an integer value denoting the total memory required. Initially, start with a value of `2`, i.e., 2GB. You may increase it if you get issues where you need more than 2GB per executor in your Spark Session (Spark will let you know about the amount of RAM being too low when loading your datasets). The maximum value allowed is `250`, i.e., 250GB. 

  > For example, if you have 128GB of total memory and 8 cores, each core gets 128/8 = 16GB of memory.

- **Singularity Image File Location**: `~/esolares/spark_py_latest_jupyter_dsc232r.sif`

- **Environment Modules to be loaded**: `singularitypro`

- **Working Directory**: `home`

- **Type**: `JupyterLab`

Once you have filled out the above fields, go ahead and click "Submit".

![Jupyter Config](images/jupyter-config2.png "Jupyter Config")

<br>
<br>

### 2.1. JupyterLab
After clicking "Submit", the SDSC Expanse Portal will put your job request in a Queue. Based on the availability of resources, this might take some time. Once the request is processed, it will open a JupyterLab session. Here you can navigate around and create your own Python3 Jupyter notebooks. 

![JupyterLab](images/jupyterlab.png "JupyterLab")

<br>
<br>

### 2.2. Spark Session Builder
Based on the configurations provided in **Jupyter** above, you need to update the following code to build your `SparkSession`. 
> For example, if you have 128GB of total memory and 8 cores, each core gets 128/8 = 16GB of memory. The driver can take 1 or more cores and executors can take the remaining cores (7 or less).
```py
sc = SparkSession.builder \
    .config("spark.driver.memory", "16g") \
    .config("spark.executor.memory", "16g") \
    .config('spark.executor.instances', 7) \
    .getOrCreate()
```

>Driver memory is the memory required by the master node. This can be similar to the executor memory as long as you are not sending a lot of data to the driver (i.e., running collect() or other heavy shuffle operation).

![Spark Session](images/spark-session.png "Spark Session")

Example Spark notebooks are available at `~/esolares/spark_notebooks`.

<br>
<br>

## 3. Active Jobs
Click on the "Active Jobs" app in the SDSC Expanse Portal. Please use this while debugging Spark jobs. Note the job `Status` and `Reason`. If the job was recently run and is dead, you will see the reason why it died under the `Reason` field. 

![Active Jobs](images/active-jobs.png "Active Jobs")


<br>
<br>


## FAQs
[FAQs](FAQs.md "FAQs")

<br>
<br>

## Support
If you are having trouble, please submit a ticket to https://support.access-ci.org/.
