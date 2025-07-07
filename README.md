**Per le istruzioni in Italiano, cliccare [qui](README.it.md).**

# *SPID SAML Check (fork)*

*SPID SAML Check* is an application suite that provides some tools for Service Providers, useful for inspecting requests shipped to an Identity Provider, checking metadata compliance and sending custom responses back to Service Provider. It includes:
 - [spid-sp-test](https://github.com/italia/spid-sp-test) to check the SPID specifications compliance
 - a web application (_`spid-validator`_) that provides an easy to use interface
 - a web application (_`spid-demo`_) that acts as a test IdP for demo purpose
 - an extension for Google Chrome that intercepts the request (deprecated)

The original [*SPID SAML Check*](https://github.com/italia/spid-saml-check) has been developed and is maintained by [AgID - Agenzia per l'Italia Digitale](https://www.agid.gov.it).

## Fork changes

This fork adds a [trustAnyCert flag](https://github.com/Montblanc0/spid-saml-check/blob/ec56c667b602fe49ee645cf77cebd48ff73b46b8/spid-validator/config/server.json#L8) in `spid-validator/config/server.json` to trust all SSL certificates.

It only works when `useHttps: true`.

Set `"trustAnyCert": true` before building to enable this feature.

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

Please use at your own discretion.


## How to build and run with Docker

```
# 1. Clone repository
git clone https://github.com/Montblanc0/spid-saml-check.git

# 2. Edit configurations and certificates
make your changes to the configuration file
 - spid-validator/config/server.json

create a private key and a certificate to use for https
 - spid-validator/config/spid-saml-check.key
 - spid-validator/config/spid-saml-check.crt

configure private key and certificate to use for response signing into:
 - spid-validator/config/idp.json
 - spid-validator/config/idp_demo.json

# 3. Build
cd spid-saml-check
docker compose up -d
```

Once the image build process end (it can take time for more minutes), the validator will be immediately available at

https://localhost:8443


To start the container after it has been stopped

```
docker start spid-saml-check
```

## How to use it as a *SPID Validator*

The application spid-validator, if invoked as a web application *as is*, provides "basic", formal validation of a Service Provider's SAML metadata.

In order to unleash the **full** set of SPID compliance tests (the proper *SPID Validator*), retrieve the metadata of *SPID Validator* at https://localhost:8443/metadata.xml and configure it on as a new Identity Provider (IdP) under your Service Provider (SP) implementation.

When used in this fashion, the *SPID Validator* can be invoked as an IdP from your SP, listing 300+ individual controls, divided into 7 families:
 * 4 families for the formal validation of the SP **metadata** (already described);
 * 3 families for the formal validation of the SP's SAML **request**;
 * 1 family (111 controls) for *interactively* validating the SP behaviour to SAML **response**s from IdP's.

To use the *SPID Validator* the AuthnRequest are thus sent from your SP, loggin in to Validator with credentials __validator__ / __validator__


### Usage steps

- Copy spid-validator metadata to the SP you want to test with.
  spid-validator can be downloaded at: [https://localhost:8443/metadata.xml](https://localhost:8443/metadata.xml)
  ````
  wget https://localhost:8443/metadata.xml -O /path/to/your/sp/metadata/folder/spid-saml-check-metadata.xml
  ````

- Start authentication request connecting to your SP, the AuthnRequest would be created and sent to spid-saml-check.
  You should access to a page like shown in the following picture
  
  <img src="doc/img/login.png" width="500" alt="login page" />
  

- Submit __validator__ / __validator__ as credential
- You would see the SAML2 Authn Request made from your SP

  <img src="doc/img/2.png" width="500" alt="authn request page" />
  

- Click on Metadata -> Download and submit your SP metadata url.<br/>
  **Warning**: If your SP is on your localhost, please use your host Docker IP and not "localhost"!

  <img src="doc/img/3.png" width="500" alt="metadata download page" />
  
- Now you'll be able to execute all the tests, in order of appareance: Metadata, Request and Response.
- To check a Response, from Response section, select in the scroll menu the test you want to execute, then mark it as done and if successful

  <img src="doc/img/4a.png" width="500" alt="response select page" />


## How to use it as a *SPID Demo*

The application spid-demo runs at: [https://localhost:8443/demo](https://localhost:8443/demo)

<img src="doc/img/demo_idp_index.png" width="500" alt="demo index page" />
   
   
Test users of spid-demo that can be used are listed at: [https://localhost:8443/demo/users](https://localhost:8443/demo/users)

<img src="doc/img/demo_idp_users.png" width="500" alt="demo users page" />


<h3>Usage steps</h3>

- Copy spid-demo metadata to the SP you want to test with.
  spid-demo metadata can be downloaded at: [https://localhost:8443/demo/metadata.xml](https://localhost:8443/demo/metadata.xml)
  ````
  wget https://localhost:8443/demo/metadata.xml -O /path/to/your/sp/metadata/folder/spid-demo.xml
  ````

- Go to https://localhost:8443 to register metadata of your SP on spid-validator.
  You should access to a page like shown in the following picture
  
  <img src="doc/img/login.png" width="500" alt="login page" />
  
  
- Submit __validator__/ __validator__ as credential

- Click on Metadata -> Download and submit your SP metadata url.<br/>
  **Warning**: If your SP is on your localhost, please use your host Docker IP and not "localhost"!
  
  <img src="doc/img/demo_download_metadata_sp.png" width="500" alt="download metadata page" />
  

- Send an authn request to spid-demo in order to use Demo environment

  <img src="doc/img/demo_idp.png" width="500" alt="demo idp" />
