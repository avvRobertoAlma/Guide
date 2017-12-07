# Introduzione a NODE

## Node.js

### Lanciare i comandi di NODE

Ci sono tre alternative per lanciare i comandi di **node**:

- REPL (digitare in una finestra di terminale ``node``)
- Eval (digitare ``node -e " /comandi javascript inline/ "``
- Lanciare codice node da un file, digitando ``node nomefile.js`` (sempre se il file si trova nella stessa cartella attiva, sennò occorre il PATH)

> N.B. Se in una cartella c'è un file ``index.js``, sarà sufficiente dare il PATH della cartella in cui è inserito (es. node ./app è equivalente a node ./app/index.js)

### Variabili globali di NODE

Nel Browser abbiamo l'oggetto globale ``window``, in NODE, invece, abbiamo differenti oggetti globali che ora esamineremo.

#### global

Esiste una variabile chiamata ``global`` che è accessibile da ogni script di NODE e si riferisce all'oggetto globale.

``global`` ha varie proprietà. Le proprietà di primo livello sono accessibili anche senza il prefisso ``global``. Ad esempio ``process`` è equivalente a ``global.process``.

### I processi di NODE

Ogni script di NODE in esecuzione è un ``process``.

- possiamo accedere alle variabili di ambiente tramite l'attributo ``env`` (digitando ``process.env``)
- in particolare si utilizza settare ``NODE_ENV`` a development o production (si tratta di una convenzione, ma spesso recepita da alcune librerie o framework)
- per accedere agli argomenti si utilizza ``process.argv`` che restituisce un *array* di argomenti
- per uscire da un processo si utilizza ``process.exit()`` (si può specificare un'uscita con errore utilizzando 1, e invece un'uscita con successo con 0)

### module.exports and require 

Obsoleti e sostituiti da ``import`` e ``export``.

```javascript
//file utility.js
module.exports = function(numbersToSum) {
  let sum = 0, 
    i = 0, 
    l = numbersToSum.length;
    while (i < l) {
        sum += numbersToSum[i++]
    }
    return sum
}
```
Si tratta di un modulo consistente in una funzione.

Il programma principale può accedere a questa funzione utilizzando la seguente istruzione:

``const sum = require('./utility.js);``

Attenzione tutto il codice presente in un file dove è presente un ``module.exports`` è eseguito una volta soltanto.

Se io inserisco ``console.log('called')`` nel file ``utility.js`` e importo il modulo in due differenti file, l'istruzione di cui sopra (poiché presente al di fuori del modulo) sarà eseguita una volta soltanto.

#### Patterns per module exports

- esportare una funzione ``module.exports = function () {}``
- esportare un oggetto ``module.exports = {}``
- esportare più funzioni ``module.exports.methodA = function() {}``
- esportare più oggetti ``module.exports.objA = {}``

Possiamo anche esportare un unico oggetto comprendente all'interno più metodi

```javascript
module.exports = {
  sayHelloInEnglish() {
    return 'Hello'
  }

  sayHelloInSwedish() {
    return 'Hej'
  }

  sayHelloInTatar() {
    return 'Isänme'
  }
}
```

Per importarli con ``require()``

```javascript
var saluti = require('./greetings.js');

console.log(saluti.sayHelloInEnglish());
```

### Moduli 'core'

Si possono utilizzare senza dover installare nulla tramite npm.

Eccoli:

- fs: module to work with the file system, files and folders
- path: module to parse file system paths across platforms
- querystring: module to parse query string data
net: module to work with networking for various protocols
- stream: module to work with data streams
- events: module to implement event emitters (Node observer pattern)
- child_process: module to spawn external processes
- os: module to access OS-level information including platform, number of CPUs, memory, uptime, etc.
- url: module to parse URLs
http: module to make requests (client) and accept requests (server)
- https: module to do the same as http only for HTTPS
- util: various utilities including promosify which turns any standard Node core method into a promise-base API
- assert: module to perform assertion based testing
- crypto: module to encrypt and hash information

#### Approfondimento su FS

FS si utilizza per eseguire operazioni sui file. Ci sono metodi sincroni e asincroni. Si preferiscono gli asincroni, più che altro per non interrompere l'esecuzione del programma.

Leggere da un file

```javascript
const fs = require('fs')
const path = require('path')
fs.readFile(path.join(__dirname, '/data/customers.csv'), {encoding: 'utf-8'}, function (error, data) {
  if (error) return console.error(error)
  console.log(data)
})
```
Scrivere su un file
```javascript
const fs = require('fs')
fs.writeFile('message.txt', 'Hello World!', function (error) {
  if (error) return console.error(error)
  console.log('Writing is done.')
})

```

#### Approfondimento su path

Il problema è che in Windows i path sono separati utilizzando \, mentre nei sistemi unix si utilizza /.

Con ``path.join()`` si creano percorsi che sono indipendenti dal sistema operativo utilizzato.

Esempi

```javascript
const path = require('path')
const server = require(path.join('app', 'server.js'))
```
Abbiamo creato un collegamento relativo a ``app/server.js``

```javascript
const path = require('path')
const server = require(path.join(__dirname, 'app', 'server.js')) 
```
Abbiamo creato un collegamento assoluto a ``app/server.js``

#### Event Emitters

In primo luogo, dichiarare la const ``EventEmitter`` 
``const EventEmitter = require('events');``

In secondo luogo, creiamo una ``class Job`` che estende la classe ``EventEmitter``

Poi si crea un'istanza della classe che abbiamo appena creato.

``job = new Job()``

A questo punto possiamo impostare un *listener* che resterà in attesa di un determinato evento, al verificarsi del quale eseguirà la funzione di callback.

```javascript
const EventEmitter = require('events')

class Job extends EventEmitter {}
job = new Job()

job.on('done', function(timeDone){
  console.log('Job was pronounced done at', timeDone)
})

job.emit('done', new Date())
job.removeAllListeners()  // remove  all observers
```

Se volessi gestire più *event listeners*. Ogni volta che effettuo l'*emitting* (con ``.emit()``), tutti i listener che sono in attesa di quell'evento, verranno automaticamente eseguiti.

```javascript
const EventEmitter = require('events')

class Emitter extends EventEmitter {}
emitter = new Emitter()

emitter.on('knock', function() {
  console.log('Who\'s there?')
})

emitter.on('knock', function() {
  console.log('Go away!')
})

emitter.emit('knock')
emitter.emit('knock')
```
Se utilizzassi ``emitter.once(eventName, eventHandler)``, la funzione *eventHandler* verrebbe eseguita una volta soltanto, a prescindere dal numero di verificazioni dell'evento.

### HTTP Client

Per utilizzare il client HTTP di NODE, occorre importare il modulo ``'http'``.
Per cui dichiareremo una variabile ``http`` che richiama il modulo HTTP.

#### richieste GET

Possiamo, quindi, utilizzare il metodo ``.get`` che accetta come argomento un ``url`` e una funzione di callback (che sarà costituita dalla risposta). 

```javascript
onst http = require('http')
const url = 'http://nodeprogram.com'
http.get(url, (response) => { }
```
Nella funzione di callback potremmo utilizzare differenti *event listeners* (ad esempio uno per l'evento ``data`` e uno per l'evento ``end``).

Possiamo anche utilizzare un *listener* specifico per il metodo ``http.get``, per captare eventuali errori, come da esempio seguente:

```javascript
http.get(url, (response) =>{}).on('error', (error)=>{
console.log(error);
});

```
#### richieste POST

Il modulo HTTP di NODE ci consente di gestire anche le richieste.

In primo luogo, è utile creare un oggetto ``options`` che conterrà tutta la configurazione della nostra richiesta (es. POST o GET, gli headers ecc.)

```javascript
const options = {
  hostname: 'mockbin.com',
  port: 80,
  path: '/request?foo=bar&foo=baz',
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Content-Length': Buffer.byteLength(postData)
  }
}
```
A questo punto, gestiamo la richiesta, dichiarando una variabile ``req`` con i suoi *event listeners*

```javascript
const req = http.request(options, (res) => {
  res.on('data', (chunk) => {
    console.log(`BODY: ${chunk}`)
  })
  res.on('end', () => {
    console.log('No more data in response.')
  })
})

req.on('error', (e) => {
  console.error(`problem with request: ${e.message}`)
})
```
A questo punto, possiamo inviare la richiesta con ``req.write([requestBody])`` e ``req.end()``

### HTTP Server

Creiamo una variabile ``port`` (3000)

Invochiamo il metodo ``http.createServer(callback)``

Nella funzione di callback indichiamo come argomenti ``request`` e ``response``, con ``response``, in particolare, possiamo inviare dati al client.

> La funzione sarà eseguita ogni volta che viene effettuata una richiesta al server

In particolare dobbiamo impostare l'header e lo status e il testo che invieremo al client.

```javascript
http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'})
  res.end('Hello World\n')
}).listen(port)
```

In particolare,  

- ``res.writeHead()``, serve per impostare il codice di stato della risposta (es. 200) e l'header (es. ``{'Content-Type': 'text-plain'}``)
- `res.end(string)` serve per inviare la risposta al client (una stringa che sarà visualizzata sul browser).
- ``listen(port)``, serve per abilitare il server ad accettare richieste.

Per gestire richieste POST, in NODE, si aggiunge un'istruzione condizionale 

```javascript
if(req.method == 'POST'){
	//do something
	request.on('end', function(){
		res.end('Accepted Body')
		});
}
```

Per quanto riguarda i metodi della risposta, finora abbiamo utilizzato solo ``res.end()``, ma avremmo potuto scomporre il tutto utilizzando:

- ``.statusCode`` che può essere impostato ad un numero di stato HTTP (es. 200)
- ``.write()`` che aggiunge dati alla risposta;
- ``.end()`` che termina la risposta

### Esempio NODE RESTful API

```javascript
var http = require('http')
var util = require('util')
var querystring = require('querystring')
var client = require('mongodb').MongoClient

var uri = process.env.MONGOLAB_URI || 'mongodb://@127.0.0.1:27017/messages'
//MONGOLAB_URI=mongodb://user:pass@server.mongohq.com:port/db_name

client.connect(uri, function(error, db) {
  if (error) return console.error(error)
  var collection = db.collection('messages')
  var app = http.createServer(function (request, response) {
    var origin = (request.headers.origin || '*')
    if (request.method == 'OPTIONS') {
      response.writeHead('204', 'No Content', {
        'Access-Control-Allow-Origin': origin,
        'Access-Control-Allow-Methods':
          'GET, POST, PUT, DELETE, OPTIONS',
        'Access-Control-Allow-Headers': 'content-type, accept',
        'Access-Control-Max-Age': 10, // In seconds
        'Content-Length': 0
      })
      response.end()
    } else if (request.method === 'GET' && request.url === '/messages') {
      collection.find().toArray(function(error,results) {
        if (error) return console.error(error)
        var body = JSON.stringify(results)
        response.writeHead(200,{
          'Access-Control-Allow-Origin': origin,
          'Content-Type':'text/plain',
          'Content-Length':body.length
        })
        console.log('LIST OF OBJECTS: ')
        console.dir(results)
        response.end(body)
      })
    } else if (request.method === 'POST' && request.url === '/messages') {
      request.on('data', function(data) {
        console.log('RECEIVED DATA:')
        console.log(data.toString('utf-8'))
        collection.insert(JSON.parse(data.toString('utf-8')),
        {safe:true}, function(error, obj) {
          if (error) return console.error(error)
          console.log('OBJECT IS SAVED: ')
          console.log(JSON.stringify(obj))
          var body = JSON.stringify(obj)
          response.writeHead(200,{
            'Access-Control-Allow-Origin': origin,
            'Content-Type':'text/plain',
            'Content-Length':body.length
          })
          response.end(body)
        })
      })
    } else {
    	response.end('Supported endpoints: \n/messages\n/messages')
    }
  })
  var port = process.env.PORT || 1337
  app.listen(port)
})
```


## Express

### Introduzione

Framework per NODE che consente di creare in pochi semplici passi un web server.

Il PLUS di Express è l'utilizzo dei *Middlewares*, ossia dei plugin che consentono di aggiungere ulteriori funzionalità ad Express. 

#### Utilizzare express-generator

Installare ``express-generator`` globalmente.

Utilizzare la CLI per creare rapidamente nuove applicazioni Express.

```javascript
express new-app
cd new-app
npm install
npm start
```
Semplice codice per creare un nuovo progetto ``new-app``

#### Struttura di un progetto EXPRESS

La best practice per organizzare un progetto EXPRESS è la seguente:

- `app.js`: main file, houses the embedded server and application logic
- `/public`: contains static files to be served by the embedded server
- `/routes`: houses custom routing for the REST API servers (not needed for a web app)
- `/routes/users.js`: endpoint/routes for resources such as users
- `/views`: contains templates that can be processed by a template engine (not needed for REST APIs)
- `/package.json`: project manifest
/www: boot up script folder

### I Middleware

Sono i plugin di Express. 
Sono una serie di unità di elaborazione unite insieme, dove l'output di una è l'input di un'altra.

```javascript
function(args, next) {
  // ... Run some code
  next(output) // Error or real output
}
```
Si tratta della struttura di base di un middleware.

La catena dei middleware può essere così sintetizzata

``request->middleware1->middleware2->...middlewareN->route->response
``
La richiesta e la risposta sono agli estremi della catena.

Per utilizzare i middleware, si ricorre al *middleware manager* nativo di EXPRESS. 

Di conseguenza, si può applicare direttamente il middleware utilizzando ``app.use()``.

> **Attenzione** I middleware sono eseguiti nell'ordine che viene specificato nel file ``app.js``.

Ci sono due tipologie di middleware:

- moduli npm utilizzati con ``app.use()`` (es. ``app.use(bodyParser.json())``)
- moduli personalizzati (es. ``app.use((req, res, next)=>{})``)

> **L'importanza di ``next``** ``next()`` viene invocato proprio in ragione dell'inserimento del middlware in una catena che si concluderà con la risposta.

#### Creare un middleware

Sostanzialmente, il middleware è una funzione.

Può essere salvato come una variabile (e sarebbe l'approccio migliore, nell'ottica di scrivere codice modulare).

```javascript
const middleware = (request, response, next) => {
  // Modify request or response
  // Execute the callback when done
  next()
}
app.use(middleware)

```
oppure, può essere creato direttamente all'interno della chiamata ad ``app.use()``, utilizzando una funzione anonima.

```javascript
app.use((request, response, next) => {
  // Modify request or response
  // Execute the callback when done
  next()
})
```

##### References

Va premesso che ``request`` è sempre lo stesso oggetto nel ciclo di vita di una qualsiasi richiesta ad un server di EXPRESS. Ciò consente di dare vita ad un buon pattern in cui si passano dati da un middleware ad un altro e ad una funzione che gestisce la richiesta.

Per esempio, ci si potrebbe connettere ad un database in un middleware e renderlo disponibile in tutti i successivi middleware e nelle *routes*.

```javascript
app.use(function (request, response, next) {  
  DatabaseClient.connect(URI, (err, db) => {
    // error handling
    request.database = db    
    next()
  })
})
```

In questo middleware, ``database`` è disponibile nell'oggetto ``request`` ed è possibile eseguire *query* come ricercare un oggetto con il suo id

```javascript
app.use((req, res, next) => {
  req.database.collection('apps').findOne({appId: req.query.appId}, (err, app) => {
    // error handling
    req.app = app
    next()
  })
})
```

#### BodyParser

Si tratta di un Middleware che consente di accedere al *body* di una richiesta HTTP

> in express-generator il bodyParser è già installato e configurato di default

Per utilizzarlo è sufficiente importarlo con ``const bodyParser = require('body-parser')`` 

e successivamente applicare il ``json`` che è principalmente utilizzato dai client REST

``app.use(bodyParser.json())``

Volendo, si potrebbe anche utilizzare un middleware per processare i ``form`` inviati tramite l'attributo ``action`` (utilizzano ``application/x-www-form-urlencoded``)

``app.use(bodyParser.urlencoded({extended: false}))``

> se fosse necessario gestire oggetti complessi (i.e. file) o array di dati, dovremmo utilizzare la sintassi ``extended: true``

#### Elenco di utili Express Middleware

- `body-parser` request payload
- `compression` gzip
- `connect-timeout` set request timeout
- `cookie-parser` Cookies
- `cookie-session Session via Cookies store
- `csurf` CSRF
- `errorhandler` error handler
- `express-session` session via in-memory or other store
- `method-override` HTTP method override
- `morgan` server logs
- `response-time`
- `serve-favicon` favicon
- `serve-index`
- `serve-static` static content
- `vhost`: virtual host
- `cookies and keygrip`: analogous to cookieParser
- `raw-body`: raw request body
- `connect-multiparty`, 
- `connectbusboy`: file upload
- `qs`: analogous to query
- `st, connect-static` analogous to staticCache
- `express-validator`: validation
- `less`: LESS CSS
- `passport`: authentication library
- `helmet`: security headers
- `connect-cors`: CORS
- `connect-redis`

### REST API Routing

#### Introduzione

Occorre creare degli *endpoint* (es /fatture o /clienti), e gestire le quattro operazioni fondamentali su ogni endpoint.

> Si parla di **CRUD** (Create, Retrieve, Update, Delete).

In EXPRESS, una *route* basica può essere costituita come nell'esempio seguente:

```javascript
app.get('/', (req, res) => {
	res.send('Ciao a tutti!')
})
```
``app.get`` si riferisce al metodo HTTP GET. Il primo argomento è una string (ossia l'endpoint), il secondo è un *handler* che accetta oggetti di tipo richiesta e risposta. In questo caso inviamo una risposta contenente una stringa.

Possiamo anche suddividere le *route* in più file, come da esempio seguente:

```javascript
const {homePage, getUsers} = require('./routes')

app.get('/', homePage)
app.get('/users', getUsers)
```

Gli altri metodi che EXPRESS ci consente di definire tramite ``app`` sono:

- `.post` (creazione di nuove entità)
- `.put` (modifica totale o parziale)
- `.patch` (modifica parziale)
- `.delete` (eliminazione di una entità)
- `.head` (richieste GET senza il BODY)
- `.options` (identificare i metodi ammessi)

Segue un esempio basico di API RESTful per un endpoint *'profile'*

```javascript
const express = require('express') 
const app = express() 

const profile = {
  username: 'azat',
  email: '[reducted]',
  url: 'http://azat.co'
}
app.get('/profile', (req, res)=>{
  res.send(profile)
})
app.post('/profile', (req, res) => {
  profile = req.body
  res.sendStatus(201)
})
app.put('/profile', (req, res)=>{
  Object.assign(profile, req.body)
  res.sendStatus(204)
})
app.delete('/profile', (req, res)=>{
  profile ={}
  res.sendStatus(204)
})

app.listen(3000)
```

#### L'oggetto richiesta

L'oggetto `request` ha più proprietà di quello nativo di NODE.

- `request.params`: URL parameters
- `request.query`: query string parameters
- `request.route`: current route as a string
- `request.cookies`: cookies, requires cookieParser
- `request.signedCookies`: signed cookies, requires cookie-parser
- `request.body`: body/payload, requires body-parser
- `request.headers`: headers

In particolare, `request.headers` ha una serie di scorciatoie.

Ad esempio:

- `request.secure` verifica se il protocollo è HTTPS
- `request.protocol` restituisce il protocollo HTTP
- `request.path` restituisce il path dell'URL
- `request.ip` restituisce l'IP della richiesta

#### L'oggetto risposta

L'oggetto `response` è il secondo argomento nel callback *handler* della route di EXPRESS

Ha dei metodi aggiuntivi rispetto a quelli classici di `http` (statusCode, writeHead, end, write)

Elenco dei metodi aggiuntivi:

- `response.redirect(url)`: redirect request
- `response.send(data)`: send response
- `response.json(data)`: send JSON and force proper headers
- `response.sendfile(path, options, callback)`: send a file
- `response.render(templateName, locals, callback)`: render a template
- `response.locals`: pass data to template
- `response.status`: specifica il codice di stato HTTP della risposta

##### Inviare una risposta

```javascript
app.get('/', (req, res) => {
	res.send('Ciao a tutti!')
})
```
Si tratta di una risposta basica, il cui contenuto è una stringa.

In questo caso il content-type è gestito automaticamente da EXPRESS in funzione del contenuto della risposta (che in questo caso, essendo una stringa, sarà `text/plain`)

Si può pre-impostare il content-type utilizzando `response.set('Content-Type','text/plain')`.

Per inviare una risposta vuota (tipicamente a fronte di un errore) si può utilizzare:

`response.status(404).end()`

##### Accedere ai parametri dell'URL

Per accedere ai parametri dell'URL si utilizza ``req.params``.
Ad esempio, dato un url del seguente tenore: GET /users/1234 per accedere a quell'*1234* sarà necessario:

1. gestire l'URI con la seguente sintassi ``app.get('/users/:id')``
2. accedere all'id utilizzando ``req.params.id``.

In presenza di più parametri es. GET /users/1234/invoices/012017, si potrà accedere a id utente e id fattura utilizzando la seguente sintassi:

1. ``app.get('/users/:id/invoices/:idInvoice')``
2. accedere ai valori di interesse con ``req.params.id`` e ``req.params.idInvoice``.

##### Accedere alle query

EXPRESS ha un proprio parser delle query. Si può accedere alle query mediante ``req.query.name``, dove ``name`` è la chiave del valore in una stringa di ricerca.

Esempio, un URL del tipo ``http://www.iusinaction.com/search?term=cassazione`` ha una query composta da una key (term) e un valore (cassazione).
Posso accedere al valore (cassazione) con ``req.query.term``.

## MongoDB e Mongoose

Mongoose è una libreria ODM (*Object Document Mapping*) per Node.js e MongoDB.

Si crea un'astrazione dal Database e l'applicazione interagisce solo con gli oggetti e i loro metodi.

### Installazione

``npm install mongoose``

Per accedere alla libreria, utilizzare:

``const mongoose = require('mongoose');``

Per la connessione al database, utilizzare il metodo ``.connect``

```javascript
mongoose.connect('mongodb://localhost:27017/test')
```
### Schema

Uno Schema di Mongoose è un oggetto che contiene informazioni sulle proprietà di un documento e su eventuali regole di validazione.

Gli Schema rappresentano una sorta di *scheletro* di un oggetto del database.

Ad esempio:
```javascript
import mongoose, { Schema } from 'mongoose'

const personSchema = new Schema({
	name: {
		type: String
	},
	surname: {
		type: String
	},
	age: {
		type: Number
	}
})

```
Negli Schema si possono inserire le seguenti tipologie di dati:

- ``String``: una sequenza di caratteri;
- ``Number``: un numero fino a 253;
- ``Array``: un array di dati;
- ``Boolean``: un valore *true* o *false*;
- ``Buffer``: un file binario di Node.js (es. PDF, Immagini ecc.);
- ``Date``: una data espressa in formato ISO es. 2017-11-25T18:23:23.009Z;
- ``Schema.Types.ObjectId``: una stringa esadecimale tipica di MongoDB (es. 52dafa354bd71b30fa12c441);
- ``Schema.Types.Mixed``: qualsiasi tipo di dato;

> In particolare ``ObjectId`` viene inserito automaticamente come ``_id`` univoco di un oggetto del DB, se non ne viene inserito uno con il metodo ``save``.

Nello Schema, si possono inserire anche dei metodi setter e getter. Ad esempio:

```javascript
const articleSchema = new mongoose.Schema({
	slug: {
		type: String,
		set: function(slug){
			return slug.toLowerCase();
			}
	}
})
```
In questo caso, al momento del salvataggio la stringa inserita viene convertita in caratteri minuscoli.

### Model

Per compilare uno Schema in un modello, si utilizza ``mongoose.model([name of model], [Schema]``

es. ``let Article = mongoose.model('Article', articleSchema)``

I modelli sono utilizzati per creare documenti di Mongoose. 

Ad esempio per creare un nuovo articolo, utilizziamo la seguente sintassi:

``let icoArticle = new Article({title: 'How to launch an Ico'})``

#### metodi dei Modelli di Mongoose

- ``Model.create(data, [callback (error, doc)])`` crea un nuovo documento di Mongoose e lo salva nel database;
- ``Model.remove(query, [callback(error)]`` rimuove il documento che soddisfa la query;
- ``Model.find(query,[fields],[options], [callback(error)]`` trova i documenti che soddisfano le query (si possono selezionare i campi specifici e utilizzare opzioni);
- ``Model.update(query,update,[options], [callback(error)]`` aggiorna documenti Mongoose;
- ``Model.populate(docs, options, [callback(error, doc)])`` popola documenti utilizzando referenze ad altre *collections*;
- ``Model.findOne(query,[fields],[options], [callback(error, doc)]`` trova il primo documento che soddisfa la query;
- ``Model.findById(id, [fields], [options], [callback(error, doc)])``: trova il primo documento il cui _id coincide con l'id desiderato;

Altri metodi sono disponibili sulla documentazione ufficiale di [Mongoose](http://mongoosejs.com/docs/api.html#model-js)

#### metodi personalizzati

Con Mongoose è possibile aggiungere metodi personalizzati.

Prima occorre definire il metodo personalizzato es. **buy()**.

```javascript
articleSchema.method({
	buy: function(quantity, customer, callback) {
		var articleToPurchase = this
		// creare un ordine di acquisto e inviare una fattura al cliente
		return callback(results)
	}
});
```
A questo punto si potrà utilizzare il metodo ``buy()``.

> I metodi personalizzati devono essere definiti prima che sia compilato il modello.


