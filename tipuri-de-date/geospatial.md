# Date geospațiale

Ceea ce permite Redis este interogarea datelor geospațiale în funcție de anumite criterii:
- interogare în baze unei anumite distanțe;
- obiective aflate într-o anumită rază;
- raza specifică unui anumit client.

În Redis, pentru fiecare pereche latitudine/longitudine, se calculează un GeoHash, care este un număr întreg de 52 de bit. Standardul GeoHash a fost creat de Gustavo Niemeyer.

Pentru datele geospațiale se pot folosi comenzile de la seturi (intersecții, etc.). Nu utiliza ZINTERSTORE și nici ZUNIONSTORE pentru că folosește scoringul pentru operații. Totuși pot fi folosite în combinație cu `AGGREGATE MIN`.

Pentru a șterge înregistrări, se folosește `ZREM`. Pentru a vedea câți membri sunt în set, se poate folosi `ZRANGE`.

## GEOADD

Comanda permite adăugarea longitudinii, latitudinii și a unui nume într-o cheie. Datele sunt stocate ca un sorted set, fapt care permite interogările folosind comanda `GEOSEARCH`. Scorul este geohash-ul.
Semnătura: `GEOADD key [NX|XX] [CH] longitude latitude member [longitude latitude member ...]`.

Comanda permite adăugarea mai multor puncte pentru un singur apel.

## GEOHASH

Comanda returnează geohashul pentru unul sau mai mulți membri ai setului.
Semnătura: `GEOHASH key member [member ...]`.

## GEOPOS

Comanda returnează longitudinea și latitudinea unuia sau a mai multor membri ai setului.
Semnătura: `GEOPOS key member [member ...]`.

## Resurse

- [Comenzile Redis pentru Geo](https://redis.io/commands#geo)
- [Haversine formula | Wikipedia](https://en.wikipedia.org/wiki/Haversine_formula)
- [Geohash | Wikipedia](https://en.wikipedia.org/wiki/Geohash)
- [EPSG:900913 Google Maps Global Mercator -- Spherical Mercator (unofficial - used in open source projects / OSGEO)](https://epsg.io/900913)
- [EPSG:3785](https://spatialreference.org/ref/epsg/popular-visualisation-crs-mercator/)
