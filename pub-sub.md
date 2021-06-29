# Pub/Sub

Această caracteristică Redis permite unor consumatori să se aboneze la un canal (*bus*) în baza unui nume sau al unui pattern. Astfel, Redis se poate comporta precum un broker pentru mai mulți clienți pentru că oferă posibilitatea ca mesajele/evenimentele să fie postate și apoi consumate. Acest mecanism poartă denumirea de *syndication*. Un canal nu trebuie neapărat să existe ca să te abonezi la el.

Pentru un singur nod de Redis, ordinea mesajelor este garantată.

În cazul în care un client se abonează la un canal după ce au fost emise mesaje în acel canal, nu va avea acces la cele anterioare momentului abonării. Mecanismul după are se face emiterea mesajelor este *fire-and-forget* ceea ce presupune ca un receptor deja să fie abonat pentru a primi mesajele.

Un receptor se poate abona la mai multe canale odată.

## Comanda PUBLISH

Această comandă este folosită pentru a introduce un mesaj pe un canal. Mesajul poate fi orice string binar. Adeseori veți vedea faptul că sunt trimise blob-uri de JSON serializat.
Semnătura: `PUBLISH canal mesaj`.

Toți receptorii vor primi mesajul și conținutul acestuia (*payload*). Dacă este trimis un mesaj de 1MB și sunt abonați 100 de clienți, prin rețea vor trece 100MB.

```text
publish ch-1 "salutare popor"
(integer) 1
```

## Comanda SUBSCRIBE

Această comandă permite *ascultarea* mesajelor dintr-un canal. Folosind această comandă, te poți abona la mai multe canale. Numele canalului trebuie să fie cunoscut pentru că nu sunt permise wildcard-urile. Căutarea unui canal folosind șabloane este posibilă folosind `PSUBSCRIBE`.
Semnătura: `SUBSCRIBE canal [canale...]`.

SUBSCRIBE este un apel care blochează firul de execuție, dar spre deosebire de celelalte comenzi care blochează, Redis va accepta și alte comenzi.

```text
subscribe ch-1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "ch-1"
3) (integer) 1
```

Mesajul primit după abonare indică prin `1) "subscribe"` faptul că ești abonat la `ch-1`, iar prin `3) (integer) 1`, câți sunt abonați la acel canal.
Când ești deja abonat și primești mesaje, vei avea în terminal un astfel de rezultat.

```text
1) "message"
2) "ch-1"
3) "salutare"
```

Vei vedea că la `1)` este indicat statutul comunicării, care în exemplul de mai sus este mesaj. Variantele pe care le poți avea sunt:
- "subscribe",
- "unsubscribe",
- "message".

Când te abonezi la mai multe canale:

```text
subscribe ch-1 ch-2
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "ch-1"
3) (integer) 1
1) "subscribe"
2) "ch-2"
3) (integer) 2
```

## Comanda UNSUBSCRIBE

Comanda oprește ascultarea mesajelor pe pe un canal.
Semnătura: `UNSUBSCRIBE [canal [canal...]]`

## Patterned syndication

Uneori ai nevoie să filtrezi mesajele în funcție de un anumit șablon.

### Comanda PSUBSCRIBE

Comanda este folosită pentru a te abona la un canal(e) pe care-l selectezi folosind un șablon prin care selectezi numele acestora.
Semnătură: `PSUBSCRIBE șablon [șablon...]`.

Șabloane pentru care există suport:
- `?` pentru un singur caracter,
- `*` pentru mai multe caractere,
- `[]` oricare dintre caracterele din parantezele pătrate,
- `^` pentru a indica prefixe; de exemplu: `[^Analize]*`.

De exemplu, `psubscribe ch-?` va avea drept efect abonare la canalele `ch-1` și `ch-2`.

```text
psubscribe ch-?
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "ch-?"
3) (integer) 1
```

Când vei primi un mesaj, conținutul rezultat este puțin diferit.

```text
// Clientul 1 emitent
publish ch-2 "Cine vine la cina?"
(integer) 2 -> indică faptul că sunt abonați doi la canal
// Clientul 2 receptor
1) "pmessage"
2) "ch-?"
3) "ch-2"
4) "Cine vine la cina?"
```

Observă faptul că la `2)` este indicat șablonul care a fost folosit, iar la `3)` este canalul care a primit mesajul.

### Comanda PUNSUBSCRIBE

Semnătură: `PUNSUBSCRIBE șablon [șablon...]`.

## Administrare `PUBSUB`

Folosești această comandă pentru a vedea la care canale ești abonat.
Semnătura: `PUBSUB subcomandă [argument [argument...]]`.

### PUBSUB channels

Comanda oferă o listă a tuturor canalelor active la care este clientul abonat.
Semnătură: `PUBSUB CHANNELS [pattern]`.

```text
pubsub channels *
```

În exemplul de mai sus sunt returnate toate canalele existente.

Dacă nu este specificat niciun șablon, vor fi afișate toate canalele. În caz contrar, dacă șablonul este prezent, vor fi aduse doar canalelel care respectă șabloanele tip glob-style.


### PUBSUB NUMSUB

Comanda returnează numărul de abonați fără cei care sunt abonați folosindu-se de un anumit șablon.
Semnătura: `PUBSUB NUMSUB [channel-1 ... channel-N]`.

### PUBSUB NUMPAT

Comanda returnează numărul de abonați folosind șabloanele.
Semnătura: `PUBSUB NUMPAT`.
