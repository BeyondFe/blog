title: Where To Have API Calls In React Flux
date: 2015-03-02
---

Working with React/Flux the last three months I couldn't decide where to make asynchronous calls. Should they be made in the component, store or action creators? I chose the action creators since dispatching of all actions come from them. A module could abstract the actual asynchronous call and return a promise.  The promise would resolve with the result of the call or be rejected if there was an error.

Here's an example of the module to make the asynchronous call. I used [superagent](https://github.com/visionmedia/superagent) to make the request.

```js
var Api = {
	get: function (url) {
		return new Promise(function (resolve, reject) {
			request
				.get(url)
				.end(function (res) {
					if (res.status === 404) {
						reject();
					} else {
						resolve(JSON.parse(res.text));
					}
			});
		});
	}
}; 
```
 
The action creator would use this module. When the promise is returned dispatch an action containing the result. 

```js
var Api = require('./Api');
var Dispatcher = require('./Dispatcher');
var ActionConstants = require('./ActionConstants');

// Define the ActionCreator.
var ActionCreator = {
	getCategories: function () {
		Api
			.get('/api/categories')
			.then(function (categories) {
			
				// Dispatch an action containing the categories.
				Dispatcher.handleViewAction({
					actionType: ActionConstants.RECEIVE_CATEGORIES,
					categories: categories
				});
			});
	};
};
```

The store would register with the dispatcher and provide a callback to handle the response from the action. It would also emit a change event so components would be notified that values have changed.

```js
var Dispatcher = require('./Dispatcher');
var ActionConstants = require('./ActionConstants');
var EventEmitter = require('events').EventEmitter;
var assign = require('object-assign');

var CHANGE_EVENT = 'change';
var _categories = [];

function setCategories (categories) {
	_categories = categories;
}

// Define the Store.
var Store = assign({}, EventEmitter.prototype, {

	emitChange: function () {
		this.emit(CHANGE_EVENT);
	},
	
	addChangeListener: function (callback) {
		this.on(CHANGE_EVENT, callback);
	},
	
	removeChangeListener: function (callback) {
		this.removeListener(CHANGE_EVENT, callback);
	},
	
	getCategories: function () {
		return _categories;
	}
});

// Store registers with dispatcher to handle actions.
Store.dispatchToken = Dispatcher.register(function (payload) {
	var action = payload.action;
	
	switch (action.actionType) {
		case ActionConstants.RECEIVE_CATEGORIES:
			
			// Callback to handle the response from the action.
			setCategories();
			break;
			
		default:
			return true;
			break;
	}
	
	Store.emitChange();
	
	return true;
});
```

Finally our components would use the store to register change listeners and the action creator for getting our categories!

```js
var Store = require('./Store');
var ActionCreator = require('./ActionCreator');

// Define the Category component.
var Category = React.createClass({

  getInitialState: function () {
    return {
      categories: []
    };
  },

  componentWillMount: function () {
    Store.addChangeListener(this._onChange);
  },

  // Use the ActionCreator to get the categories.
  componentDidMount: function () {
    ActionCreator.getCategories();
  },

  componentWillUnmount: function () {
    Store.removeChangeListener(this._onChange);
  },

  /**
  * Update the state of categories for this component.
  * This will get called when our store handles the response
  * from the action.
  */
  _onChange: function () {
    this.setState({
      categories: Store.getCategories()
    });
  },

  // Display a drop-down containg the categories.
  render: function () {
    var categories;

    if (this.state.categories) {
      categories = this.state.categories.map(function (category) {
        return <option key={ category.id }
            value={ category.name }>
            { category.name }</option>;
      });
    }

    return (
      <div>
        <select name="category">
          <option value="">Select a Category</option>
          { categories }
        </select>
      </div>  
    );
  }
});
```


## final thoughts
The result from an asynchronous call should create an action. This keeps the data flowing through the application the flux way (Actions -> Dispatcher -> Stores -> Views).

One issue having the category component make a Api call once it's mounted is it will be rendered twice. Having components get their data from props is better but sometimes the data has to be from an external service.

Check out the repo on github: [react-flux-api-calls](https://github.com/schempy/react-flux-api-calls)

Questions? Comments? Tweet me [@schempy](https://www.twitter.com/schempy) or email [schempysays@gmail.com](mailto:schempysays@gmail.com)
