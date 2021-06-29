# Strings

În Redis, șirurile de caractere sunt secvențe binare de bytes sigure - *binary safe sequences*. Acest lucru permite stocarea a mai multor reprezentări de date:

- numere întregi,
- valori binare,
- comma separated values,
- JSON serializat,
- XML.

Pentru că sunt secvențe binare, pot fi stocate chiar imagini, înregistrări video și audio. Valoarea unui `String` poate fi de 512MB.

Cazul cel mai comun în care sunt folosite este cel al realizării mecanismelor de caching:

- răspunsuri API,
- stocarea sesiunilor,
- pagini HTML,
- etc.

String-urile Redis pot fi incrementate, dacă valoarea *seamănă* cu o cifră.

## Structurarea numelor cheilor

O bază de date Redis este o structură plată de date. Atunci când ai nevoie să introduci o structură privind cine cui aparține ca valoare sau de care subset/colecție o anumită cheie aparține, poți folosi o compunere de tipul `user:id:comments` pentru a modela date în adâncime, unde:

- `user` este numele obiectului,
- `id` este numărul de identificare al respectivului user,
- `comments`, fiind un obiect cu o structură proprie.

Acestea sunt convenții pe care o echipă le poate adopta pentru a păstra consistența și structura unei scheme a datelor. Pentru că și cheile sunt *binary safe*, șirurile de caractere folosite sunt *case sensitive*. Următoarele chei sunt diferite.

```text
User:234:comments
user:215:comments
```

Reține faptul că serverul face o comparație la nivel binar pentru a detecta diferențele.

## Stocare și accesare

Pentru a crea o valoare în Redis, se folosește comanda `set`.
Semnătura: `set <cheie> <valoare>`.

## Crearea cheilor

Un scenariu întâlnit este accesarea unei înregistrări dintr-o bază de date. Pentru a introduce o cheie în Redis, vom folosi comanda `SET`.

```text
SET numeCheie valoarea [EX în secunde] [PX în milisecunde] [NX|XX]
```

Pentru a seta valoarea în cazul ipotetic al unui utilizator, vom folosi următoarea comandă:

```text
SET user:12 ionel
```

Pentru a obține valoarea, vom folosi comanda `GET`.

```text
GET user:12
"ionel"
```

### Verificarea existenței unei chei

În Redis, pentru a introduce o cheie, nu trebuie creată anterior. O simplă comandă `SET` o va crea. Totuși, dacă ai nevoie să verifici preexistența unei chei, vei folosi comanda `EXISTS` pentru a verifica dacă respectiva cheie există deja. Dacă o cheie există, va fi returnată valoarea `1` (`(integer) 1`). Dacă nu există, va fi returnată valoarea `0` (`(integer) 0`).

```text
exists user:145
(integer) 1
```

Totuși a avea două comenzi poate conduce la inconsistențe în procedura de verificare pentru că între un `EXISTS` și un `SET` pot apărea chei sau pot dispărea.

Comanda `SET` are opțiuni de verificare a existenței unei chei prin `[NX|XX]`. Pentru a ne asigura că o cheie nu există, vom introduce comanda `SET` cu `NX` în final.

```text
set user:543 "Viviana Andreescu" NX
OK
```

Dacă altcineva va încerca crearea cheii ulterior, rezultatul comenzii va fi `(nil)`. Opțiunea `XX` permite actualizarea valorii unei chei doar dacă aceasta există deja.

```text
set user:543 "Alexandru Birlic" XX
OK
get user:543
"Alexandru Birlic"
```

## Incrementare și decrementare

Pentru a incrementa și decrementa o valoare, se vor folosi comenzile `INCR` și `INCRBY`. Poți iniția o valoare folosind comanda `EXISTS` urmată de numele cheii.

```text
127.0.0.1:6379> EXISTS ceva:expira
(integer) 0
127.0.0.1:6379> EXISTS ceva:expira
(integer) 0
127.0.0.1:6379> INCR ceva:expira
(integer) 1
127.0.0.1:6379> INCR ceva:expira
(integer) 2
127.0.0.1:6379> GET ceva:expira
"2"
```

Dacă o structură de date nu există, va fi creată în momentul în care scrii în aceasta.

Folosind comanda `INCRBY cheie valoare_incrementare` poți incrementa o valoare cu o alta, fie aceasta pozitivă sau negativă. Pentru a decrementa valoarea unei chei, se poate folosi comanda `DECRBY cheie valoare_decrementare`. Dacă valoarea cheii este un număr cu virgulă mobilă, se va utiliza comanda `INCRBYFLOAT cheie valoare_incrementare`.

## Accesarea cheilor în REDIS

Pentru a accesa cheile, poți folosi comenzile `KEYS` sau `SCAN` pentru a itera toate cheile.

### Comanda KEYS

Va introduce un bloca baza de date pentru a itera toate cheile. În cazul a câtorva milioane, baza de date va fi blocată. Nu folosi niciodată `KEYS` în sisteme de producție. Este o comandă foarte utilă pentru debugging local.

```text
set user:112 ionuț
set user:342 alina
set user:145 lenuța
keys user:1*
1) "user:10:time-zone"
2) "user:112"
3) "user:145"
```

### Comanda SCAN

Face o iterare folosind un cursor și returnează o referință slot. Poate găsi `0` sau mai multe chei.
Semnătura: `SCAN slot [MATCH pattern] [COUNT count]`
Opțiunea `MATCH` este mandatorie, iar `COUNT` precizează câte înregistrări să aducă, fiind opțională.

Această comandă este cea care trebuie utilizată în sistemele de producție.

```text
set user:112 ionuț
set user:342 alina
set user:145 lenuța
scan 0 MATCH user:1* COUNT 1000000
1) "0"
2) 1) "user:10:time-zone"
   2) "user:145"
   3) "user:112"
```

Valoarea lui COUNT să fie foarte mare pentru a extinde căutarea. Trebuie remarcat faptul că această comandă, fără COUNT, va returna prima înregistrare găsită. La `1)` va fi numărul slotului găsit, iar la `2)` este cheia găsită ce potrivește șablonul de căutare. În cazul în care `SCAN` nu mai găsește înregistrări, va returna valoarea zero la `1)`.

## Ștergerea unei chei

Pentru a șterge o cheie se va folosi comanda `DELETE`. Semnătura este `DELETE cheie [cheie...]`. Comanda `DELETE` este una ca blochează baza de date.
Tot pentru a realiza o eliberare a memoriei, se poate folosi și comanda `UNLINK`, având următoarea semnătură: `UNLINK cheie [cheie...]`. Acest mod de a elibera memoria este o procedură asincronă, care nu blochează baza de date.

```text
unlink user:10:time-zone
```

Aplicarea lui `UNLINK` are ca efect returnarea unei valori `(integer) 1` care indică câte chei au fost șterse. În cazul de mai sus, a fost ștearsă una singură. Dacă încerci să ștergi o cheie care nu există, va fi returnată valoarea `null`.

## Resurse

- [Redis Strings Explained | Redis University | youtube.com](https://www.youtube.com/watch?v=n0LQREq4GrY)
- [Data types | redis.io](https://redis.io/topics/data-types)
