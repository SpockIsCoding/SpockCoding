# [ANTT] - VIYA

## Inventário - Ambiente SAS

| HOSTNAME | IP | CPU |MEMÓRIA |NOTA |
| -------- | -------- | -------- |-------- |-------- |
|vcnso104.antt.gov.br|	192.168.56.11|	16|	126|	CAS Controller |
|vcnso105.antt.gov.br|	192.168.56.12|	16|	126|	Microserviço|
|vcnso106.antt.gov.br|	192.168.56.13|	16|	126|	CAS Worker 02|

## Volumes - Discos

| Path | Tipo | tamanho | hosts |
| -------- | -------- | -------- | -------- |
|/opt/sas|	LOCAL| 1000G | vcnso104,vcnso105 |
|/sastmp/cascache |	LOCAL | 1000G |  vcnso104,vcnso105 |
|/saswork |	LOCAL| 500G |  vcnso104,vcnso105|
|/sasdata|	NFS| 500G |	vcnso104,vcnso105,vcnso106 |
|/backup |	NFS| 500G |	vcnso104,vcnso105,vcnso106 |

## Para Instalar SAS VIYA
Quando é confirmado a compra do SAS VIYA 3.5, o cliente recebe um e-mail com a licença e um arquivo SAS_Viya_deployment_data.zip .

Esse arquivo é usado para baixar os binários mirrormgr (responsável por baixar os pacotes dos produtos comprados) e sas-orchestration ( responsável por baixar o pacote de configurações do produto para o Ansible).

Abaixo o e-mail recebido pela SMF:
![](https://i.imgur.com/kMCYaTj.png)

- [x] Baixar pacotes de instalação - Mirrormgr

Para baixar o mirrormgr, entrar na seguinte URL:

https://support.sas.com/en/documentation/install-center/viya/deployment-tools/35/mirror-manager.html

![](https://i.imgur.com/EH3NSGo.png)

```bash=
[root@vcnso104 /]# cd /opt/sas/install
[root@vcnso104 install]# wget https://support.sas.com/installation/viya/35/sas-mirror-manager/lax/mirrormgr-linux.tgz
[root@vcnso104 install]# tar -xvf mirrormgr-linux.tgz
[root@vcnso104 install]# chmod +x mirrormgr
[root@vcnso104 install]# mkdir viya_mirror
[root@vcnso104 install]# ./mirrormgr mirror --deployment-data SAS_Viya_deployment_data.zip --path viya_mirror --platform x64-redhat-linux-6 --latest
```

- [x] Criar playbook - SAS-ORCHESTRATION

Para baixar o mirrormgr, entrar na seguinte URL: 

https://support.sas.com/en/documentation/install-center/viya/deployment-tools/35/command-line-interface.html


```bash=
[root@vcnso104 /]# cd /opt/sas/install
[root@vcnso104 install]# wget https://support.sas.com/installation/viya/35/sas-orchestration-cli/lax/sas-orchestration-linux.tgz
[root@vcnso104 install]# tar -xvf sas-orchestration-linux.tgz
[root@vcnso104 install]# chmod +x 
[root@vcnso104 install]# ./sas-orchestration build --input SAS_Viya_deployment_data.zip --platform redhat --deployment-type default --architecture x64

```

## Install Ansible
```bash=
[root@vcnso104 install]# pip3.6 install setuptools
[root@vcnso104 sas_viya_playbook]# pip3.6 install setuptools
[root@vcnso104 sas_viya_playbook]# pip3.6 install setuptools_rust
[root@vcnso104 sas_viya_playbook]# source env/bin/activate
(env) [root@vcnso104 sas_viya_playbook]# pip3.6 install ansible==2.10.7
```

## Editar arquivos de configuração - playbook
```bash=
(env)[root@vcnso104 sas_viya_playbook]# cd ..
(env)[root@vcnso104 install]# mkdir playbook_config/
(env)[root@vcnso104 install]# cd playbook_config/
(env)[root@vcnso104 playbook_config]# mv ../sas_viya_playbook/vars.yml
(env)[root@vcnso104 playbook_config]# mv ../sas_viya_playbook/inventory.ini
(env)[root@vcnso104 playbook_config]# cd ../sas_viya_playbook/
(env)[root@vcnso104 sas_viya_playbook]# ln -s ../playbook_config/inventory.ini .
(env)[root@vcnso104 sas_viya_playbook]# ln -s ../playbook_config/vars.yml .
(env)[root@vcnso104 sas_viya_playbook]# ls -ltr 

drwxr-xr-x   2 sas sas    4096 Out 20 13:37 utility
-r--r--r--   1 sas sas     765 Out 20 13:37 update-only.yml
drwxr-xr-x   2 sas sas    4096 Out 20 13:37 tasks
-r--r--r--   1 sas sas     679 Out 20 13:37 system-assessment.yml
-r--r--r--   1 sas sas     377 Out 20 13:37 site.yml
drwxr-xr-x   2 sas sas      86 Out 20 13:37 scripts
-r--r--r--   1 sas sas   12854 Out 20 13:37 SASViyaV0300_9CMBZV_Linux_x86-64.txt
-r--r--r--   1 sas sas   21639 Out 20 13:37 SASViyaV0300_9CMBZV_70248140_Linux_x86-64.jwt
-r-xr-xr-x   1 sas sas    2155 Out 20 13:37 SAS_CA_Certificate.pem
drwxr-xr-x   2 sas sas     139 Out 20 13:37 samples
drwxr-xr-x 166 sas sas    8192 Out 20 13:37 roles
-r--r--r--   1 sas sas   38345 Out 20 13:37 renew-security-artifacts.yml
drwxr-xr-x   4 sas sas      73 Out 20 13:37 module_utils
drwxr-xr-x   2 sas sas    4096 Out 20 13:37 library
drwxr-xr-x   2 sas sas   12288 Out 20 13:37 internal
-r--r--r--   1 sas sas     286 Out 20 13:37 install-only.yml
drwxr-xr-x   2 sas sas    4096 Out 20 13:37 group_vars
-r-xr-xr-x   1 sas sas    3013 Out 20 13:37 entitlement_certificate.pem
-r--r--r--   1 sas sas     355 Out 20 13:37 deploy-cleanup.yml
-r--r--r--   1 sas sas    6411 Out 20 13:37 deploy-casworker.yml
-r--r--r--   1 sas sas  130271 Out 20 13:37 checksums.txt
-r--r--r--   1 sas sas     589 Out 20 13:37 apply-license.yml
-r--r--r--   1 sas sas     431 Out 20 13:37 ansible.cfg
drwxr-xr-x   4 sas sas    4096 Out 20 13:39 viya-ark-master
drwxrwxr-x   5 sas sas     100 Out 22 22:05 env
drwxr-xr-x   3 sas sas     119 Out 22 22:18 filter_plugins
-rw-r--r--   1 sas sas     576 Out 22 22:19 cluster_defn_vars.yml
drwxr-xr-x   3 sas sas      49 Out 22 22:19 action_plugins
drwxrwxr-x   3 sas sas      24 Out 22 22:22 snapshot
-rw-r--r--   1 sas sas      86 Out 22 23:25 ssh_defn_vars.yml
lrwxrwxrwx   1 sas sas      27 Out 27 19:44 vars.yml -> ../playbook_config/vars.yml
lrwxrwxrwx   1 sas sas      32 Out 27 19:44 inventory.ini -> ../playbook_config/inventory.ini
-rw-rw-r--   1 sas sas 1423358 Out 27 22:02 deployment.log

```
### Acessar os arquivos inventory.ini e vars.yml e aplicar as devidas alterações conforme arquitetura proposta ao cliente. 
#### Os arquivos estão versionado no gitlab.vert.com.br


## Instalar Viya

```bash=
(env)[root@vcnso104 sas_viya_playbook]# ansible-playbook system-assessment.yml
(env)[root@vcnso104 sas_viya_playbook]# ansible-playbook ansible-playbook site.yml

```
elaborado por:Alexandre Carvalho da Silva