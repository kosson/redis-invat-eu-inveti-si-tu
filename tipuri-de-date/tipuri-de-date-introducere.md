# Structuri de date

Redis oferă suport pentru polimorfism, ceea ce înseamnă că o cheie poate avea valori de diferite tipuri în timp.

Pentru a verifica tipul de date al unei chei, se poate folosi comanda `TYPE nume_cheie` care va returna tipul valorii atribuire unei chei. Dacă valoarea este un număr întreg sau o valoare pe care nu o cunoști, poți interoga folosind comanda `OBJECT ENCODING nume_cheie`.

```text
get user:543
"Alexandru Birlic"
object encoding user:543
"embstr"
set user:394:likes 10
object encoding user:394:likes
"int"
```

## Strings

Folosirea șirurilor de caractere este mecanismul cel mai simplu de stocare a datelor. Acestea sunt introduse în Redis ca perechi cheie/valoare. În cazul în care ai reprezentări numerice ca string, Redis poate face automat operațiuni de incrementare sau decrementare.

## Liste

Sunt colecții ordonate de șiruri de caractere. Pot să conțină elemente duplicat și se asemuiesc array-urilor din JavaScript. Se pot face toate operațiunile caracteristice array-rilor.

## Hash-uri

Hash-urile sunt perechi câmp/valoare care sunt alocate unei singure chei Redis. Numele câmpurilor și valorile acestora sunt tot stringuri. Poți face set sau get pe câmpuri. Poți verifica dacă un câmp există. Poți obține doar o listă de chei sau o listă de valori.
Atenție, TTL-ul este aplicat întregii chei, nu doar unui câmp. Este un tip de date simpiar unui Map din JavaScript.

```javascript
const redis = require('redis');
const client = redis.createClient();

client.hmset('user01', 'name', 'bob', 'interest', 'football', redis.print);
client.hmset('user02', 'name', 'edward', 'interest', 'basketbal', redis.print);
client.hget('user01', 'name', redis.print);
client.hget('user02', 'name', redis.print);
client.hgetall('user01', (err, user)=>console.log(user));
```

## Set-uri

Acestea sunt similare set-urilor din JavaScript. În seturi vei avea numai valori unice. Poți adăuga și șterge din set și poți verifica dacă există un element. Poți face diff/union/intersection pe seturi dacă dorești.

```javascript
const redis = require('redis');
const client = redis.createClient();

client.sadd('daysOfWeek', 'Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday');
client.smembers('daysOfWeek', (err, value)=>console.log(value))
```

Pentru a obține membri ai setului, folosești funcția `smembers()`.

## Sorted sets

Este o combinație de hash-uri și map-uri. În loc de perechi field/values, un Sorted Set are perechi score/value. Valorile trebuie să fie unice, dar scores nu.
Această structură de date permite obținerea numărului de elemente, adăugarea și/sau eliminarea lor. Mai poți obține elemente în baza unei plaje de valori determinată de scores.

```javascript
const redis = require('redis');
const client = redis.createClient();

client.zadd('daysOfWeek', 1, 'Sunday', 2, 'Monday', 3, 'Tuesday', 4, 'Wednesday', 5, 'Thursday');
client.zrange('daysOfWeek', 0, -1, (err, value) => console.log(value))
```

## Geolocalizări

Cheile de GeoLocation sunt folositoare pentru stocarea de chei latitude/longitude. Aceste perechi sunt structurate precum Sorted Sets.

## Resurse

- [Data types | redis.io](https://redis.io/topics/data-types)
