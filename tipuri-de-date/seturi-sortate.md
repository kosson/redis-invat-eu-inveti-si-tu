# Seturi sortate

Seturile ordonate pot primi doar valori care sunt unice. Ordinea de sortare este determinată de valoarea scorului asociat fiecărui element. Spre deosebire de seturi, seturile sortate nu pot stoca rezultatele decât în alte seturi sortate.

Seturile sortate se pretează cel mai bine pentru a ține evidența utilizatorilor logați. Fiecare valoare a unui set ordonat are o valoare care reprezintă scorul și o valoare de identificare pentru element. Valoarea scorului este un număr cu virgulă mobilă. Dacă două elemente au același scor, departajarea se va face în funcție de ordinea lexicală. Acest lucru înseamnă că se va calcula valoarea caracterelor care alcătuiesc elementul și se vor compara. De exemplu, dacă elementul `a` are același scor cu `b`, elementul `a` va sta în fața (mai sus) de `b`.

Manipularea seturilor ordonate se poate face prin modificarea valorii scorului, a poziției sau la nivel lexicografic. Precum seturile, seturile ordonate au comenzi cu ajutorul cărora poți realiza uniunea sau intersecția. Drept valori nu poți introduce alte liste, seturi sau hash-uri. Este respectată regula string-urilor.

Această structură este perfectă pentru realizarea de tabele de scor pentru jocuri sau aplicații care implică o anumită ierarhizare. Un set ordonat poate fi accesat într-o ordine ascendentă sau descendentă. Valoarea scorului poate fi incrementată sau decrementată, fapt care conduce la modificarea ordinii setului.

Seturile ordonate sunt serii de numere ordonate. De exemplu, `scor id_player`.

## Adăugarea unei înregistrări

Comanda `ZADD` permite introducerea unei noi înregistrări: `ZADD scor:user val_scor id_player`.
Semnătura este `ZADD nume_oset [NX|XX] [CH] [INCR] SCORE MEMBER [SCORE MEMBER...]`.
Opțiunea `NX` adaugă elementul dacă acesta nu există deja.
Opțiunea `XX` permite doar actualizarea unui element care există deja.

Atunci când elementele sunt adăugate într-un set ordonat, valoarea returnată este numărul elementelor care au fost adăugate.

```text
zadd "subway:Keiyo Line" 5 Shin-Kiba 3 Etchujima
(integer) 2
```

Acest comportament poate fi modificat folosind opțiunea `CH`. Opțiunea `CH` informează asupra numărului de elemente modificate de comandă.

Opțiunea `INCR` oferă posibilitatea actualizării unei singure perechi scor element.

## Investigarea unui set ordonat

Folosind această comandă poți investiga componența unui set ordonat.
Semnătura: `ZSCAN key cursor [MATCH pattern] [COUNT count]`.

```text
zadd "subway:Keiyo Line" 1 Tokyo
(integer) 1
ZSCAN "subway:Keiyo Line" 0 match *
1) "0"
2) 1) "Tokyo"
   2) "1"
```

## Incrementarea unei valori

Comanda `ZINCRBY scor:user valoarea_de_incrementare id_player`

## Parcurgerea unui set sortat

Pentru parcurgerea unui set sortat, ai la îndemână comenzile `ZRANGE` și `ZREVRANGE`.

### ZRANGE

Această comandă este folosită pentru a vedea valorile unui set ordonat. Se folosește adesea pentru investigarea acestor valori.

Comanda parcurge un set ordonat de la valoarea indexului. Reține faptul că ordinea totuși o dă valoarea scorului.
Semnătura: `ZRANGE nume_oset idx_start idx_stop [WITHSCORES]`. Indexul într-un set ordonat pornește de la valoarea `0`. Pentru a obține toate elementele, se va alege valoarea `-1` pentru al doilea index.

Dacă vom folosi parametrul `WITHSCORES`, va fi returnat și scorul fiecărui element.

```text
ZRANGE "subway:Keiyo Line" 0 10 WITHSCORES
1) "Tokyo"
2) "1"
3) "Etchujima"
4) "3"
5) "Shin-Kiba"
6) "5"
```

### ZREVRANGE

Comanda face opusul lui `ZRANGE`, ordonând valorile de la cea mai mare, la cea mai mică. Comanda este urmată de numele setului ordonat, apoi indexul de la care să se pornească ordonarea urmat de indexul până la care se iau în considerare valorile.

```text
ZREVRANGE "subway:Keiyo Line" 0 10 WITHSCORES
1) "Shin-Kiba"
2) "5"
3) "Etchujima"
4) "3"
5) "Tokyo"
6) "1"
```

### ZRANGEBYSCORE

Comanda `ZRANGEBYSCORE` permite aducerea elementelor care sunt într-o plajă de scor definită printr-o valoare minimă și una maximă.
Semnătura: `ZRANGEBYSCORE nume_oset min max [WITHSCORES] [LIMIT offset count]`.

### ZRANGEBYLEX

Comanda `ZRANGEBYLEX` permite aducerea elementelor în baza calculării valorii lexicale.
Semnătura: `ZRANGEBYSCORE nume_oset min max [WITHSCORES] [LIMIT offset count]`.

## Comanda ZRANK

Rangul (*rank*) la care se află un element într-un set ordonat este chiar indexul la care stă elementul în funcție de scorul său care-l poziționează acolo.

Pentru a vedea poziția/indexul în care se află un anumit element dintr-un set ordonat, se va folosi comanda `ZRANK scor:user id_player`. Atenție, valoarea returnată nu este cea a scorului, ci a indexului la care se află elementul.

## Comanda ZREVRANK

Comanda va fi folosită pentru a vedea ordinea în formă înversă pornind de la valoarea scorului.

## Comanda ZSCORE

Pentru a afla valoarea scorului unui player, vom folosi comanda `ZSCORE scor:user id_player`.

## Comanda ZCOUNT

Această comandă este folosită pentru a returna elementele între valoarea scorului de start și cea de încheiere.
Semnătură: `ZCOUNT nume_oset val_min val_max`.

## Șterge elemente

### ZREM

Folosind această comandă poți șterge un element dintr-un set ordonat folosind valoarea elementului, nu valoarea scorului.
Semnătură: `ZREM nume_oset membru [membru...]`.

### ZREMRANGEBYLEXs

### ZREMRANGEBYRANK

Această comandă este echivalentul lui `LTRIM` al listelor, dar spre deosebire de aceasta, `ZREMRANGEBYRANK` indică elementele pe care dorim să le eliminăm.
Semnătura: `ZREMRANGEBYRANK nume_oset idx_start idx_stop`.

Să presupunem că setul ordonat are cinci elemente. Pentru a șterge ultimul element, vei preciza drept indexul de start 5 și indexul de final -1. Pentru a șterge ultimele două elemente, primul index va fi 4, iar ultimul -1.

Motivul pentru care ai folosi comanda este în combinațiile cu `zadd nume_set număr_scor valoare_element`, când introduci un element care reface ordinea în set. Voi dori ca scorul cel mai mic să fie eliminat și astfel să ții numărul setului ordonat același. Pentru a face ajustarea, vei folosi comanda `ZREMRANGEBYRANK nume_oset 0 0`.

### ZREMRANGEBYSCORE

## Operațiuni pe seturi

### Intersecția seturilor ordonate cu ZINTERSTORE

Fiecare set ordonat are propriile scoruri pentru fiecare dintre elemente. Atunci când faci o intersecție, trebuie să decizi asupra scorurilor elementelor comune.

Pentru a realiza o intersecție a două seturi ordonate, se va folosi `ZINTERSTORE`. Comanda returnează un număr întreg, care reprezintă câte elemente are noul set ordonat format reprezentând intersecția.
Semnătură: `ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]`.

- destination este un set ordonat care va fi populat cu rezultatul intersecției. Dacă setul ordonat există deja, acesta va fi suprascris.
- NUMKEYS este numărul de seturi ordonate din care se va constitui uniunea. Acest paramentru absolut necesar.
- WEIGHTS este factorul de multiplicare pentru fiecare element al seturilor care intră în intersecție, fiindu-i necesar mai departe lui `AGGREGATE`. Când nu este precizată o valoare, cea din oficiu va fi `1`. Ceea ce se petrece este că fiecare element al fiecărui set este înmulțit cu valoarea lui `WEIGHT`. Trebuie precizate valori pentru fiecare set ordonat care intră în intersecție.

```text
ZADD oset1 21 "unu" 54 "doi" 22 "trei" 34 "alfa" 654 "beta"
ZADD oset2 43 "alfa" 833 "beta" 3 "teta"
ZADD oset3 45 "primo" 51 "secundo" 98 "terzo"
ZINTERSTORE osetinter1 2 oset1 oset2 WEIGHTS 10 10
ZRANGE osetinter1 0 -1 WITHSCORES
```

Intersecția funcționează și în cazul în care dorim să lucrăm și cu un set simplu. În acest caz, va trebui să atribuim un score weighted pentru set pentru că acesta nu are un scor pentru fiecare element.

### ZUNIONSTORE

Pentru a realiza o uniune a două seturi ordonate, se va folosi `ZUNIONSTORE`. Aceeași comandă poate fi folosită pentru a realiza uniunea unui set ordonat cu un set normal, caz în care trebuie adaugate valori de ponderare pentru toate seturile care intră în uniune.
Semnătură: `ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]`.

## Resurse

- [Sorted Sets | redis.io/commands](https://redis.io/commands#sorted_set)
