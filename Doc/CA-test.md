###
# This doc is used for replacing API server certificates to customer private certificate
###

# create a root ca private key
`openssl genrsa -out rootca.key 4096`
# generate a csr
`openssl req -new -sha256 -key rootca.key -out intermediateca.csr -subj "/C=CN/L=F/O=ACM/OU=QE/CN=*.dev09.red-chesterfield.com"`
# request key for the csr
`openssl req -nodes -x509 -newkey rsa:4096 -in intermediateca.csr -keyout key.pem -out cert.pem -days 365 -addext "subjectAltName = DNS:api.qe6-vmware-ibm.install.dev09.red-chesterfield.com" -subj "/C=CN/L=F/O=ACM/OU=QE/CN=*.dev09.red-chesterfield.com"`

# login hub cluster
`oc login -u kubeadmin -p XFFfk-UJoQS-zz5Yb-WUwhE --server=https://api.qe6-vmware-ibm.install.dev09.red-chesterfield.com:6443`
# create a secret that contains cert and key
`oc create secret tls test-ca --cert=cert.pem --key=key.pem -n openshift-config`
# update the api server to reference the created secret
`oc patch apiserver cluster --type=merge -p '{"spec":{"servingCerts": {"namedCertificates": [{"names": ["api.qe6-vmware-ibm.install.dev09.red-chesterfield.com"], "servingCertificate": {"name": "test-ca"}}]}}}'`
# wait at least 15 minutes for apiserver to restart
`oc get pods -n openshift-kube-apiserver`
