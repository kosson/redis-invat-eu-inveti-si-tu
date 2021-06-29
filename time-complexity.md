# Time complexity

Redis a fost proiectat să fi eficient în ceea ce privește folosirea memoriei, a timpului de acces, dar și în ceea ce privește aspectele de rulare concurențială. Pentru fiecare comandă, la momentul proiectării s-au luat în considerare aceste aspecte.

Pentru fiecare comandă vei vedea în documenația oficială o notă privind *time complexity*.

De exemplu, pentru comanda `SADD nume_set valori` vei avea un *time complexity* de O(1). Notația folosită este **Big O**. Big O este un sistem de notație matematică care descrie comportamentul limitativ al unei funcții. O vine de la cuvântul în limba germană *ordung* - *ordinea*.

Atenție, chiar dacă două comenzi au aceelași Big O, timpul de execuție va fi diferit.

Fiecare comandă este proiectată să fie atomică. Acest lucru înseamnă că, fie va fi executată până la final, fie va eșua returnând starea sistemului de dinaintea executării. Pentru a rezolva, Redis folosește un singur fir de execuție, fiecare comandă fiind procesată una după alta în ordinea în care sunt primite. Din acest motiv este necesar să cunoști cât timp va lua execuția unei comenzi pentru că acel timp va bloca firul de execuție pentru celelalte, care chiar dacă au propriul timp de execuție mai mic, îl vor adăuga pe cel al comenzii anterioare.

## O(N)

De exemplu, setarea mai multor seturi odată folosind comanda `MSET` va avea un cost O(N).

```text
MSET set_nou1 "ceva" set_nou2 "altceva"
```

La fel se petrece și cu ștergerea cheilor din Redis folosind comanda `DEL`. Time complexity va depinde de numărul de chei care trebuie șterse.

Pentru o comandă `HMGET`, suplimentar numărului de câmpuri cerute, un alt factor care influiențează timpul este dimensiunea datelor stocate în fiecare câmp.

## O(M)

În cazul în care lucrezi cu liste, seturi, seturi ordonate sau hash-uri, pentru un `DEL`, de exemplu, vei avea O(M) pentru că va trebui ca funcția de ștergere să fie aplicată fiecărui membru al setului. Adu-ți aminte de faptul că Redis execută comenzile atomic, acest lucru însemnând că va bloca executarea altor comenzi. Pentru `DEL`, mai bine folosești `UNLINK` pentru că această comandă nu produce blocaje (background thread).

## O(N*M)

Este cazul unei intersecții `SINTER`, O(N*M) fiind cazul în care sunt folosite cele mai multe resurse. N reprezintă numărul elementelor setului cu cele mai puține (cardinalitatea), iar M este numărul seturilor care vor intra în intersecție. Dacă setul mai mic are 5 element, iar cel mai mare are 23, atunci vom avea un O(5\*2).

## O(S+N)

Este cazul lui `LRANGE lista1 idx_start idx_stop`, de exemplu. S va fi numărul elementelor care sunt înaintea celui de la care se pornește, iar N este numărul de elemente cerut. De exemplu, pentru un set de 10 elemente, dacă extragi un subset de la indexul 4, la 6, vei avea 5 elemente înaintea de la cel care pornește subsetul dorit. Deci, S va avea valoarea 5. N va avea valoarea de 3, adică numărul elementelor din subset.
Subsetul pornește având prima valoare pe cea de la indexul precizat drept limită. Ultima valoare va include pe cea de la indexul de stop. Propriu-zis ai `[4, 6]`.
