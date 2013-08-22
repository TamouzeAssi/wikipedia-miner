The ''languages.xml'' file (located in the ''config'' directory of the toolkit) describes language-dependent variables that are required for processing Wikipedia dumps. It is often necessary to modify and update this file to process new dumps of Wikipedia.

##Format

The language configuration file contains one Language element per language version of Wikipedia. Each of these contains RootCategory, DisambiguationCategory, and DisambiguationTemplate elements as children.

###Language

The Language element must provide the following attributes:
||code|A short (typically two letter) code for the language (e.g. en, de, fr)
||name|A name (in English) for the language version
||localName|A name (in the given language) for the language version

###RootCategory
Describes the root category, from which all other content-based categories descend (e.g. [this one|http://en.wikipedia.org/wiki/Category:Fundamental_categories] for the ''en'' version)

There should be only one root category per language.

###DisambiguationCategory

Describes a category to which disambiguation pages often belong to (e.g. [this one|http://en.wikipedia.org/wiki/Category:Disambiguation] for the ''en'' version)

There may be multiple disambiguation categories per language.

###DisambiguationTemplate

Describes a template that is often invoked on disambiguation pages (e.g. [this one|http://en.wikipedia.org/wiki/Template:Disambig] for the ''en'' version)

There may be multiple disambiguation templates per language.

###RedirectIdentifier

Describes the syntax used to create redirects. For example, if redirects are identified like this:

```
#REDIRECT [[target]]
```

then you should use the following redirect identifier
```xml
<RedirectIdentifier>REDIRECT</RedirectIdentifier>
```

There may be multiple redirect identifiers per language.


##Current languages

Below are examples of Language elements for processing different versions of Wikipedia. Please feel free to add and modify these.

###Full English Wikipedia
 
```xml
        <Language code="en" name="English" localName="English">
        
               <RootCategory>Fundamental categories</RootCategory>
               
               <DisambiguationCategory>disambiguation</DisambiguationCategory>
               
               <DisambiguationTemplate>disambiguation</DisambiguationTemplate>
               <DisambiguationTemplate>disambig</DisambiguationTemplate>
               <DisambiguationTemplate>geodis</DisambiguationTemplate>
               <DisambiguationTemplate>hndis</DisambiguationTemplate>
               <DisambiguationTemplate>hospitaldis</DisambiguationTemplate>
               <DisambiguationTemplate>mathdab</DisambiguationTemplate>
               <DisambiguationTemplate>mountianindex</DisambiguationTemplate>
               <DisambiguationTemplate>numberdis</DisambiguationTemplate>
               <DisambiguationTemplate>roaddis</DisambiguationTemplate>
               <DisambiguationTemplate>schooldis</DisambiguationTemplate>
               <DisambiguationTemplate>shipindex</DisambiguationTemplate>
               <DisambiguationTemplate>SIA</DisambiguationTemplate>
               
               <RedirectIdentifier>REDIRECT</RedirectIdentifier>
               
       </Language>
```

###Simple English Wikipedia

```
        <Language code="simple" name="Simple English" localName="Simple English">
        
               <RootCategory>articles</RootCategory>
               
               <DisambiguationCategory>disambiguation</DisambiguationCategory>
               
               <DisambiguationTemplate>disambiguation</DisambiguationTemplate>
               <DisambiguationTemplate>disambig</DisambiguationTemplate>
               <DisambiguationTemplate>2CC</DisambiguationTemplate>
               <DisambiguationTemplate>3CC</DisambiguationTemplate>
               <DisambiguationTemplate>hndis</DisambiguationTemplate>
               
               <RedirectIdentifier>REDIRECT</RedirectIdentifier>
               
       </Language>
```

###German Wikipedia

```
        <Language code="de" name="German" localName="Deutsch">

		<RootCategory>!Hauptkategorie</RootCategory>

		<DisambiguationCategory>Begriffsklärung</DisambiguationCategory>

		<DisambiguationTemplate>Begriffsklärung</DisambiguationTemplate>
		
		<RedirectIdentifier>REDIRECT</RedirectIdentifier>
		<RedirectIdentifier>WEITERLEITUNG</RedirectIdentifier>
		
	</Language>
```
