### Adicionar o repositório Ingress-Nginx ao Helm

```
helm repo update
helm repo list
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm repo list
helm search repo ingress-nginx --versions

```
* Vamos trabalhar com a última versão do ingress, caso queira outra é só especificar

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/0bc25bfe-8ac2-4a34-b61a-0c27678429ad)

* Criaremos um arquivo .yaml com as especificações do chart. Este arquivo será analisado para ver quais imagens iremos ter que baixar separadamente

* No nosso exemplo, teremos que baixar  defaultbackend-amd64 e ingress-nginx/kube-webhook-certgen


```
helm show values ingress-nginx/ingress-nginx --version 4.7.1 > helm.yml
vim helm.yml
```

* Anotaremos o nome da imagem e sua tag.

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/0c0f9818-730f-439c-a1da-c89e8072d844)

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/f5532b53-880c-4590-881d-c88277709b53)

* Utilizaremos o docker pull para baixar cada imagem com sua respectiva tag.

```
sudo docker pull registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407

```

* **Observação:** Pode acontecer da rede estar bloqueada, se isso ocorrer, terá que fazer uso de um computador que tenha o docker instalado para fazer o docker pull da imagem, depois docker save

### Se estiver bloqueado, fazer conforme abaixo. 

* Temos que baixar a imagem com o docker pull que vimos nos passos anteriores. Depois iremos salvar essa imagem para que possamos enviá-la para nosso ambiente via protocolo SCP

```
sudo docker save -o /home/trojan/webhook.tar registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407

sudo docker save -o /home/amd64.tar registry.k8s.io/defaultbackend-amd64:1.5

```

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/eac0317f-f4fc-44f2-962d-b6dd5a6040e1)


* Usando um client SCP de sua escolha, envie as imagens empacotadas para o servidor.

  ![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/a9a00f00-25ff-47b1-85eb-906c44e35e7c)

* Após o envio da imagem executar o docker load e o retag da imagem.

```
sudo docker load -i webhook.tar

sudo docker tag registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407 registry.rancher.tcu.gov.br/rancher/ingress-nginx/kube-webhook-certgen:v20230407
```

* logar no docker registry da instituição e fazer o docker push

```
sudo docker login -u <usuário> <registry server>

sudo docker push

sudo docker push registry.rancher.tcu.gov.br/rancher/ingress-nginx/controller:v1.8.1
```

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/1374629e-fc6b-4d4c-b4b2-54c76a8fbeb8)


### Enviando a nova imagem do Ingress-Controller para o Harbor.

* Fazer o Helm Pull, copiar o arquivo com o SCP para o computador local

```
helm pull ingress-nginx/ingress-nginx --version 4.7.1
```
![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/b5775828-9b06-4eaa-8086-6c97a718f6bd)

* De posse do arquivo, logar no servidor Harbor

https://registry.rancher.tcu.gov.br/

* Ir para Projetos > Rancher > Helm Charts > Enviar

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/ec3f2b15-1cf7-4752-b12a-3fe2610be12c)

* Selecionar a imagem que está no computador local.

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/f8f112f0-fba5-43b1-99a1-d401ecbd55f0)

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/4ea68b5f-6163-4485-8aef-97ca2c14a749)

* Acessar o Ingress e ver a nossa imagem que acabamos de subir

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/7c9ca544-b3bf-4842-9bea-2b92da561b51)


### Alterando a versão do Chart no Fleet do GitOps

* Acessar o servidor GitLab da instituição

https://srv-gitlab-1a.tcu.gov.br/

* Acessar o Fleet do Cluster Local e de Aplicações

https://srv-gitlab-1a.tcu.gov.br/SINAP/gitops/-/blob/master/lab1/local/helm/ingress-nginx/fleet.yaml

* Vamos alterar a versão do ingress-nginx para o chart que enviamos para o Harbor.

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/0a78f6f6-8910-408e-98de-26c6ebb30e84)

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/8079f964-d794-4cc0-bd4b-42903c09e9ed)


### Altear o Digest das imagens (Ingress, Webhook) no Values do GitOps

* Primeiro vamos coletar o Digest da imagem no Harbor. Acessar o diretório da imagem desejada > Artefatos > Comando de Pull na imagem. Assim, copiamos o comando de pull com o Digest

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/6fff0cf5-8ac1-4425-82a2-62b7cd711df6)

* Algo assim deve aparecer

docker pull registry.rancher.tcu.gov.br/rancher/ingress-nginx/controller@sha256:bd54c330f73b17d0bf19f3ec3832b285d43a4c9fa5fe15f5a7accd3de706b438

* aí usa-se a partir de sha256:bd54c330f73b17d0bf19f3ec3832b285d43a4c9fa5fe15f5a7accd3de706b438

* Vamos alterar desta vez o Values.yaml tanto do cluster local quanto do cluster remoto.

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/e54ab28f-3f81-4645-a748-1c5228ed9f5c)

**Ingress**

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/a5330fc7-97c6-4a0a-b040-bb91d5d4e153)

**Webhook**

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/7d20426c-d5fb-42c6-8a28-0e2bc31809d7)




![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/a6a0cc72-4ead-4176-bd7b-58035b72e332)


* Feito o commit, iremos verificar no Rancher (Cluster Local e Aplicações) se a alteração surtiu o efeito desejado.

https://lab1.rancher.tcu.gov.br/


Local > WorkLoad > Pods > NameSpace **Ingress-Nginx** > Seleciona 1 Pod de amostra

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/5c5d5dcc-53ac-4f0a-8534-bf1818ebdc7c)


* Está Feito!! O Pod está com a versão desejada, está com o digest correto e está **Running**

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/01d22fde-72fa-4c08-af59-7757f68a127a)


* **Observação:** Se por acaso mesmo após mudar o Digest no Value.yaml não alterar no POD será necessário alterar diretamente no DaemonSet do Pod.

* Bastar acessar o cluster desejado via CLI, e com o Kubectl Edit Daemonsets -n <namespace> <nome do deployment> . Alterar o valor do Digest desejado. 

```
kubectl edit daemonsets.apps -n ingress-nginx ingress-nginx-controller
```

![image](https://github.com/SpockIsCoding/PrivateKnowlegdeBase/assets/97816800/b2c7cf21-2b07-4516-bb02-8c436f7e031f)






















