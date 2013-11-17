If you can't find a suitable set of csv files [[here|Downloads]], then will have to extract them yourself. This page explains how.

The instructions assume you have already obtained an [XML dump from Wikipedia](http://en.wikipedia.org/wiki/Wikipedia:Database_download).

**Note:** we recommend you try out this process on one of the smaller Wikipedia dumps (for example the [Simple English Wikipedia](http://dumps.wikimedia.org/simplewiki/latest/simplewiki-latest-pages-articles.xml.bz2)) before you move up to the larger ones.


##Install Hadoop

Download and install [Hadoop](http://hadoop.apache.org). Follow the [single node guide](http://hadoop.apache.org/common/docs/current/single_node_setup.html) to get it running on a 'pseudo cluster' of one computer.

The extraction process requires Hadoop to be running on at least a small cluster of computers, so follow the [cluster setup guide](http://hadoop.apache.org/common/docs/current/cluster_setup.html) to make that happen. It's not as bad as it sounds, just a matter of putting files in a common or shared location on the computers and configuring some xml files.

For larger dumps, each Task node will need to have 2-3Gb memory available to it. So, ensure that the machines in your Hadoop cluster are up to snuff, and configure `mapred.child.ulimit` and `mapred.child.java.opts` appropriately.


##Organize and upload data to HDFS

Hadoop provides a distributed filesystem called HDFS, which chunks files up and shares them across the computers in the cluster. Its a good idea to create an input and output directory to organize the files

```bash
hadoop fs -mkdir input
hadoop fs -mkdir output
```

Upload the Wikipedia dump you want to process (unzipped) to the input directory

```bash
hadoop fs -put dump.xml input
```

##Compile Wikipedia Miner JAR file

Go to the `ant` directory of the toolkit, and use the ant target ''package-hadoop'' to compile a JAR file from the Wikipedia Miner project that includes all dependencies required by hadoop.

```bash
ant package-hadoop
```

This will create the jar file `wikipedia-miner-hadoop.jar` in the `build/jar` directory.

> **Version 1.2 contains a bug** that prevents the jar file from being created correctly. It can be fixed by replacing the `ant/build.xml` with the version found [here](http://wikipedia-miner.svn.sourceforge.net/viewvc/wikipedia-miner/trunk/ant/build.xml?revision=211&view=markup). This patch will be included in the next version.

##Modify and upload language configuration file

In the `config` directory of the toolkit, locate `languages.xml`. Make sure it contains a *Language* element for the dump version you want to process. If not, you will need to [[modify it|Language dependent configuration]].

Even if the version you want to process is already described, it is a good idea to double check the *RootCategory* parameter. This should be the name of the category that contains high-level categories like *Nature* and *Society* (e.g. [Category:Fundamental categories](http://en.wikipedia.org/wiki/Category:Fundamental_categories) in the current English wikipedia). This category changes names and is re-organized fairly often, so just confirm it by greping over the dump you have downloaded, or browsing around Wikipedia online if you are processing a recent dump.

Once you are done, upload the xml file to the HDFS input directory.

```bash
hadoop fs -put languages.xml input
```

##Upload model for sentence detection

Upload an OpenNLP sentence detection model for the appropriate language to the HDFS input directory. Models for English (`en-sent.bin`) and German (`de-sent.bin`) are included in the toolkit under the `models` directory. Other languages are available [here](http://opennlp.sourceforge.net/models). 

##Run the Dump Extractor

Now, ask hadoop to run the DumpExtractor, using the jar you compiled and the input files you uploaded earlier

```bash
hadoop jar wikipedia-miner-hadoop.jar org.wikipedia.miner.extraction.DumpExtractor input/DUMP_FILE input/LANG_FILE LANGUAGE_CODE input/SENTENCE_MODEL output
```

...where *DUMP_FILE*, *LANG_FILE* and *SENTENCE_MODEL* are the names of the files you uploaded earlier, and *LANGUAGE_CODE* is the short code that identifies the language version of the Wikipedia dump (e.g. *en*, *de*, *fr*).

This is going to take a while. As a rough guide, a cluster of 30 machines can process the latest en dumps (27G) in about 2.5 hours. You can expect the time to scale linearly with the number of machines you use and the size of Wikipedia. If you have read the Hadoop documentation, you will know how to monitor progress via the cool web services it provides.

##Download summaries

Finally, download the directory output/final from HDFS
```
hadoop fs -get output/final
```