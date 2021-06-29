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

Comanda introduce o intrare în stream și returnează id-ul atribuit acesteia.
Semnătura: `XADD key [NOMKSTREAM] [MAXLEN|MINID [=|~] threshold [LIMIT count]] *|ID field value [field value ...]`

Un posibil exemplu de scriere în stream: `XADD mesaje_primite * user_id 4302 departamentul 5436 mesaj "Salutare popor!"`. În exemplul nostru, faptul că am folosit steluța după numele cheii, semnalizează Redis să construiască automat id-ul folosind un timestamp și o valoare numerică ce indică ordinea de intrare.

## Interogarea unui interval - XRANGE

Folosești comanda `XRANGE timestamp_start timestamp_stop` pentru a vedea care sunt înregistrările din acea fereastră de timp. Intrările sunt aduse în ordine începând cu cel mai vechi primul. Pentru a limita numărul de înregistrări aduse, se va folosi opțiunea COUNT.

## Cele mai noi cu XREVRANGE

Dacă vrei să aduci cele mai noi înregistrări mai întâi, folosești comanda `XREVRANGE`. Dacă nu știi perioada de timp reprezentată de intrările din stream, se pot folosi împreună `XRANGE` și `XREVRANGE` folosind operatorii `- +` care reprezintă cel mai mare și cel mai mic timestamp: `XRANGE checkins - + COUNT 2`.

## XREAD

Comanda `XREAD` poate consuma unul sau mai multe stream-uri și se oprește până la apariția de noi intrări în stream.
De exemplu, `XREAD STREAMS nume_stream id_de_pornire` => `XREAD STREAMS mesaje_primite 0` - adu mesajele noi care au id-uri mai mari de `0`.

## Resurse

- [Introduction to Redis Streams](https://redis.io/topics/streams-intro)
