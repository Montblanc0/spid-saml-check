# *SPID SAML Check (fork)*

*SPID SAML Check* è una suita applicativa che fornisce diversi strumenti ai Service Provider SPID, utili per ispezionare le request di autenticazione SAML inviate all'Identity Provider, verificare la correttezza del metadata e inviare response personalizzate al Service Provider. SPID SAML Check è costituito da:
 - [spid-sp-test](https://github.com/italia/spid-sp-test), per eseguire i test di conformità alle specifiche SPID
 - una web application (_`spid-validator`_) che fornisce una interfaccia grafica per l'esecuzione dei test e l'invio delle response
 - una web application (_`spid-demo`_) che implementa un IdP di test per eseguire demo di autenticazione
 - un'estensione per Google *Chrome* che permette di intercettare le richieste SAML (deprecata)

Il [repository originale](https://github.com/italia/spid-saml-check) è sviluppato e mantenuto da [AgID - Agenzia per l'Italia Digitale](https://www.agid.gov.it).

## Fork changes

Questo fork aggiunge il [flag trustAnyCert](https://github.com/Montblanc0/spid-saml-check/blob/ec56c667b602fe49ee645cf77cebd48ff73b46b8/spid-validator/config/server.json#L8) in `spid-validator/config/server.json` per validare qualsiasi certificato SSL.

Funziona solo quando anche `useHttps: true`.

Imposta `"trustAnyCert": true` prima della build per abilitare questa funzione.

### Example config

```json
{
  "host": "https://my.domain.com",
  "port": 8443,
  "useProxy": false,
  "useHttps": true,
  "httpsPrivateKey": "./config/spid-saml-check.key",
  "httpsCertificate": "./config/spid-saml-check.crt",
  "trustAnyCert": true
}

```

Utilizzare a proprio rischio.


## Come costruire l'immagine Docker ed eseguire il container

```
# 1. Clone del repository
git clone https://github.com/Montblanc0/spid-saml-check.git

# 2. Aggiornamento configurazione e certificati
modifica il file di configurazione
 - spid-validator/config/server.json

crea chiave privata e certificato per il protocollo https
 - spid-validator/config/spid-saml-check.key
 - spid-validator/config/spid-saml-check.crt

configura chiave privata e certificato per la firma delle response nei seguenti file
 - spid-validator/config/idp.json
 - spid-validator/config/idp_demo.json

# 3. Esecuzione della build
cd spid-saml-check
docker compose up -d
```

Un volta terminato il processo di build dell'immagine (che potrebbe durare
diversi minuti), il validator sarà immediatamente disponibile al seguente indirizzo:

https://localhost:8443


Per avviare il container dopo averlo stoppato

```
docker start spid-saml-check
```

## Come usare *SPID Validator*

L'applicazione spid-validator, se invocata *così com'è*, effettua un collaudo formale del solo metadata SAML del SP.
Per utilizzare l'intero set di controlli (il vero e proprio *SPID Validator*), va scaricato il metadata SAML disponibile all'indirizzo https://localhost:8443/metadata.xml installandolo come un nuovo IdP presso la propria implementazione di SP.
Usato in questo modo, lo *SPID Validator* può essere invocato come un IdP dal proprio SP, presentando un insieme di più di 300 controlli individuali, divisi in 7 famiglie:
 * 4 famiglie per la convalida formale del **metadata** SP (come sopra descritto);
 * 3 famiglie per la convalida formale delle **request** SAML;
 * 1 famiglia (111 controlli) per la convalida *interattiva* del comportamento del SP alle **response** dall'IdP.

Per usare lo *SPID Validator* pertanto occorre inviare una richiesta di autenticazione dal proprio SP e autenticarsi al *Validator* con le credenziali __validator__ / __validator__


### Passi per l'utilizzo

- Copia il metadata di spid-validator presso il metadata store del tuo SP.
  I metadata di spid-validator possono essere scaricati qui: [https://localhost:8443/metadata.xml](https://localhost:8443/metadata.xml)
  ````
  wget https://localhost:8443/metadata.xml -O /path/to/your/sp/metadata/folder/spid-saml-check-metadata.xml
  ````

- Connettendoti al tuo SP invia una richiesta di autenticazione, questa ti porterà alla schermata di autenticazione di spid-validator. 

  <img src="doc/img/login.png" width="500" alt="login page" />

- Inserisci come credenziali __validator__/ __validator__
- Al primo accesso potrai vedere la AuthnRequest appena inviata dal tuo SP

  <img src="doc/img/2.png" width="500" alt="authn request page" />

- Clicca su Metadata e scarica i metadata del tuo SP per la validazione.<br/>
  **Attenzione**: Se il tuo SP non è disponibile mediante un FQDN, usa l'ip dell'host Docker e non "localhost"!
  
  <img src="doc/img/3.png" width="500" alt="metadata download page" />

- Adesso potrai eseguire tutti i test in ordine di apparizione: Metadata, Request and Response.
- Per testare una Response, dalla sezione Response, seleziona dal menu a tendina il test che desideri eseguire, e marcalo come eseguito e con successo se lo ritieni tale.

  <img src="doc/img/4a.png" width="500" alt="response select page" />


## Come usare *SPID Demo*

L'applicazione spid-demo viene eseguita all'indirizzo: [https://localhost:8443/demo](https://localhost:8443/demo)

<img src="doc/img/demo_idp_index.png" width="500" alt="demo index page" />
   
   
Gli utenti di test di spid-demo che è possibile utilizzare sono elencati su: [https://localhost:8443/demo/users](https://localhost:8443/demo/users)

<img src="doc/img/demo_idp_users.png" width="500" alt="demo users page" />


### Passi per l'utilizzo

- Copia il metadata di spid-demo presso il metadata store del tuo SP.
  Il metadata di spid-demo può essere scaricato qui: [https://localhost:8443/demo/metadata.xml](https://localhost:8443/demo/metadata.xml)
  ````
  wget https://localhost:8443/demo/metadata.xml -O /path/to/your/sp/metadata/folder/spid-demo-metadata.xml
  ````

- Vai su https://localhost:8443 e registra il metadata del tuo SP tramite l'interfaccia di spid-validator.<br/>
  Dovresti poter accedere tramite una pagina di login come mostrato nella figura seguente
  
  <img src="doc/img/login.png" width="500" alt="login page" />
  
  
- Inserisci come credenziali __validator__/ __validator__

- Clicca su Metadata e scarica i metadata del tuo SP.<br/>
  **Attenzione**: Se il tuo SP non è disponibile mediante un FQDN, usa l'ip dell'host Docker e non "localhost"!
  
  <img src="doc/img/demo_download_metadata_sp.png" width="500" alt="download metadata page" />
  

- Invia una richiesta di autenticazione dal tuo SP all'IdP spid-demo per utilizzare l'ambiente demo

  <img src="doc/img/demo_idp.png" width="500" alt="demo idp" />
