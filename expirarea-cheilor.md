# Expirarea cheilor

Înregistrările care nu se modifică des pot fi cache-uite în Redis. Atunci când faci caching de date, vei seta o perioadă după care datele să expire. Atunci când folosești comanda `SET` poți preciza și timpul de expirare cu `PX` pentru milisecunde și `EX` pentru secunde.

```text
SET date:caz1023 '{ceva: true}' EX 7200
```

Pentru exemplul de mai sus, datele vor expira în 7200 de secunde (2 ore).

Pentru a vedea cât timp mai are până la expirare, poți verifica folosind `TTL`, care returnează numărul de secunde cât mai are o înregistrare înainte să expire.

```text
TTL date:caz1023
```

Perioda după care o cheie expiră poate fi menționată în secunde, milisecunde sau timestamp-uri UNIX.
Comenzi TTL:

- EXPIRE cheie secunde
- EXPIREAT cheie timestamp
- PEXPIRE cheie milisecunde
- PEXPIREAT cheie milisecunde-timestamp

Pentru a examina timpul de expirare, poți folosi:

- TTL cheie
- PTTL cheie

Folosești `TTL` pentru secunde și `PTTL` pentru milisecunde.

Pentru a șterge timpul de expirare, folosești comanda `PERSIST cheie`. Această comandă va returna `-1` care înseamnă că acea cheie ar trebui să fie păstrată.
