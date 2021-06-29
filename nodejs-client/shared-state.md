# Shared state

```javascript
const cluster = require('cluster');
const http = require('http');
const redis = require('redis').createClient();

if (cluster.isMaster) {
  cluster.fork();
  cluster.fork();
} else {
  http.createServer((req, res) => {
    redis.incr('counter', (error, data) => {
      res.end(JSON.stringify({counter: data, pid: process.pid}));
    });
  }).listen(9999);
}
```

## Resurse

- [Redis and Node Part 2: Shared State](https://thomashunter.name/posts/2017-07-12-redis-and-node-part-2-shared-state)
