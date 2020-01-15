## Expiring Certs Pipeline

Often we see our Operators struggling to monitor and rotate non-configurable and configurable certs.  Alot of changes have been introduced into the Pivotal Application Service Platform to make monitoring easier.  This pipline aims to exploit those features and provide an example for Operators to use in their environments.


### Useful references

* [om cli](https://github.com/pivotal-cf/om)
    * `om --env env.yml expiring-certificates --expires-within "6m"`
* [Rotating certifications docs](https://docs.pivotal.io/platform/security/pcf-infrastructure/api-cert-rotation.html)
* [platform Automation expiring-certificates](https://docs.pivotal.io/platform-automation/v4.2/tasks.html#expiring-certificates)
    * The expiring certificates task will run a om command that checks the opsman expiring certificates endpoint.  If a expired certificate is found the task will not exit cleanly resulting in a pipeline failure. The pipeline in this repository simply modified this task so you can ship the results in an email.

### Known Bugs

* Bosh Nats
    * Certificate can only be rotated via opsman API `POST /api/v0/certificate_authorities/active/regenerate` in versions v2.2.17~,v2.3.10~,v2.4.4~, v2.5~
    * Starting PCF 2.1 Bosh nats CA cert validity was set to 2yrs and increased to 4 years in PCF 2.3.10~, 2.4.4~, 2.5~.  It is automatically rotated when Opsman Root CA is rotated.
* Bosh DNS
    * As of Opsman 2.3~ bosh-dns leaf certifications can be rotated using opsman api `POST /api/v0/certificate_authorities/active/regenerate`
    * After upgrading to Opsman [2.3](https://docs.pivotal.io/pivotalcf/2-3/pcf-release-notes/opsmanager-rn.html#bosh-dns-certs) a new 4 year Bosh DNS CA cert is automatically generated.  But you will still need to regenerate the leaf certificates using opsman api `POST /api/v0/certificate_authorities/active/regenerate` after upgrading to 2.3.
* Rotating Opsman SAML Certricate is not supported via API
    * [How to check and rotate Ops Manager SAML Certificate before it expires](https://community.pivotal.io/s/article/how-to-check-and-rotate-ops-manager-saml-certificate-before-it-expires)
* Credhub managed CAs are not rotatable via opsman API, however a cli tool called credhub maestro will allow you to rotate these certs
    * [https://docs-pcf-staging.cfapps.io/platform/2-8/release-notes/opsmanager-rn.html#maestro-rotate-ca](https://docs-pcf-staging.cfapps.io/platform/2-8/release-notes/opsmanager-rn.html#maestro-rotate-ca)
* om expiring-certificates will not detect Opsman Root CA cert until PAS 2.7 or later
    * To check if Opsman Root CA cert needs to be rotated is by downloading it from opsman api endpoint `https://OPS-MAN-FQDN/download_root_ca_cert`.  Then checking its validity

```
om -e ~/Downloads/05.yml curl -p /download_root_ca_cert > /tmp/root.crt

openssl x509 -in /tmp/root.crt -dates -noout
notBefore=Dec 13 21:38:30 2019 GMT
notAfter=Dec 14 21:38:30 2023 GMT
```

### Deploying pipeline

first update `params.yml` or add the relvent variables to your credhub/vault keystore.  Then deploy the pipeline

```
fly -t lab set-pipeline -p expiring-certs -c pipeline.yml -l params.yml
```


### Example Email

This is what the email will look like when the `check-expiring-certs` tasks detects an expired cert.

SUBJECT

```
ALERT: Expiring certs were found in Pivotal Application Service Platform
```

BODY

```
Getting expiring certificates...
Found expiring certificates in the foundation:

[X] Credhub
    /p-bosh/service-instance_a9a03b88-9115-49e4-be6a-06a09708f40f/mysql_server_tls: expiring on 13 Jan 21 22:07 UTC
    /p-bosh/service-instance_a9a03b88-9115-49e4-be6a-06a09708f40f/agent_server_tls: expiring on 13 Jan 21 22:07 UTC
    /p-bosh/service-instance_a9a03b88-9115-49e4-be6a-06a09708f40f/agent_client_tls: expiring on 13 Jan 21 22:07 UTC
.
.
.
```





