This code example demonstrates how to automatically annotate snippets of text with links to relevant Wikipedia articles. 

##What we are aiming for

Here is some sample output from the program we are going to create.

```bash
Enter snippet to annotate (or ENTER to quit):
Mixed martial artists competing in Pride parade around the ring during the tournament`s opening ceremony

All detected topics:
 - The Ring (magazine)
 - Pride parade
 - Martial arts
 - Parade
 - Mixed martial arts
 - Tournament
 - 2008 Summer Olympics opening ceremony
 - Wrestling ring
 - Pride Fighting Championships

Topics that are probably good links:
 - Martial arts[70%]
 - Mixed martial arts[67%]
 - Pride Fighting Championships[56%]

Augmented markup:
[[Mixed martial arts|Mixed martial artists]] competing in [[Pride Fighting Championships|Pride]]
parade around the ring during the tournament`s opening ceremony
```


```
Enter snippet to annotate (or ENTER to quit):
The second annual lesbian, gay, bisexual and transgender (LGBT) pride parade
in southern Taiwan will take place in Kaohsiung this Saturday

All detected topics:
 - Kaohsiung
 - Gay
 - Lesbian
 - Pride parade
 - Southern United States
 - Parade
 - LGBT
 - Bisexuality
 - Gay pride
 - Taiwan
 - Transgender

Topics that are probably good links:
 - LGBT[81%]
 - Transgender[80%]
 - Bisexuality[80%]
 - Gay[79%]
 - Lesbian[75%]
 - Pride parade[62%]

Augmented markup:
The second annual [[LGBT|lesbian, gay, bisexual and transgender (LGBT)]] [[pride parade]] 
in southern Taiwan will take place in Kaohsiung this Saturday
```

##Disambiguation and Detection

First, a little background. The annotation task involves two problems: disambiguation and detection.

**Disambiguation** involves choosing the correct senses (or link destinations) for ambiguous terms. You can see this in action in the examples above, which use the phrase *pride parade* in very different ways. 

**Detection** involves deciding whether a concept is worth linking to or not. You can see this in the difference between the *All detected topics* and *Topics that are probably good links* lists above. 

Wikipedia articles provides millions of examples of how to solve these two problems, because every article in Wikipedia is a piece of text that has been manually linked to the appropriate Wikipedia topics. For each link a human has manually chosen its destination (disambiguation), and for each phrase a person has chosen to whether or not to create a link at all (detection). 

Wikipedia Miner's annotation algorithms are machine learned, so they are trained using these articles

##Class setup

Let's create a `SnippetAnnotator` class

```java
public class SnippetAnnotator {
	
	DecimalFormat _df = new DecimalFormat("#0%") ;

	public SnippetAnnotator(Wikipedia wikipedia) throws Exception {
	    //TODO
	}
	
	public void annotate(String originalMarkup) throws Exception {
            //TODO
	}
	
	public static void main(String args[]) throws Exception {
		
		WikipediaConfiguration conf = new WikipediaConfiguration(new File(args[0])) ;
		Wikipedia wikipedia = new Wikipedia(conf, false) ;
		
		SnippetAnnotator annotator = new SnippetAnnotator(wikipedia) ;
		
		BufferedReader reader = new BufferedReader(new InputStreamReader(System.in)) ;
		while (true) {
			System.out.println("Enter snippet to annotate (or ENTER to quit):") ;
			String line = reader.readLine();
			
			if (line.trim().length() == 0)
				break ;
			
			annotator.annotate(line) ;
		}
	}
}
```

This will initialise an instance of Wikipedia, which in turn will be used to construct a new SnippetAnnotator. It will then continuously prompt the user for snippets, and send these to the `annotate` method. 

##Preprocessing existing markup

The first thing this annotate method needs to do is construct a [preprocessed document](../../doc/org/wikipedia/miner/annotation/preprocessing/PreprocessedDocument.html). This is a copy of the original snippet, in which regions of markup have been blanked out if they are ineligible for tagging. Our annotator expects snippets of mediawiki markup, so we need a [WikiPreprocessor](../../doc/org/wikipedia/miner/annotation/preprocessing/WikiPreprocessor.html) to blank out existing links, templates and so on. Without this step, we risk producing invalid markup.

So, we add the following class member...
```java
    DocumentPreprocessor _preprocessor ; 
```

...create it in the constructor...
```java
    _preprocessor = new WikiPreprocessor(wikipedia) ;
```
    
...and use it in the ```annotate``` method.
```java
    PreprocessedDocument doc = _preprocessor.preprocess(originalMarkup) ;
```

##Gathering topics (a rough first pass)

Now the cleaned-up snippet can be fed into a [TopicDetector](../../doc/org/wikipedia/miner/annotation/TopicDetector.html) and [Disambiguator](../../doc/org/wikipedia/miner/annotation/Disambiguator.html) to gather all of the terms and phrases that could possibly match to Wikipedia topics.

The `TopicDetector` acts as a rough filter, by identifying terms and phrases that are used frequently as links in Wikipedia articles. For example, *New Zealand* is found in about 75000 articles, and is used as a link in 48% of these. *New*, in contrast, occurs in hundreds of thousands of articles, and is hardly ever used as a link. Depending on the configuration you used to construct the Wikipedia instance, the TopicDetector may also use a stopword file to discard extremely common terms like *and*, *or*, *the* and so on.

Many of the terms identified will be ambiguous, in that they could refer to multiple articles. To resolve these, the `TopicDetector` consults a `Disambiguator` to choose the most likely sense for each ambiguous term. You can see it in action in the first example above, where it encounters the term *the ring*. This could have referred to the [2002][ring1] or [1927 horror movie][ring2], but a more likely sense, given what the rest of the snippet talks about, is the [boxing magazine][ring3]. 

[ring1]: http://en.wikipedia.org/wiki/The_Ring_(2002_film)
[ring2]: http://en.wikipedia.org/wiki/The_Ring_(1927_film)
[ring3]: http://en.wikipedia.org/wiki/The_Ring_(magazine)

The `Disambiguator` uses machine learning, and is trained using the article-to-article links that Wikipedia already contains. Each existing link provides one positive example, namely its chosen destination, and several negative examples, namely the destinations that have been chosen for this link text in other articles but not this one. Because it needs to be trained, the Disambigutor will not work unless your [[Wikipedia configuration|Configuring a single Wikipedia instance]] specifies a *topicDisambiguationModel*.

Let's add the following class members...

```java
Disambiguator _disambiguator ;
TopicDetector _topicDetector ;
```

...create them in the constructor...

```java
_disambiguator = new Disambiguator(wikipedia) ;
_topicDetector = new TopicDetector(wikipedia, _disambiguator, true, false) ;
```

...and use them in the annotate method...

```java
Collection<Topic> allTopics = _topicDetector.getTopics(doc, null) ;
System.out.println("\nAll detected topics:") ;
for (Topic t:allTopics)
    System.out.println(" - " + t.getTitle()) ;
```

This produces a list of [Topics](../../doc/org/wikipedia/miner/annotation/Topic.html), which are simply [Articles](../../doc/org/wikipedia/miner/model/Article.html) extended with additional information about where the article is mentioned, how strongly it relates to the other topics discussed, and various other features for judging its relevance to the text.

If you try running the code at this point (or look at the examples above) you will see that the list is rough and contains many irrelevant topics. At this stage, no effort has been made to distinguish those that are central to the document from ones that are only mentioned in passing. We will tackle this problem in the next step.

##Refining the detected topics.

To refine the list of topics, we will use a [LinkDetector](../../doc/org/wikipedia/miner/annotation/weighting/LinkDetector.html). This class is based on the idea that every existing Wikipedia article is an example of how to separate relevant topics—ones that authors chose to link to—from irrelevant ones. For each topic in a new document, the link detector calculates a weight based on how well it fits the model of what Wikipedians would choose to link to if the document were a Wikipedia article. Like the `Disambiguator`, this class is machine-learned and must be trained. It will not work unless your Wikipedia configuration specifies a *linkDetectionModel*.

Lets add one as a class member...
```java
LinkDetector _linkDetector ;
```

...create it in the constructor...
```java
_linkDetector = new LinkDetector(wikipedia) ;
```

...and use it in the `annotate` method.
```java
ArrayList<Topic> bestTopics = _linkDetector.getBestTopics(allTopics, 0.5) ;
System.out.println("\nTopics that are probably good links:") ;
for (Topic t:bestTopics)
    System.out.println(" - " + t.getTitle() + "[" + _df.format(t.getWeight()) + "]" ) ;
```

Running the code now should produce much tidier lists of topics. By this point, many users will have achieved what they need: a list of Wikipedia topics for the given document, weighted by their relevance to it. This list could be used as a concept-based representation of the document in applications like categorisation, clustering, retrieval, and so on. 

##Inserting topics back into the markup

The final step is to feed the `PreprocessedDocument` and the list of weighted `Topics` into a [DocumentTagger](../../doc/org/wikipedia/miner/annotation/tagging/DocumentTagger.html), which produces a new version of the original document, marked up to identify the topics. The tagger assumes that tags should not be nested, and resolves collisions or overlapping mentions of topics. You can see this in action in the 2nd example above, in which *lesbian*, *gay*, *bisexual* and *transgender* have all been detected as relevant topics, but are replaced with a single link to *LGBT*.

Like the `DocumentPreprocessor`, the `DocumentTagger` needs to be extended to handle different kinds of markup. We will use the [WikiTagger](../../doc/org/wikipedia/miner/annotation/tagging/WikiTagger.html), since we are processing media-wiki markup.

Lets add it as a class member...
```java
DocumentTagger _tagger ;
```

...create it in the constructor...
```java
_tagger = new WikiTagger() ;
```

...and use it in the `annotate` method.
```java
String newMarkup = _tagger.tag(doc, bestTopics, RepeatMode.ALL) ;
System.out.println("\nAugmented markup:\n" + newMarkup + "\n") ;
```

##That's it (or is it?)

* The annotator probably runs slowly, so you will want to [[cache some databases to memory|Configuring a single Wikipedia instance]] (particlarly *label* and *pageLinksInNoSentences*).  
* If the annotator doesn't work very well on your documents, then look into [[retraining it|Experimenting with annotation]].
* If you need to process a different kind of markup, or have the tagger create links in a different format, then look into [[extending the preprocessor and tagger|Annotating custom markup]].
