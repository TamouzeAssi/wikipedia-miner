###Obtaining the toolkit

#### Latest version

* [wikipedia-miner-1.2.0.tar.gz](http://sourceforge.net/projects/wikipedia-miner/files/wikipedia-miner/wikipedia-miner_1.2/wikipedia-miner-1.2.0.tar.gz/download) 

#### Previous versions

* [wikipedia-miner-1.1.0.tar.gz](http://sourceforge.net/projects/wikipedia-miner/files/wikipedia-miner/wikipedia-miner_1.1/wikipedia-miner_1.1.tar.gz/download)
* [wikipedia-miner-1.0.0.tar.gz](http://sourceforge.net/projects/wikipedia-miner/files/wikipedia-miner/wikipedia-miner_1.0/wikipedia-miner_1.0.tar.gz/download)

####SVN Checkout

The absolute latest (and likely broken) code can be checked out using svn 

```bash
    svn co https://wikipedia-miner.svn.sourceforge.net/svnroot/wikipedia-miner/trunk wikipedia-miner 
```

Releases can be obtained in a similar way, eg:

```bash
    svn co https://wikipedia-miner.svn.sourceforge.net/svnroot/wikipedia-miner/tags/1.2.0 wikipedia-miner 
```


###Obtaining Wikipedia data

The toolkit requires both the original XML dumps of Wikipedia, and extracted CSV Summaries. Both need to be from the same edition (same language and release date). 

If you can't find the edition you want in the table below, you can download the xml dumps directly from the Media Wiki foundation and extract the CSV summaries yourself. Check [[here|Obtaining wikipedia data]] for details. 

| Language | Edition | CSV Summaries  | XML Dump
|:---------|:--------|:---------------|:-----------
| en  | 22 July 2011 | [enwiki-20110722-csv.tar.gz](http://sourceforge.net/projects/wikipedia-miner/files/data/en/enwiki-20110722-csv.tar.gz/download) | [enwiki-20110722-pages-articles.xml.bz2](http://download.wikimedia.org/enwiki/20110722/enwiki-20110722-pages-articles.xml.bz2)
| en  | 1 September 2011 | - | [enwiki-20110901-pages-articles.xml.bz2](http://download.wikimedia.org/enwiki/20110901/enwiki-20110901-pages-articles.xml.bz2)   
| de  | blah | - | - 

