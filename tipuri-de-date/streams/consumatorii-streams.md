# Consumatorii de stream-uri

Pentru a consuma un stream, vei folosi comanda `XREAD`.
Semnătura: `XREAD [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] ID [ID ...]`.

Dacă privești această comandă fără implicațiile sale mai adânci, ai putea spune că este un înlocuitor pentru `XRANGE`.
Această comandă nu poate fi folosită cu `-` și `+`, acestea fiind utilizate exclusiv de `XRANGE` și `XREVRANGE`.

Comanda `XREAD` oferă mult mai mult decât `XRANGE`. De exemplu, răspunsul unei comenzi `XREAD` va fi numărul de mesaje specificat prin `COUNT` care au ajuns după ID-ul cerut. Poți folosi un id parțial (`0`) sau `0-0` pentru a specifica mesajul anterior primului mesaj din stream: `XREAD COUNT 1 STREAMS nume_stream 0-0`. Poți folosi id-ul returnat pentru a cere următorul mesaj și astfel poți consuma înregistrările din stream una câte una sau mai multe într-un calup în funcție de valoarea de la `COUNT`.

La opțiunea `STREAMS` pot fi menționate mai multe stream-uri din care să se facă citirea.

Comanda `XREAD` este una multicheie acest lucru fiind remarcat prin faptul că numele cheii din care s-a făcut citirea include numele stream-ului.

```text
XREAD COUNT 2 STREAMS mystream writers 0-0 0-0
```

## Funcționarea în regim de blocare

Adevăratul avantanj al comenzii `XREAD` este că poate fi parametrat să funcționeze într-un mod blocking anulând necesitatea consumatorilor de a face polling pentru resursele stream-ului. Propriu-zis opțiunea `BLOCK` menționează numărul de milisecunde cât execuția comenzii va întârzia și astfel, i se permite stream-ului să acumuleze mesaje noi.

În cazul în care valoarea pentru opțiunea `BLOCK` este `0`, citirea stream-ului va fi blocată până la momentul în care stream-ul este populat cu o nouă intrare. Pentru a aduce doar ultimul mesaj din stream, va fi pasat după numele cheii, caracterul special `$`.

```text
XREAD BLOCK 0 STREAMS nume_stream $
```

## Consumatori concurenți

Atunci când mai mulți consumatori ascultă noi mesaje dintr-un stream, fiecare mesaj nou este trimis fiecărui consumator care blochează și care consumă stream-ul. Acest mod de operare se numește *fan-out*.

Acest model permite distribuția în paralel a mesajelor prin împărțirea procesării în mai multe task-uri concurente care se execută asincron.

Fiecare consumator care blochează consumă separat streamul în termenul logicii pe care o rulează și a resurselor pe care le alocă. Totuși, aceștia au la dispoziție același stream și așteaptă aceleași mesaje să apară. Din moment ce fiecare consumator își termină procesarea, reintră în *blocking read state* pentru a procesa următorul mesaj.

## Grupuri de consumatori
