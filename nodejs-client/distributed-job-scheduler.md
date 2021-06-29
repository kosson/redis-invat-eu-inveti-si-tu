# Distributed job scheduler

Un programator de joburi (*job scheduler*) va funcționa în contextul unui pool de instanțe Node. Fiecare instanță Node.js va verifica la un anumit interval dacă are de rezolvat vreun job apărut. Joburile vor fi instroduse într-un *Sorted Set* care va avea drept *score* un timestamp UNIX la nivel de milisecundă. Valoarea va fi un string care descrie jobul care trebuie făcut.

Fiecare aplicație care se conectează la Redis are propria conexiune privată cu serverul. Pentru realizarea unui jeb scheduler va trebui să ambalăm comenzile către server în `MULTI` și `EXEC`. În momentul în care Redis întâlnește un `MULTI`, nu va emite comenzi până când nu este întâlnit un `EXEC` care să-i corespundă. Astfel, sunt eliminate *racing conditions*.

```javascript
const redis = require('redis').createClient();
const JOBS = 'jobs'; // Sorted Set care va gestiona joburile

//simularea adăugării joburilor
redis.zadd(JOBS, Date.now() + 5 * 1000, 'trimite mail lui Ion'); // primul job
redis.zadd(JOBS, Date.now() + 10 * 1000, 'trimite mail lui Ana');// al doilea job

setInterval(() => {
  let now = Date.now(); // setează baza de timp
  // conceptul de operare este similar DB transactions
  redis.multi()
    .zrangebyscore(JOBS, 0, now) // get jobs până în acest moment (o secundă)
    .zremrangebyscore(JOBS, 0, now) // delete jobs până în acest moment (o secundă)
    .exec((error, data) => {
      if (error) console.error;
      let jobList = data[0];
      console.log('jobs', jobList.length ? jobList : 'N/A', process.pid); // execută joburile
    });
}, 1 * 1000);
```

Nu poți lua rezultatul unei comenzi și să-l introduci ca input al următoarei.
