# Hashes

Sunt colecții de chei-valori. Seamănă cu un obiect JSON. Hash-urile nu sunt create în baza unei scheme și nici nu au o ordine internă.

Hash-urile sunt mutabile. Poți adăuga, șterge, modifica, incrementa, etc. oricând valorile lor. Hash-urile sunt pe un singur nivel. Nu au adâncime. Acest lucru înseamnă că nu poți introduce în interior seturi sau liste. Atunci când ștergi cheia hash-ului toate valorile sale sunt șterse.

Pot fi folosite pentru a ține evidența numărului de accesări ale unui endpoint al unui API.

```text
HSET endpoint-nume "/records/{id}" 1000
```

Odată setată o astfel de structură, pentru fiecare acces poți decrementa valoarea. Poți să și expiri hash-ul după o zi, dacă faci un reset.

```text
EXPIRE endpoint-nume 86400
```

O altă aplicație este realizarea unui cache pentru sesiuni de lucru.

## Crearea unui hash - HSET

Pentru a crea un hash, folosești comanda `HSET nume_hash nume_cheie1 valoare_1 nume_cheie2 valoare_2`.

```text
HSET user:12 nume "Adelina Madgearu" vârsta 22 grupa începători
```

Comanda returnează numărul de câmpuri stocat în hash: `(integer) 3`.

## HGET

Pentru a inspecta valoarea unei singure chei a hash-ului, folosești comanda `HGET nume_cheie nume_cheie_din_hash`.

```text
HGET user:12 grupa
"\xc3\xaencep\xc4\x83tori"
```

## HMGET

Folosind comanda, poți aduce multiple valori ale câmpurilor menționate.
Semnătura: `HMGET nume_hash câmp1 câmp2`. Sunt returnate valorile câmpurilor menționate.

Are un time complexity de O(1). Durata de execuție depinde de dimensiunea datelor ca valoare a câmpului.

## HGETALL

Pentru a vedea întreaga înregistrarea, putem folosi comanda `HGETALL nume_cheie`. Time complexity este O(n) ceea ce înseamnă că depinde de numărul câmpurilor din hash. Reține că este o operațiune blocking. Întregul hash va fi transferat prin rețea. Pentru un hash de 100 de câmpuri cu valorile sale, `HGETALL` funcționează bine chiar și într-un sistem de producție.
Semnătură: `HGETALL nume_hash`.

```text
HGETALL user:12
1) "nume"
2) "Adelina Madgearu"
3) "v\xc3\xa2rsta"
4) "22"
5) "grupa"
6) "\xc3\xaencep\xc4\x83tori"
```

## HLEN

Comanda returnează numărul câmpurilor din hash. Comanda returnează `0` dacă nu sunt câmpuri.
Semnătura: `HLEN nume_hash`.

## HEXISTS

Această comandă determină dacă respectivul câmp există în hash. Comanda returnează valoarea `1` dacă respectivul câmp există, iar în caz contrar `0`.
Semnătura: `HEXISTS nume_hash câmp`.

## HSETNX

Comanda setează câmpul doar dacă acesta nu există. Dacă a introdus câmpul, va fi returnată valoarea `1`. Dacă respectivul câmp deja există, valoarea returnată este `0`.
Semnătura: `HSETNX nume_hash câmp valoare`.

## HSCAN

O comandă mai eficientă pentru a aduce valorile unui hash este `HSCAN cheie cursor [MATCH șablon] [COUNT număr_înregistrări]`.

## HKEYS

Comanda `HKEYS nume_hash` aduce numai numele cheilor, iar `HVALS nume_hash` aduce numai valorile.

## Adaugă câmp - HSETNX

Pentru a adăuga un câmp nou, folosești tot `HSET`. Pentru a verifica dacă un anumit câmp există pentru a-l actualiza, folosești comanda `HSETNX`.

```text
HSET user:12 absolvent false
(integer) 1
```

Acum înregistrarea va apărea astfel:

```text
HGETALL user:12
1) "nume"
2) "Adelina Madgearu"
3) "v\xc3\xa2rsta"
4) "22"
5) "grupa"
6) "\xc3\xaencep\xc4\x83tori"
7) "absolvent"
8) "false"
```

## Șterge un câmp - HDEL

Pentru a șterge un câmp dintr-un hash, se va folosi comanda `HDEL nume_hash nume_câmp`.
Semnătura: `HDEL nume_hash câmp [câmpuri...]`

```text
HDEL user:12 absolvent
(integer) 1
```

## Incrementarea valorii unui câmp

În cazul în care valoarea unui câmp al hash-ului este un număr întreg, se va folosi comanda `HINCRBY`. Pentru valorile cu virgulă, se va folosi comanda `HINCRBYFLOAT`.

```text
HINCRBY user:12 v\xc3\xa2rsta 1
(integer) 23
```

## Strategii de lucru

În cazul în care dorești să realizezi legături ale unui hash cu un altul care al putea reprezenta un subdocument al primului, vei proiecta două obiecte separate pe care le vei actualiza prin logica programului. Relațiile, care sunt echivalentul tabelelor de relație, ar putea fi stocate într-un set cu scopul de a găsi obiecte înrudite mai rapid.

În cazul subdocumentelor, am putea recurge la o așa-zisă aplatizare în care cheile în adâncime pot fi adăugată la rădăcină separându-le prin două puncte. Avantaje:

-  actualizări atomice ale valorilor (într-o singură comandă);
-  ștergere la nivel atomic
-  nu sunt necesare tranzacții
-  încapsulare în forma cea mai simplă posibil. Acest lucru înseamnă că atunci când ștergi cheia rădăcină, ștergi totul.

Dezavantaje:

- ștergerea unui subdocument, de fapt înseamnă ștergerea multor câmpuri în funcție de datele pe fiecare nivel de adâncime.

## Resurse

- [Hashes | redis.io](https://redis.io/commands#hash)
