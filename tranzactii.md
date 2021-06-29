# Tranzacții

În Redis, toate comenzile care sunt incapsulate într-o tranzacție sunt serializate și executate secvențial. Acest lucru garantează executarea comenzilor într-o singură operațiune izolată. În timpul executării unei tranzacții nicio altă conexiune nu poate face vreo modificare în timpul unei tranzacții.

Dacă sunt comenzi în linie, iar conexiunea este pierdută, Redis nu va executa niciuna.

Redis nu are suport pentru tranzacții nested. Asta înseamnă că din moment ce pornești o tranzație cu `MULTI` nu mai poți emite comanda `MULTI`.

Într-o tranzacție Redis, modificările făcute de o comandă sunt vizibile următoarelor comenzi ale aceleiași tranzacții.

## Pornirea unei tranzacții cu MULTI

Această comandă indică pornirea unei tranzacții.

## EXEC

Este comanda care execută comenzile programate. Dacă tranzacția eșuează, comanda returnează `(nil)`.

## DISCARD

Comanda șterge toate comenzile programate.

## Optimistic concurency control

Ce se petrece atunci când ai pornit o tranzacție, dar între timp, un alt proces sau aplicație a modificat o anumită cheie care deja a fost citită.
Mecanismul *optimistic concurency control* este cel care permite specificarea unui interes legat de un anumit obiect care se concluzionează cu o notificare în cazul în care respectivul obiect a fost modificat.

Keyspace notifications este mecanismul care poate fi utilizat de Redis pentru a satisface această necesitate.

Mecanismul de *optimistic locking* este realizat prin folosirea comenzii `WATCH`.

### WATCH

Folosești această comandă pentru a-ți arăta interesul privind una sau mai multe chei. Astfel, atunci când este apelată comanda `EXEC`, tranzacția va eșua dacă una sau mai multe chei a fost modificată.
Semnătura: `WATCH cheie [chei...]`.

`WATCH` trebuie apelată înainte de începerea tranzacției. Pot fi executate mai multe comenzi `WATCH` înainte de `MULTI`, iar cele ulterior lansate nu le suprascriu pe cele care au început să fie *păzite* deja (sunt cumulative).

Comenzile `WATCH` sunt localizate la nivel de client și conexiune. Nu au o caracteristică globală.

Dacă tranzacția reușește în urma lansării comenzii `EXEC`, atunci toate cheile vor deveni automat *unwatched*.

Nu poți lansa o comandă `WATCH` într-o tranzație.

### UNWATCH

Folosești această comandă pentru a elimina toate cheile care sunt *păzite*.

## Resurse

- [Transactions | redis.io/commands](https://redis.io/commands#transactions)
- [Redis Clients Handling](https://redis.io/topics/clients)
