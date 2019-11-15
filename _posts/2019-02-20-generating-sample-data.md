---
layout: post
title: Generating sample data in Java applications
author: joe
author-jobtitle: Demo Savant
---

When writing code most usecases or requirements have a very limited amount of sample and testdata. When writing test cases or creating a technical demo this is a problem, cause to get a real impression about how your application looks you need realistic test data. In most cases after I use _Foo_ and _Bar_ or the infamous _Alice_ and _Bob_ cause I am not the creative. 

If I need more testdata I have found [jfairy](https://github.com/Devskiller/jfairy)

It is available as maven dependency
``` xml
	<groupId>com.devskiller</groupId>
	<artifactId>jfairy</artifactId>
	<version>0.6.0</version>
```

The code to create sample data is static and pretty easy to use
``` java
Fairy fairy = Fairy.create();
Person person = fairy.person();

System.out.println(person.getFirstName()); 
```
More information is available in the [README](https://github.com/Devskiller/jfairy/blob/master/README.md) file.

You can use it to generate random:
* Person
* Company
* IBAN
* Phone Numbers
* Emails
* VAT
* Domains

Jfairy also contains locale support, so you can also create german users by passing a local tag in the `create` method
``` java
Fairy plFairy = Fairy.create(Locale.forLanguageTag("de"));
``` 

{% include twitter.html %}
