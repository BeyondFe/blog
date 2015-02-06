Here are some tiny backend node modules I like to glue together to build
webapps.

Check out the repo on github:
[substack-flavored-webapp](https://github.com/substack/substack-flavored-webapp).

``` js
var alloc = require('tcp-bind');
var minimist = require('minimist');
var argv = minimist(process.argv.slice(2), {
    alias: { p: 'port', u: 'uid', g: 'gid' },
    default: { port: require('is-root')() ? 80 : 8000 }
});
```
