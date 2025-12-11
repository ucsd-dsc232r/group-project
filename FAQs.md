# FAQs

1. **How to create a Shared folder in SDSC server?**

    You would need to create a `shared` folder inside of `uci150/$USER` and then run: 
    ```shell
    chmod g+w shared
    ```
    
    This is only for the `shared` folder inside `uci150/$USER`. Creating a folder inside of `home` directory is extremely slow and has a small quota comparatively. However, `uci150` is fast and on the distributed file system. 

    <br>
    <br>

2. **How do I install AWS CLI in SDSC server?**
   
    Since this is a shared resource, you cannot install packages into the operating system. Instead you need to install them local in your `$HOME/bin` folder.

    ```shell
    # first create the `bin` folder in your home directory. This is to be done without singularity
    mkdir -p $HOME/bin/aws-cli
    cd $HOME/bin/aws-cli
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    ./aws/install --bin-dir $HOME/bin/aws-cli --install-dir $HOME/bin/aws-cli --update
    ```

    We grabbed this from the aws website and modified it for hpc install. You will need to go to https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html.

    In order to put it in your path, you will need to run:
    
    ```shell
    PATH=$HOME/bin/aws-cli:$PATH
    ```    
    You will need to do this everytime you log in.

    <br>
    <br>

3. **What is the maximum memory available?**

    You have a total of 250GB of memory available.

    <br>
    <br>


4. **What is the maximum number of executors available?**

    You can have a maximum of 128 executors. 

    <br>
    <br>


5. **Please provide an example SparkSession configuration.**

    **The Formula:**
    ```
    Driver memory = 1-2GB (fixed, small)
    Executor instances = Total Cores - 1  (reserve 1 for driver)
    Executor memory = (Total Memory - Driver Memory) / Executor Instances
    ```

    **Example:** 8 cores with 128GB memory:
    - Driver: **2GB** (fixed)
    - Executors: 8 - 1 = **7 instances**
    - Per executor: (128GB - 2GB) / 7 = **18GB each**

    ```py
    spark = SparkSession.builder \
        .config("spark.driver.memory", "2g") \
        .config("spark.executor.memory", "18g") \
        .config('spark.executor.instances', 7) \
        .getOrCreate()
    ```

    For more examples and detailed explanations, see the **[Spark HPC Best Practices Guide](SPARK_HPC_BEST_PRACTICES.md)**.

    <br>
    <br>

6. **Why should driver memory be small (1-2GB)?**

    The driver coordinates the job but doesn't process data—executors do. The driver only:
    - Builds the execution plan (DAG)—just metadata
    - Schedules tasks—lightweight tracking
    - Receives small results like `count()` or `mean()`

    **1-2GB is sufficient** for most workloads. Giving the driver more memory wastes resources that could go to executors where the actual data processing happens.

    **Exceptions (consider 4GB driver):**
    - Interactive Jupyter analysis with frequent `toPandas()` for visualization
    - Broadcasting lookup tables > 1GB
    - ML training pipelines with large models
    - Processing 10,000+ small files

    See the **[Spark HPC Best Practices Guide](SPARK_HPC_BEST_PRACTICES.md)** for detailed caveats and trade-offs.

    <br>
    <br>

7. **I'm getting out-of-memory errors. What should I do?**

    1. **Increase memory per core:** Request more total memory in your Jupyter session
    2. **Check your code:** Avoid `collect()` on large datasets
    3. **Use Parquet:** Write intermediate results to Parquet instead of keeping in memory
    4. **Repartition:** If data is skewed, use `repartition()` to balance partitions

    See the **[Spark HPC Best Practices Guide](SPARK_HPC_BEST_PRACTICES.md)** for detailed troubleshooting.

    <br>
    <br>

