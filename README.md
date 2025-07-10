# ansible-k3s-install


## Install roles:

 ansible-galaxy install -r requirements.yml


## Install a new cluster

 ansible-playbook -i inventory.yaml  cluster.yml

### sealed secrets 

  kubectl get secret sealed-secrets-keyXXXX   -n kube-system   -o jsonpath="{.data['tls\.crt']}" | base64 -d > /opt/files/pub-cert.pem

  kubectl create secret generic dbsuper \
  --from-literal=password='SenhaUltraSegura123' \
  --namespace=cnpg-system \
  --dry-run=client -o yaml > /tmp/secret.yaml
  
  


  kubeseal --cert /opt/files/pub-cert.pem --format=yaml < /tmp/secret.yaml > sealed-superuser.yaml