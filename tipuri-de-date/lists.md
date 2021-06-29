# Lists

O listă este o suită de string-uri organizate într-o manieră ordonată prin indecși. Valorile duplicate sunt permise. Nu pot exista elemente în adâncime. O listă are drept element doar string-uri. Implementarea unei liste în Redis este *double linked*. Acest lucru înseamnă că un element va fi legat de elementul din dreapta, care la rândul său este legat de elementul din stânga sa. Acesta este și motivul pentru care comenzile au prefixul `L` de la *left* (`LPUSH`) și `R` de la *right* (`RPOP`).

## Comanda LPUSH

Adaugă un element sau mai multe la stânga listei. Este și comanda pe care o folosești inițial pentru a crea lista.

```text
LPUSH nume_listă valoare [valoare...]
```

## Comanda LSET

Comanda `LSET nume_listă index valoare` introduce o valoare la indexul specificat.

## Comanda LREM

Comanda `LREM nume_listă count valoare` șterge un anumit număr de elemente pentru o anumită valoare.

## Comanda RPUSH

Va adăuga un element sau mai multe la dreapta listei. Poți să te gândești ca la comanda cu care adaugi la listă elemente.

## Comanda LPOP

Comanda elimină un element de la stânga.

```text
LPOP nume_listă
```

## Comanda RPOP

Comanda elimină un element de la dreapta.

```text
RPOP nume_listă
```

## Comanda LRANGE

În Redis, listele au un index care pornește de la `0`.

```text
LRANGE nume_cheie 0 4
```

Dacă specfici drept index final valoarea `-1`, spui comenzii să selecteze toate eleentele de la indexul de pornire specificat, până la ultimul element al listei.

## Verifică dimensiunea listei

Comanda returnează un număr care indică câte elemente sunt în listă.

```text
LLEN nume_cheie_listă
```

## Comanda LINDEX

Comanda `LINDEX cheie număr_index` va aduce valoarea de la indexul specificat.

## Comanda LINSERT

Folosind `LINSERT nume_listă BEFORE|AFTER pivot valoare`

## Comanda LTRIM

Comanda `LTRIM nume_listă index_start index_stop` face o tăiere a elementelor care nu mai sunt dorite în listă. Să presupunem că ai nevoie doar de primele trei elemente din listă. Dacă introduci o valoare negativă pentru al doilea index, se numără de la coada listei începând cu `-1` care este ultimul element.

```text
ltrim nume_listă 0 -2
```

În exemplul de mai sus vom obține toate elementele mai puțin cel care era la final. Pentru a reține ultimele trei elemente, vei lansa comanda `LTRIM nume_lista -3 -1`.

Atenție, comanda modifică lista prin pierderea elementelor excluse.
