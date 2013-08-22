This code example demonstrates how to create and use a [text processor](../../doc/org/wikipedia/miner/util/text/TextProcessor.html), for resolving conflation issues like CasE vAriaTions, âćçëňŧș, and plural(s). 

##What we are aiming for: 

If you followed the [[Command-line Thesaurus|A command-line thesaurus]] tutorial to the end, you will have produced a class that looks up terms in Wikipedia, and provides details about the concepts (or Wikipedia articles) these terms refer to. 

However, searching can be a bit hit-and-miss. Variations in capitalization, use of accents, and pluralization greatly effect the results returned. For example

```bash
Please enter a term to look up in Wikipedia (or ENTER for none)
metallica
'metallica' could mean several things:
 - [1] Green-head ant
...

Please enter a term to look up in Wikipedia (or ENTER for none)
Metallica
'Metallica' could mean several things:
 - [1] Metallica
 - [2] Metallica (album)
 - [3] For Whom the Bell Tolls (Metallica song)
```

So without capitalization *metallica* is matched only to the *Green-head ant*, or *Rhytidoponera metallica*. With capitalization, it is matched only to the band. Users really shouldn't have to bother with such subtleties. 

In this exercise, we are going to improve our 'thesarus' by making the search function use stemming, case- and accent-folding.

##Create a text processor

Create a new class `TextFolder` (not an ideal name), which extends [TextProcessor](../../doc/org/wikipedia/miner/util/text/TextProcessor.html). 

```java
import org.wikipedia.miner.util.text.*;

public class TextFolder extends TextProcessor{

    @Override
    public String processText(String text) {
        return text ;
    }
}
```

Text processors perform one job; they take a piece of input text and remove any variations they don't care about. To specify how this job is done, you need to implement the abstract method `processText`, which in the class above does nothing. 

There are already several text processors implemented for you in the `util.text` package, including a [CaseFolder](../../doc/org/wikipedia/miner/util/text/CaseFolder.html) and [PorterStemmer](../../doc/org/wikipedia/miner/util/text/PorterStemmer.html). Lets start by just using these. 

```java
import org.wikipedia.miner.util.text.*;

public class TextFolder extends TextProcessor{

    private CaseFolder caseFolder = new CaseFolder() ;
    private PorterStemmer stemmer = new PorterStemmer() ;

    @Override
    public String processText(String text) {
        return stemmer.processText(caseFolder.processText(text)) ;
    }
}
```

And finally, we will add the accent folding. There isn't currently an AccentFolder included in the toolkit, which is probably an oversight. However, this [stack overflow thread](http://stackoverflow.com/questions/1008802/converting-symbols-accent-letters-to-english-alphabet) describes a pretty elegant way to implement one. 

```java
public class TextFolder extends TextProcessor{

    private CaseFolder caseFolder = new CaseFolder() ;
    private PorterStemmer stemmer = new PorterStemmer() ;
    private Pattern pattern = Pattern.compile("\\p{InCombiningDiacriticalMarks}+");

    @Override
    public String processText(String text) {
		
        String normalizedText = Normalizer.normalize(text, Normalizer.Form.NFD); 
	normalizedText = pattern.matcher(normalizedText).replaceAll("") ;
	normalizedText = caseFolder.processText(normalizedText) ;
	normalizedText = stemmer.processText(normalizedText) ;
		
	return normalizedText;
    }
}
```

Done! For extra credit, write a main method that lets you type in text and see the processed output

##Applying the text processor at index time

The text processor must be applied at both index time and query time. By *index time*, I mean that we need the labels stored in the database need to be processed, and any conflicts merged. In the case of our example above *Metallica* and *metallica* need to become one single label. 

Doing this is kind of a big deal; it needs to be done over all labels in the database (6M or so in the English wikipedia). So, the call below might take a while. It also takes a fair amount of memory, so make sure to set ``-Xmx`` to be 2G or so.

```java
public static void main(String args[]) throws Exception {
		
    TextFolder folder = new TextFolder() ;
		
    WikipediaConfiguration conf = new WikipediaConfiguration(new File(args[0])) ;
    WEnvironment.prepareTextProcessor(folder, conf, new File("tmp"), true, 3) ;
}
```

The arguments to `prepareTextProcessor` are, in order:
* The text processor to prepare
* A configuration describing where to find the wikipedia database.
* A directory that can be created and deleted, for storing temporary files.
* *true* if the operation should be continued when there is already a text processor of the same name prepared, *false* if it should be aborted.
* The number of passes to perform the operation in (this depends on the amount of memory you have available) 

> This creates a new copy of the label database, named after the text processor's class name. You can maintain many different text processors at once, as long as they have different names. 


##Applying the text processor at query time

Now we just need to make the 'thesaurus' use our new TextFolder. One way to accomplish this would be to modify the call that creates each label. In the `run` method, this line

```java
    Label label = _wikipedia.getLabel(term) ;
```

Simply becomes

```java
    Label label = _wikipedia.getLabel(term, _folder) ;
```

Where `_folder` is a TextFolder created in the Wikisaurus constructor.

However, that annoyingly requires us to edit the Wikisaurus class. A simpler way is to just alter the configuration to specify our text folder as the default text processor. 

```java
public class FoldedWikisaurus extends Wikisaurus{

	
    public FoldedWikisaurus(WikipediaConfiguration conf) {
        super(conf);
    }

    public static void main(String args[]) throws Exception {
        WikipediaConfiguration conf = new WikipediaConfiguration(new File(args[0])) ;
        conf.setDefaultTextProcessor(new TextFolder()) ;

        FoldedWikisaurus thesaurus = new FoldedWikisaurus(conf) ;
        thesaurus.run() ;
    }
}
```
