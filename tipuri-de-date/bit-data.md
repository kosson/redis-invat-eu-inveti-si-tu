# Bit data

Redis oferă suport pentru bit fields și bit array-uri. Acestea sunt structuri compacte și optimizate care seamănă cu niște hărții de biți. Pentru un exemplu ai putea să te gândești la biții care setează permisiunile în Linux.

Nu există un tip de date bit (*bitmaps*), ci un set de comenzi de manipulare a biților care se aplică pe tipul string. Adu-ți mereu aminte că un string în Redis este o valoare binary safe blob, ceea ce permite, de fapt înmagazinarea a 2<sup>32</sup> biți. Pentru a verifica, după ce ai setat o valoare, introdu comanda `type nume_cheie`. Faptul că șirurile de caractere sunt binary safe, permite ca prin mutarea unui bit într-un string să poți aplica operațiuni pe biți.

Dacă vei avea curiozitatea să cauți care este codarea obiectelor setate ca bit data lansând comanda `object encoding nume_cheie`, rezultatul va fi valoarea `raw`.

Aplicativitate:

- histograme
- înregistrarea de vârfuri în citirea unor senzori
- setari biților permisiunilor
- crearea de măști.

## Bit arrays

Bit arrays permit manipularea de biți individuali dintr-un tip de date string. Biții sunt adresați folosind un offset care pornește de la 0. Poți să te gândești la un array de biți ca la un șir de caractere care pornește de la stânga spre dreapta, care are bitul ce indică semnul la indexul 0. Numărul de biți este limitat la dimensiunea cea mai mare pe care o poate avea un șir.
Pentru a stoca și utiliza un bit anume, se pot folosi comenzile:
- `GETBIT key offset` și
- `SETBIT key offset value`.

`BITFIELD` poate fi considerată a fi un superset al acestor comenzi, iar în cazul în care o folosești specifi tipul `u1`.

### Comanda SETBIT

Folosind această comandă poți seta sau elimina valoarea bit-ului de la o anumită poziție (*offset*) care este stocat la o anumită cheie. Valoarea poate fi `0` sau `1`. Dacă respectiva cheie nu există, aceasta va fi creată. Poziția (*offset-ul*) trebuie să fie mai mare de 0 și mai mic de 2<sup>32</sup>. Deci, limita este la 512MB.
Semnătura: `SETBIT key offset value`.

```text
SETBIT cheie2 7 1
(integer) 0
SETBIT cheie2 7 0
(integer) 1
GET cheie2
"\x00"
```

În cazul în care valoarea este pusă la un anumit offset, șirul de caractere este creat pentru a permite acest lucru.

Sunt cazuri în care ai nevoie să setezi toți biții unui bitmap deodată. Acest lucru este posibil prin mai multe comenzi `SETBIT`, dar poți face acest lucru dintr-o singură comandă. Pentru că operațiunile pe biți le faci folosind comenzi pe șiruri de caractere, poți seta bitmap-uri utilizând comenzile `GET` și `SET` ale string-urilor. Astfel, un bitmap este stocat ca un byte stream (fragemnte de 8 biți). Primul byte al șirului corespunde unui offset de la 0 la 7, al doilea de la 8, la 15 ș.a.m.d.

Pentru a seta un singur bit, poți folosi și `BITFIELD` cu mențiunea ca setarea câmpului să se facă la tipul `u1`.

```text
BITFIELD bitsinabitmap SET u1 2 1 SET u1 3 1 SET u1 5 1 SET u1 10 1 SET u1 11 1 SET u1 14 1
```

## Comanda BITFIELD

Această comandă oferă posibilitatea de a obține, seta sau incrementa valoarea unui câmp de biți. Comanda tratează un șir ca fiind un array de biți. Comanda poate opera cu mai multe câmpuri de biți deodată care, potențial au un număr diferit de biți. Acest tip de câmp este unul pozițional, fiecare bit fiind identificat prin index.
Semnătura: `BITFIELD key [GET type offset] [SET type offset value] [INCRBY type offset increment] [OVERFLOW WRAP|SAT|FAIL]`.

Comanda permite specificare dimensiunii datelor pe care dorești să operezi și mai mult, poți specifica dacă valoarea este `signed` ori `unsigned`.
Alte opțiuni permit tratarea datelor care sunt mai mari decât cele care pot fi scrise - *overflowing* sau *wrapping*.

GET, SET și INCRBY necesită specificarea tipului de date cu care se lucrează. Dacă valoarea este signed, se va marca cu `i`, dacă este unsigned, se va marca cu `u`. Următoarea mențiune este numărul de biți. Poți specifica 64 de biți în cazul signed (`i64`) și 63 de biți în cazul unsigned (`u64`).

```text
bitfield cheie_test set u8 0 42
```

În exemplul de mai sus, am setat o valoare de 8 biți fără semn la offset-ul 0 și am introdus drept valoarea numărul întreg `42`. În cazul în care introduci valoarea unui bit la un offset diferit de 0, toate pozițiile până la offset vor fi completate automat cu 0. Pentru că `BITFIELD` este un superset al comenzilor `GETBIT` și `SETBIT`, poți lucra cu array-uri de biți câtă vreme menționezi tipul `u1`.

```text
bitfield cheie4 set u1 5 1
bitcount cheie4
(integer) 1
```

În cazul în care nu dorești să calculezi singur offset-ul, poți folosi simbolul pound pentru a indica lui Redis să facă acest calcul singur.

```text
bitfield cheie3 set u8 #1 5
```

BITFIELD este variadic ceea ce înseamnă că pot fi specificate multiple GET-uri și SET-uri.

```text
bitfield cheie3 get u8 #1 5 get u8 8
(integer) 5
(integer) 5
```

Poți seta multiplu precum în exemplul de mai jos. Vom seta trei byte separați.

```text
bitfield cheie_bf set u1 7 1 set u1 15 1 set u1 23 1
1) (integer) 0
2) (integer) 0
3) (integer) 0
bitcount cheie_bf
(integer) 3
```

Fii foarte atent căci spre deosebire de alte comenzi pe biți, `BITCOUNT` folosește o numărătoare în byte pentru offset. Folosind exemplul de mai sus, vom avea următorul rezultat.

```text
bitcount cheie_bf 1 2
(integer) 2
```

După cum se observă au fost luați în considerare cel de-al doilea și cel de-al treilea byte. Sunt permise și valori negative pentru offset.

### Incrementarea valorii

```text
bitfield cheie_test incrby u8 0 1
```

În exemplul de mai sus am incrementat valoarea câmpului de biți cu 1.

Motivul pentru care ai folosi această comandă este stocarea numere întregi într-un singur bitmap mare sau care este segmentat pe mai multe chei. Această abilitate permite folosirea Redis-ului în analize în timp real.

## Comanda BITCOUNT

Folosești comanda pentru a număra numărul de biți care au fost setați la 1 dintr-un șir de caractere. Toți byții din șirul de caractere sunt luați în evaluare. Poți preciza și un interval de biți.
Semnătura: `BITCOUNT key [start end]`.

```text
set cheie1 "ceva"
BITCOUNT cheie1
BITCOUNT cheie1 0 0
BITCOUNT cheie1 1 1
```
Poți căuta și de la coadă spre start, menționând valori negative, unde `-1` este primul caracter.

Bitmap-uprile sunt reprezentări foarte eficiente ale diferitelor tipuri de informații. Un exemplu poate fi o aplicație web care are nevoie să țină evidența vizitelor unui utilizator.

## Comanda BITOP

Folosești această comandă pentru a iniția comenzi bitwise pe multiple chei, iar la final stochezi rezultatul într-o cheie nouă.
Semnătura: `BITOP operation destkey key [key ...]`.

Comenzi valide:

- BITOP AND destkey srckey1 srckey2 srckey3 ... srckeyN
- BITOP OR destkey srckey1 srckey2 srckey3 ... srckeyN
- BITOP XOR destkey srckey1 srckey2 srckey3 ... srckeyN
- BITOP NOT destkey srckey

```text
BITFIELD cheiea1 set u1 6 1
cheie1   |0 0 0 0 0 0 1 0|
BITFIELD cheiea2 set u1 7 1
cheie2   |0 0 0 0 0 0 0 1|
BITOP OR rezultatx cheiea1 cheiea2
rezultat |0 0 0 0 0 0 1 1|
BITCOUNT rezultatx
(integer) 2
```

## Comanda BITPOS

Returnează poziția primului bit setat la 1 sau la 0 dintr-un string. Poziția este returnată gândindu-ne la șirul de caractere ca la un array de biți pornind de la stânga la dreapta, unde primul bit semnificativ al primului byte se află la poziția 0, cel de-al doilea bit semnificativ al celui de-al doilea byte se află la poziția 8 ș.a.m.d.
Semnătura: `BITPOS key bit [start] [end]`

```text
redis> SET mykey "\xff\xf0\x00"
"OK"
redis> BITPOS mykey 0
(integer) 12
redis> SET mykey "\x00\xff\xf0"
"OK"
redis> BITPOS mykey 1 0
(integer) 8
redis> BITPOS mykey 1 2
(integer) 16
redis> set mykey "\x00\x00\x00"
"OK"
redis> BITPOS mykey 1
(integer) -1
```

## Resurse

- [REDIS BITMAPS – FAST, EASY, REALTIME METRICS](http://blog.getspool.com/2011/11/29/fast-easy-realtime-metrics-using-redis-bitmaps)
- [Bit field](https://en.wikipedia.org/wiki/Bit_field)
