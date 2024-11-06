# postal

A helm-chart to deploy [Postal](https://postal.atech.media/) on kubernetes.

## Introduction

This chart bootstraps a deployment of Postal, MariaDB and RabbitMQ on a
[Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Prerequisites

- Kubernetes 1.27+
- PV provisioner support in the underlying infrastructure

## Optional prerequisites

- An ingress controller
- A functioning [cert-manager](https://github.com/jetstack/cert-manager) for certificate management

## Installing the Chart

To install the chart with the release name `my-release`, git clone this repository:

```console
git clone https://github.com/enguerr/postal-kubernetes.git
```

and install the chart:

```console
export MARIADB_REPLICATION_PASSWORD=$(kubectl get secret --namespace "septeo-postal-test" postal-mariadb -o jsonpath="{.data.mariadb-replication-password}" | base64 -d)
helm upgrade postal . -f values.yaml -n septeo-postal-test --kubeconfig=/home/enguer/.kube/config.kub-comput --set mariadb.auth.replicationPassword=$MARIADB_REPLICATION_PASSWORD
```

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```console
helm delete my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration - MariaDB and RabbitMQ

Please refer to the respective documentation for configuration parameters of those components.

## Configuration

We set the following default configuration parameters for MariaDB and RabbitMQ:

Parameter | Description | Default
--- | --- | ---
`mariadb.auth.password` | Password for the MariaDB root user. Change this from the default! | see [values.yaml](values.yaml)
`mariadb.metrics.enabled` | Enable prometheus metrics | `true`
`rabbitmq.replicaCount` | Number of RabbitMQ replicas. | `3`
`rabbitmq.auth.sername` | Username for RabbitMQ. | `postal`
`rabbitmq.auth.password` | Password for RabbitMQ. Change this from the default! | see [values.yaml](values.yaml)

The following table lists the configurable parameters of the postal chart and their default values.

Parameter | Description | Default
--- | --- | ---
`postal.nameOverride` | override the name of the chart | ``
`postal.config` | A postal configuration yaml to apply on top of postal's default configuration. See [Postal's default configuration](https://github.com/atech/postal/blob/master/config/postal.defaults.yml) for available options. | `{}`
`postal.image` | postal container image repository | `linkyard/postal`
`postal.imageTag` | postal container image tag | `1.0.0`
`postal.imagePullPolicy` | postal container image pull policy | `Always`
`postal.resources` | CPU/Memory resource requests/limits  | `{}`
`postal.signingKey` | RSA private key in PEM format used for DKIM signing. Change this from the default! | see [values.yaml](values.yaml)
`postal.railsSecretKey` | The secret key for rails. Change this from the default! | see [values.yaml](values.yaml)
`postal.letsEncryptKey` | RSA private key in PEM format. Used by Postal to acquire and renew certificates for the click-tracking-server from Let's Encrypt. Change this from the default! | see [values.yaml](values.yaml)
`postal.smtpPassword` | Password for the SMTP server. Change this from the default! | see [values.yaml](values.yaml)
`postal.web.ingress.enabled` | if an `ingress` resource should be deployed for the web interface | `true`
`postal.web.ingress.hostname` | public hostname for the web interface; this is a required value  | ``
`postal.web.ingress.ingressClass` | ingress class to use | `nginx`
`postal.web.ingress.tlsEnabled` | enable TLS on the ingress | `true`
`postal.web.ingress.certManager.enabled` | enable management of the TLS secret with [cert-manager](https://github.com/jetstack/cert-manager) | `true`
`postal.web.ingress.certManager.ingressClass` | ingress class to use for HTTP01 challenge | `nginx`
`postal.web.ingress.certManager.issuerName` | name of the cert-manager issuer; this is a required value | ``
`postal.web.ingress.certManager.issuerKind` | kind of the cert-manager issuer; this is a required value | ``
`postal.web.ingress.existingTlsSecret` | name of an existing TLS secret to use for the ingress (if cert-manager is not used); must be in the same namespace | ``
`postal.smtp.hostname` | public hostname of postal's SMTP server; this is a required value | ``
`postal.smtp.serviceType` | what kind of service the SMTP server is exposed as | `LoadBalancer`
`postal.smtp.certManager.enabled` | enable management of the TLS secret with [cert-manager](https://github.com/jetstack/cert-manager) | `true`
`postal.smtp.certManager.ingressClass` | ingress class to use for HTTP01 challenge | `nginx`
`postal.smtp.certManager.issuerName` | name of the cert-manager issuer; this is a required value | ``
`postal.smtp.certManager.issuerKind` | kind of the cert-manager issuer; this is a required value | ``
`postal.smtp.ingress.existingTlsSecret` | name of an existing TLS secret to use for the SMTP server (if cert-manager is not used); must be in the same namespace | ``

## CREATE SELF SIGN SECRET
```
openssl genrsa -out cert_key.pem 2048
openssl req -new -key cert_key.pem -out cert_csr.pem -subj "/CN=example.com"
openssl x509 -req -in cert_csr.pem -sha256 -days 365 -extensions v3_ca -signkey cert_key.pem -CAcreateserial -out cert_cert.pem
kubectl create secret tls postal-smtp-secret --cert=cert_cert.pem --key=cert_key.pem -n septeo-postal-prod
``