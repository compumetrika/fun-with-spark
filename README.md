# Fun With Spark - README

Parallel computation is one of the easiest ways to throw computational power at a problem. Technological development has made it cheaper and cheaper for the average person to do this. 

Here's an example. A long time ago [PiCloud](https://web.archive.org/web/20130805174353/http://www.picloud.com/) made access to "effectively unlimited" parallel computation (on Amazon Web Service Service) astonishingly easy from Python. Their homepage (if it still loads well enough from archive.org) shows the simple example: load a [library](https://pypi.python.org/pypi/cloud), spool up some remote machines, run your function on the remote mahines, gather results. It really was that easy -- more than once I executed hundreds of machine-hours of dissertation research over an evening. [Pricing](https://web.archive.org/web/20130727210823/http://www.picloud.com:80/pricing/) was calculated by the *millisecond*, starting at $0.05 per 1.2 Ghz CPU-hour of work.

Unfortunately their business model did [not appear sustainable](https://www.crunchbase.com/organization/picloud/timeline#/timeline/index) -- they were bought out by Dropbox (success for the creators!) and since then I havev found no equivalent, open-source tool to get bespoke computation-intensive Python running on scalable cloud computing, which *didn't* require extensive human capital creation and lots of time to set up. There are plenty of powerful ways to set up a grid and interface, but they require lots of skilled effort. PiCloud was painless. 

I'm hoping Spark will turn out to be the near-next-best thing. It's survived the rough "3-year over-hype" period that many open-source techcnologies with lots of *promise* experience, so I think it's around to stay. Let's see what we can do with it. This repo is my "sandbox" to answer that question. 

(I'm also very intrigued by the Jupyter/IPython parallelization options, and hope *that* might also become an easy drop-in replacement for PiCloud. My experiences creating a Jupyternotebook on AWS, eg. following [this](http://ipyparallel.readthedocs.io/en/latest/) and others, was a lot of work but doable. Another repo will explore that option.)

Here are some sections:

## 1.0 Installing PySpark

These steps are for (X)Ubuntu 16.04. They will likely work for other Ubuntu/Debian flavors, and high-level variations on this theme should work for non-Linux systems. I use the steps from [Datacamp here](https://www.datacamp.com/community/tutorials/apache-spark-python), as inspiration, with my own variations.

Note: all instructions assume you are using the command line in Ubuntu or a Ubuntu variant. 

Steps:

1. Open a command-line terminal, create a directory for this project and navigate there. These commands will create a folder in your Ubuntu home directory:

    ```
    cd ~                                # Navigate to home directory    
    mkdir spark-project
    cd spark-project
    ```
    
2. Ensure you have Java JDK by typing the following on the terminal:

    ```
    java -version # or    
    javac -version
    ```

    - If nothing comes up, you need to install java. On Ubuntu, you can follow these steps to install. On the command line type these one at a time. This will add the Java repository and install Java 8: 

    ```
    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update
    sudo apt-get install oracle-java8-installer 
    ```
    
3. Download the latest stable Spark. Two options: (i) manually download via the webpage of (ii) use the command line. 
    a. Manual web browser: navigate the the [Spark download page](https://spark.apache.org/downloads.html) and download the most recent default (I used a .edu mirror). Use the default options (Spark release 2.2.0; package type pre-built for Hadoop 2.7+; download type direct download) and click through the links to download both the compressed file and the checksum to your "spark-project" directory. For checksum I use sha; read more [here](https://www.openoffice.org/download/checksums.html).
    b. Command line options: 
    
    ```
    cd ~/spark-project                  # if not already there
    wget http://www.gtlib.gatech.edu/pub/apache/spark/spark-2.2.0/spark-2.2.0-bin-hadoop2.7.tgz  # download spark
    wget https://archive.apache.org/dist/spark/spark-2.2.0/spark-2.2.0-bin-hadoop2.7.tgz.sha     # download sha checksum
    ```
    
4. [Optional but recommended] Generate the file checksum: 

    ```
    cd ~/spark-project                  # if not already there
    sha512sum spark-2.2.0-bin-hadoop2.7.tgz > my_checksum_output   # save checksum output
    ```

5. [Optional but recommended] To compare checksums, I'll use Python. The ">>>" lines are in the python command line. You can copy all of the lines with ">>>" as a block, and in ipython type "%paste" and they will execute. You'll want to see "True" printed out by Python. If you get False you should manually compare the checksums; if they do not agree this may indicate that the file you downloaded has been tampered with in some way. You may need to re-download from an alternate source.
    
    ```
    cd ~/spark-project                  # if not already there
    ipython
    ```
    
    
    ```python
    >>> with open("spark-2.2.0-bin-hadoop2.7.tgz.sha", "r") as f:
    >>>     webpage_checksum_raw_string = f.read().replace('\n', '')  # Read entire line into a string. For windows may need import os; os.linesep
    >>> webpage_checksum = ''.join(webpage_checksum_raw_string.split()[1:]).lower()  # 4 string manipulations in one line!
    >>> with open("my_checksum_output", "r") as f:
    >>>     my_checksum_raw_string = f.read().replace('\n', '')  # Read entire line into a string. Ubuntu endline.
    >>> my_checksum = my_checksum_raw_string.split()[0]
    >>> print("\nChecksums are equal: " + str(webpage_checksum == my_checksum))
    ```

6. Unpack it and move to correct location:

    ```
    cd ~/spark-project                  # if not already there    
    tar -xf spark-2.2.0-bin-hadoop2.7.tgz
    mv spark-2.2.0-bin-hadoop2.7 ~/
    ```

7. Update your .bashrc to point to the spark/bin:

    ```
    cd ~                                # move to home directory    
    cp ~/.bashrc ~/.bashrc-pyspark.bak  # create backup .bashrc, always good idea
    sed -i '$a export PATH="~/spark-2.2.0-bin-hadoop2.7/bin/:$PATH"' .bashrc    # Append this export command to end of file   
    ```

8. Open a **new terminal command line window**  and ensure that you can start pyspark from command line, or [spark-submit](https://spark.apache.org/docs/latest/submitting-applications.html) a job. These should spew a lot of text and eventually say "Spark 2.2.0" 

    ```
    pyspark  # or        
    pyspark --master local[4]
    ```
    
8. Let's test it with an example file -- if successful should see "Pi is roughly 3.1xxxx..." in mess of results:

    ```
    cd ~/spark-2.2.0-bin-hadoop2.7/examples/src/main/python
    spark-submit pi.py
    ```
    
9. Now the good stuff: we can use a Spark-powered [IPython command line](https://spark.apache.org/docs/latest/programming-guide.html#using-the-shell) as follows:

    ```
    PYSPARK_DRIVER_PYTHON=ipython pyspark    
    ```
    
10. ... or start a Spark-powered Jupyter notebook by executing the following three lines on your command line; hints [here](https://github.com/zipfian/spark-install/blob/master/README.md):

    ```    
    export PYSPARK_DRIVER_PYTHON=jupyter
    export PYSPARK_DRIVER_PYTHON_OPTS="notebook --NotebookApp.open_browser=True --NotebookApp.ip='localhost' --NotebookApp.port=8898"
    pyspark
    ```


## 2.0 Running a Basic, Local Example

Please see this notebook: [Spark Simple Regression Example](https://github.com/compumetrika/fun-with-spark/blob/master/Spark-Regression-Example.ipynb)

## 3.0 Running on the Cloud

Coming soon.

## Appendix A: Some Resources

Pages I found useful:

- [Install Java 8, Ubuntu 16.04](Some resource links: https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-get-on-ubuntu-16-04)
- Python Spark [docs from Apache](https://spark.apache.org/docs/latest/)
    - [Programming guide](https://spark.apache.org/docs/latest/programming-guide.html)
- [Install Spark](https://www.datacamp.com/community/tutorials/apache-spark-python)
- OpenOffice has a nice page on [checksums](https://www.openoffice.org/download/checksums.html) here. I try to use SHA512, which is commonly used. See [here](https://askubuntu.com/questions/61826/how-do-i-check-the-sha1-hash-of-a-file) for how to check/generate sha checksums. Note that sha1sum is the example stackoverflow provides, but typically you'll encounter sha512sum files. 
- Running IPython/Jupyter notebooks under PySpark:
    - [Spark homepage instructions](https://spark.apache.org/docs/0.9.0/python-programming-guide.html)
    - Tutorials: [Supergloo tutorial](https://www.supergloo.com/fieldnotes/apache-spark-ipython-notebook-easy-way/), [Dataquest.io](https://www.dataquest.io/blog/pyspark-installation-guide/), [John Ramey](http://ramhiser.com/2015/02/01/configuring-ipython-notebook-support-for-pyspark/)
- Running Spark clusters on AWS -- things to try:
    - Insight Data Labs Blog: 
        - [Spark Cluster Step by Step](http://blog.insightdatalabs.com/spark-cluster-step-by-step/)
        - [Running a massive job via Jupyter](http://blog.insightdatalabs.com/jupyter-on-apache-spark-step-by-step/)
    - [Jupyter notebook on AWS](https://medium.com/@josemarcialportilla/getting-spark-python-and-jupyter-notebook-running-on-amazon-ec2-dec599e1c297)
- [Old Faithful data from R](http://www.stat.cmu.edu/~larry/all-of-statistics/=data/faithful.dat)
    - s/eruptions/duration  (for clarity)


### A.1 - A Note on Checksums

Assume you've downloaded an ".sha" file, but you you may notice that Ubuntu has multiple sha checksum programs -- sha1, sha128, sha512, etc.  

You can determine which to use by opening up the {filename}.sha and counting -- the number of characters in the checksum times 4 is the checksum bitnumber. 

For example, the checksum I downloaded for spark looks like this:

    spark-2.2.0-bin-hadoop2.7.tgz: 7A186A2A 007B2DFD 880571F7 214A7D32 9C972510
                                   A460A8BD BEF9F7F2 A8910193 43C020F7 4B496A61
                                   E5AA42BC 9E9A79CC 99DEFE5C B3BF8B6F 49C07E01
                                   B259BC6B

They helpfully arange it into groups of eight, in rows of five, so I know that there are 8\*16 characters. Since sha uses hexidecimal (numbers and letters A-F), each character needs 4 bits to represent it (sum([2\*\*x for x in range(4)]) + 1), so total bits needed = 8\*16\*4 = 512 -- we'll need to use the sha512sum function. 
