# Create a cert for the new user

First we need to generate an rsa key
```
openssl genrsa -out bob.key 4096
```

In the CN: you must add the user name
In the O: you must add the team/group

```
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
[ dn ]
CN = bob
O = dev
[ v3_ext ]
authorityKeyIdentifier=keyid,issuer:always
basicConstraints=CA:FALSE
keyUsage=keyEncipherment,dataEncipherment
extendedKeyUsage=serverAuth,clientAuth
```
Save this as csr.cnf 
CSR=Certificate Signing Request

```
openssl req -config ./csr.cnf -new -key bob.key -nodes -out bob.csr
```

When the CSR is ready, Bob must send it to the admin of the cluster

# Certificate Signing Request

```
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: mycsr
spec:
  groups:
  - system:authenticated
  request: ${BASE64_CSR}
  usages:
  - digital signature
  - key encipherment
  - server auth
  - client auth
```

get the Base64 value
```
export BASE64_CSR=$(cat ./bob.csr | base64 | tr -d '\n')
```

apply the resource
```
cat csr.yaml | envsubst | kubectl apply -f -
```

Check the status of the request
```
kubectl get csr
```

Approve the request
```
kubectl certificate approve mycsr
```

Building the config to the new user

```
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: ${CLUSTER_CA}
    server: ${CLUSTER_ENDPOINT}
  name: ${CLUSTER_NAME}
users:
- name: ${USER}
  user:
    client-certificate-data: ${CLIENT_CERTIFICATE_DATA}
contexts:
- context:
    cluster: ${CLUSTER_NAME}
    user: bob
  name: ${USER}-${CLUSTER_NAME}
current-context: ${USER}-${CLUSTER_NAME}
```

export USER="Bob"
export CLUSTER_NAME=$(kubectl config view --minify -o jsonpath={.current-context})
kubectl get csr mycsr -o jsonpath='{.status.certificate}'

CLUSTER_CA and CLUSTER_ENDPOINT
kubectl config view --raw -o json | jq -r '.clusters[] 

Using the config for temporary:
export KUBECONFIG=$PWD/kubeconfig

Using the config for permanent:
adding a new entry in the default $HOME/.kube/config