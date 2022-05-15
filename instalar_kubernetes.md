## Instalando o Kuberentes na AWS (com instâncias rodando o sistema operacional Debian)

Você pode acompanhar o passo a passo desse procedimento no vídeo abaixo:

https://youtu.be/7k_LG2Rk8PU

Vamos quebrar a instalação em 5 seções:

1 - Subindo as instâncias na AWS e configurando o security group

2 - Configurando o sistema operacional

3 - Instalando o Docker

4 - Instalando o Kubernetes

5 - Subindo alguns pods para testar


## Subindo as instâncias
Nessa seção não vou colocar o passo a passo, pois é mais fácil acompanhar pelos vídeos. Mas em suma:

* Vamos subir 3 instâncias na AWS com o sistema operacional Debian (type 2);

* E vamos configurar o secutiry group para as instâncias se comunicarem entre sim.

## Configurando o SO
Assim que subir as instâncias, conecte nos servidores com o terminal de sua preferência e execute:

__1 - Altere o nome das instâncias, para ficar fácil entender onde você está via prompt de comando__

a) Na master:

$ sudo su -

$ hostname k8s-master

$ echo k8s-master > /etc/hostname

$ bash

b) No worker 01:

$ sudo su -

$ hostname k8s-worker-01

$ echo k8s-worker-01 > /etc/hostname

$ bash

c) No worker 02:

$ sudo su -

$ hostname k8s-worker-02

$ echo k8s-worker-02 > /etc/hostname

$ bash

__2 - Faça o update do sistema operacional nas 3 instâncias__

$ apt-get update

## Instalando o Docker
__3 - Instale o Docker nas 3 instâncias__

$ curl -fsSL https://get.docker.com | bash

__4 - Teste se o Docker está ok nas 3 instâncias__

$ docker version

$ docker container ls

__5 - Configure o driver cgroup nas 3 instâncias__

$ cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

__6 - Crie o diretório nas 3 instâncias__
                                      
$ mkdir -p /etc/systemd/system/docker.service.d

__7 - Reinicie o docker nas 3 instâncias__
                                      
$ systemctl daemon-reload
                                      
$ systemctl restart docker

## Instalando o Kubernetes
O site de instalação do Kubernetes é: https://kubernetes.io/pt-br/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

__8 - Faça o update do sistema operacional nas 3 instâncias__
                                      
$ sudo apt-get update

__9 - Instale os módulos necessários para o Kubernetes funcionar (normalmente todos eles já estão instalados, mas não custar certificar) nas 3 instâncias__
                                      
$ sudo apt-get install -y apt-transport-https ca-certificates curl

__10 - Faça o download da assinatura pública do Google nas 3 instâncias__
                                      
$ sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

__11 - Adicione o repositório apt do Kubernetes nas 3 instâncias__
                                      
$ echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

__12 - Atualize e instale os pacotes do Kubernetes nas 3 instâncias__
                                      
$ sudo apt-get update
                                      
$ sudo apt-get install -y kubelet kubeadm kubectl
                                      
$ sudo apt-mark hold kubelet kubeadm kubectl

__13 - Faça o download das imagens dos componentes do Kubernetes, somente na instância MASTER__
                                      
$ kubeadm config images pull

__ATENÇÃO: SE DER ERRO NESSE PASSO FAÇA O PROCEDIMENTO ABAIXO NAS 3 INSTÂNCIAS__
                                      
$ rm /etc/containerd/config.toml
$ systemctl restart containerd
                                      
$ kubeadm config images pull

__14 - Inicie o Kubernetes, somente na instância MASTER__
                                      
$ kubeadm init

__ATENÇÃO: NÃO ESQUEÇA DE DEIXAR GUARDADO A CHAVE DE CONEXÃO DOS WORKERS COM O MASTER__

__15 - Crie os diretórios que o Kubernetes solicita, somente na instância MASTER__
                                      
$ mkdir -p $HOME/.kube
                                      
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
                                      
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

__16 - Instale o pod que vai gerenciar a rede, no nosso caso vamos usar o Calico, somente na instância MASTER__
                                      
$ kubectl create -f https://docs.projectcalico.org/manifests/calico.yaml

__17 - Adicione os nodes workes, executando nos WORKERS o comando que você guardou no passo 14. É algo do tipo__
                                      
$ kubeadm join 172.31.83.12:6443 --token o7ilzx.3bfmozsp743itdfx \
        --discovery-token-ca-cert-hash sha256:ffe2688651341e5c7fd6594d40100a42ea2858f1f8ee981169263fb9dc85a72d

__18 - Agora é só testar deployando pods do NGINX ou qualquer outro que você quiser__
                                      
$ kubectl run nginx1 --image nginx
                                      
$ kubectl run nginx2 --image nginx
