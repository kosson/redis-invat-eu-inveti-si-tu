# Faceted search

Indexarea este o formă de a obține performațe sporite în regăsirea datelor. Căutarea fațetată se materializează prin crearea unui index inversat. Redis nu area indexuri secundare, ceea ce înseamnă că nu poți constitui un index la nivel de câmp.

```json
{
  'id':"23-dre-325",
  'name': "Final la paralele băieți",
  'disabled_access': True,
  'medal_event': True,
  'venue': "Stadionul Olimpia",
  'categorie': "Gimnastică"
},
{
  'id':"52-dsa-343",
  'name': "Final la paralele fete",
  'disabled_access': True,
  'medal_event': True,
  'venue': "Stadionul Olimpia",
  'categorie': "Gimnastică"
},
{
  'id':"76-rsa-984",
  'name': "100 de metri garduri",
  'disabled_access': False,
  'medal_event': True,
  'venue': "Stadionul Național",
  'categorie': "Atletism"
}
```

Obiectele JSON pot fi serializate și introduse drept valoare a unei chei. Căutarea prin obiecte s-ar putea face folosind comanda `SCAN CURSOR [MATCH PATTERN][COUNT COUNT]`. Această comandă nu este cea mai potrivită în sistemele de producție.

La `MATCH` se va introduce un șablon de tip *glob*. Acest tip de șablon folosește caractere cu rol special așa cum este steluță pentru a face înlocuiri a oricărui caracter/înșiruire de caractere care ar putea fi în acea poziție. De exemplu, `*.txt` este un *șablon general* - *glob pattern*. Alt caracter cu o funcție de înlocuire este `?` care desemneză strict un singur caracter.

```text
set user:112 ionuț
set user:342 alina
set user:145 lenuța
scan 0 MATCH user:1* COUNT 1000000
1) "0"
2) 1) "user:10:time-zone"
   2) "user:145"
   3) "user:112"
```

Valoarea lui COUNT să fie foarte mare pentru a extinde căutarea. Trebuie remarcat faptul că această comandă, fără COUNT, va returna prima înregistrare găsită. La `1)` va fi numărul slotului găsit, iar la `2)` este cheia găsită ce potrivește șablonul de căutare. În cazul în care `SCAN` nu mai găsește înregistrări, va returna valoarea zero la `1)`.

Căutarea fațetată:
- navigare;
- căutare după criterii multiple;
- filtrare după mai multe criterii.

Punctul principal ar fi că trebuie constituite seturi care să reflecte combinațiile de atribute care ar fi cerute la cerere. Astfel, vei putea folosi comanda `SINTER` pentru a realiza intersecții ale acestor seturi după câmpurile care prezintă profilul căutării. O strategie utilă de căutare prin realizarea de intersecții este să pornești de la setul cu cele mai puține valori pe care îl intersectezi cu următorul ș.a.m.d.
`SINTER nume_set1 nume_set2` va returna un subset al valorilor conținute în ambele seturi.

### Time complexity

Time complexity este o funcție a cardinalității setului (numărul elementelor din set) și numărul de seturi care sunt intersectate.
De exemplu, atunci când facem o căutare după două atribute, costul de a face un `SINTER` este dat de cardinalitatea celui mai mic set. Multiplicând cardinalitatea celui mai mic set (`SCARD`) cu numărul seturilor pe care le comparăm pentru a extrage elementelor comune, vom obține valoarea a ceea ce numim *time complexity*.

### Concluzii

Folosind metoda de căutare fațetată, factorii limitativ sunt:

- numărul de seturi care trebuie inspectate și comparate;
- distribuția datelor pentru valorile atributelor după care se constituie intersecția

## Hashed faceting

Sunt seturi care sunt create pentru a răspunde la căutări ale căror date știm că se află în respectiva interesecție (*compound indexes*). Sunt intersecții care răspund strict anumitor scenarii de căutare. Să presupunem că ai nevoie de un set care cuprinde id-urile unor elemente din alte indexuri, dar care în noul index reprezintă o anumită realitate/instanță/realitate.

```text
sadd cazcautare:234pd432p565mnjj34343434 10-triatlon-id543543 432-atletism-id43300
```

Putem coda numele cheii (setului) cu un *nume de domeniu* (`cazcautare`) urmat de o valoare hash care aparține strict doar cazului respectiv de căutare.

Totuși, pentru datele care se modifică destul de mult, această tehnică nu este prea utilă, pentru că ar trebui să modifici mereu hash-urile.

## Resurse

- [Faceted search | Wikipedia](https://en.wikipedia.org/wiki/Faceted_search)
- [Inverted index | Wikipedia](https://en.wikipedia.org/wiki/Inverted_index)
- [glob (programming) | WIkipedia](https://en.wikipedia.org/wiki/Glob_(programming))
