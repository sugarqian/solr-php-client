

# About the Library and its Authors #
Information on this project, its authors, and its roots.

---


## What is Solr? ##
> Solr is an open source enterprise search server based on the Lucene Java search library, with XML/HTTP and JSON APIs, hit highlighting, faceted search, caching, replication, a web administration interface and many more features. It runs in a Java servlet container such as Tomcat.

For more information about Solr, please see the [Solr Web Page](http://lucene.apache.org/solr/) .

This client was originally in Solr's issue tracker: https://issues.apache.org/jira/browse/SOLR-341

---


## Why the New BSD license? ##
Conduit originally submitted the PHP client to the Solr project under the Apache 2.0 license so that it was compatible to be included in the Solr distribution. Since then, we've been asked several times to allow the client to use other licenses so it could be included in other projects (e.g. a Drupal search plugin).  We think that the new BSD license is the most compatible with the largest number of other OSS licenses.

---


## Who is Servigistics? ##
[![](http://www.servigistics.com/images/global/header/LogoMain.jpg)](http://servigistics.com)

Servigistics, Inc. is the company where I work. We started using Solr to augment our search capabilities in 2007 and continue to use it today. At the time there were PHP code samples for how to interface with Solr, but we saw a need for something more - particularly in the handling of the nitty gritty HTTP requests and response serializations. I asked my employer for permission to distribute the client source as OSS, and they happily allowed it.

For more information on Conduit, please see the [Servigistics Web Page](http://servigistics.com).

---


# General Usage #
General usage questions.

---


## Which Version Should I use ? ##
Trunk is kept very stable, so I would actually recommend using a checkout of that directly if you can. However, periodically, usually after a number of bug fixes and / or features we will wrap up the current version and publish it as zip and tarball download. If you don't want to use trunk then use the most recently released archive of your choosing.

Please limit your submission of issues to those that are present in trunk. If you're having an issue, make sure its not resolved already in the the current trunk.

---


## What Example Code is Available? ##
See the ExampleUsage wiki page for some PHP code as well as a link to an IBM Developer Works article.

---


## How Can I Use Additional Parameters (like fq, facet, etc) ##
The Apache\_Solr\_Service::search() has its first three arguments mapped to q, start, and rows repsectively. These are the three most common parameters for any query. However, there are a number of additional parameters available to control the behavior of Solr request handlers. Any of these addtional parameters can be passed along in the search method's 4th argument. This argument takes an array of key / value pairs where the key is the additional parameter name (e.g. 'fq') and the value is a single scalar value (will be converted to a string for the query) or an array of scalar values - use arrays when a parameter has multiple values (e.g. you are sending multiple 'facet.field' values).

```
$solr = new Apache_Solr_Service();

$query = "your query";
$start = 0;
$rows = 10;

$additionalParameters = array(
    'fq' => 'a filtering query',
   'facet' => 'true',
   // notice I use an array for a muti-valued parameter
   'facet.field' => array(
        'field_1',
        'field_2'
    )
);

$results = $solr->search($query, $start, $rows, $additionalParameters);
```

The call above would be mapped to the URL:
http://localhost:8180/solr/select?q=your+query&start=0&rows=10&fq=a+filtering+query&facet=true&facet.field=field_1&facet.field=field_2

---


## How Do I Use A Non-Default Request Handler? ##
All named request handlers can be accessed by using the 'qt' additional query parameter in your search call (see the section on additional parameters above). For example, to use the DisMax request handler configured in the distributed Solr configuration you would pass a value of 'dismax' for the the 'qt' parameter.

---


## How do I Add Document / Field Boosts? ##
There are two levels of boost available in every document. Document level boosts apply to the whole document when searching. Field level boosts apply only to a particular document field.

The Apache\_Solr\_Document class contains methods for setting a document level boost as well as individual field boosts. You'll need to use the document class for adding documents to solr that you want boosts on. The methods used below should all be documented in their related documentation blocks in source as well.

```
...

$document = new Apache_Solr_Document();

// set a boost for the document
$document->setBoost(2);

// get the current document boost
$document->getBoost();

// add a field with a boost as the last parameter
$document->addField("field", "value", 2);

// or set a field boost separately
$document->field = "value";
$document->setFieldBoost("field", 2);

// get a current field's boost
$boost = $document->getFieldBoost("field");

// or get all field boosts - array indexed by field name
$boosts = $document->getFieldBoosts();

// document boosting can be removed by setting to false
$document->setBoost(false);

// field boosting can also be removed by setting to false
$document->setFieldBoost("field", false);

...
```

---
