This page documents the CSV summaries extracted from Wikipedia, in case you want to use them independently from the toolkit. 

These files are all written in the CSV format specified by Hadoop's *record* package. We use this because we were looking for a format that
* Could be stored in a human-readable way (for dump files)
* Could also be stored in a highly compressed binary format (for database and in-memory caching)
* Could be deserialized from binary format to java objects efficiently

Hadoop's record package fit the bill, and had the added advantages
* Doesn't require any additional libraries (the toolkit already uses Hadoop)
* Can serialised into different languages, such as C++.

We also found it to be surprisingly efficient (in our tests the binary format compressed smaller and was faster to deserialize than Google's prototype buffers, for example). However, the package is deprecated, so we may look at alternatives in future. 

#The Hadoop CSV Serialization format

> These notes are cribbed from [here](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/record/package-summary.html)

The CSV serialization format has a lot more structure than the "standard" Excel CSV format, but we believe the additional structure is useful because

* it makes parsing a lot easier without detracting too much from legibility
* the delimiters around composites make it obvious when one is reading a sequence of Hadoop records 

Serialization formats for the various types are detailed in the grammar that follows. The notable feature of the formats is the use of delimiters for indicating the certain field types.

* A string field begins with a single quote (').
* A buffer field begins with a sharp (#).
* A class, vector or map begins with 's{', 'v{' or 'm{' respectively and ends with '}'. 

The CSV format can be described by the following grammar:

```
record = primitive / struct / vector / map
primitive = boolean / int / long / float / double / ustring / buffer

boolean = "T" / "F"
int = ["-"] 1*DIGIT
long = ";" ["-"] 1*DIGIT
float = ["-"] 1*DIGIT "." 1*DIGIT ["E" / "e" ["-"] 1*DIGIT]
double = ";" ["-"] 1*DIGIT "." 1*DIGIT ["E" / "e" ["-"] 1*DIGIT]

ustring = "'" *(UTF8 char except NULL, LF, % and , / "%00" / "%0a" / "%25" / "%2c" )

buffer = "#" *(BYTE except NULL, LF, % and , / "%00" / "%0a" / "%25" / "%2c" )

struct = "s{" record *("," record) "}"
vector = "v{" [record *("," record)] "}"
map = "m{" [*(record "," record)] "}"
```


## The files

Each file simply contains key/value pairs, one per line. The table below lists all of the different files, and the key and value record types.

|file | key | value | purpose
|:----|:---:|:-----:|:-------
|articleParents.csv | int | [DbIntList](../../doc/org/wikipedia/miner/db/struct/DbIntList.html) | associates article id with ids of categories it belongs to
|categoryParents.csv | int | [DbIntList](../../doc/org/wikipedia/miner/db/struct/DbIntList.html) | associates category id with ids of categories it belongs to
|childArticles.csv | int | [DbIntList](../../doc/org/wikipedia/miner/db/struct/DbIntList.html) | associates category id with ids of articles that belong to it
|childCategories.csv | int | [DbIntList](../../doc/org/wikipedia/miner/db/struct/DbIntList.html) | associates category id with ids of categories that belong to it
|label.csv | String | [DbLabel](../../doc/org/wikipedia/miner/db/struct/DbLabel.html) | associates word or phrase with statistics of use and the different sense articles it could refer to
|page.csv | int | [DbPage](../../doc/org/wikipedia/miner/db/struct/DbPage.html) | associates id of page with details like title, page type, etc
|pageLabel.csv | int | [DbLabelForPageList](../../doc/org/wikipedia/miner/db/struct/DbLabelForPageList.html) | associates id of page with list of labels that refer to it
|pageLinkIn.csv | int | [DbLinkLocationList](../../doc/org/wikipedia/miner/db/struct/DbLinkLocationList.html) | associates id of page with list of pages that link to it, and indexes of sentences where those links are found
|pageLinkOut.csv | int | [DbLinkLocationList](../../doc/org/wikipedia/miner/db/struct/DbLinkLocationList.html)| associates id of page with list of pages that it links to, and indexes of sentences where those links are found
|redirectSourcesByTarget.csv | int | [DbIntList](../../doc/org/wikipedia/miner/db/struct/DbIntList.html) | associates article id with ids of redirects that target it
|redirectTargetsBySource.csv | int | int | associates redirect id with the id of the article it targets
|sentenceSplits.csv | int | [DbIntList](../../doc/org/wikipedia/miner/db/struct/DbIntList.html) | associates id of page with character indexes of sentence splitting tokens.
|stats.csv | String | long | assocates statistic name with value of the statistic
|translations.csv | int | [DbTranslations](../../doc/org/wikipedia/miner/db/struct/DbTranslations.html) | associates id of page with links to same page in other language editions of Wikipedia


To process any of these files from within Java, use the following code: 

```java
//Set up a UTF8 reader
File dataFile = new File("path/to/data/file.csv") ;
BufferedReader input = new BufferedReader(new InputStreamReader(new FileInputStream(dataFile), "UTF-8")) ;

//read file line by line
String line ;
while ((line=input.readLine()) != null) {
    CsvRecordInput cri = new CsvRecordInput(new ByteArrayInputStream((line + "\n").getBytes("UTF-8"))) ;

    //read key 
    Integer id = record.readInt(null) ;
    
    //read value (substitue DbPage for appropriate record type if not processing page.csv)
    DbPage page = new DbPage() ;
    page.deserialize(record) ;

    //do something with the key/value pair
    System.out.println(id + " " + page.getTitle()) ; 
}
input.close() ;
```

If you work with another language, the record types are specified in a language-independent way in `src/structures.ddl`. Refer to [this page](http://blog.foofactory.fi/2006/12/record-your-data.html) for information on compiling this, and how to use the resulting data structures. 


