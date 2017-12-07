# Creare un container docker con express e mongo

## Installare Docker

Seguire le istruzioni sul sito di [docker.com](https://docs.docker.com/engine/installation/#supported-platforms)

## Introduzione

* un **Immagine** è un pacchetto che contiene tutto ciò che è necessario per eseguire un determinato software (incluso sorgenti, librerie, variabili d'ambiente e files di configurazione)
* un **Container** è un'istanza di un'Immagine, al momento dell'esecuzione. Viene eseguita in un ambiente isolato dall'elaboratore che la ospiuta, può accedere ai file e alle porte dell'elaboratore HOST solo se espressamente confiugurata allo scopo.

Per verificare se docker è installato e configurato correttamente, digitare:
``docker run hello-world``
Se il messaggio di risposta é:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
...(snipped)...
```
allora l'installazione è corretta e il sistema è configurato per la corretta esecuzione.

## Architettura di una Docker app con Docker-compose

### Introduzione

Prima di iniziare lo studio di una Docker App è necessario comprenderne la gerarchia:

1. Stack (Definisce l'interazione tra i vari Servizi)
2. Services (Definiscono come i Container si comportano in produzione)
3. Container (è il livello più basso).

Lo Studio inizia dai Container

### I Container

Il Container viene definito da un ``Dockerfile``.

Nel ``Dockerfile`` viene definito tutto ciò che deve essere presente nell'ambiente dell'applicazione.

Ad esempio, si potrà definire l'accesso alle interfacce di rete o ai dischi fissi.

Un esempio di ``Dockerfile`` è  quello riportato qui di seguito e si tratta di un ambiente di sviluppo Node.js

```
# Use an official Node runtime as a parent image
FROM node:8-alpine

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install from package.json
RUN npm install --production

# Make port 80 available to the world outside this container
EXPOSE 80

# Run app when application launches
CMD ["node", "index.js"]
```

Per creare l'immagine Docker si usa il seguente comando:

``docker build -t [nomeImmagine]``

> solitamente si usa vendorName/appName:1.0 .

### Servizi (configurazione in docker-compose)

La configurazione dei servizi in docker-compose viene effettuata all'interno del file ``docker-compose.yml`` 

Un esempio di configurazione basica è la seguente:

```
version: '3'
services: 
  app: 
    image: foo/bar:1.0 //il vendorName/appName/version che era stato indicato in fase di build
    env_file:
    - .env //ricerca un file di configurazione che abbia l'estensione .env
    ports:
    - 8080:8080
```
Poi si crea il file ``.dockerignore``

```
cat .dockerignore
.env
node_modules/
```
Poi si crea il file ``.gitignore``

```
cat .gitignore
.env
node_modules/
```

Per eseguire l'immagine:

Digitare ``docker-compose up`` dalla directory del progetto

> docker-compose a questo punto scaricherà l'immagine di node, costruirà l'immagine dell'applicazione ed avvierà i servizi indicati nel file ``.yml``

Per interrompere l'esecuzione del container, digitare:

``docker stop [containerID]``
oppure ``docker-compose down`` sempre dalla directory del progetto


