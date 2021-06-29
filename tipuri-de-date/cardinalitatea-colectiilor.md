# Cardinalitatea colecțiilor

Poți afla cardinalitatea folosind următoarele comenzi:

- `LLEN` pentru liste,
- `SCARD` pentru seturi,
- `ZCARD` pentru seturi ordonate.

## Obținerea de subseturi

Atunci când avem nevoie să obținem un subset de elemente, vorbim despre colecții limitate (*capped collections*).

Cazurile în care am avea nevoie de subseturi (*capped collections*) folosind date Redis ar fi dictate de menținerea scorului cu primele n intrări pentru jocuri sau chiar articole de blog.

### Cazul listelor

În cazul listelor, se va folosi comanda `LTRIM`, care permite specificarea unui număr limitat de elemente pe care dorești să le ții în continuare ca parte a listei.

Listele sunt structuri de date poziționale și astfel poți indica indexul de la care dorești să pornească selecția și apoi menționezi indexul până la care se vor păstra elementele. Semnătura: `LTRIM nume_set idx_start idx_stop`. Indexul din stânga trebuie să fie pozitiv, dar indexul din dreapta poate fi un număr negativ. Pornind de la ultimul element din listă, care poartă valoarea de index `-1`, poți număra oricâte elemente către indexul de start folosind numerele negative pentru identificare. Acest lucru este util atunci când elementele care vor rămâne se află mai spre dreapta dar nu toate până la finalul listei.

```text
LPUSH testlist1 a b c d e f
LRANGE testlist1 0 -1
1) "f"
2) "e"
3) "d"
4) "c"
5) "b"
6) "a"
```

Un caz tipic în obținerea de *capped lists* este să elimini elemente folosind fie `LPUSH`, fie `RPUSH` urmat de un `LTRIM` la dimensiunea dorită.

### Cazul seturilor sortate

Pentru seturile sortate, echivalentul lui `LTRIM` este `ZREMRANGEBYRANK nume_oset idx_start idx_stop` cu diferența că prin această comandă indici elementele pe care vrei să le elimini, nu să le păstrezi. Adu-ți mereu aminte că seturile ordonate sunt și ele poziționale, elementele având un index.

Dacă ai un set ordonat de 4 elemente și dorești să-l elimini pe ultimul, asta însemnănd că are un rank mic, vei specifica la indexul de start valoarea de index a ultimului element, în cazul nostru 3 iar la indexul de stop, vei pune valoarea negativă a indexului acestuia, adică -1. Drept efect, ultimul element va fi eliminat. Dacă am fi dorit eliminarea a ultimelor două elemente, am fi precizat valoarea de index pozitivă a elementului, adică 2 și drept index de stop -1. Astfel, ștergi ultimele două elemente dintr-un set ordonat de 4.

În cazul în care dorești să ții o tabelă de scor, după adăugarea unui element în set cu `ZADD`, acesta va fi urmat de un `ZREMRANGEBYRANK` și astfel ții un set la aceeași dimensiune. Totuși, această soluție poate fi împovărătoare și își dovedește eficiența doar când numărul elementelor pe care le păstrezi este mai mare decât cel pe care le elimini.
