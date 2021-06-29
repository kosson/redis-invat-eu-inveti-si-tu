# Seturi

În Redis, un set este o suită de valori unice care nu au index.

## Comanda SADD

Adăugarea unei valori la un set se face folosind comanda `SADD nume_set valoare [valori...]`. Pentru a obține valorile dintr-un set, se va aplica comanda `SMEMBERS nume_set`. Un mecanism mai eficient este `SSCAN nume_set cursor [MATCH șablon] [COUNT câte_elemente]`.

```text
sadd setnou ceva undeva cineva cândva
(integer) 4
sscan setnou 0 match *
1) "0"
2) 1) "ceva"
   2) "c\xc3\xa2ndva"
   3) "cineva"
   4) "undeva"
   5) "gigi"
```

## Comanda MSET

Folosești această comandă pentru a seta mai multe seturi deodată.

```text
MSET set_nou1 "ceva" set_nou2 "altceva"
```

## Comanda SCARD

Pentru a vedea numărul valorilor existente în set, se folosește comanda `SCARD nume_set` (**set cardinality**). Comanda va returna un număr.

```text
scard setnou
(integer) 5
```

## Comanda SISMEMBER

Pentru a verifica dacă o valoare face parte dintr-un set existent, folosești comanda `SISMEMBER nume_set valoare`.

```text
SISMEMBER setnou cândva
(integer) 1
```

În cazul în care se confirmă că valoarea face parte din set, comanda va returna valoarea 1.

## Ștergerea unui element cu SREM

Pentru a șterge un element dintr-un set, se va folosi comanda `SREM nume_set valoare [valori...]`.

```text
SREM setnou cândva
(integer) 1
```

Dacă valoarea a fost ștearsă din set, Redis va returna valoarea 1.

## Ștergerea unui element random

Pentru a șterge un element random dintr-un set, se va folosi comanda `SPOP nume_set [număr_elem_dorite_șterse]`.

## Operațiuni pe seturi

### Diferența între seturi

Comanda `SDIFF` oferă diferență a două seturi. Diferența este constituită din elementele primului set care nu se regăsesc în cel de-al doilea.

### Intersecție

Pentru a verifica intersecția a două seturi, folosim `SINTER nume_set1 nume_set2`. Comanda va returna un subset al valorilor conținute în ambele seturi.

### Uniunea dintre seturi

Comanda `SUNION`.

### Mutare dintr-un set în altul

Comanda `SMOVE set_sursă set_destinație valoare_din_primul` mută un element dintr-un set în altul specificat la destinație.

## Cazuistică

### Evidența utilizatorilor online

Seturile se pretează la ținerea evidenței utilizatorilor online.

Când utilizatorul se loghează, va fi adăugat în set cu `SADD nume_set valoare` - `SADD players:online:100 245`.
Când utilizatorul se deloghează, va fi apelată comanda `SRAM nume_set valoare` pentru a-l scoate din set.

Atunci când conexiunea utilizatorului cu serverul este afectată sau întreruptă, trebuie pregătit un mecanism care să șteargă utilizatorul din set automat după o anumită perioadă. Mecanismul ar implica expirarea unui set întreg la un anumit interval concomitent cu realizarea unuia nou.

- #1 Adaugi userul la o fereastră de timp (set): `SADD players:online:100 245`
- #2 Adaugi același user și la următoarea fereastră de timp (set): `SADD players:online:105 245`
- #3 Adaugi un moment la care expiră fereastra de timp a celor prezenți: `EXPIREAT players:online:100 1588241100`
- #4 Adaugi momentul la care expiră acea de-a doua fereastră de timp: `EXPIREAT players:online:105 1588241400`
- #5 Verifici dacă userul face parte din prima fereastră: `SISMEMBER players:online:100 245`

## Resurse

- [Sets | redis.io/commands](https://redis.io/commands#set)
