###
# This doc is used for replacing API server certificates to customer private certificate
###

1. create a root ca private key
```
openssl genrsa -out rootca.key 4096
```
2. generate a csr
`openssl req -new -sha256 -key rootca.key -out intermediateca.csr -subj "/C=CN/L=F/O=ACM/OU=QE/CN=*.dev09.red-chesterfield.com"`
3. request key for the csr
`openssl req -nodes -x509 -newkey rsa:4096 -in intermediateca.csr -keyout key.pem -out cert.pem -days 365 -addext "subjectAltName = DNS:api.qe6-vmware-ibm.install.dev09.red-chesterfield.com" -subj "/C=CN/L=F/O=ACM/OU=QE/CN=*.dev09.red-chesterfield.com"`

4. login hub cluster
`oc login -u kubeadmin -p XFFfk-UJoQS-zz5Yb-WUwhE --server=https://api.qe6-vmware-ibm.install.dev09.red-chesterfield.com:6443`
5. create a secret that contains cert and key
`oc create secret tls test-ca --cert=cert.pem --key=key.pem -n openshift-config`
6. update the api server to reference the created secret
`oc patch apiserver cluster --type=merge -p '{"spec":{"servingCerts": {"namedCertificates": [{"names": ["api.qe6-vmware-ibm.install.dev09.red-chesterfield.com"], "servingCertificate": {"name": "test-ca"}}]}}}'`
7. wait at least 15 minutes for apiserver to restart
`oc get pods -n openshift-kube-apiserver`
8. check the managedcluster status
`oc get mcl`

### Troubleshooting: if the hub cluster only has one master node, the managedcluster may get offline. Recover the managedcluster by replacing bootstrap.
1. get import yaml from hub cluster
`oc login <hubcluster>`
`oc get secret -n <cluster_name> <cluster_name>-import -ojsonpath='{.data.import\.yaml}' | base64 --decode  > import.yaml`
3. apply the import yaml to the managedcluster
`oc login <managedcluster>`
`oc apply -f import.yaml`
