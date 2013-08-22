Here is a small sample from the infinite list of things Wikipedia Miner can't do. 

* **Live updates**: This toolkit requires you to download and preprocess you own static dump of Wikipedia. Because of that, you will always be a few days behind. For real-time access to Wikipedia, try the [Wikipedia API](http://www.mediawiki.org/wiki/API).

* **RDF/Semantic Web compliance**: The toolkit is for developers and researchers who want closely examine the raw structure of Wikipedia. If you are looking for a pre-built Wikipedia-based ontology, then something like [DBpedia](http://www.dbpedia.org) or [FreeBase](http://www.freebase.com) will probably be more relevant to you. 

* **Anything much with templates, info-boxes or revision history**:Although these aspects of Wikipedia have been very fruitful for researchers, we simply haven't had a chance to look into them. 

* **Comprehensive parsing of MediaWiki syntax** This toolkit includes code to strip out MediaWiki syntax to obtain plain text or html versions of Wikipedia's content, but this is far from comprehensive. Other (better) parsers are available [here](http://www.mediawiki.org/wiki/Alternative_parsers).

* **Anything with revision history**: The toolkit can handle multiple editions of Wikipedia, so you could use it to track changes over time, but we have not done a lot in this area. Check out the [JWPL](http://www.ukp.tu-darmstadt.de/software/jwpl) for better support for revision history.

Remember, Wikipedia Miner is entirely open source, and is free to evolve as you see fit.
