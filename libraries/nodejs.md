# NodeJS server API client

NodeJS client located [on Github](https://github.com/centrifugal/jscent).

Basic example:

```javascript
Client = require("jscent");

var c = new Client({url: "http://localhost:8000", secret: "secret"});

c.publish("$public:chat", {"input": "test"}, function(err, resp){console.log(err, resp)});
```
