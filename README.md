## <img width="50" height="30" alt="image" src="https://github.com/user-attachments/assets/11c8d7ca-307b-43ea-b1b0-bc68f9e17df3" /> Instalación de Rancher sobre K3s (Single Node)

Este documento describe paso a paso la instalación de Rancher sobre K3s en un solo nodo,
utilizando un certificado TLS propio (por ejemplo DigiCert o CA corporativa)
y configurando agent-tls-mode en system-store.

---

# 1. Requisitos del servidor

## Hardware recomendado
- CPU: 4 vCPU
- RAM: 8 GB
- Disco: 50 GB
- Sistema operativo: Rocky Linux / RHEL / Ubuntu

## Puertos requeridos
| Puerto | Uso |
|-------|-----|
| 80 | HTTP |
| 443 | HTTPS |
| 6443 | Kubernetes API |
| 10250 | Kubelet |
| 8472 | Flannel VXLAN |﻿# install-rancher-k3s



## Instalacion de k3s

```
curl -sfL https://get.k3s.io | sh -s - --docker --data-dir /opt/k3s
```

#### exportar el kube-config de k3s


```
echo 'export KUBECONFIG=/etc/rancher/k3s/k3s.yaml' >> ~/.bashrc
source ~/.bashrc
```

#### Verificar que k3s esta corriendo

```
systemctl status k3s
kubectl get nodes
```

#### Instalacion de Helm

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

####  Instalar cert-manager
 ```
kubectl create namespace cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.4/cert-manager.crds.yaml

helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager   --namespace cert-manager   --version v1.14.4
 ```
####  Verificar que los conenedores de cert-manager suban correctamente
```
kubectl get pods -n cert-manager
```

####  Crear secret con el certificado del dominio respectivo

El tls.crt debe tener toda la cadena de certificacion completa

 ```
kubectl -n cattle-system create secret tls tls-rancher-ingress \
  --cert=tls.crt \
  --key=tls.key
```

##  Instalacion de rancher

#### Instalacion de repositorios de rancher en helm

```
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
```
#### Instalar el rancher utilizando el values.yaml

```
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  -f values-rancher.yaml
```

#### Verficar que los contenedores suban correctamente

```
kubectl get pods -n cattle-system
```
#### Obtener la contraseña inicial

```
kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'
```






