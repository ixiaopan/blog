---
title: "Semantic Web"
date: "2021-05-08"
description: "In semester 2, I took a module called Semantic Web Technologies. There are some reasons why I choose this module. At first glance, the name itself sounds not appealing. Actually, it does. But it's still on my shortlist because I've been working on the web for many years and I wondered what the semantic web was. "
# tags: []
categories: [
    "Machine Learning",
]
katex: true
---



In semester 2, I took a module called **Semantic Web Technologies**. There are some reasons why I choose this module. At first glance, the name itself sounds not appealing. Actually, it does. But it's still on my shortlist because I've been working on the web for many years and I wondered what the semantic web was. 


On the other hand, I was not interested in modules like Advanced Databases since I've learnt database during my undergraduate degree in Computer Science. Besides, I hope I could have more free time to do other things, and I don't think I can deal with a more difficult module like Reinforcement Learning though it's indeed useful and interesting. Maybe I could learn it later when needed.



Okay, let's go back to this post. The goal of this post is to go through the most important parts of this module for the final exam preparation.



## Web for machine



We all know that the current web pages are written in HTML. You can use `<p>` to define a paragraph like this.



```html
<p>Hello world</p>
```



HTML looks more like a natural language as it is designed to be readable and understandable for human, not machines. However, we hope machines can understand information as well. But why? Because we want machines to do reasoning about data rather than save data only.



Web pages can be linked using the markup `<a>`, but not all relevant documents are connected. Anyone can publish a new web page anytime. Moreover, not everyone uses the same language to describe the same thing. Thus, the interconnected web pages are simply a limited bundle of documents. More importantly, the data are not connected, and we don't have explicit schemas that enable machines to understand documents.



On the contrary, the goal of Semantic Web is to make data readable and understandable for machines. To achieve this, the first step is to encourage people to share their data. Then things are denoted by their unique identifiers. Thus, data from various contexts can be connected through the identifiers, making a huge net finally (also known as Linked Data). Therefore, in Semantic Web, machines learn from data directly instead of documents that are written for people.



What are the applications of Semantic web? Well, we can utilise the vocabulary of a domain to build a knowledge graph, which encodes the semantics of domain knowledge in this network. Search engines can augment their search results from such knowledge graphs since they provide domain-specific knowledge.



## RDF



RDF is short for the Resource Description Framework. It's a triple-based data model for knowledge representation. Figure 1 shows a triple composed of 3 components, which means Bob has a friend named Alice. 

- `Bob` is the subject

- `Alice` is the object 
- `hasFriend` is the predicate that connects them



![](/blog/post/images/rdf.png "Figure 1: A triple-based data model")



The above model is an abstract framework. Subjects and predicates can be anything. To describe the specific subjects and the relationship between them, we need to find a way to identify them explicitly. We also need a data format that defines and parses this model.



### URI

In the World Wide Web, URI is used to define a unique object. Here are some examples,



```javascript
https://www.example.com
mailto:aaa@xxx.com
urn:isbn:123456
```



Well, URI can also be followed by an optional fragment identifier like this,



```javascript
https://www.example.com/index.html#introduction
```



### Namespace

The above method has one problem. A term may have different meanings in different domains. If we include them all in a file, it will cause name conflicts. We can solve it using namespace. Specifically, a resource can be represented by a namespace, a colon and a local name. 

```html
<http://example.org/ontology#hasFriend>

// is equivalent to

@prefix d: <http://example.org/ontology#> .
d:hasFriend 
```



### Turtle

The last thing is to define a data structure so that machines can parse and serialize. One of the popular data  formats is Turtle. The following are some syntaxes defined by Turtle,

- Resource URI are written in angle brackets
- Literal values  are written in double quotes
- Triples end with a full stop



```html
<http://example.org/data#Bob> <http://example.org/ontology#hasFriend> <http://example.org/data#Alice> .

```



### RDF/XML

Apart from Turtle, we can also express RDF using RDF/XML, which is similar to XML. But it's not easy for humans to read compared to Turtle.



```HTML
<?xml version="1.0" encoding="utf-8" ?>
<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
         xmlns:ns0="http://example.org/ontology#">

  <rdf:Description rdf:about="http://example.org/data#Bob">
    <ns0:hasFriend rdf:resource="http://example.org/data#Alice"/>
  </rdf:Description>

</rdf:RDF>
```



### Tools

The following are some useful tools for understanding RDF.



- [EasyRDF Converter](https://www.easyrdf.org/converter)
  - Tool for converting between different RDF syntaxes 
- [W3C RDF Validator](http://www.w3.org/RDF/Validator/)
  - Validator for RDF/XML, which can also visualise RDF as a graph



Generally, we are required to know how to read them and write some simple statements in this module.



## RDFS



RDF only tells us some specific instances. For example, Bob knows Alice, Bob works for Google, etc. However, they don't define the vocabulary used in those triples. From the perspective of object-oriented programming, we want to define the classes that generate these instances. Strictly speaking, we want to define the ontology of a domain. Moreover, if we define some rules in this domain, say  `Person worksFor Company`, then we can infer that Bob is a person and Google is a company under the defined rule. 

RDFS and OWL are two common ontology languages to do this. We will talk RDFS first in this section. RDFS allows us to define classes and properties as well as the characteristics of classes and properties.



### class definition

In OOP, we use keyword `class` to declare a class, for example, 



```javascript
class Person {}
```



In RDFS, we use predefined classes, such as `rdfs:Class`, `rdfs:Property`, to declare the corresponding object. 

```javascript
ex:Person rdf:type rdfs:Class .
```



![](/blog/post/images/rdfs-class.png "Figure 2: Declare a class")



Furthermore, we use `rdfs:subClassOf` to declare that a class is a subclass of another class. It's clear that `rdfs:subClassOf` is transitive. For example, if  `Teacher` is a subclass of `UniStaff` and `UniStaff` is a subclass of `Person` , then `Teacher` is a subclass of `Person`, as shown in Figure 3.



```javascript
ex:Teacher rdf:type rdfs:Class ;
		   rdf:subClassOf ex:UniStaff .

ex:UniStaff rdf:type rdfs:Class ;
		   rdf:subClassOf ex:Person .
```



![](/blog/post/images/rdfs-superclass.png "Figure 3: RDFS class semantics ")



On the other hand, `rdf:type` distributes over `rdf:subClassOf`. This means that if  `C rdf:type A` and `A rdfs:subClassOf B` , then `C rdf:type B`. For instance, Figure 4 illustrates that  if Bob is an instance of `Teacher`, then Bob is also an instance of `Unistaff` and `Person`.



![](/blog/post/images/rdfs-type.png "Figure 4: rdf:type distributes over rdfs:subClassOf")



### property definition

Property declaration is similar to class declaration. The following code declares a property named `ex:hasStreet`. 



```javascript
ex:hasStreet rdf:type rdfs:Property .
```



But not every object has an addresss. Besides, we sometimes want to limit the domain and range of a property. In this case, we would say, only people can have an address.



```javascript
ex:hasStreet rdf:type rdfs:Property ;
             rdf:domain ex:Person .
```



We also notice that `ex:hasStreet` is a more specific attribute of `ex:hasAddress` since addresses are usually composed of several components like towns and streets. So we define `ex:hasStreet` as a subproperty of `ex:hasAddress`.



```javascript
ex:hasStreet rdf:type rdfs:Property ;
             rdf:domain ex:Person ;
             rdf:subPropertyOf ex:hasAddress .
```



Figure 5 illustrates that if Bob lives in a street named West Road, it implies that West Road is Bob's address. Furthermore, we have already stated that only **Person** can have street names, which means Bob must be a  person under this semantics. 



![](/blog/post/images/rdfs-superpropery.png "Figure 5: RDF Schema property semantics ")



## SPARQL



So far, we are able to represent data. But how do we retrieve it from databases? Well, SPARQL is a SQL-like query language that allows us to make a query. 



### basic syntax



```SQL
SELECT <variable>
FROM <graph>
WHERE { <triple patterns> }
```



- variables are prefixed with `?`
- triple patterns are expressed in Turtle



For example, the following codes return the names of the people who have street addresses.



```sql
SELECT ?name
WHERE {
	?name ex:hasStreet ?street .
}
```



### FILTER

We can also query the names of the people who live on the streets only containing 'Baker'.



```sql
SELECT ?name
WHERE {
	?person ex:hasStreet ?street .
	FILTER regex(?street, "Baker") 
	?person ex:hasName ?name .
}
```



Another example is to filter all the items whose prices are lower than 5 pounds.



```sql
SELECT ?name
WHERE {
	?item ex:hasPrice ?price .
	FILTER (?price >= 5)
	?item ex:hasName ?name .
}
```



### UNION

`UNION` allows us to either match this condition or that one, which is equivalent to the operation of `OR`



```sql
SELECT ?name
WHERE {
	{ ?person ex:hasStreet "Baker Street" . }
	UNION
	{ ?person ex:hasStreet "West Road" . }

	?person ex:hasName ?name .
}
```



We won't dive into SPARQL deeper because SPARQL has quite similar syntaxes to SQL. If you are familiar with SQL, you'll know how to use SPARQL instantly.



## Description Logic (DL)



TODO



