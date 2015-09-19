I've been experimenting with Node.js to build a http server for handling POST requests. Particularly for uploading files. I would send a multipart-form/data POST request containing one or more files. The route handler would parse and stream any file to a destination. That destination could be a directory on the server or a storage service like Amazon S3.

I used [routes](https://www.npmjs.com/package/routes) for handling routing and [busboy](https://www.npmjs.com/package/busboy) for parsing HTML form data. Busyboy is a perfect module for this application. It emits a **file** event for incoming files and a **field** event for each non-file field. If a file is found Busboy will emit a **data** event when streaming and an **end** event when the streaming is complete.

 Here's an example of the http server:

```js
var http = require('http');
var router = require('routes')();
var Busboy = require('busboy');
var port = 5000;

// Define our route for uploading files
router.addRoute('/images', function (req, res, params) {
	if (req.method === 'POST') {
	
		// Create an Busyboy instance passing the HTTP Request headers.
		var busboy = new Busboy({ headers: req.headers });
		
		// Listen for event when Busboy finds a file to stream.
		busboy.on('file', function (fieldname, file, filename, encoding, mimetype) {
		
			// We are streaming! Handle chunks
			file.on('data', function (data) {
				// Here we can act on the data chunks streamed.
			});
			
			// Completed streaming the file.
			file.on('end', function () {
				console.log('Finished with ' + fieldname);
			});
		});
		
		// Listen for event when Busboy finds a non-file field.
		busboy.on('field', function (fieldname, val) {
			// Do something with non-file field.
		});
		
		// Listen for event when Busboy is finished parsing the form.
		busboy.on('finish', function () {
			res.statusCode = 200;
			res.end();
		});
		
		// Pipe the HTTP Request into Busboy.
		req.pipe(busboy);
	}
});

var server = http.createServer(function (req, res) {

	// Check if the HTTP Request URL matches on of our routes.
	var match = router.match(req.url);
	
	// We have a match!
	if (match) match.fn(req, res, match.params);
});

server.listen(port, function () {
	console.log('Listening on port ' + port);
});
```


This was a fun experiment. Handling multipart/form-data was pretty easy and the ability to stream file uploads is awesome!

I created a gist for this example [stream-file-uploads-nodejs.js](https://gist.github.com/schempy/0fe3abab3d834c94b3be).


Questions? Comments? Tweet me [@schempy](https://www.twitter.com/schempy) or email [schempysays@gmail.com](mailto:schempysays@gmail.com)