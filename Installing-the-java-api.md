This page describes how to start interacting with the Wikipedia Miner toolkit from within Java code. It assumes you have already [[obtained the data|Obtaining wikipedia data]] (both XML dump and corresponding CSV summaries).

##Unpack data

Place the uncompressed XML file and all of the extracted csv summaries into a single directory

##Create configuration file

Copy the `wikipedia-template.xml` file from the configs directory included in the toolkit. It is likely that you will reuse this file every time you interact with Wikipedia miner, so name it something memorable and put it somewhere safe (the `configs` directory is a good place).

Fill in the file with appropriate details. The file is heavily commented, so it should be self explanatory. At a minimum, you should at least enter the fields that are mandatory: **langCode** (e.g. "en") and **databaseDirectory** (e.g. "db", which will be created in the next step). Also fill in the **dataDirectory** field to refer to the folder containing an uncompressed wikipedia dump and extracted summaries.

##Build the berkeley database

Edit the `ant/build.properties` file to include a reference to the configuration file you just created. Add a line like this:

```
wiki.conf=path/to/conf/file
```

The path needs to be fully qualified, or relative to the root folder of the wikipedia miner project. 

Now run the ant target `build-database`
```
ant build-database
```

If you did everything right, you will get lots of status updates, and the whole process should be over in an hour or so depending on the size of the dump and the specs of your computer.

##Start playing around

To verify that everything is working ok, try creating a new java project that we can play around with. 

Start by using the ant target `package`

```
    ant package
```

This will create the file `bin/jar/wikipedia-miner.jar`

> **Version 1.2 contains a bug**that prevents the jar file from being created correctly. It can be fixed by replacing the `ant/build.xml` with the version found [here](http://wikipedia-miner.svn.sourceforge.net/viewvc/wikipedia-miner/trunk/ant/build.xml?revision=211&view=markup)
This patch will be included in the next version.

Using your preferred programming environment (Eclipse, emacs, [butterflies](http://xkcd.com/378), whatever), create a new project, whose classpath includes `wikipedia-miner.jar` and all of the libraries in the `lib` directory of wikipedia miner.

Then try creating the *WikipediaDefiner* class:

```java
import java.io.File;

import org.wikipedia.miner.model.Article;
import org.wikipedia.miner.model.Wikipedia;
import org.wikipedia.miner.util.WikipediaConfiguration;


public class WikipediaDefiner {

    public static void main(String args[]) throws Exception {
		
        WikipediaConfiguration conf = new WikipediaConfiguration(new File("path/to/conf")) ;
			
        Wikipedia wikipedia = new Wikipedia(conf, false) ;
	    
        Article article = wikipedia.getArticleByTitle("Wikipedia") ;
	    
        System.out.println(article.getSentenceMarkup(0)) ;
	    
        wikipedia.close() ;
    }
}
```

If everything is working well, you should be able to run this and print out a pithy definition of what Wikipedia is. 


> Don't be worried if this first call takes a while to run. It seems to take a while for the database behind the toolkit to iron out once it is first built. It will soon be running quickly.


Check out [[Using the Java API]] for more ideas, and the [[JavaDoc|]] for the gritty details.

Java API is very much useful
[Java Training in Chennai](http://wisentechnologies.com/it-courses/java-training.aspx)