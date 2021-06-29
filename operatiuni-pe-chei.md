## Operațiuni pe chei

Pentru a seta o cheie în Redis ai comanda `set`.

```javascript
const redis = require('redis');
const client = redis.createClient();
client.set('user', 'admin', redis.print);
client.get('user', redis.print);
// Update our user
client.set('user', 'bob', redis.print);
client.get('user', redis.print);
client.del('user', redis.print);
client.get('user', redis.print);
```

## Incrementare și decrementare

```javascript
client.set('key1', 10, function() {
    client.incr('key1', function(err, reply) {
        console.log(reply); // 11
    });
});
```

Funcția `incr` este folosită pentru a crește o valoare cu o unitate. Opusul este realizat folosind funcția `decr()`. Pentru a crește valoarea cu o valoare specificată, vei folosi funcția `incrby()`, iar opusul `decrby()`.

Ștergi sau expiri durata de viață a cheilor.

```javascript
// șterge o cheie
client.del('frameworks', function(err, reply) {
    console.log(reply);
});
// poți să expiri o cheie
client.set('key1', 'val1');
client.expire('key1', 30);
```

Poți verifica existența unei chei.

```javascript
client.exists('key', function(err, reply) {
    if (reply === 1) {
        console.log('exists');
    } else {
        console.log('doesn\'t exist');
    }
});
```
