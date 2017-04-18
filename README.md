# Fun With Spark

Parallel computation is one of the easiest ways to throw computational power at a problem. Technological development has made it cheaper and cheaper for the average person to do this. 

Here's an example. A long time ago [PiCloud](https://web.archive.org/web/20130805174353/http://www.picloud.com/) made access to "effectively unlimited" parallel computation (on Amazon Web Service Service) astonishingly easy from Python. Their homepage (if it still loads well enough from archive.org) shows the simple example: load a [library](https://pypi.python.org/pypi/cloud), spool up some remote machines, run your function on the remote mahines, gather results. It really was that easy -- more than once I executed hundreds of machine-hours of dissertation research over an evening. [Pricing](https://web.archive.org/web/20130727210823/http://www.picloud.com:80/pricing/) was calculated by the *millisecond*, starting at $0.05 per 1.2 Ghz CPU-hour of work.

Unfortunately their business model did [not appear sustainable](https://www.crunchbase.com/organization/picloud/timeline#/timeline/index) -- they were bought out by Dropbox (success for the creators!) and since then I havev found no equivalent, open-source tool to get bespoke computation-intensive Python running on scalable cloud computing, which *didn't* require extensive human capital creation and lots of time to set up. There are plenty of powerful ways to set up a grid and interface, but they require lots of skilled effort. PiCloud was painless. 

I'm hoping Spark will turn out to be the near-next-best thing. It's survived the rough "3-year over-hype" period that many open-source techcnologies with lots of *promise* experience, so I think it's around to stay. Let's see what we can do with it. This repo is my "sandbox" to answer that question. 

(I'm also very intrigued by the Jupyter/IPython parallelization options, and hope *that* might also become an easy drop-in replacement for PiCloud. My experiences creating a Jupyternotebook on AWS, eg. following [this](http://ipyparallel.readthedocs.io/en/latest/) and others, was a lot of work but doable. Another repo will explore that option.)

Here are some sections:

## 1.0 Installing PySpark

These steps are for (X)Ubuntu 16.04. They will likely work for other Ubuntu/Debian flavors, and high-level variations on this theme should work for non-Linux systems. I use the steps from [Datacamp here](https://www.datacamp.com/community/tutorials/apache-spark-python), as inspiration, with my own variations.

Steps:

1. Ensure you have Java JDK (currently 8 is most stable/recommended):

    java -version # or
    
    javac -version

2. If not, install:

    sudo add-apt-repository ppa:webupd8team/java
    
    sudo apt-get update
    
    sudo apt-get install oracle-java8-installer 

3. Download the latest stable Spark:
    - From the [download page](https://spark.apache.org/downloads.html), grab most recent default (I used a .edu mirror)
        - I use default options (Spark release 2.1.0; package type pre-built for Hadoop 2.7+; download type direct download)
    - Pull appropriate checksum from the link at step 5 and check it (see more in the section below, "A Note on Checksums")
        - I use sha; read more [here](https://www.openoffice.org/download/checksums.html) for general info
        - Download the appropriate ".sha" file
        - On ubuntu, I use "sha512sum 'filename'"
        - This spits out a long single lowercase string. The "{filename}.sha" file has the number in uppercase ad split into chunks. 
        - To compare, I manually construct a single string from the "{filename}.sha", and use Python to compare them. See the code snippet below.

4. Unpack it and move to correct location:

    tar -xf spark-2.1.0-bin-hadoop2.7.tgz
    
    mv spark-2.1.0-bin-hadoop2.7  ~/

5. Update your .bashrc to point to the spark/bin:

    cp ~/.bashrc ~/.bashrc-pyspark.bak  # create backup .bashrc, always good idea
    
    vim ~/.bashrc     # Sorry Chris!

6. Add this line at the end of the file, where you can replace "~/" with "/home/{your_username}/" if you want.

    export PATH="~/spark-2.1.0-bin-hadoop2.7/bin/:$PATH"

7. Open new command line window and ensure that you can start pyspark from command line, or [spark-submit](https://spark.apache.org/docs/latest/submitting-applications.html) a job:

    ~/spark-2.1.0-bin-hadoop2.7/bin/pyspark  # or
    
    ~/spark-2.1.0-bin-hadoop2.7/bin/pyspark --master local[4]

8. Let's test it with an example file -- if successful should see "Pi is roughly 3.14xxx..." in mess of results:

    cd ~/spark-2.1.0-bin-hadoop2.7/examples/src/main/python
    
    ~/spark-2.1.0-bin-hadoop2.7/bin/spark-submit pi.py

9. Now the good stuff: we can use a Spark-powered [IPython command line](https://spark.apache.org/docs/latest/programming-guide.html#using-the-shell) as follows:

    PYSPARK_DRIVER_PYTHON=ipython ~/spark-2.1.0-bin-hadoop2.7/bin/pyspark    

10. ... or start a Spark-powered Jupyter notebook, hints [here](https://community.hortonworks.com/articles/75551/installing-and-exploring-spark-20-with-jupyter-not.html):
    
    PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS='notebook' ~/spark-2.1.0-bin-hadoop2.7/bin/pyspark # or 
    
    PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS='notebook' PYSPARK_PYTHON=python2 ~/spark-2.1.0-bin-hadoop2.7/bin/pyspark # if py3 
    



### 1.1 Code snippet comparing checksums:
        
    # Manually constructed hash:
    hash_from_file = "3FC94096AE34F9A1A148D37E5ED640A7E5DE1812F1F2ECD715D92BBF2901E895CF4B93E6D8EE0D64DEBB5DF7C56D673C0A36E5FC49503EC0F4507EB0EDF961A4"
    # Hash from checksum utility:
    hash_from_sha512sum = "3fc94096ae34f9a1a148d37e5ed640a7e5de1812f1f2ecd715d92bbf2901e895cf4b93e6d8ee0d64debb5df7c56d673c0a36e5fc49503ec0f4507eb0edf961a4"
    # Compare using Python string utilities:
    hash_from_file.upper() == hash_from_sha512sum.upper()

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

Assume you've downloaded an ".sha" file, but you don't know which checksum it is -- sha1, sha128, sha512? 

You can check by opening up the {filename}.sha and counting -- the number of characters in the checksum times 4 is the checksum bitnumber. 

For example, the checksum I downloaded for spark looks like this:

    spark-2.1.0-bin-hadoop2.7.tgz: 3FC94096 AE34F9A1 A148D37E 5ED640A7 E5DE1812
                                   F1F2ECD7 15D92BBF 2901E895 CF4B93E6 D8EE0D64
                                   DEBB5DF7 C56D673C 0A36E5FC 49503EC0 F4507EB0
                                   EDF961A4

They helpfully arange it into groups of eight, in rows of five, so I know that there are 8\*16 characters. Since sha uses hexidecimal (numbers and letters A-F), each character needs 4 bits to represent it (sum([2\*\*x for x in range(4)]) + 1), so total bits needed = 8\*16\*4 = 512 -- we'll need to use the sha512sum function. 
