# Cert Manager Tutorials

## Install Cert-Manager
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.9.1/cert-manager.yaml
```

## Create Certificate Authority Issuer
```
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: hello-ca-issuer
spec:
  selfSigned: {}
```

```
kubectl apply -f hello-ca-issuer.yaml
```

## Create CA Certificate
```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
 name: hello-ca-certificate
spec:
 secretName: hello-ca-certificate-key-pair
 isCA: true
 issuerRef:
   name: hello-ca-issuer
   kind: Issuer
 commonName: "hello.certificate.authority"
 dnsNames:
 - hello.certificate.authority
```

```
kubectl apply -f hello-ca-certificate.yaml
```

## Verify CA certificate
```
kubectl get secrets hello-ca-certificate-key-pair -o json | jq -r '.data["tls.crt"]' | base64 -D | keytool -printcert | grep Version -B 10
```

```
Owner: CN=hello.certificate.authority
Issuer: CN=hello.certificate.authority
Serial number: 8c7b14f4b93eb17bf507293be63c7abf
Valid from: Tue Oct 11 22:17:27 AEDT 2022 until: Mon Jan 09 22:17:27 AEDT 2023
Certificate fingerprints:
	 SHA1: 10:C1:74:B9:65:13:F4:40:30:E2:20:4D:24:33:C3:CF:34:DE:12:8D
	 SHA256: BF:40:86:DF:4D:A4:14:8B:85:A3:DF:96:24:4B:F2:28:E2:8C:0F:90:39:66:CC:A2:31:4D:16:AA:17:31:3A:82
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 3
```

## Create Certificate Issuer
```
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: hello-certificate-issuer
spec:
  ca:
    secretName: hello-ca-certificate-key-pair
```

## Create Certificate
```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
 name: hello-certificate
spec:
 secretName: hello-certificate-key-pair
 isCA: true
 issuerRef:
   name: hello-certificate-issuer
   kind: Issuer
 commonName: "hello.certificate"
 dnsNames:
 - hello.certificate
```

## Verify Certificate
```
kubectl get secrets hello-certificate-key-pair -o json | jq -r '.data["tls.crt"]' | base64 -D | keytool -printcert | grep Version -B 10
```

```
Owner: CN=hello.certificate
Issuer: CN=hello.certificate.authority
Serial number: e90e27bdf1c531409b58fce74ee715c0
Valid from: Tue Oct 11 22:17:27 AEDT 2022 until: Mon Jan 09 22:17:27 AEDT 2023
Certificate fingerprints:
	 SHA1: 57:52:E4:B0:9B:72:1A:29:AE:9F:9A:81:E5:46:3C:F3:FE:AD:84:9F
	 SHA256: 5E:12:BD:86:96:4B:73:BD:33:ED:C8:7E:63:0F:97:B5:5E:8F:91:2E:5F:50:DB:1F:5F:26:2D:CF:CF:99:95:48
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 3
```