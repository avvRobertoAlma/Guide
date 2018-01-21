# Nginx

## Installazione

1. Aggiornare il sistema ``sudo apt-get update && sudo apt-get upgrade -y``
2. Installare nginx ``sudo apt-get install nginx -y``
3. Avviare nginx ``sudo systemctl start nginx``
4. Avviare nginx all'avvio del sistema ``sudo systemctl enable nginxy``

> Per verificare se nginx è attivo, digitare ``sudo systemctl status nginx``

Per verificare che l'installazione sia andata a buon fine, digitare nella barra degli indirizzi del browser, l'indirizzo IP del server. Comparirà un messaggio di benvenuto.



## Configurazione BASICA

### Nginx come reverse proxy

[Guida di riferimento] (https://medium.com/@utkarsh_verma/configure-nginx-as-a-web-server-and-reverse-proxy-for-nodejs-application-on-aws-ubuntu-16-04-server-872922e21d38) su medium.

- Rimuovere e aggiungere un file di configurazione di default

```javascript
$ sudo rm /etc/nginx/sites-available/default
$ sudo vi /etc/nginx/sites-available/default
```

- Aggiungere il seguente contenuto:

```javascript
server {
    listen 80;
    server_name your_domain.com;
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
     }
}
```
> In questo modo abbiamo configurato un proxy in modo che le richieste dirette alla porta 80 siano automaticamente reindirizzate alla porta 3000 (ossia quella in cui sarà attiva la nostra app express).

- Testare la configurazione di nginx: ``sudo nginx -t``

- Riavviare nginx ``sudo /etc/init.d/nginx reload``

## Nginx come reverse proxy in caso di unico dominio collegato a più applicazioni express

Sostanzialmente, seguire l'intera guida del paragrafo precedente, con i seguenti accorgimenti:

- Aggiungere altre ``location`` all'interno del medesimo ``server``

- Utilizzare la direttiva ``rewrite``

```javascript
location /utils {
 	rewrite ^/utils/(.*) /$1 break;

```
Per dubbi seguire la guida di [soheilhy](https://gist.github.com/soheilhy/8b94347ff8336d971ad0).

Il concetto è che occorre riscrivere gli indirizzi. 
Nginx invia le richieste a ``/utils`` mentre le app Express le gestiscono a partire da ``/``. 

> N.B. rispetto alla guida ho aggiunto un ``/`` alla fine di utils perché a tutte le richieste veniva aggiunto un ``/`` aggiuntivo.  

## Nginx come reverse proxy in caso di più domini collegati ad un unico indirizzo IP

