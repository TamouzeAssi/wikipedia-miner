This code example demonstrates how to train and evaluate classifiers for document annotation.

##What we are aiming for

If you followed the [[Command-line Annotator|A command-line document annotator]] example, you would have produced a class that automatically augments snippets of text with links to the Wikipedia topics they discuss. 

This involved using two different classifiers; one for disambiguation, and another for link detection. Both are machine learned, and have to be given training data&mdash;in the form of wikipedia articles&mdash;to work. 

The performance of these classifiers depends on how closely this training data matches the way you want them to behave. If you train using lengthy articles, for example, then they won't perform well on short snippets of text. 

This page will demonstrate a robust way to gather appropriate training data, experiment with different classification algorithms, and measure their performance.

##Class setup

Lets set up a class called `AnnotationWorkbench` with four main stages: 

* Generating sets of Wikipedia articles that are suitable for training and testing
* Generating ARFF files (feature data) from the training data, which can be fed into weka.
* Generating model files that describe the logic of a trained classifier. 
* Evaluating the trained classifiers.

First, the class and constructor:

```java
public class AnnotationWorkbench {

	private Wikipedia _wikipedia ;
	
	//directory in which files will be stored
	private File _dataDir ;
	
	//classes for performing annotation
	private Disambiguator _disambiguator ;
	private TopicDetector _topicDetector ;
	private LinkDetector _linkDetector ;
	
	//article set files
	private File _artsTrain, _artsTestDisambig, _artsTestDetect ;
	
	//feature data files
	private File _arffDisambig, _arffDetect ;
	
	//model files
	private File _modelDisambig, _modelDetect ;


	public AnnotationWorkbench(File dataDir, Wikipedia wikipedia) throws Exception {
		
		_dataDir = dataDir ;
		_wikipedia = wikipedia ;
		
		_disambiguator = new Disambiguator(_wikipedia) ;
		_topicDetector = new TopicDetector(_wikipedia, _disambiguator, false, false) ;
		_linkDetector = new LinkDetector(_wikipedia) ;
		
		_artsTrain = new File(_dataDir.getPath() + "/articlesTrain.csv") ;
		_artsTestDisambig = new File(_dataDir.getPath() + "/articlesTestDisambig.csv") ;
		_artsTestDetect = new File(_dataDir.getPath() + "/articlesTestDetect.csv") ;
		
		_arffDisambig = new File(_dataDir.getPath() + "/disambig.arff") ;
		_arffDetect = new File(_dataDir.getPath() + "/detect.arff") ;
		
		_modelDisambig = new File(_dataDir.getPath() + "/disambig.model") ;
		_modelDetect = new File(_dataDir.getPath() + "/detect.model") ;
	}
}
```

Next, some placeholders for the four steps we need to perform:

```java
    private void gatherArticleSets() throws IOException{
    
    }
    
    private void createArffFiles(String datasetName) throws IOException, Exception {
    
    }
    
    private void createClassifiers(String configDisambig, String configDetect) throws Exception {
		
    }
    
    private void evaluate() throws Exception {
    
    }
```

And finally, a main method that will construct the workbench and allow the user to perform each step:

```
	public static void main(String args[]) throws Exception {
		
		File dataDir = new File(args[0]) ;
		
		WikipediaConfiguration conf = new WikipediaConfiguration(new File(args[1])) ;
		conf.addDatabaseToCache(DatabaseType.label) ;
		conf.addDatabaseToCache(DatabaseType.pageLinksInNoSentences) ;
		
		Wikipedia wikipedia = new Wikipedia(conf, false) ;
		
		AnnotationWorkbench trainer = new AnnotationWorkbench(dataDir, wikipedia) ;
		BufferedReader input = new BufferedReader(new InputStreamReader(System.in)) ;
		
		while (true) {
			System.out.println("What would you like to do?") ;
			System.out.println(" - [1] create article sets.") ;
			System.out.println(" - [2] create arff files.") ;
			System.out.println(" - [3] create classifiers.") ;
			System.out.println(" - [4] evaluate classifiers.") ;
			System.out.println(" - or ENTER to quit.") ;
			
			String line = input.readLine() ;
			
			if (line.trim().length() == 0)
				break ;
			
			Integer choice = 0 ;
			try {
				choice = Integer.parseInt(line) ;
			} catch (Exception e) {
				System.out.println("Invalid Input") ;
				continue ;
			}
			
			switch(choice) {
			case 1:
				trainer.gatherArticleSets() ;
				break ;
			case 2:
				System.out.println("Dataset name:") ;
				String datasetName = input.readLine() ;
				
				trainer.createArffFiles(datasetName) ;
				break ;
			case 3:
				System.out.println("Disambiguation classifer config (or ENTER to use default):") ;
				String configDisambig = input.readLine() ;
				
				System.out.println("Detection classifer config (or ENTER to use default):") ;
				String configDetect = input.readLine() ;
				
				trainer.createClassifiers(configDisambig, configDetect) ;
				break ;
			case 4:
				trainer.evaluate() ;
				break ;
			default:
				System.out.println("Invalid Input") ;
			}
		}
	}
```

The main function takes as arguments:
* path to a directory in which files (article sets, feature data and models) can be saved
* path to a wikipedia configuration file.

It constructs the workbench with the data directory and a Wikipedia instance in which important databases ( *linksInNoSentences* and *label*) have been cached to memory. It then enters a loop asking the user what they want to do.

##Gather article sets

Lets fill in the method for gathering suitable training articles. 

```java
    private void gatherArticleSets() throws IOException{
        int[] sizes = {200,100,100} ;

        ArticleSet[] articleSets = new ArticleSetBuilder()
            .setMinOutLinks(25)
            .setMinInLinks(50)
            .setMaxListProportion(0.1)
            .setMinWordCount(1000)
            .setMaxWordCount(2000)
            .buildExclusiveSets(sizes, _wikipedia) ;
		
        articleSets[0].save(_artsTrain) ;
        articleSets[1].save(_artsTestDisambig) ;
        articleSets[2].save(_artsTestDetect) ;
    }
```

This constructs three separate [ArticleSets](../../doc/org/wikipedia/miner/util/ArticleSet.html), the minimum needed to conduct a fair experiment. 

The first (and largest) will be used to train both the disambiguator and link detector. It can also use it for tweaking the algorithm&mdash;trying out different classifiers or text processors, for example, using [cross-validation][crossvalidation]. The latter two are for the final evaluation of the disambiguator and detector respectively; if we use either of these more than once, we are doing something wrong.

[crossvalidation]: http://en.wikipedia.org/wiki/Cross-validation_(statistics)

The [ArticleSetBuilder](../../doc/org/wikipedia/miner/util/ArticleSetBuilder.html) is used to generate these three article sets randomly, but with certain constrains to suit the kinds of documents you want to annotate (so feel free to edit these to suit you). 

You can try running the code at this point and selecting the first option. The end result should be three csv files containing the ids of suitable articles for training and testing.

##Create ARFF files (woof!)

The next step produces [ARFF Files](http://weka.wikispaces.com/ARFF+%28stable+version%29) that can be loaded into Weka. These files expand on the training articles, turning each article id into a large list of positive and negative instances (senses that should or should not be chosen by the disambiguator; topics that should or should not be chosen by the linkDetector), and all of the features available for distinguishing between them. 

```java
    private void createArffFiles(String datasetName) throws IOException, Exception {
		
        if (!_artsTrain.canRead()) 
            throw new Exception("Article sets have not yet been created") ;
		
        ArticleSet trainingSet = new ArticleSet(_artsTrain, _wikipedia) ;
		
        _disambiguator.train(trainingSet, SnippetLength.full, datasetName + "_disambiguation", null) ;
        _disambiguator.saveTrainingData(_arffDisambig) ;
        _disambiguator.buildDefaultClassifier();
		
        _linkDetector.train(trainingSet, SnippetLength.full, datasetName + "_detection", _topicDetector, null) ;
        _linkDetector.saveTrainingData(_arffDetect) ;
    }
```

Running the code and selecting option 2 will generate two ARFF files (one for disambiguation, one for detection) from the training set. 

WEKA provides a nice environment called the [Explorer](http://www.slideshare.net/dataminingtools/weka-the-explorer) for loading up these files and experimenting with different filters and classifiers. It is a good idea to use this to quickly run some rough experiments and come up with classifiers that work well for each of the two tasks.

##Create classification models

Having tested some classifiers in WEKA, the next step is to build models for them using the entire training set:

```java
    private void createClassifiers(String configDisambig, String configDetect) throws Exception {
		
        if (!_arffDisambig.canRead() || !_arffDetect.canRead())
            throw new Exception("Arff files have not yet been created") ;
		
        _disambiguator.loadTrainingData(_arffDisambig) ;
        if (configDisambig == null || configDisambig.trim().length() == 0) {
            _disambiguator.buildDefaultClassifier() ;
        } else {
            Classifier classifier = buildClassifierFromOptString(configDisambig) ;
            _disambiguator.buildClassifier(classifier) ;
        }
        _disambiguator.saveClassifier(_modelDisambig) ;
		
        _linkDetector.loadTrainingData(_arffDetect) ;
        if (configDetect == null || configDisambig.trim().length() == 0) {
            _linkDetector.buildDefaultClassifier() ;
        } else {
            Classifier classifier = buildClassifierFromOptString(configDisambig) ;
            _linkDetector.buildClassifier(classifier) ;
        }
        _linkDetector.saveClassifier(_modelDetect) ;
    }

    private Classifier buildClassifierFromOptString(String optString) throws Exception {
        String[] options = Utils.splitOptions(optString) ;
        String classname = options[0] ;
        options[0] = "" ;
        return (Classifier) Utils.forName(Classifier.class, classname, options) ;
    }
```

Running the code and selecting option 3 will produce two `.model` files, one for disambiguation and one for detection. These models capture the logic of the fully trained classifier. If they work well (which we will find out in the next step) then you can use these instead of the models included in the toolkit. 

Before each model is created, you will be prompted for a *Disambiguation classifier config* and a *Detection classifier config*. These are the strings found at the top of the *Classify* tab of Weka's explorer, which describe the classifier you constructed and how you configured it. If you didn't bother playing around with Weka, you can just press ENTER to use the default classifier configuration.

##Evaluate the classifiers

The final step (finally!) is to run these classifiers over the training data, to see how well they perform on documents they have never seen before. 

```java
    private void evaluate() throws Exception {
		
        if (!_modelDisambig.canRead() || !_modelDetect.canRead()) 
            throw(new Exception("Classifier models have not yet been created")) ;
		
        if (!_artsTestDisambig.canRead() || !_artsTestDetect.canRead()) 
            throw(new Exception("Article sets have not yet been created")) ;
		
        ArticleSet disambigSet = new ArticleSet(_artsTestDisambig, _wikipedia) ;
        _disambiguator.loadClassifier(_modelDisambig) ;
        Result<Integer> disambigResults = _disambiguator.test(disambigSet, _wikipedia, SnippetLength.full, null) ;
		
        ArticleSet detectSet = new ArticleSet(_artsTestDetect, _wikipedia) ;
        _linkDetector.loadClassifier(_modelDetect) ;
        Result<Integer> detectResults = _linkDetector.test(detectSet, SnippetLength.full, _topicDetector, null) ;
		
        System.out.println();
        System.out.println("Disambig results: " + disambigResults) ;
        System.out.println("Detect results: " + detectResults) ;
    }
```

Running the code and selecting option 4 will produce [precision, recall and f-measure](http://en.wikipedia.org/wiki/Precision_and_recall) scores for both the disambiguator and detector. Your milage may vary, but you are doing well if you get >95% f-measure for disambiguation and >75% f-measure for detection. 

###That's it (or is it?) 

* Try modifying the [[Command-line Annotator|A command-line document annotator]] to use these models, so you can see how they work first-hand.
* Try creating an `evaluateWithDefaultModels` method that uses the models packaged in the toolkit. Do your new models perform better for the kinds of documents you want to process?
* What happens if the classifiers have more training data to learn from?
