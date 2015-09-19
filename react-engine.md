Previously I wrote an article about [rendering react server side with seo](http://schempy.com/2015/02/16/reactjs_flux_and_routing_for_seo/). It was an experiment that worked but needed improvement. Lately I've been using [react-engine](https://github.com/paypal/react-engine) to render react views client and server side. It's a rendering engine for [express](https://github.com/strongloop/express) that uses [react-router](https://github.com/rackt/react-router).

I wanted to use the same react components in my previous example and modify as little as possible.
The demo will display a html select element with a list of categories. Once a category is selected a list of products, within that category, will be displayed. The select element change will fire-off an api call to get the products.

Here's the routing I am using:

```js
var React = require('react');
var Router = require('react-router');
var AppHandler = require('./handlers/AppHandler.jsx');
var ProductHandler = require('./handlers/ProductHandler.jsx');

var Routes = (
	<Router.Route name="app" path="/" handler={AppHandler}>
		<Router.Route name="products" path="/products/:category" handler={ProductHandler} />
	</Router.Route>
);
```

I had to change **AppHandler** to use a new component, **Layout**, which serves as the main html template. 

Here's the **AppHandler**:

```js
var React = require('react');
var Router = require('react-router');
var Layout = require('../components/Layout.jsx');

var AppHandler = React.createClass({
	render: function() {
		return (
			<Layout {...this.props} >
				<Router.RouteHandler {...this.props} />
			</Layout>
		);
	}
});

module.exports = AppHandler;
```

I pass all the properties into **Layout** and also into the **RouteHandler**. The **RouteHandler** is where the active route will be rendered. I use [spread attributes](https://facebook.github.io/react/docs/jsx-spread.html) to pass properties.

Here's the **Layout** component:

```js
var React = require('react');
var CategoryList = require('./CategoryList.jsx');

var Layout = React.createClass({
	render: function() {
		return (
			<html>
				<head>
					<meta charSet='utf-8' />
					<title>{this.props.title}</title>
				</head>
				<body>
					<CategoryList categories={this.props.categories} />
					{this.props.children}
				</body>
				<script src='/bundle.js'></script>
			</html>
		);
	};
});

module.exports = Layout;
```

I'll be displaying the **CategoryList** component in all the views. An important part is the **{this.props.children}** which represents the children of the **Layout** component. In this case it's the **RouteHandler**. 

##server-side setup

1. Require [node-jsx](https://github.com/petehunt/node-jsx) to allow node to require **jsx** files.
3. Create a template engine using [react-engine](https://github.com/paypal/react-engine).
3. Register the template engine for **jsx** files.
4. Set the template/view directory.
5. Set **jsx** as the default view engine.

```js
// Make jsx file requirable by node.
require('node-jsx').install();

var express = require('express');
var app = express();
var path = require('path');

// Create the template engine with react-engine.
var engine = require('react-engine').server.create({
	reactRoutes: path.normalize(path.join(__dirname, '/src/Routes.jsx'))
});

// Registers the template engine for jsx files.
app.engine('.jsx', engine);

// Set the template/view directory.
app.set('views', path.normalize(path.join(__dirname, '/src/handlers')));

// Set jsx as the view engine.
app.set('view engine', 'jsx');

// Set the custom view
app.set('view', require('react-engine/lib/expressView'));
```
Rendering the views on the server is easy. Pass the **url** and any **data** the view requires. Here's an example of rendering the main route:

```js
var express = require('express');
var router = express.Router;

router.get('/', function(req, res) {
	res.render(req.url, {
		title: 'Welcome!'
	});
});
```

##client-side setup

1. Require the [react-router](https://github.com/rackt/react-router) routes.
2. Require all view handler files. For this example I used [require-globify](https://github.com/capaj/require-globify) which allows you to require files with globbing expressions. I use [browserify](https://github.com/substack/node-browserify) which is why this step is necessary.
3. Set boot options.
4. Boot!

```js
var Routes = require('./Routes.jsx');
var Client = require('react-engine/lib/client');

// Include all view handler files.
require('./handlers/*.jsx', { glob: true });

// Boot options
var options = {
	routes: Routes,
	
	// Supply a function that can be called to resolove the file that
	// was rendered.
	viewResolver: function(viewName) {
		return require('./handlers/' + viewName);
	}
};

// Boot when you are ready! The DOMContentLoaded event is fired when
// the initial HTML has been completely loaded without waiting for
// stylesheets, images and subframes to finish loading.
document.addEventListener('DOMContentLoaded', function onLoad() {
	Client.boot(options);
});
```

I created a github repo [react-flux-routing-seo-revisited](https://github.com/schempy/react-flux-routing-seo-revisited) for this example.
