---
title: Using SPARQL to access Linked Open Data
authors:
- Matthew Lincoln
date: 2015-06-21
layout: default
categories: [lessons]
---

Lesson Goals
------------

This lesson explains why many cultural institutions are adopting graph
databases, and how researchers can access these data though the query language
called SPARQL.

# Graph Databases, RDF, and Linked Open Data

Many cultural institutions now offer access to their collections information
through [web Application Programming Interfaces][api]. While these APIs are a
powerful way to access individual records in a machine-readable manner, they are
not ideal for cultural heritage data because they are structured to work for a
predetermined set of queries. For example, a museum may have information on
donors, artists, artworks, exhibitions, and provenance, but its web API may
offer only object-wise retrieval, with associated data about donors, artists,
provenance, etc. embedded within subfields of each object's JSON data. This
structure is great if you come looking for information about particular objects.
However, it makes it difficult to aggregate information about every artist or
donor that happens to be described in the dataset as well.

[api]: /lessons/intro-to-the-zotero-api.html

RDF databases (also called graph databases) are well-suited to expressing
complex relationships between many entities, like people people, places, events,
and concepts tied to individual objects. Because RDF databases support the use
of URLs (weblinks), they can be made available online and linked to other
databases, hence the term "Linked Open Data". Major art collections including
the [British Museum][bm], [Europeana], the [Smithsonian American Art
Museum][saam], and the [Paul Mellon Center for British Art][mellon] have
published their collections data as LOD, and they have been joined by the [Getty
Vocabulary Program][getty].

[getty]: http://vocab.getty.edu

[bm]: http//collection.britishmuseum.org

[Europeana]: http://data.eurpoeana.org

[saam]: http://americanart.si.edu

[mellon]: http://britishart.yale.edu/collections/using-collections/technology/linked-open-data

SPARQL is the language used to query these databases. This language is
particularly powerful because it does not presuppose the perspectives that users
will bring to the data. A query about objects and a query about donors is
basically equivalent to such a database. Unfortunately, many tutorials on SPARQL
use extremely simplified data models that don't resemble the more complex
datasets released by cultural heritage institutions. This tutorial gives a crash
course on SPARQL using a dataset that a humanist might actually find in the
wilds of the Internet. In this tutorial, we will learn how to query the British
Museum Linked Open Data collection.

## RDF in brief

RDF represents information in a series of three-part "statements" that comprise
a subject, predicate, and an object, e.g.:

```
<The Nightwatch> <was created by> <Rembrandt van Rijn> .
```

(Note that just like any good sentence, they each have a period at the end.)

Each of these elements is a node within a vast network. A subject in one
statement may be an object (or even a predicate) in another statement. A
psuedo-RDF database might contain interrelated statements like these:

```
...
<The Nightwatch> <was created by> <Rembrandt van Rijn> .
<The Nightwatch> <was created in> <1642> .
<The Nightwatch> <has medium> <oil on canvas> .
<Rembrandt van Rijn> <was born in> <1606> .
<Rembrandt van Rijn> <has nationality> <Dutch> .
<Johannes Vermeer> <has nationality> <Dutch> .
<Woman with a Balance> <was created by> <Johannes Vermeer> .
<Woman with a Balance> <has medium> <oil on canvas> .
...
```

However, if we were to visualize this as a network graph, it would appear like
so:

{% include figure.html src="/images/sparql01.png" caption="A network visualization of the pseudo-RDF shown above." %}

A traditional relational database might split attributes about artworks and
attributes about artists into separate tables. In an RDF/graph database, all
these data points belong to the same interconnected graph, which allows users
maximum flexibility in deciding how they wish to query it.

## Searching RDF with SPARQL

SPARQL lets us translate heavily interlinked, graph data into normalized,
tabular data with rows and columns you can open in programs like Excel, or
import into a visualization suite such as [plot.ly](http://plot.ly) or
[Palladio].

It is useful to think of a SPARQL query as a [Mad
Lib](https://en.wikipedia.org/wiki/Mad_Libs) - a set of sentences with blanks in
them. The database will take this query and find every set of matching
statements that correctly fill in those blanks, returning the matching values to
us as a table. Take this SPARQL query:

```
SELECT ?painting
WHERE {
  ?painting <has medium> <oil on canvas> .
}
```

On receiving this query, the database will search for all values of `?painting`
that properly complete the RDF statement `<has medium> <oil on canvas> .` We
might visualize this query like so:

{% include figure.html src="/images/sparql02.png" caption="A visualization of the SPARQL query, with mentioned nodes and edges in orange, and selected nodes (those that will be returned in the results) in red." %}

And our results might look like this table:

| painting             |
|----------------------|
| The Nightwatch       |
| Woman with a Balance |

What makes RDF and SPARQL powerful is the ability to create complex queries that
reference many variables at a time. For example, we could search our pseudo-RDF
database for  paintings by any artist who is Dutch:

```
SELECT ?artist ?painting
WHERE {
  ?artist <has nationality> <Dutch> .
  ?painting <was created by> ?artist .
}
```

Here we've introduced a second variable, `?artist`. The RDF database will return
all matching combinations of `?artist` and `?painting` that fulfill both of
these statements.

| artist             | painting             |
|--------------------|----------------------|
| Rembrandt van Rijn | The Nightwatch       |
| Johannes Vermeer   | Woman with a Balance |

## URIs

So far, we have been looking at a toy representation of RDF that uses
easy-to-read text. However, RDF is primarily stored as URIs (Uniform Resource
Identifiers, otherwise recognizable as hyperlinks) that separate conceptual
entities from their plain-English (or other langauage!) labels. In real RDF, our
original statement:

```
<The Nightwatch>   <was created by>   <Rembrandt van Rijn> .
```

would more likely look something like this:

```
<http://data.rijksmuseum.nl/item/8909812347> <http://purl.org/dc/terms/creator>  <http://dbpedia.org/resource/Rembrandt>.
```

_N.B. the Rijksmuseum has not (yet) built their own Linked Data site, so the URL in this query is just for demo purposes._

In order to get the _labels_ for each of these URI's, what we're really doing is
just retrieving more RDF statements:

```
<http://data.rijksmuseum.nl/item/8909812347> <http://purl.org/dc/terms/title> "The Nightwatch" .

<http://purl.org/dc/terms/creator> <http://www.w3.org/1999/02/22-rdf-syntax-ns#label> "was created by" .

<http://dbpedia.org/resource/Rembrandt> <http://xmlns.com/foaf/0.1/name> "Rembrandt van Rijn" .
```

The _objects_ of these statements in quotation marks are just strings of text,
known as _literals_. Other literal values in RDF include dates and numbers.

See the _predicates_ in these statements, with domain names like `purl.org`,
`w3.org`, and `xmlns.com`? These are some of the many providers of ontologies
that help standardize the way we describe relationships between bits of
information like "title", "label", "creator", or "name". The more RDF/LOD that
you work with, the more of these providers you'll find.

URIs can become unwieldy when composing SPARQL queries, which is why we'll
use _prefixes_. These are shortcuts that allow us to skip typing out entire long
URIs. For example, remember that predicate for retrieving the title of the
*Nightwatch*, `<http://purl.org/dc/terms/title>`? With these prefixes, we just
need to type `dct:title` whenever we need to use a `purl.org` predicate. `dct:`
stands in for `http://purl.org/dc/terms/`, and `title` just gets pasted onto the
end of this link.

For example, with the prefix `PREFIX rkm: <http://data.rijksmuseum.nl/>`,
appended to the start of our SPARQL query,
`<http://data.rijksmuseum.nl/item/8909812347>` becomes `rkm:item/8909812347`
instead.

Many live databases, such as the British Museum database that we will try our
first new queries on, already define these prefixes for us, so we won't have to
explicitly state them in our queries. However, you should now be able to
recognize whenever we use one in a SPARQL query.

## Terms to review

- **SPARQL** - _Protocol and RDF Query Language_ - The language used to query RDF graph databases
- **RDF** - _Resource Description Framework_ - A method for structuring data as a graph or network of connected statements, rather than a series of tables.
- **LOD** - _Linked Open Data_ - LOD is RDF data published online with dedicated URIs in such a manner than developers can reliably reference it.
- **URI** - _Uniform Resource Identifier_ - Also known as a URL (uniform resource locator), or a web link.
- **node** - A node in a graph database represents some entity that has relationships to other entities within the database.
- **prefix** - In order to simplify SPARQL queries, a user may specify prefixes that act as abbreviations for full URIs.
- **statement** - Sometimes also called a "triple", an RDF statement is a quantum of knowledge comprising a _subject_, _predicate_, and _object_.

# Real-world queries

## All the statements for one object

Let's start our first query using the [British Museum SPARQL endpoint][bms]. A
SPARQL endpoint is a web address that accepts SPARQL queries and returns
results. The BM endpoint is like many others: if you navigate to it in a web
browser, it presents you with a text box for composing queries.

[bms]: http://collection.britishmuseum.org/sparql

{% include figure.html src="/images/sparql03.png" caption="The BM SPARQL endpoint webpage. For all the queries in this tutorial, make sure that you have left the 'Include inferred' and 'Expand results over equivalent URIs' boxes unchecked." %}

When starting to explore a new RDF database, it helps to look at the
relationships that stem from a single [example
object](http://collection.britishmuseum.org/id/object/PPA82633).

(For each of the following queries, click on the "Run query" link below to see
the results. Click on the "Edit query" link to go to a page with the query
automatically pasted the query into the BM's endpoint. You can then run it as
is, or modify it before requesting the results. Remember when editing the query
before running to uncheck the 'Include inferred' box.)

```
SELECT ?p ?o
WHERE {
  <http://collection.britishmuseum.org/id/object/PPA82633> ?p ?o .
}
```
[Run query](http://collection.britishmuseum.org/sparql?query=SELECT+*%0D%0AWHERE+%7B%0D%0A++%3Chttp%3A%2F%2Fcollection.britishmuseum.org%2Fid%2Fobject%2FPPA82633%3E+%3Fp+%3Fo+.%0D%0A++%7D&_implicit=false&_equivalent=false&_form=%2Fsparql) /
[Edit query](http://collection.britishmuseum.org/sparql?sample=SELECT+*%0D%0AWHERE+%7B%0D%0A++%3Chttp%3A%2F%2Fcollection.britishmuseum.org%2Fid%2Fobject%2FPPA82633%3E+%3Fp+%3Fo+.%0D%0A++%7D)

By calling `SELECT ?p ?o` we're asking the database to return the values of `?p`
and `?o` as described in the `WEHRE {}` command This query returns every
statement for which our example artwork,
`<http://collection.britishmuseum.org/id/object/PPA82633>`, is the subject.

{% include figure.html src="/images/sparql04.png" caption="An initial list of all the predicates and objects associated with one artwork in the British Museum." %}

The BM endpoint formats the results table with hyperlinks for every variable
that is itself an RDF node, so by clicking on any one of these links you can
shift to seeing all the predicates and objects for that newly-selected node.
Note that BM automatically includes a wide range of SPARQL prefixes in its
queries, so you will find many hyperlinks are displayed in their abbreviated
versions; if you mouse over them your browser will display their unabbreviated
URIs.

Let's find out how they store the object type information: look for the
predicate `<bmo:PX_object_type>` (highlighted in the figure above) and click on
the link for `thes:x8577` to navigate to the node describing the particular
object type "print":

{% include figure.html src="/images/sparql05.png" caption="The resource page for thes:x8577 ('print') in the British Museum LOD." %}

You'll note how this node has an English label, as well as ties to related
artwork type nodes within the database.

## Complex queries

To find other objects of the same type with the preferred label "print", we can
call this query:

```
SELECT ?object
WHERE {

  # Search for all values of ?object that have a given "object type"
  ?object bmo:PX_object_type ?object_type .

  # That object type should have the label "print"
  ?object_type skos:prefLabel "print" .
}
```
[Run query](http://collection.britishmuseum.org/sparql?query=SELECT+%3Fobject%0D%0AWHERE+%7B%0D%0A++%3Fobject+bmo%3APX_object_type+%3Fobject_type+.%0D%0A++%3Fobject_type+skos%3AprefLabel+%22print%22+.%0D%0A%7D&_implicit=false&implicit=false&_equivalent=false&_form=%2Fsparql) / [Edit query](http://collection.britishmuseum.org/sparql?sample=PREFIX+bmo%3A+%3Chttp%3A%2F%2Fcollection.britishmuseum.org%2Fid%2Fontology%2F%3E%0APREFIX+skos%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23%3E%0ASELECT+%3Fobject%0D%0AWHERE+%7B%0D%0A++%3Fobject+bmo%3APX_object_type+%3Fobject_type+.%0D%0A++%3Fobject_type+skos%3AprefLabel+%22print%22+.%0D%0A%7D)

{% include figure.html src="/images/sparql06.png" caption="A one-column table returned by our query for every object with type 'print'" %}

In this query, `?object_type` is a _blank node_. Because it is not present in
the `SELECT` command, it will not show up in the results table. However, it is
essential to structuring our query, because it connects the dots from object to
type label.

## FILTER

In the previous query, our SPARQL query searched for an exact match for the
object type with the text label "print". However, often we want to match values
within a certain range, such as dates. For this, we'll use the `FILTER` command.

To find URIs for all the prints in the BM created between 1580 and 1600, we'll
need to first figure out where the database stores dates in relationship to the
object node, and then add references to our query that describe where to find
the dates that objects were created. Similar to the way that we followed a
single link to determine an object type, we must hop through several nodes to
find the production dates associated with a given object:

{% include figure.html src="/images/sparql07.png" caption="Visualizing part of the British Museum's data model where production dates are connected to objects." %}

```
# Return object links and creation date
SELECT ?object ?date
WHERE {

  # We'll use our previous command to search only for
  # objects of type "print"
  ?object bmo:PX_object_type ?object_type .
  ?object_type skos:prefLabel "print" .

  # We need to link though several nodes to find the
  # creation date associated with an object
  ?object ecrm:P108i_was_produced_by ?production .
  ?production ecrm:P9_consists_of ?date_node .
  ?date_node ecrm:P4_has_time-span ?timespan .
  ?timespan ecrm:P82a_begin_of_the_begin ?date .

  # As you can see, we need to connect quite a few dots
  # to get to the date node! Now that we have it, we can
  # filter our results. Because we are filtering by date,
  # we must attach the xsd:date tag to our date strings
  # so that the database can parse and compare them.

  FILTER(?date >= "1580-01-01"^^xsd:date &&
         ?date <= "1600-01-01"^^xsd:date)
}
```

[Run query](http://collection.britishmuseum.org/sparql?query=%23+Return+object+links+and+creation+date%0D%0ASELECT+%3Fobject+%3Fdate%0D%0AWHERE+%7B%0D%0A%0D%0A++%23+We%27ll+use+our+previous+command+to+search+only+for+objects+of+type+%22print%22%0D%0A++%3Fobject+bmo%3APX_object_type+%3Fobject_type+.%0D%0A++%3Fobject_type+skos%3AprefLabel+%22print%22+.%0D%0A%0D%0A++%23+We+need+to+link+though+several+nodes+to+find+the+creation+date+associated%0D%0A++%23+with+an+object%0D%0A++%3Fobject+ecrm%3AP108i_was_produced_by+%3Fproduction+.%0D%0A++%3Fproduction+ecrm%3AP9_consists_of+%3Fdate_node+.%0D%0A++%3Fdate_node+ecrm%3AP4_has_time-span+%3Ftimespan+.%0D%0A++%3Ftimespan+ecrm%3AP82a_begin_of_the_begin+%3Fdate+.%0D%0A%0D%0A++%23+Yes%2C+we+need+to+connect+quite+a+few+dots+to+get+to+the+date+node%21+Now+that%0D%0A++%23+we+have+it%2C+we+can+filter+our+results.+Because+we+are+filtering+a+date%2C+we%0D%0A++%23+must+attach+the+xsd%3Adate+tag+to+our+date+strings+so+that+SPARQL+knows+how+to%0D%0A++%23+parse+them.%0D%0A%0D%0A++FILTER%28%3Fdate+%3E%3D+%221580-01-01%22%5E%5Exsd%3Adate+%26%26+%3Fdate+%3C%3D+%221600-01-01%22%5E%5Exsd%3Adate%29%0D%0A%7D&_implicit=false&_equivalent=false&_form=%2Fsparql) / [Edit query](http://collection.britishmuseum.org/sparql?sample=%23+Return+object+links+and+creation+date%0D%0APREFIX+bmo%3A+%3Chttp%3A%2F%2Fcollection.britishmuseum.org%2Fid%2Fontology%2F%3E%0APREFIX+skos%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23%3E%0APREFIX+ecrm%3A+%3Chttp%3A%2F%2Ferlangen-crm.org%2Fcurrent%2F%3E%0APREFIX+xsd%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23%3E%0ASELECT+%3Fobject+%3Fdate%0D%0AWHERE+%7B%0D%0A%0D%0A++%23+We%27ll+use+our+previous+command+to+search+only+for+objects+of+type+%22print%22%0D%0A++%3Fobject+bmo%3APX_object_type+%3Fobject_type+.%0D%0A++%3Fobject_type+skos%3AprefLabel+%22print%22+.%0D%0A%0D%0A++%23+We+need+to+link+though+several+nodes+to+find+the+creation+date+associated%0D%0A++%23+with+an+object%0D%0A++%3Fobject+ecrm%3AP108i_was_produced_by+%3Fproduction+.%0D%0A++%3Fproduction+ecrm%3AP9_consists_of+%3Fdate_node+.%0D%0A++%3Fdate_node+ecrm%3AP4_has_time-span+%3Ftimespan+.%0D%0A++%3Ftimespan+ecrm%3AP82a_begin_of_the_begin+%3Fdate+.%0D%0A%0D%0A++%23+Yes%2C+we+need+to+connect+quite+a+few+dots+to+get+to+the+date+node%21+Now+that%0D%0A++%23+we+have+it%2C+we+can+filter+our+results.+Because+we+are+filtering+a+date%2C+we%0D%0A++%23+must+attach+the+xsd%3Adate+tag+to+our+date+strings+so+that+SPARQL+knows+how+to%0D%0A++%23+parse+them.%0D%0A%0D%0A++FILTER%28%3Fdate+%3E%3D+%221580-01-01%22%5E%5Exsd%3Adate+%26%26+%3Fdate+%3C%3D+%221600-01-01%22%5E%5Exsd%3Adate%29%0D%0A%7D)

{% include figure.html src="/images/sparql08.png" caption="All BM prints made between 1580 and 1600" %}

## Aggregation

So far we have only used the `SELECT` command to return a table of objects.
However, SPARQL allows us to do more advanced analysis such as grouping,
counting, and sorting.

Say we would like to keep looking at objects made between 1580 and 1600, but we
want to understand how many objects of each type the BM has in its collections.
Instead of limiting our results to objects of type "print", we will instead use
`COUNT` to tally our search results by type.

```
SELECT ?type (COUNT(?type) as ?n)
WHERE {
  # We still need to indicate the ?object_type variable,
  # however we will not require it to match "print" this time

  ?object bmo:PX_object_type ?object_type .
  ?object_type skos:prefLabel ?type .

  # Once again, we will also filter by date
  ?object ecrm:P108i_was_produced_by ?production .
  ?production ecrm:P9_consists_of ?date_node .
  ?date_node ecrm:P4_has_time-span ?timespan .
  ?timespan ecrm:P82a_begin_of_the_begin ?date .
  FILTER(?date >= "1580-01-01"^^xsd:date &&
         ?date <= "1600-01-01"^^xsd:date)
}
# The GROUP BY command designates the variable to tally by,
# and the ORDER BY DESC() command sorts the results by
# descending number.
GROUP BY ?type
ORDER BY DESC(?n)
```

[Run query](http://collection.britishmuseum.org/sparql?query=SELECT+%3Ftype+%28COUNT%28%3Ftype%29+as+%3Fn%29%0D%0AWHERE+%7B%0D%0A++%23+We+still+need+to+indicate+the+%3Fobject_type+variable%2C+however+we+will+not%0D%0A++%23+require+it+to+match+%22print%22+this+time%0D%0A%0D%0A++%3Fobject+bmo%3APX_object_type+%3Fobject_type+.%0D%0A++%3Fobject_type+skos%3AprefLabel+%3Ftype+.%0D%0A%0D%0A++%23+Once+again%2C+we+will+also+filter+by+date%0D%0A++%3Fobject+ecrm%3AP108i_was_produced_by+%3Fproduction+.%0D%0A++%3Fproduction+ecrm%3AP9_consists_of+%3Fdate_node+.%0D%0A++%3Fdate_node+ecrm%3AP4_has_time-span+%3Ftimespan+.%0D%0A++%3Ftimespan+ecrm%3AP82a_begin_of_the_begin+%3Fdate+.%0D%0A++FILTER%28%3Fdate+%3E%3D+%221580-01-01%22%5E%5Exsd%3Adate+%26%26+%3Fdate+%3C%3D+%221600-01-01%22%5E%5Exsd%3Adate%29%0D%0A%7D%0D%0A%23+The+GROUP+BY+command+designates+the+variable+to+tally+by%2C+and+the+ORDER+BY%0D%0A%23+DESC%28%29+command+sorts+the+results+by+descending+number.%0D%0AGROUP+BY+%3Ftype%0D%0AORDER+BY+DESC%28%3Fn%29&_implicit=false&_equivalent=false&_form=%2Fsparql) / [Edit query](http://collection.britishmuseum.org/sparql?sample=PREFIX+bmo%3A+%3Chttp%3A%2F%2Fcollection.britishmuseum.org%2Fid%2Fontology%2F%3E%0APREFIX+skos%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23%3E%0APREFIX+ecrm%3A+%3Chttp%3A%2F%2Ferlangen-crm.org%2Fcurrent%2F%3E%0APREFIX+xsd%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23%3E%0ASELECT+%3Ftype+%28COUNT%28%3Ftype%29+as+%3Fn%29%0D%0AWHERE+%7B%0D%0A++%23+We+still+need+to+indicate+the+%3Fobject_type+variable%2C+however+we+will+not%0D%0A++%23+require+it+to+match+%22print%22+this+time%0D%0A%0D%0A++%3Fobject+bmo%3APX_object_type+%3Fobject_type+.%0D%0A++%3Fobject_type+skos%3AprefLabel+%3Ftype+.%0D%0A%0D%0A++%23+Once+again%2C+we+will+also+filter+by+date%0D%0A++%3Fobject+ecrm%3AP108i_was_produced_by+%3Fproduction+.%0D%0A++%3Fproduction+ecrm%3AP9_consists_of+%3Fdate_node+.%0D%0A++%3Fdate_node+ecrm%3AP4_has_time-span+%3Ftimespan+.%0D%0A++%3Ftimespan+ecrm%3AP82a_begin_of_the_begin+%3Fdate+.%0D%0A++FILTER%28%3Fdate+%3E%3D+%221580-01-01%22%5E%5Exsd%3Adate+%26%26+%3Fdate+%3C%3D+%221600-01-01%22%5E%5Exsd%3Adate%29%0D%0A%7D%0D%0A%23+The+GROUP+BY+command+designates+the+variable+to+tally+by%2C+and+the+ORDER+BY%0D%0A%23+DESC%28%29+command+sorts+the+results+by+descending+number.%0D%0AGROUP+BY+%3Ftype%0D%0AORDER+BY+DESC%28%3Fn%29)

{% include figure.html src="/images/sparql09.png" caption="Counts of objects by type produced between 1580 and 1600." %}

## Export results to CSV

So far, we have only viewed results in the browser window. However, SPARQL
endpoints are designed to return structured data to be used by other programs.
In the top right corner of the results page for the BM endpoint, you will find
links for both JSON and XML downloads. Other endpoints may also offer the option
for a CSV/TSV download, however this option is not always available.

To quickly convert JSON results from a SPARQL endpoint, I recommend the free
command line utility [jq](http://stedolan.github.io/jq/download/). (For a
tutorial on using command line programs, see ["Introduction to the Bash Command
Line"](/lessons/intro-to-bash.html).) The following query will convert the
special JSON RDF format into a CSV file:

```sh
jq -r '.head.vars as $fields | ($fields | @csv), (.results.bindings[] | [.[$fields[]].value] | @csv)' sparql.json > sparql.csv
```

## Export results to Palladio

The popular data exploration platform [Palladio] can directly load data from a
SPARQL endpoint. On the "Create a new project" screen, a link at the bottom to
"Load data from a SPARQL endpoint (beta)" will provide you a field to enter the
endpoint address, and a box for the query itself. Depending on the endpoint, you
may need to specify the file output type in the endpoint address; for example,
to load data from the BM endpoint you must use the address
`http://collection.britishmuseum.org/sparql.json`. Try pasting in the
aggregation query we used above to count artworks by type and clicking on "Run
query". Palladio should display a preview table.

[Palladio]: http://palladio.designhumanities.org/

{% include figure.html src="/images/sparql10.png" caption="Palladio's SPARQL query interface." %}

After previewing the data returned by the endpoint, click on the "Load data"
button at the bottom of the screen to begin manipulating it.

## Further reading

In this tutorial we got a look at the structure of LOD as well as a real-life
example of how to write SPARQL queries for the British Museum's database. You
also learned how to use aggregation commands in SPARQL to group, count, and sort
results rather than simply list them.

There are even more ways to modify these queries, such as introducing `OR`
statements, full-text searching, or doing other mathematical operations more
complex than counting. For a more complete rundown of the commands available in
SPARQL, see these links:

- [How to SPARQL](http://rdf.myexperiment.org/howtosparql?)
- [Wikibooks SPARQL tutorial](http://en.wikibooks.org/wiki/XQuery/SPARQL_Tutorial)