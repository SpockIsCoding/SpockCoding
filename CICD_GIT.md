####   O exemplo de código do .gitlab-ci.yaml a seguir é um passo da pipeline no git que faz um push de sua aplicação no dockerhub empacotando tudo, assim, é possível no kubernetes, docker swarn, docker, fazer um pull dessa imagem de sua aplicação. 

* Toda ver que houver um commit no repositório o Git irá executar essa pipeline fazendo esse push dessa aplicação docker.

#################################.gitlab-ci.yml ##################################

```bash=
stages:
  - build

build_images:
  stage: build
  image: docker:20.10.16

  services:
    - docker:20.10.16-dind
  
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  
  before_script:
    - docker login -u  $REGISTRY_USER -p $REGISTRY_PASS

  script:
    - docker build -t spockiscoding/jarjarbinx:1.0 app/.
    - docker push spockiscoding/jarjarbinx:1.0
  ```
    
 ################################# dockerfile  ##################################

```bash=
FROM httpd:latest

WORKDIR /usr/local/apache2/htdocs/

COPY index.html /usr/local/apache2/htdocs/

EXPOSE 80
```

######################## Configuração da Pipeline no Gitlab #####################

![image](https://user-images.githubusercontent.com/97816800/210676860-bda00605-af7e-4d6f-afab-87636e8d3d88.png)




