Wikipedia Miner provides a suite of REST-like XML-over-HTTP web services that make it easy for applications (particularly web applications) to communicate with it.

To deploy these web services yourself, start by [[installing the Java API| Installing the Java API]], then do the following:


##Create a hub configuration file

Make a copy of `configs/hub-template.xml` and name it something memorable. 

Edit it to describe the different editions of Wikipedia that you have [[built previously|Installing the Java API]] and want to be available via the web services, and how to authenticate with your proxy, if necessary. Make sure you any paths you specify are fully qualified.

The file is well-commented and should be self-explanatory, but more details are available [[here|Configuring a hub of Wikipedias]]. 

##Create a Tomcat web configuration file

Make a copy of `configs/web-template.xml` and name it web.xml (in the same location). Open it and locate the param that talks about `hubConfigFile`

```xml
    <context-param>
      <param-name>hubConfigFile</param-name>
      <param-value></param-value>
      <description>
        An XML file describing the wikipedia dumps that are available via these web services,
	and how they should be configured.
      </description>
    </context-param>  
```

Fill in the `param-value` element with the fully qualified path to the hub config file you created in the previous step. You can also edit the `web.xml` file at this point to swap out any web services that you don't want to deploy.

Check out [[this page|http://tomcat-configure.blogspot.com/2009/01/tomcat-web-xml.html]] for more information about Tomcat web.xml files.

##Create a WAR file

Navigate to the `ant` directory of the toolkit, and run the `deploy` target
```
    ant deploy
```

This will create `build/wikipedia-miner.war`, which contains everything needed to host the wikipedia-miner website. You can put it wherever you like. 

##Install Tomcat

Obtain a copy of [Apache Tomcat](http://tomcat.apache.org). Get it up and running (instructions [here](http://tomcat.apache.org/tomcat-7.0-doc/setup.html)), and check it (make sure you see something when you go to [[http://localhost:8080]] in your browser).

Next, configure tomcat to add the war file created earlier as a new context. There are many ways to do this, but I find the easiest is to navigate to tomcat's `conf/Catalina/localhost` directory, and create a new file called `wikipediaminer.xml`, with this content:

```xml
<Context docBase="path/to/wikipedia/miner/war" reloadable="true" allowLinking="true" />
```

##Caching and performance

Some services will run abysmally slowly unless the correct databases are cached to memory. To fix this, you need to configure the toolkit to cache certain databases, and give tomcat the memory it requires. 

###Configuring the toolkit

Edit each [[Wikipedia XML configuration file|Configuring a single Wikipedia instance]] to tell it which databases to cache to memory. Which databases you cache will depend on your needs and the memory you have available, but the table below gives you a rough guide. 

service    | databases to cache
:----------|:------
*search*   | `label`
*compare*  | depends on the article comparison dependencies used, but generally `pageLinksInNoSentences`
*annotate* | all of the above
*suggest*  | the same as *compare*, plus `articleParents`

###Configuring tomcat

You also need to give tomcat the memory it needs to cache these databases, by editing the CATALINA_OPTS environment variable. 

Playing around with wikipedia miner [[from within java|Installing the Java API]] will give you a clear idea of how much is needed. It depends on the size of the wikipedia editions you deploy, which databases you cache to memory, and the different options you use to exclude items from the cache. Adding additional memory won't make a big impact on performance: so just use enough to avoid running out of heap space.

Under linux or OSX, you can set the environment variable by editing your `~/.bash_profile` to include the line:

```
export CATALINA_OPTS='-Xmx 2g'
```

That will set the environment variable next time the terminal is restarted. You can force this to happen now with this command:

```
source ~/.bash_profile
```

And verify that the variable is set with this command:
```
echo $CATALINA_OPTS
```
which should print the value of the variable to the screen.

##Try the services out

Once you restart tomcat, you should be able to go to [[http://localhost:8080/wikipediaminer/services]] for documentation on the available web services, or [[http://localhost:8080/wikipediaminer/demos]] to try out the demos that call these services. 

Pick one, and try it out!
