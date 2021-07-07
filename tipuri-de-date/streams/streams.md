# Streams

Acesta este un nou tip de date introdus odată cu versiunea 5 a lui Redis. Stream-urile Redis sunt o structură de date la care doar poți să adaugi lucruri. Poți să-ți imaginezi ca este ca un fișier de logging. Are un comportament *append only log*. Poți să te gândești la un stream și ca la o serie de evenimente la care adaugi altele noi. Ordinea intrărilor nu poate fi modificată. Redis permite ștergerea de intrări dintr-un stream.

Când folosești stream-uri, de fapt folosești date care sunt generate în mod continuu. De regulă aceste date vin din surse concurente diferite. Faptul că un stream este de fapt o cheie în Redis, acest lucru permite aplicatea tuturor operațiunilor caracteristice cheilor. De exemplu, poți folosi comanda `DEL` pentru a șterge un stream. Poți să aplici un TTL unui stream, dacă acest lucru este util pentru a nu permite o creștere lipsită de control a stream-ului.

Fiecare intrare dintr-un stream are un id unic. Din oficiu aceste elemente de identificare sunt prefixate cu un timestamp. Acest lucru înseamnă că intrările dintr-un stream constituie o serie organizată temporal. Stream-urile Redis nu au o schemă. Chiar dacă o intrare în stream trebuie să aibă cel puțin un nume și o valoare, înregistrările nu trebuie să aibă aceeași structură, aceleași câmpuri.

Redis permite consumatorilor să citească intrările dintr-un stream în ordine. Consumatorii pot căuta orice intrare din stream în baza id-ului. Pentru că id-urile conțin un timestamp, putem afirma că interogările rezultă în serii temporale (*time range*).

Stream-urile pot fi consumate și procesate de multiple seturi de consumatori distincți. Aceștia sunt numiți *consumer groups*. Id-ul poate conține un element de identificare a grupului de consumator căruia întregistrările sunt destinate. Un grup de consumatori poate fi responsabil cu introducerea datelor într-o bază de date, altul cu notificări, ș.a.m.d.
Toate grupurile de consumatori citesc streamul în întregime și vor primi toate intrările. Fiecare grup de consumatori poate conține unul sau mai mulți consumatori fiecare lucrând împreună pentru a procesa streamul. Consumatorii individuali dintr-un grup vor primi câte un subset de intrări pentru procesare.

## Arhitectura

În cazul unei arhitecturi pentru o aplicație distribuită, componentele care scriu date într-un stream sunt numite *producers*. Datele generate de aceștia sunt adăugate într-un stream. Fiecare înregistrare are propriul id și setul propriu de câmpuri.

Cei care citesc aceste date din stream se numesc *consumers*. Aceștia citesc datele din stream și le prelucrează. Mai mulți consumatori pot citi datele dintr-un stream. Un consumator poate juca rol de consumator, altul are în sarcină popularea unei baze de date, iar altul pur și simplu citește datele, le prelucrează și la rândul său devin un *producător* constituind un nou stream cu un subset de date sau chiar altele noi cu totul.

Producătorii și consumatorii lucrează la propriile capacități, ceea ce face ca stream-ul să se interpună ca un buffer. Acest lucru permite ca un producător să nu fie nevoit să cunoască nimic din modul de funcționare al unui consumator.

Odată introdusă o înregistrare în stream, aceasta nu poate fi modificată (*immutable*).

Procesarea stream-urilor este opusă *batch processing*.

## Adăugarea unei intrări în stream - XADD

Comanda introduce o intrare în stream și returnează id-ul atribuit acesteia. Această comandă este considerată a fi API-ul folosit de producătorul de stream-uri.
Semnătura: `XADD key [NOMKSTREAM] [MAXLEN|MINID [=|~] threshold [LIMIT count]] *|ID field value [field value ...]`

Un posibil exemplu de scriere în stream: `XADD mesaje_primite * user_id 4302 departamentul 5436 mesaj "Salutare popor!"`. În exemplul nostru, faptul că am folosit steluța după numele cheii (*mesaje_primite*), semnalizează Redis să construiască automat id-ul folosind un timestamp și o valoare numerică ce indică ordinea de intrare.

Numele cheii în baza căruia se constituie stream-ul, poate fi considerat ca un contract între producătorul stream-ului și consumatorii acestuia. Dacă această cheie nu există, va fi creată automat. În cazul în care nu se dorește crearea stream-ului, se va menționa opțiunea `[NOMKSTREAM]` imediat după numele stream-ului.

Comanda se adresează unui singure chei, adică unui singur stream.

Al doilea argument după numele cheii este `*|ID`, care în cazul introducerii unei steluțe, Redis va crea un identificator al mesajului în baza unui timestamp. Acest timestamp va fi stocat ca număr întreg pe 64 de bit. Aceasta este și caracteristica care este căutată în cazul stream-urilor pentru că se dorește ca mesajele să fie introduse în stream într-o anumită ordine, iar cea mai bună este și cea cronologică.

```text
7 XADD streamtest * ceva:1 10 ceva:2 21
"1625056387700-0"
"1625056387700-1"
"1625056387701-0"
"1625056387701-1"
"1625056387701-2"
"1625056387701-3"
"1625056387701-4"
```

În exemplul de mai sus, s-a folosit un truc al *redis-cli*-ului pentru a introduce în stream 7 mesaje. Ceea ce se observă este faptul că există un id format dintr-un timestamp, iar în cazul în care introducerea mesajului s-a realizat într-un moment la nivel de milisecundă, după timestamp id-ul este completat cu o secvență crescătoare care identifică unic mesajul, chiar dacă are același fragment de timestamp. Cu fiecare milisecundă care trece contorul crescător este resetat la zero așa cum se vede și în exemplu. Precum timestamp-ul, și contorul este implementat ca un număr întreg de 64 de biți.

Restul argumentelor sunt perechi de nume câmp - valoare similar introducerii valorilor într-un `HSET`.

## Interogarea unui interval - XRANGE

Folosești comanda `XRANGE timestamp_start timestamp_stop` pentru a vedea care sunt înregistrările din acea fereastră de timp. Intrările sunt aduse în ordine începând cu cel mai vechi primul. Pentru a limita numărul de înregistrări aduse, se va folosi opțiunea COUNT.
Semnătura: `XRANGE key start end [COUNT count]`.

Pentru a obține toate înregistrările pornind de la cea mai mică, la cea mai mare, se pot folosi pentru ID-uri `-` și `+`. Astfel, vor fi aduse toate înregistrările din stream.

```text
XRANGE somestream - +
```

În cazul acesta, semnul minus este echivalent unui ipotetic ID `0-0`, iar semnul plus este pentru `18446744073709551615-18446744073709551615`. În cazul unui ID, se poate folosi doar partea care specifică numărul de milisecunde. În cazul în care pentru valoarea de început și pentru cea finală este același număr de milisecunde, Redis va returna toate înregistrările introducând automat partea de id dedicată contorului, pornind la `-0`, până la `-18446744073709551615`. Astfel, vor fi aduse toate milisecundele.

Folosind opțiunea `COUNT`, poți limita numărul de mesaje care sunt aduse dintr-un interval.

```text
XRANGE nume_stream 0 + COUNT 5
```

Observă că în exemplu, am folosit valoarea `0`, un echivalent valid al lui minus (`-`). Este echivalentul unui *partial id*, adică a părții dedicate milisecundelor. Astfel, punând valoarea `0` de la care să pornești, este echivalentul id-ului `0-1`.

Comanda reciprocă este `XREVRANGE`.

## Cele mai noi cu XREVRANGE

Dacă vrei să aduci cele mai noi înregistrări mai întâi, folosești comanda `XREVRANGE`. Dacă nu știi perioada de timp reprezentată de intrările din stream, se pot folosi împreună `XRANGE` și `XREVRANGE` folosind operatorii `- +` care reprezintă cel mai mare și cel mai mic timestamp: `XRANGE checkins - + COUNT 2`.

Pentru a aduce cel mai recent mesaj: `XREVRANGE nume_stream + - COUNT 1`.

## XREAD

Comanda `XREAD` poate consuma unul sau mai multe stream-uri și se oprește până la apariția de noi intrări în stream.
De exemplu, `XREAD STREAMS nume_stream id_de_pornire` => `XREAD STREAMS mesaje_primite 0` - adu mesajele noi care au id-uri mai mari de `0`.

## XLEN

Comanda returnează numărul de intrări dintr-un stream.
Semnătura: `XLEN key`.

```text
xlen streamtest
(integer) 7
```

Pot exista streamuri de dimensiune 0. Astfel că ar trebui să apelezi `TYPE` sau `EXIST` pentru a verifica dacă o anumită cheie există sau nu. Stream-urile nu sunt șterse dacă nu mai au intrări pentru că un stream ar putea avea asociate grupuri de consumatori. Acesta este un comportament specific doar acestor chei care sunt stream-uri spre deosebire de alte tipuri de date din Redis.

```text
127.0.0.1:6379> XLEN cevacenuexista
(integer) 0
127.0.0.1:6379> exists cevacenuexista
(integer) 0
```

## XDEL

Comanda șterge mesaje din stream și returnează numărul mesajelor care au fost șterse. Acest mecanism de ștergere poate fi util în cazul în care din motive de confidențialitate trebuie să ștergem datele.
Semnătura: `XDEL key ID [ID ...]`

```text
7 XADD streamtest * ceva:1 10 ceva:2 21
"1625056387700-0"
"1625056387700-1"
"1625056387701-0"
"1625056387701-1"
"1625056387701-2"
"1625056387701-3"
"1625056387701-4"
XDEL streamtest 1625056387700-0 1625056387700-1
(integer) 2
XLEN streamtest
(integer) 5
xrange streamtest - +
1) 1) "1625056387701-0"
   2) 1) "ceva:1"
      2) "10"
      3) "ceva:2"
      4) "21"
2) 1) "1625056387701-1"
   2) 1) "ceva:1"
      2) "10"
      3) "ceva:2"
      4) "21"
3) 1) "1625056387701-2"
   2) 1) "ceva:1"
      2) "10"
      3) "ceva:2"
      4) "21"
4) 1) "1625056387701-3"
   2) 1) "ceva:1"
      2) "10"
      3) "ceva:2"
      4) "21"
5) 1) "1625056387701-4"
   2) 1) "ceva:1"
      2) "10"
      3) "ceva:2"
      4) "21"
memory usage streamtest samples 0
(integer) 675
```

XDEL este o metodă impractică pentru a ține sub control dimensiunea unui stream pentru că intrările sunt șterse, dar memoria nu este eliberată. Pentru a ține sub control dimensiunea unui stream, se va folosi comanda `XTRIM`.

## XTRIM

Comanda face triming la o anumită dimensiune și va recupera memoria blocată cu date. Comanda returnează numărul înregistrărilor care au fost șterse.
Semnătura: `XTRIM key MAXLEN|MINID [=|~] threshold [LIMIT count]`.

Pentru că trimingul are o amprentă în ceea ce privește performața, sunt posibile două tipuri de triming. Un trimming care lasă stream-ul exact la o anumită dimensiune sau o tăietură aproximativă care doar aproximează încercarea de redimensionare.
De regulă, ceea ce se dorește este eliminarea intrărilor ceva mai vechi (ID-uri mai mici).

Opțiunea `MAXLEN` șterge înregistrările atunci când dimensiunea stream-ului depășește pragul specificat, unde pragul este un număr întreg pozitiv.

```text
XTRIM mystream MAXLEN 1000
```

Stream-ul va fi tăiat la ultimele 1000 de înregistrări.

Opțiunea `MINID` șterge înregistrările care au ID-ul mai mic de pragul specificat, iar pragul este un ID.

```text
XTRIM mystream MINID 649085820
```

În exemplul de mai sus, vor fi șterse toate înregistrările care au un id mai jos de valoarea 649085820-0

### Redimensionarea cu aproximație

Pentru că tăierea cu exactitate necesită un efort suplimentar din partea serverului Redis, este oferit un argument opțional `~` care este mult mai eficient.

```text
XTRIM mystream MAXLEN ~ 1000
```

Acest lucru înseamnă că Redis va tăia la o dimensiune cel puțin egală cu pragul specificat.

Comanda de triming poate fi invocată periodic, dar dacă se dorește, funcționalitatea sa poate fi invocată direct în comanda `XADD`.

```text
XADD nume_cheie MAXLEN ~ 9345 * nume_camp 13232423
```

În acest caz, de fiecare dată când un mesaj va fi adăugat la stream, acesta va fi trunchiat automat.

## Resurse

- [Introduction to Redis Streams](https://redis.io/topics/streams-intro)
