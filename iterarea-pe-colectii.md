# Iterarea pe colecții

Atunci când ai nevoie de a itera colecții de date în Redis, vei folosi, în funcție de tipul datelor următoarele comenzi:

- `SCAN` cu ajutorul căreia poți itera un set de chei (returnează un array de elemente care reprezintă cheile);
- `SSCAN` cu ajutorul căreia poți itera elementele unor date tip set (returnează un array de elemente care reprezintă membrii unui Set);
- `HSCAN` care permite iterarea câmpurilor unor date de tip hash și valorile lor asociate (returnează un array de elemente, fiecare având două componente: câmpul și valoarea);
- `ZSCAN` cu care iterezi elementele unui Sorted Set și a scorurilor asociate (returnează un array de elemente, fiecare având două componente: un membru și scorul asociat).

Aceste comenzi enumerate funcționează în mod similar.

Avantajul utilizării acestor comenzi este că iterarea se face incremental, fapt care permite obținerea unui număr mai mic de înregistrări la un singur apel. Un alt mare avantanj este că nu blochează serverul.

## Comanda SCAN

Această comandă este una care permite iterarea folosind un cursor. Acest lucru înseamnă că de fiecare dată când este apelată, serverul returnează un cursor cu date actualizate, iar acesta trebuie folosit în următorul apel pentru a accesa următorul calup de date. Pentru că este utilizat un cursor pentru iterare, este posibil ca un număr mare de clienți să itereze aceeași colecție. Acest lucru este posibil pentru că starea iterării este ținută de cursor, fără ca serverul să mențină informație privind fiecare iterare în parte. O iterare pornită, poate fi oprită fără ca serverul să cunoască acest lucru. Doar cursorul va avea această informație.

Iterarea începe când cursorul este la `0` și se încheie atunci când cursorul returnat de server este tot `0`. Când cursorul returnat este `0`, spunem că s-a făcut o iterare completă.

Semnătura: `SCAN cursor [MATCH pattern] [COUNT count] [TYPE type]`.

```text
127.0.0.1:6379> scan 0
1) "5120"
2)  1) "seatmap:NXJO-WCKP-ZNGO-GSPM:General:CY"
    2) "seatmap:XKHI-SUPX-DWAY-LLBM:General:ABI"
    3) "seatmap:QAZR-MDKP-ERYX-ZZGC:General:AO"
    4) "seatmap:RIPC-UJGD-KLZC-ULGG:General:C"
    5) "sales_order:LPKNGD-YPMXJB"
    6) "sales_order:HZZASX-XNULEZ"
    7) "seatmap:QARA-YRXA-IUUD-TZUY:General:MY"
    8) "seatmap:ENAH-YJOO-VAPF-WXYK:Reserved:P"
    9) "seatmap:RIPC-UJGD-KLZC-ULGG:General:PP"
   10) "invoice:GEFEAY-CDTOYA"
127.0.0.1:6379> scan 5120
1) "12800"
2)  1) "seatmap:QARA-YRXA-IUUD-TZUY:General:GN"
    2) "seatmap:YBZU-MHWA-LGXM-VJAB:General:EC"
    3) "invoice:NSWMPH-LWLXWS"
    4) "seatmap:FMSG-NFBG-UCCM-MFXH:Reserved:AJ"
    5) "seatmap:XZPW-IRVO-CEDT-DLYL:General:IG"
    6) "seatmap:YBZU-MHWA-LGXM-VJAB:General:CA"
    7) "seatmap:AKLY-HHEG-GIBT-CCCM:Reserved:CH"
    8) "invoice:OONDSS-OAJAYH"
    9) "sales_order:YIHWHI-BCVIKY"
   10) "seatmap:PDMM-JUOT-FPFF-BBLO:Reserved:ML"
   11) "seatmap:MUUY-HDVB-LTIJ-EKWI:General:BL"
   12) "seatmap:LPPO-ZJLB-XYEE-PZDQ:General:G"
```

În exemplul de mai sus sunt aduse cheile existente în bază. O iterate completă aduce toate elementele care existau la momentul începerii iterării. Iterarea nu aduce elemetele care nu au fost prezente în colecție de la bun început.

Este posibil ca elementele aduse să fie dublate și din acest motiv, se recomandă ca logica programului să detecteze aceste situații.

### Opțiunea COUNT

Pentru a limita numărul de elemente adus la o apelare, se poate folosi `COUNT` căruia i se dă o valoare ce reprezintă numărul de înregistrări dorite. Valoarea din oficiu pentru COUNT este 10. Un lucru important de reținut este că nu trebuie să folosești aceeași valoare pentru `COUNT` la fiecare apel.

Atunci când sunt iterate Set-uri care sunt codate ca *intsets* (seturi reduse compuse doar din numere întregi), sau Hash-uri și Sorted Sets codate ca *ziplists* (hash-uri de dimensiune redusă compuse din valori individuale reduse), de regulă sunt aduse toate înregistrările la prima apelare a lui `SCAN` indiferent de valoarea lui `COUNT`.

### Opțiunea MATCH

Este posibilă iterarea elementelor care se potrivesc unui șablon glob-style similar comportamentului comenzii KEYS care are drept unic argument un șablon.

Pentru a face această potrivire, introduci argumentul la finalul comenzii `SCAN`. Acest argument funcționează doar cu familia comenzilor `SCAN`.

```text
redis 127.0.0.1:6379> sadd myset 1 2 3 foo foobar feelsgood
(integer) 6
redis 127.0.0.1:6379> sscan myset 0 match f*
1) "0"
2) 1) "foo"
   2) "feelsgood"
   3) "foobar"
redis 127.0.0.1:6379>
```

Trebuie menționat faptul că filtrul pe care `MATCH` îl impune, se aplică abia după ce elementele sunt aduse din colecție chiar înainte de a fi trimise clientului.


### Opțiunea TYPE

Poți aduce înregistrări de un anumit tip dacă folosești această opțiune. Opțiunea este disponibilă doar pentru scanarea unei întregi baze de date. În consecință, nu se poate folosi cu `HSCAN` sau `ZSCAN`.

Argumentul `TYPE` este același string pe care comanda îl returnează. Abaterile pe care le vei întâlni sunt în cazul GeoHash-urilor, HyperLogLog-urilor, Bitmap-urilor și a Bitfield-urilor, care sunt implementate folosind alte tipuri Redis, precum string-uri sau zset-uri.

Filtrarea după `TYPE` este aplicată după ce elementele sunt aduse din baza de date.


## Resurse

- [SCAN cursor | redis.io/commands/scan](https://redis.io/commands/scan)
