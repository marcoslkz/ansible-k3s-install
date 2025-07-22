# ansible-k3s-install


## Install roles:

 ansible-galaxy install -r requirements.yml


## Install a new cluster

 ansible-playbook -i inventory.yaml  cluster.yml

### sealed secrets 

  kubectl get secret sealed-secrets-keyXXXX   -n kube-system   -o jsonpath="{.data['tls\.crt']}" | base64 -d > /opt/files/pub-cert.pem
  
## new user
  export USERPASS='xxx'
  export SECRETNAME='xxx'
  export NAMESPACE=app_namespace 

  kubectl create secret generic ${SECRETNAME} \
  --from-literal=password='${USERPASS}' \
  --namespace=${NAMESPACE} \
  --dry-run=client -o yaml > /tmp/secret.yaml


  kubeseal --cert /opt/files/pub-cert.pem --format=yaml < /tmp/secret.yaml > sealed-superuser.yaml
  
  
## new dbuser
  export DBUSER=xxx
  export DBPASS='xxx'
  export NAMESPACE=app_namespace 
  
  envsubst <<EOF | kubectl create -f - --dry-run=client -o yaml > /tmp/db-app-access.yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-app-access
  namespace: ${NAMESPACE}
type: Opaque
stringData:
  username: ${DBUSER}
  password: ${DBPASS}
  owner: ${DBUSER}
  database: ${DBUSER}
  dbname: ${DBUSER}
  host: db-rw.cnpg-system
  jdbc-uri: jdbc:postgresql://db-rw.cnpg-system:5432/${DBUSER}?password=${DBPASS}&user=${DBUSER}
  pgpass: db-rw:5432:${DBUSER}:${DBUSER}:${DBPASS}\n
  port: "5432"
  uri: postgresql://${DBUSER}:${DBPASS}@db-rw.cnpg-system:5432/${DBUSER}
  user: ${DBUSER}
EOF
  
  
  kubeseal --cert /opt/files/pub-cert.pem --format=yaml < /tmp/db-app-access.yaml > sealed-db-app-access.yaml
  