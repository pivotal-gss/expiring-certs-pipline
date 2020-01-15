## Expiring Certs Pipeline

Often we see our Operators struggling to monitor and rotate non-configurable and configurable certs.  Alot of changes have been introduced into the Pivotal Application Service Platform to make monitoring easier.  This pipline aims to exploit those features and provide an example for Operators to use in their environments.


### Useful references

* [om cli](https://github.com/pivotal-cf/om)
* [Rotating certifications docs](https://docs.pivotal.io/platform/security/pcf-infrastructure/api-cert-rotation.html)
* [platform Automation expiring-certificates](https://docs.pivotal.io/platform-automation/v4.2/tasks.html#expiring-certificates)
    * The expiring certificates task will run a om command that checks the opsman expiring certificates endpoint.  If a expired certificate is found the task will not exit cleanly resulting in a pipeline failure. The pipeline in this repository simply modified this task so you can ship the results in an email.

### Known Bugs

*
*
*

### Deploying pipeline

first update `params.yml` or add the relvent variable to your credhub/vault keystore.  Then deploy the pipeline

```
fly -t lab set-pipeline -p expiring-certs -c pipeline.yml -l params.yml
```


### Example Email

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





