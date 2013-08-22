This tutorial demonstrates how to measure semantic relatedness between articles. 

##What we are aiming for

If you followed the [[Command-line Thesaurus|A command-line thesaurus]] tutorial to the end, you will have produced a class that looks up terms in Wikipedia, and provides details about the concepts (or Wikipedia articles) these terms refer to. 

Once the user has arrived at a concept, the 'thesaurus' lists other related concepts by looking at the outgoing links from the corresponding article. 
 
There are a few problems with this approach. 

1. Some links are better than others. The article *Kiwi* links to *New Zealand* which is great, but it also links to *University of Oxford*, which is not so informative.
2. Not all relations are represented by outgoing links. *Kiwi* doesn't link to *Flightless Bird*, for example.  

In this code example, we will look at some other sources of related topics, and we will use semantic relatedness measures to prioritise relations so we only show the strongest ones.

##Semantic Relatedness Measures

First, a little background on [semantic relatedness measures](http://en.wikipedia.org/wiki/Semantic_similarity). 

It is fairly common in computer science to describe relations in a formal way, like this:

* *Kiwi* **isA** *Flightless Bird*
* *Flightless Bird* **isA** *Bird*

These structures (triples, in semantic-webby type speak) allow computers to reason, for example, that all Kiwis are birds. The problem is, they are difficult to obtain and require complex algorithms to use. 

A less common and less ambitious way to represent relations is through numbers, by quantifying them. We might say that

* *Kiwi* is **90%** related to *Flightless Bird*
* *Flightless Bird* is **95%** related to *Bird*
* *Kiwi* is **75%** related to *Bird*

These numbers don't express any solid facts, but they are helpful none-the-less. We can use them for word-sense disambiguation, topic indexing, clustering, and many other tasks (just have a look at these [[publications]]). 

In this code example, we are going to use them to filter out weak relations so we only show the strongest to the user.

##Casting a wider net for relations

Create a new class `ComparingWikisaurus`, which extends our original Wikisaurus

```java
public class ComparingWikisaurus extends Wikisaurus {

        public ComparingWikisaurus(WikipediaConfiguration conf) {
		super(conf);
	}
	
	public static void main(String args[]) throws Exception {
		WikipediaConfiguration conf = new WikipediaConfiguration(new File(args[0])) ;
		
		ComparingWikisaurus thesaurus = new ComparingWikisaurus(conf) ;
		thesaurus.run() ;
	}
}
```

Now create a method that will build up a list of articles that are connected in some way (either through in-links, out-links or category siblings) to an input article.

```java
	private List<Article> gatherRelatedTopics(Article art) {
		HashSet<Integer> relatedIds = new HashSet<Integer>() ;
		relatedIds.add(art.getId()) ;

		ArrayList<Article> relatedTopics = new ArrayList<Article>() ;

		//gather from out-links
		for (Article outLink:art.getLinksOut()) {
			if (!relatedIds.contains(outLink.getId())) {
				relatedIds.add(outLink.getId()) ;
				relatedTopics.add(outLink) ;
			}
		}

		//gather from in-links
		for (Article inLink:art.getLinksIn()) {
			if (!relatedIds.contains(inLink.getId())) {
				relatedIds.add(inLink.getId()) ;
				relatedTopics.add(inLink) ;
			}
		}   

		//gather from category siblings
		for (Category cat:art.getParentCategories()){
			for (Article sibling:cat.getChildArticles()) {
				if (!relatedIds.contains(sibling.getId())) {
					relatedIds.add(sibling.getId()) ;
					relatedTopics.add(sibling) ;
				}
			}
		}
		
		return relatedTopics ;
	}
```

Then override the `displayRelatedTopics` method to call this rather than just using out links.

```java
    @Override
	protected void displayRelatedTopics(Label.Sense sense) {

		List<Article> relatedTopics = gatherRelatedTopics(sense) ;

		System.out.println("\nRelated topics:") ;
		for (Article art:relatedTopics) 
			System.out.println(" - " + art.getTitle()) ;
	}
```

Running the code at this point should hammer home the importance of prioritizing relations: Kiwi now has hundreds of related topics, but we can only expect users to wade through the top 20 or so. 

##The article comparer

The [ArticleComparer](../../doc/org/wikipedia/miner/comparison/ArticleComparer.html) is the class responsible for generating semantic relatedness measures between articles. We need to set one up when we first construct the Wikisaurus, so change the class members and constructor to:

```java
	ArticleComparer _comparer ;
	
	public ComparingWikisaurus(WikipediaConfiguration conf) throws Exception {
		super(conf);
		_comparer = new ArticleComparer(_wikipedia) ;
	}
```

The comparer can work with or without machine learning, and can take advantage of different [data dependencies](../../doc/org/wikipedia/miner/comparison/ArticleComparer.DataDependency.html), or structural elements of Wikipedia in order to generate the measures. It is a trade-off, where using more dependencies and machine learning is more accurate, but also more expensive. 

In most cases we recommend not using machine learning, and using only the `pageLinksIn` dependency; this provides a good balance between accuracy and efficiency. 

All of these options can be set by editing the [[Wikipedia configuration|Configuring a single Wikipedia instance]], or by adding more arguments to the `ArticleComparer` constructor call. 

##Using the article comparer

Now lets use the comparer. Create a new method `sortTopics`, which uses the article comparer to sort a list of articles by how strongly they relate to a query article.

```java
private List<Article> sortTopics(Article queryTopic, List<Article>relatedTopics) throws Exception {
    //weight the related articles according to how strongly they relate to sense article
    for (Article art:relatedTopics) 
        art.setWeight(_comparer.getRelatedness(art, queryTopic)) ;
	    
    //Now that the weight attribute is set, sorting will be in descending order of weight.
    //If weight was not set, it would be in ascending order of id.  
    Collections.sort(relatedTopics) ;
    return relatedTopics ;
}
```

And modify the `displayRelatedTopics` method to use this. 

```java
@Override
protected void displayRelatedTopics(Label.Sense sense) throws Exception {

    List<Article> relatedTopics = gatherRelatedTopics(sense) ;
    relatedTopics = sortTopics(sense, relatedTopics) ;

    //now trim the list if necessary
    if (relatedTopics.size() > 25)
        relatedTopics = relatedTopics.subList(1,25) ;
		
    System.out.println("\nRelated topics:") ;
    for (Article art:relatedTopics) 
        System.out.println(" - " + art.getTitle() + " " + _df.format(art.getWeight())) ;
}
```

You'll need to add the `DecimalFormat _df` as a class member:

```java
    DecimalFormat _df = new DecimalFormat("#0%") ;
```

##That's it (or is it?)

You should be able to run the Wikisaurus again, and get much better lists of related topics.

For extra credit:
* If the thesaurus runs slowly, you might want to [cache some databases to memory](Configuring a single Wikipedia instance) (particlarly `pageLinksInNoSentences`).
* Is there a good chance that the same relatedness measure will be getting calculated multiple times? Then check out [RelatednessCache](../../doc/org/wikipedia/miner/util/RelatednessCache.html).
* Do you want to measure relatedness between ambiguous terms rather than unambiguous articles? Then check out [LabelComparer](../../doc/org/wikipedia/miner/comparison/LabelComparer.html).
