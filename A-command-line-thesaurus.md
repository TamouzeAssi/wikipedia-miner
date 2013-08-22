This code example demonstrates how Wikipedia can be used as a thesaurus, in a similar fashion as WordNet, but much larger and somewhat messier. It assumes you have already [[installed the Java API|Installing the Java API]].


##What we are aiming for

Here is some sample output from the program we are going to create.

```bash
Please enter a term to look up in Wikipedia (or ENTER for none): Kiwi

'Kiwi' could mean several things:
 - [1] Kiwi
 - [2] Kiwi (people)
 - [3] Kiwifruit
   ...
So which do you want? (1 - 16 or ENTER for none): 2

==Kiwi (people)==
'''[[Culture of New Zealand#Kiwi|Kiwi]]''' is the nickname used internationally for 
[[New Zealanders|people from New Zealand]], as well as being a relatively common 
self-reference.

Alternative labels:
 - Kiwi
 - kiwi
 - Kiwis
 - New Zealander
 - Kiwi (people)
 - Kiwipedians
   ...

Related topics:
 - Kiwi
 - Kiwifruit
 - Māori language
 - Culture of New Zealand
 - Aussie
 - Pākehā
 - National symbol
   ...

Please enter a term to look up in Wikipedia (or ENTER for none)
...


```


##Class and utility functions

First, the boring stuff. Let's set up a class called Wikisaurus (wrawww!), which is going to be able to read input from the command-line, and get information from Wikipedia. It will have one main method ''run'', which is going to prompt the user for terms they want to look up in the "thesaurus", and keep on prompting until the user tells it to shut up. 

```java
public class Wikisaurus {

	BufferedReader _input ;
	Wikipedia _wikipedia ;
	
	public Wikisaurus(WikipediaConfiguration conf) {
		_input = new BufferedReader(new InputStreamReader(System.in)) ;
		_wikipedia = new Wikipedia(conf, false) ; 
	}
	
	public void run() {
		//TODO
	}
}
```


The most important thing happening here is constructing an instance of [Wikipedia](../../doc/org/wikipedia/miner/model/Wikipedia.html), which is pretty much where all things start when working with the toolkit.

And let's write some utility methods for retrieving information from the command-line

```java
	private String getString(String prompt) throws IOException {

		System.out.println(prompt + "(or ENTER for none)") ;

		String line = _input.readLine() ;

		if (line.trim().equals(""))
			line = null ;

		return line ;		
	}
```

```java
	private Integer getInt(String prompt, int min, int max) throws IOException {

		while (true) {

			System.out.println(prompt + " (" + min + " - " + max + " or ENTER for none)") ;

			String line = _input.readLine() ;
			if (line.trim().equals(""))
				return null ;

			try { 
				Integer val = Integer.parseInt(line) ;
				if (val >= min && val <= max)
					return val ;
			} catch (Exception e) {

			}

			System.out.println("Invalid input, try again") ;
		}

	}
```

##Intitializing the Wikisaurus

Let's make this an executable class with a main method, which initializes an instance of the WikiSaurus, and calls run

```java
	public static void main(String args[]) throws Exception {
		WikipediaConfiguration conf = new WikipediaConfiguration(new File(args[0])) ;
		
		Wikisaurus thesaurus = new Wikisaurus(conf) ;
		thesaurus.run() ;
	}
```

We assume the user has given a path to a Wikipedia xml configuration file as the first argument. A WikipediaConfiguration can also be constructed programmatically, without the use of an external file.

###Looking up labels

Lets start working on that run method:

```java
	public void run() throws IOException {
		String term ;
		while ((term = getString("Please enter a term to look up in Wikipedia"))!= null) {
			
			Label label = _wikipedia.getLabel(term) ;
			
			if (!label.exists()) {
				System.out.println("I have no idea what '" + term + "' is") ;
			} else {
				Label.Sense[] senses = label.getSenses() ;
				if (senses.length == 0) {
					displaySense(senses[0]) ;
				} else {
					System.out.println("'" + term + "' could mean several things:") ;
					for (int i=0 ; i<senses.length ; i++) {
						System.out.println(" - [" + (i+1) + "] " + senses[i].getTitle()) ;
					}
					Integer senseIndex = getInt("So which do you want?", 1, senses.length) ;
					if (senseIndex != null)
						displaySense(senses[senseIndex-1]) ;
				}
			}
		}
	}
```

This method, as we planned, will ask the user over and over for terms they want to look up. These get turned into [Labels](../../doc/org/wikipedia/miner/model/Label.html), which is a helpful class that associates a term or phrase with all of the articles (or senses) it could refer to. 

The first thing we do is check if the label exits; whether it is ever used in Wikipedia&mdash;in an article title, redirect title, or in the anchor of a link that points to an article&mdash;to refer to anything. If not, we just move on.

Otherwise, the next thing we do is check if it is ambiguous; if it refers to a single article, or to multiple ones. If there is only one sense article, we display it. Otherwise we ask the user to choose which sense to display. 

##Displaying details of a sense

Now let's write the method that will display pertinent details about the chosen sense of a term. 

The [Label.Sense](../../doc/org/wikipedia/miner/model/Label.Sense.html) class inherits functionality from the [Article](../../doc/org/wikipedia/miner/model/Article.html) class, which in turn inherits from [Page](../../doc/org/wikipedia/miner/model/Page.html). We could ask for a lot of information, but lets just aim for:

* A short definition of what the topic is
* A list of alternative labels (other ways we could have looked for this)
* A list of related topics we can look up

Since we are really dealing with a Wikipedia article here, we can get these (respectively) from
* The first sentence of the article
```java
	protected void displayDefinition(Label.Sense sense) throws Exception {
		System.out.println(sense.getSentenceMarkup(0)) ;
	}
```

* Other labels that point to this article
```java
	protected void displayAlternativeLabels(Label.Sense sense) throws Exception {
		System.out.println("\nAlternative labels:") ;
		for (Article.Label label:sense.getLabels()) 
			System.out.println(label.getText()) ;
	}
```

* Other articles that this article links to 
```java
	protected void displayRelatedTopics(Label.Sense sense) throws Exception {
		System.out.println("\nRelated topics:") ;
		for (Article art:sense.getLinksOut()) 
			System.out.println(" - " + art.getTitle()) ;
	}
```

And finally, we just need to implement the `displaySense` method referred to by `run`
```java
	protected void displaySense(Label.Sense sense) throws Exception {
		
		System.out.println("==" + sense.getTitle() + "==") ;
		
		displayDefinition(sense) ;
		displayAlternativeLabels(sense) ;
		displayRelatedTopics(sense) ;
	}
```


##That's it (or is it?)

You should be able to run the Wikisaurus and get similar output as the sample at the top of the page. 

After playing around for a while, you are likely to have the following questions:

* Presenting the definitions in markup format is pretty messy, [[How do I tidy them up?|The markup stripper]]
* Weither I get a match or not depends on silly things like letter case and plurals. [[How do I fix that?|Text processors]]
* Many of the related topics are irrelevant. [[Can I filter out the weird ones?|Article relatedness measures]]
