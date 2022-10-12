# Cert-Manager Tutorials

[Cert-Manager](https://cert-manager.io/docs/) adds certificates and certificate issuers as resource types in Kubernetes clusters, and simplifies the process of obtaining, renewing and using those certificates.
I created a set of tutorials for illustrating how cert-manager can be used in Kubernetes environments for above purposes.

## Prerequisites
Before starting below tutorials you will first need to understand what cert-manager is and how it works. Please go through below sections to understand that first:

### Cert Manager Custom Resource Definitions
Following are the list of Custom Resource Definitions (CRDs) that cert-manager uses in cert-manager 1.7.3:
```
kubectl -n cert-manager get pods -o json | jq .items[].spec.containers[].image

"quay.io/jetstack/cert-manager-controller:v1.7.3"
"quay.io/jetstack/cert-manager-cainjector:v1.7.3"
"quay.io/jetstack/cert-manager-webhook:v1.7.3"
```
```
kubectl get crd | grep cert-manager

certificaterequests.cert-manager.io                 2022-10-06T01:32:34Z
certificates.cert-manager.io                        2022-10-06T01:32:34Z
challenges.acme.cert-manager.io                     2022-10-06T01:32:34Z
clusterissuers.cert-manager.io                      2022-10-06T01:32:35Z
issuers.cert-manager.io                             2022-10-06T01:32:35Z
orders.acme.cert-manager.io                         2022-10-06T01:32:36Z
```

### Certificate Creation Process
Cert-manager has the concept of Certificates that define a desired X.509 certificate which will be renewed and kept up to date. A Certificate is a namespaced resource that references an Issuer or ClusterIssuer that determine what will be honoring the certificate request.

When a Certificate is created, a corresponding CertificateRequest resource is created by cert-manager containing the encoded X.509 certificate request, Issuer reference, and other options based upon the specification of the Certificate resource.

Here is one such example of a Certificate resource:
```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: acme-crt
spec:
  secretName: acme-crt-secret
  dnsNames:
  - example.com
  - foo.example.com
  issuerRef:
    name: letsencrypt-prod
    # We can reference ClusterIssuers by changing the kind here.
    # The default value is Issuer (i.e. a locally namespaced Issuer)
    kind: Issuer
    group: cert-manager.io
```

This Certificate will tell cert-manager to attempt to use the Issuer named letsencrypt-prod to obtain a certificate key pair for the example.com and foo.example.com domains. If successful, the resulting TLS key and certificate will be stored in a secret named acme-crt-secret, with keys of tls.key, and tls.crt respectively. This secret will live in the same namespace as the Certificate resource.

When a certificate is issued by an intermediate CA and the Issuer can provide the issued certificate's chain, the contents of tls.crt will be the requested certificate followed by the certificate chain.

Additionally, if the Certificate Authority is known, the corresponding CA certificate will be stored in the secret with key ca.crt. For example, with the ACME issuer, the CA is not known and ca.crt will not exist in acme-crt-secret.

Reference: https://cert-manager.io/docs/concepts/certificate

### Certificate Lifecycle
This diagram shows the lifecycle of a Certificate named cert-1 using an ACME / Let's Encrypt issuer. You don't need to understand all of these steps to use cert-manager; this is more of an explanation of the logic which happens under the hood for those curious about the process.

https://cert-manager.io/docs/concepts/certificate/#certificate-lifecycle

## List of Tutorials
- [Self-Signed Certificate Authority Issuer](/self-signed-ca-issuer)
- [Let's-Encrypt Certificate Authority Issuer](/lets-encrypt-ca-issuer)
