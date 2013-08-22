<dl>
<dt>Summarizes Wikipedia's link structure, category structure, page types, etc.</dt>
<dd>A sequence of Hadoop jobs are provided to extract statistics and summaries from Wikipedia's static xml dumps. These scripts scale in roughly linear time, depending on the size of Wikipedia and the number of machines available to the Hadoop cluster.</dd>

<dt>Models Wikipedia as easy to understand Java classes</dt>
<dd>such as Article, Category, Anchor, etc. See the Java Doc for details.</dd>

<dt>Indexes data for efficient access</dt>
<dd>The summarized data is stored persistently in a Java BerlekeyDb database environment. You can access it immediately, without waiting for anything to load.</dd>

<dt>Caches summaries to memory if required</dt>
<dd>Sometimes you will rather spend time pre-loading the summaries to memory, so you can avoid the overhead of constantly querying the database. The toolkit allows you to flexibly cache databases to memory, depending on the needs of your application</dd>

<dt>Provides flexible searching, via link anchors, titles and redirects</dt>
<dd>as they occur or via stemming, case-folding, etc. You can also add your own search methods, and prepare the data so that they can be used efficiently.</dd>

<dt>Measures how Wikipedia's concepts relate to each other</dt>
<dd>The toolkit includes proven semantic relatedness measures that efficiently and accurately measure how topics relate to each other. </dd>

<dt>Detects Wikipedia topics when they are mentioned in documents</dt>
<dd>This includes machine-learned approaches for disambiguating ambiguous terms, and identifying the topics that are most likely to be of interest to the reader.</dd>
</dl>