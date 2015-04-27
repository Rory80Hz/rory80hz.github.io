---
layout: post
title: Architect
excerpt: "What the hell is an architect anyway?"
modified: 2013-03-01
tags: [software architecture, architect, noscope]
comments: true
image:
  feature: lord-architect.jpg
---

Below is a copy and paste job from a project I worked on where we wanted to get across some guiding principles for everyone. The format is based on the Open Design Proposals idea from Austin Bingham, that I got us to adopt on the project. More on that some other time.

# 0015 - API Design Principles

###### Author(s) - Rory Hanratty, Will Hamill

``` 
Status: In Progress
```

## Introduction
Whilst being the awesome l33t programmers that we are, and clearly guidelines are for chumps, sometimes, just sometimes it make sense to have some principles to which we adhere, because even though WS* turned into a monster, it did get some things right, in that it at least had an opinion about how things should work.

This is not a suggestion that we go full tilt for auto-generated clients from exposed service descriptions returned by resources, with mad type safe shenanigans, rather that we adopt some set of principles around how we build and use our APIs. What follows is a pragmatic guide in 8 parts to developing REST(ish) endpoints on this project.

* Part 1. [Pragmatic REST](#PragmaticREST)
* Part 2. [Building](#Building)
* Part 3. [Testing](#Testing)
* Part 4. [Healthchecks](#Healthchecks)
* Part 5. [Dogfooding](#Dogfooding)
* Part 6. [Deploy all the things](#WalkingSkeleton)
* Part 7. [Orchestration](#Orchestration)
* Part 8. [References to other peoples ideas](#References)

## Part 1. Pragmatic REST
######  <a name="PragmaticREST">There is no one true way, but there are definitely wrong ways.</a>

These guidelines aim to support a truly RESTful API. They are subject to refinement as we identify which paradigms are most suitable and feasible for our situation.

### General guidelines for RESTful URLs
* A URL identifies a resource. (A resource is a representation of some part of a business or problem domain)
* URLs should include nouns, not verbs.
* Use plural nouns only for consistency (no singular nouns).
* Use HTTP verbs (GET, POST, PUT, DELETE) to operate on the collections and elements.
* You shouldn’t need to go deeper than resource/identifier/resource.
* Put the version number at the base of your URL, for example http://example.com/1.0/path/to/resource.
* URL v. header:
  * If it changes the logic you write to handle the response, put it in the URL.
  * If it doesn’t change the logic for each response, like OAuth info, put it in the header.
* Specify optional fields in a comma separated list.
* Formats should be in the form of api/v2/resource/{id}.json
* For resources consisting of multiple words use this form api/accounts/mortgage-invoices/1234 with hyphens and all lowercase rather than camelCase

### Good URL examples
* List of magazines:
GET http://www.example.org/api/1.0/magazines.json
* Filtering is a query:
GET http://www.example.org/api/1.0/magazines.json?year=2011&sort=desc
GET http://www.example.org/api/1.0/magazines.json?topic=economy&year=2011
* A single magazine:
GET http://www.example.org/api/1.0/magazines/1234
* All articles in (or belonging to) this magazine:
GET http://www.example.org/api/1.0/magazines/1234/articles
* All articles in this magazine in XML format:
GET http://example.org/api/1.0/magazines/1234/articles
* Specify optional fields in a comma separated list:
GET http://www.example.org/api/1.0/magazines/1234?fields=title,subtitle,date
* Add a new article to a particular magazine:
POST http://example.org/api/1.0/magazines/1234/articles

### Bad URL examples
* Non-plural noun:
http://www.example.org/api/magazine
http://www.example.org/api/magazine/1234
http://www.example.org/api/publisher/magazine/1234
* Verb in URL:
http://www.example.org/api/magazine/1234/create
* Filter outside of query string
http://www.example.org/api/magazines/2011/desc

### Content types
Where supported, content which can be returned in multiple types should be negotiated using the 'Accept' HTTP header, e.g.

    GET http://example.org/api/magazine/1234
    Accept: text/json

Should return a JSON representation of the resource.

    GET http://example.org/api/magazine/1234
    Accept: application/xml

Should return an XML representation of the resource.

### HTTP Verbs
HTTP verbs, or methods, should be used in compliance with their definitions under the HTTP/1.1 standard. The action taken on the representation will be contextual to the media type being worked on and its current state. Here's an example of how HTTP verbs map to create, read, update, delete operations in a particular context:

* GET: Obtains either a list of items of a single item and never changes data.
	* This should always be a safe operation, that is, a GET does not affect the data it acts on.	
* POST: Creates an object: inputs should be in the request body, not the url.  The response body should then include the created object.
* PUT: Updates data
* DELETE: Deletes data

### Response
* No values in keys
* No internal-specific names (e.g. "node" and "UserAuthBaseImpl") - values returned should meaningfully describe the objects involved
* Metadata should only contain direct properties of the response set, not properties of the members of the response set

#### Good examples
No values in keys: 

```json
	"Organisations": [
		{
			"id": "125",
			"name": "Farms Ltd"
		}, {
			"id": "834",
			"name": "Pigs and Chickens"
		}
	]
```

#### Bad examples
Values in keys:

```json
	"tags": [
		{"125": "Farms Ltd"}, 
		{"834": "Pigs and Chickens"}
	]
```

### HTTP Status Codes
Generally, HTTP status codes can be used to infer the result of the operation using this guide: 2XX = OK, 3XX = moved, 4XX = client error, 5XX = server error.

When defining an API endpoint, be clear in the documentation supplied as to which status codes your resource can return and in which situations. Bear in mind that many clients' default behaviour is to treat any non-200 status code as an error, so while 201 and 204 may be relevant for object creation and deletion, ensure this is clearly annotated for third party clients.

Look at [http://httpstatus.es](http://httpstatus.es) for succinct descriptions of the various HTTP status codes. 

It is important that you use the correct classification of response. Remember if a dependency fails, let you consumer know that was what happened, don't just return blank 500 responses for all faults.

### Error Handling
Error responses should include the relevant HTTP status code, and a meaningful message in the case of an exception. Avoid exposing implementation details or other data in the error message.

For example, if a request is sent to a POST endpoint that does not have the correct JSON format, an appropriate response should be like:

    Status: 400 (BAD_REQUEST)
    Response Body:
    {
      error: "unable to parse request body as JSON"
    }

Try and use the most relevant status code rather than just returning 400 for all requests that failed. For example, a request that fails syntactic validation (can't parse as JSON) should return a 400 BAD_REQUEST but a request that fails semantic validation (future date of birth supplied) should return a 422 UNPROCESSABLE_ENTITY.

### Versions
Major changes and product releases should coincide with a major version revision, and minor changes and updates should coincide with a minor version revision.
API calls made without the version number included will default to accessing the latest version of the API
  e.g. if version 2.0 is the latest API version:
  http://example.org/api/resource/123 will access resource/123 on version 2.0 of the API
  http://example.org/api/1.1/resource/123 will access resource/123 on version 1.1 of the API
Activity monitoring should be used to determine when to decommission an API. When an old version of the API has not been used at all for a significant period of time it should be removed.
Note: for now all APIs will be versioned at 1.0 only

### Record limits
If no limit is specified, return results with a default limit (this should be set to 20 and defined on the API documentation).
To get records 51 through 75 do this:
http://example.org/api/magazines?limit=25&offset=50
offset=50 means, ‘skip the first 50 records’
limit=25 means, ‘return a maximum of 25 records’
Ensure that paging/limiting parameters are consistently named across the API, e.g. "limit" vs "pageSize", "offset" vs "startAt".
Information about record limits and total available count should also be included in the response. Example:

```json
{
    "metadata": {
        "resultset": {
            "count": 227,
            "offset": 25,
            "limit": 25
        }
    },
    "results": [ ]
}
```

### Request & Response Example
#### Request
	GET https://example.org/api/person/20/summary

#### Response
```json
{
	"_data": {
		"lastName": "Dreyfus",
		"firstName": "Shosanna",
		"dateOfBirth": "10011990",
		"nationalInsuranceNumber": "AA123458C",
		"acceptedTerms": true,
		"confirmed": true,
		"personalIdentifiers": [{
			"personalIdentifier": "200000002",
			"personId": 20
		}, {
			"personalIdentifier": "200000000",
			"personId": 20
		}, {
			"personalIdentifier": "200000001",
			"personId": 20
		}],
		"title": "Mrs",
		"customerReference": "1000000206",
		"internalUser": false,
		"userId": 20,
		"middleName": null,
		"lastModified": null,
		"digitalContacts": [{
			"id": 14264,
			"partyId": 20,
			"digitalContactType": "Email",
			"digitalAddress": "selenium-test-active-2@test.kainos.com",
			"lastModified": null
		}],
		"phoneContacts": [{
			"id": 21813,
			"partyId": 20,
			"phoneContactTypeId": 1,
			"stdCode": "744",
			"internationalDiallingCode": null,
			"lastModified": null,
			"number": "7123456"
		}, {
			"id": 21814,
			"partyId": 20,
			"phoneContactTypeId": 0,
			"stdCode": "244",
			"internationalDiallingCode": null,
			"lastModified": null,
			"number": "7427436"
		}],
		"preferredContactType": {
			"id": 0,
			"type": "Not Specified"
		},
		"addresses": [{
			"id": 14283,
			"partyId": 20,
			"address1": "Willow Farm",
			"address2": "Green Lane",
			"address3": null,
			"address4": null,
			"city": "Wilmslow",
			"country": "United Kingdom",
			"county": "Cheshire",
			"postalCode": "SK9 1LD",
			"lastModified": null,
			"primaryAddress": true
		}],
		"partyTypeId": 0,
		"id": 20
	}
}
```

## Part 2. Building APIs
###### <a name="Building">Pragmatism in practice aka µ-services</a>

### Keep it well weapon you banana

We all know (at least we should) the SOLID principles. Lets take the first one, the single responsibility principle. This applies in abundance when deciding the unit of construction for your service. Have it do one thing, and do it well. We all love our solutions to be as clever as us, but this isn't always the right way to build things, start stupid (like Rory) then make it smart when the solution dictates it needs to be.

There is a fashion now to confuse new frameworks with solid (as in good, although the acronym also applies) foundations when building a service.

Again, to reiterate the point, before you reach for the massive framework, because clearly you need to buy a whole pig farm to get some bacon, don't. The less moving parts you have the better. People are really good at understanding lots of simple things, not so much one massively complex thing. SCIENCE FACT.

Also, just an aside, if you are not disciplined enough to build a composed monolith you really shouldn't start down the path of building micro-service style architectures.

### Dropwizard
For good or ill, this project has Java at the heart of it's codebase. For that reason, we have decided to use [Dropwizard](http://dropwizard.io/) to build our services. This is because it has basically, well, let Dropwizard tell you...

> Dropwizard pulls together stable, mature libraries from the Java ecosystem into a simple, light-weight package that lets you focus on getting things done.
> 
> &nbsp; 
> 
> Dropwizard has out-of-the-box support for sophisticated configuration, application metrics, logging, operational tools, and much more, allowing you and your team to ship a production-quality web service in the shortest time possible

So it is a collection of things that are good at their job, and it gives you sweet sweet metrics, which amongst other things allow you to do some fairly decent monitoring of your running services, as well as give you ideas on how your platform is being used, aka data driven decision making for Product Owners.

Make sure you use the latest version of things, and use Dropwizard modules to do stuff, not spring.

### APIs and Clients
In the context of building our services, APIs are packages that contain common request and response objects that can be leveraged by both the Resource implementations, and the reference clients. 

As mentioned we are all about java based things so this makes sense for us, as we get the benefits of having the people that build the service, provide a nice client for others to use, that has also been used for testing the service. Such collaborate, so dog food, much amaze.

Adopting this pattern across all our services means that we also get a bit of serendipitous understanding, in that someone that builds one service, should be able to expect the same pattern to be applied for all other services they encounter.

Beats having to figure out why in the name of Odin's beard we are varying between returning DAOs, Decorators, DTOs or whatever other pattern we shoehorned in there as a display of our clearly buff programming skills.

### Naming Conventions
Ah naming, personally I would have preferred if we adopted awesome codenames for our services like "Raptor",  "Trebuchet", "Immolation" or 
"Buddhist Monk" people remember that mad stuff, and ultimately have to ask the people that built it what it does, which promotes discussion, but sadly we didn't.

Instead, use sensible descriptive names for a service, don't be generic, because it hides intent. Again, if you are doing one job, make it clear what that job is.

Also, lets stop calling everything capd-whatever, because it keeps autocorrecting to cape- and also, the name doesn't make any sense anymore. _This one is project specific but the idea holds, dont use shitty prefixes__

### Documentation
Nothing says "this is what is going on" like a good set of tests, and the code itself, but sometimes other pesky people want to know what a service does. 

Also, advertising what it is you do to the outside world is not so terrible. Again, it raises the spectre of WS* type madness, but we don't need to go that far.

To that end, all resources should be documented using a set of annotations defined in the (INSERT LINK HERE) package. More info on this (AT SOME PLACE).

## Part 3. Testing

######  <a name="Testing">Because assuming you are awesome doesn't always work</a>

Testing is a great idea, because, nobody wants their hero developer status being tarnished by something as petty as a massive production outage. 

Also, if you go full TDD it can help drive your thinking towards building a piece of software that is pretty much a series of orchestrated function calls, rather than a nest of logic branches. This will drive out reusable components, and simplify your code greatly.


## Part 4. Health checks
######  <a name="Healthcheck">Sometimes a burning wreck is not the best way to find out its broken</a>

So we use Dropwizard. It provides a very nice way to perform health checks. These checks should basically test any external resources it uses (think DB connections, other services, file based IO).

Make sure you consider these as part of building the service, and figure out with your friendly Ops person how to include this in our monitoring platform.

Whats the best way to find out your service is working? 

PRO-TIP: The answer is not for a customer to ring up the help-desk to say they can't do anything.

Set up a service that allows a monitoring platform to ask it how it is getting on, and allowing that monitoring platform, via automated alerts, to tell your support team things aren't so cool is better, simple right?

## Part 5. Dogfooding
###### <a name="Dogfooding">Eat your own dog food aka build it once</a>
##### Also known as 'drinking your own champagne'

So, lets say for example, we have built a service that lets us deal with customers and their delicious data. 

We want to build another service that loads in some data, then creates a customer.

Do we:

A) Re-use our freakin sweet mad smart ORM based DAOs in the new service 'cause we are code-reusing ninjas

or

B) Call the service that deals with Customers.

The answer is B. If you chose A, your adventure is over, your original service ran a schema update and broke the new service on the next deployment to production, because you, heroic programmer, do all your unit testing on Prod. You have been consumed by the distributed ball of mud, go back to page 1 and start again.

Eating our own dog food encourages us think a bit more about service boundaries, and responsibilities. We become consumers of our own services, which will make us all a bit more responsible when it comes to designing and building.

It also means that we can have one service do one well understood job well. It owns its own schema, and it's own boundaries, it makes it less fragile, and more open to change. In a word agile.

## Part 6. Deploy all the things
###### <a name="WalkingSkeleton">The amazing walking skeleton</a>

Why wait until the end of development to see if this awesome new thing actually works when you put it on a machine that doesn't have the amazing resilience of your own personal ironclad blessed by the Gods machine.

Seriously, don't. Build a walking skeleton, get all the bits of your service in place, and deploy them straight away, you can usefully deploy something with all the health checks you need, as well as mocked data to allow your consumers to do some super basic integration testing.

This avoids last minute plumbing terror, and tests all the more difficult things right away. Well it at least tests all the bits you would usually blame on everyone else if it goes wrong.

In fact go a bit further, include a basic terrible version of your UI that invokes the service too if it is a nice vertical slice of functionality.

* [Alistair Cockburn - A Walking Skeleton](http://alistair.cockburn.us/Walking+skeleton)
* [Gojko Adzic - Skelton on Crutches](http://gojko.net/2014/06/09/forget-the-walking-skeleton-put-it-on-crutches/)

## Part 7. Orchestration
###### <a name="Orchestration">Pulling the strings</a>

Again, this is particular to this project, we are in the happy position of building the customer facing components.

This also has implications with how other components delivered as part of the overall platform communicate with us. We are the masters of customer information, and as such orchestrate other services to bend them to our will.

Avoid 2 way dependencies where possible, that means things like utterly terrible views in the database, or us exposing services for other "Off The Shelf" products to call.

It's the classic don't call us we'll call you thing...except not a job interview, more of a...eh....well this analogy held up well.

You get the idea. We accept data from the Customer, everywhere else we ask for it and act on the response.

## Part 8. References to other peoples ideas
###### <a name="References">There are no new ideas</a>

The above is in no small part derived some some other peoples excellent ideas.

* [Gov.UK - Using and creating Application Programming Interfaces](https://www.gov.uk/service-manual/making-software/apis.html)
* [APIGee API Facade Pattern](/attachments/api-facade-pattern-ebook-2012-06.pdf)
* [APIGEE Creating APIs developers will love](/attachements/api-design-ebook-2012-03.pdf)
* [Alistair Cockburn - A Walking Skeleton](http://alistair.cockburn.us/Walking+skeleton)
* [Gojko Adzic - Skelton on Crutches](http://gojko.net/2014/06/09/forget-the-walking-skeleton-put-it-on-crutches/)
* [James Hughes - Micro Service Architecture](http://yobriefca.se/blog/2013/04/28/micro-service-architecture/)
* [James Lewis, Martin Fowler - Micro Services](http://martinfowler.com/articles/microservices.html)