title: Simple Async With RxJS
date: 2015-10-14
---

I had a previous project that involved a lot of asynchronous javascript code. I wasn't entirely pleased with the result but it worked. "I wish things could be a little simpler." I kept saying to myself. The part of the application in question was a piece that grabbed data from a API call returning xml. That data had to be parsed and then inserted into a database. I used [hyperquest](https://github.com/substack/hyperquest) to make the API call, [xml-stream](https://github.com/assistunion/xml-stream) to parse the xml response and [sequelize](https://github.com/sequelize/sequelize) for the database logic. 

### The Problem
[xml-stream](https://github.com/assistunion/xml-stream) emits events while parsing and [sequelize](https://github.com/sequelize/sequelize) uses promises during it's querying. Having that work together and be readable/maintainable was a challenge.

<!-- more -->

### Enter RxJS
A few months after that project I started to notice [RxJS](https://github.com/Reactive-Extensions/RxJS). It seemed like a really good way to program asynchronous code. After reading through the docs and examples, I wanted to take it for a test run on that project. A good reference on what reactive programming Checkout [this article explaining reactive programming](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754) written by Andre Staltz.

I'll use the following xml to demonstrate what a successful and error api result would look like.

Success:
```xml
<?xml version="1.0"?>
<api>
	<row>This is the stuff I want</row>
	<row>More stuff I want</row>
</api>
```
Error:
```xml
<?xml version="1.0"?>
<api>
	<error>Something went wrong</error>
</api>
```

[xml-stream](https://github.com/assistunion/xml-stream) will parse and stream the xml response chunk by chunk firing off events for specific xml nodes. For this example I want to look for row nodes or error nodes. Row nodes will be inserted into the database and error nodes will not.

To use [RxJS](https://github.com/Reactive-Extensions/RxJS) for this I'll create two streams. One stream will make the http request to the api, parse the xml and emit streams for the matching row or error xml nodes. I'll call that apiStream. The other stream takes values from apiStream and will insert into a database (remember only values from the row xml nodes). I'll call that stream databaseStream. Finally I'll subscribe to the databaseStream from where I can handle the success/error/completed events.


```javascript
var hyperquest = require('hyperquest');
var Rx = require('rx');

// Create custom Observable to handle making the api call
// and parsing the xml result. This will push each matching
// xml node, row and error. This will also push when the parsing
// is completed.
var apiStream = Rx.Observable.create(function(observer) {
	var apiUrl = 'https://www.testapi.com';
	
	// Let's stream the API response. This will fire the event
	// updateElement for the mathcing nodes, row and error.
	// This will also fire the end event when the parsing has
	// been completed.
	var xml = new XmlStream(hyperquest(apiUrl));

	// A error xml node was found.
	xml.on('updateElement: error', function (error) {

		// Push the error.
		observer.onError(error);
	});

	// A row xml node was found.
	xml.on('updateElement: row', function (row) {

		// Push the row.
		observer.onNext(row);
	});

	// Call when completed parsing.
	xml.on('end', function() {

		// Push
		observer.onCompleted();
	});
});

// Takes each value from the apiStream and insert
// into a database. Returns an Observable I can subscribe to.
// Once I subscribe I can handle the success, error and
// completed events that have been pushed.
var databaseStream = apiStream
	.flatMap(function (data) {

		// Make a call to the database to insert. It will
		// return a promise.
		var databaseInsert = some db logic here;

		// Return an Observable that I can subscribe to.
		// This creates an Observable from the promise
		// returned from the database insert.
		return Rx.Observable.fromPromise(databaseInsert);
	});

// Subscribe to the database stream. I will handle the
// success, error and completed events.
var subsciption = databaseStream.subscribe(
	function (x) { console.log('onNext: success'); },
	function (e) { console.log('onError %s', e); },
	function () { console.log('onCompleted'); });
);

```

I'm still learing but this was a big improvement from my previous version. For  me it's simpler understanding what's going on. I'm excited to include [RxJS](https://github.com/Reactive-Extensions/RxJS) in future projects!