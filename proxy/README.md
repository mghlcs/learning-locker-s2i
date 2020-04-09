## Deploying Nginx as a proxy

To create or replace everything but the configuration and security:

    oc create|replace -f service.yaml -f deployment.yaml -f imagestream.yaml
    oc process -f route-template.yaml -p \
       HOSTNAME=<hostname>.partners.org \
       ROUTE_NAME=<name> \
       | oc create|replace -f -

## Configuration and Certificates

How to set up and deploy the configuration file and certs.

### Configuration

From the `config` subdirectory, to deploy initially:

    oc create configmap proxy-config --from-file=./
    oc label cm/proxy-config application=learning-locker component=proxy

To update an existing configmap:

    oc create configmap proxy-config --from-file=./ --dry-run -o yaml | oc replace -f -
    oc label cm/proxy-config application=learning-locker component=proxy

After making changes, redeploy:

    oc rollout latest proxy

### Certificates

After downloading the domain cert to proxy/ssl, then cd into the
environment-specific subdirectory, e.g. proxy/ssl/prod, and concatenate
the domain cert and intermediate:

    cat ../<cert path>.cer <path to intermediate>/interm_root_only.cer > proxy.cer

The domain cert should be at the top of the file so it goes first in
cat list.

See [the Nginx docs](http://nginx.org/en/docs/http/configuring_https_servers.html#chains)
for details on the Nginx configuration file.

There should be two files in the directory - proxy.key and
proxy.cer. The name of the files will become keys in the secret and
the content of the files become the values.

To deploy initially from the `proxy/ssl/<env>` subdirectory
corresponding to the environment you are working on:

    oc create secret generic media-ssl --from-file=./
    oc label secret/media-ssl application=learning-locker component=proxy

These commands will generate a base64-encoded set of secrets in a
Kubernetes secret resource object, which will inserted into the Nginx 
container at the specified mount point (see deployment configuration)
when the container starts up. The end result is that the keys become
file names and the values become the data in the file (i.e. the key
and certs).

To update the key and or cert:

    oc create secret generic proxy-ssl --from-file=./ --dry-run -o
        yaml |  oc replace -f -
    oc label secret/proxy-ssl application=learning-locker component=proxy

After making changes, redeploy:

    oc rollout latest proxy

### Finally...

Commit and push all changes to the infra repo.

### Deletion

To delete all resources... @@@ TODO
