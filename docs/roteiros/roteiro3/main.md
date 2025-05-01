## Objetivo

O objetivo principal desse roteiro é:

- entender os conceitos básicos de Private Cloud
- aprofundar conceitos sobre redes virtuais SDN

## Roteiro

Para visualizar o arquivo PDF feito e entregue, consultar o link a seguir: **[Roteiro 3 - PDF](./Roteiro_3_de_Cloud.pdf)**.

## Montagem do Roteiro

O OpenStack é um conjunto de componentes de software que oferecem serviços comuns para a infraestrutura de cloud.

![OpenStack](imgs/roteiro3-openstack.png)
/// caption
Figura 1 - OpenStack. Fonte: [https://www.openstack.org/](https://www.openstack.org/){target=_blank}
///

## Infra (Nuvem VM) - Servidor Virtual Privado (VPS)

O OpenStack permitirá distribuir virtual machines usando os nós disponíveis no kit, mas antes de iniciar a instalação, deve-se verificar se o MAAS está configurado corretamente.

A documentação oficial do OpenStack está presente no link a seguir: [Implantação do OpenStack](https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/install-openstack.html#openstack-release){target=_blank}

### Implantação do OpenStack

- Nesse ambiente, o Juju controller deve estar instalado no server1;
- Para monitorar o status da instalação, usar o comando:
``` sh
watch -n 2 --color "juju status --color"
```

!!! failure "Erro de instalação"
    
    No caso de problemas durante a instalação, dependendo da gravidade do problema, é mais simples limpar a instalação:
    
    ``` sh
    juju kill-controller maas-controller
    ```
    
    E reiniciar desde [1 - Juju Controller](#1-juju-controller).

!!! Aviso

    O roteiro feito, em 2024.2, possui uma versão antiga dos comandos utilizados. Durante essa documentação do roteiro, serão apresentados os comandos utilizados na realização do roteiro, e os comandos atualizados estarão apresentados em notas intituladas **Nova Versão**.

### 1 - Juju Controller

Apenas se ainda não tiver um Juju Controller, adicionar a tag controller na máquina server1:

``` sh
juju bootstrap --bootstrap-series=jammy --constraints tags=controller maas-one maas-controller
```

### 2 - Modelo de deploy

``` sh
juju add-model --config default-series=jammy openstack
juju switch maas-controller:openstack
```

### 3 - Ceph OSD

O aplicativo ceph-osd é implantado em 3 nós usando o charm ceph-osd, com os dispositivos de armazenamento ```/dev/sda``` e ```/dev/sdb``` configurados para uso em todos os nós no arquivo **ceph-osd.yaml**. É utilizada a tag compute, definida previamente nos nós via MAAS, com o comando juju deploy especificando as 3 máquinas e a configuração das mesmas. Para fazer o deploy da aplicação ceph-osd:

``` sh
juju deploy -n 3 --channel reef/stable --config ceph-osd.yaml --constraints tags=compute ceph-osd
```

??? "Nova Versão"
    ``` sh
    juju deploy -n 3 --channel quincy/stable --config ceph-osd.yaml --constraints tags=compute ceph-osd
    ```

### 4 - Nova Compute

O nova-compute, responsável por provisionar instâncias de computação no OpenStack, deve ser implantado nos nós de computação usando os IDs das máquinas (0, 1 e 2), pois não há mais nós MAAS disponíveis. Isso implica em compartilhar os mesmos nós entre múltiplos serviços. É feito o arquivo de configuração **nova-compute.yaml**, e para o deploy:

``` sh
juju deploy -n 2 --to 1,2 --channel 2023.2/stable --config nova-compute.yaml nova-compute
```

??? "Nova Versão"
    ``` sh
    juju deploy -n 3 --to 0,1,2 --channel yoga/stable --config nova-compute.yaml nova-compute
    ```

### 5 - MySQL InnoDB Cluster

MySQL InnoDB Cluster requer no mínimo 3 unidades de banco de dados, e ele deve implantado com o atributo mysql-innodb-cluster, sendo conteinerizados nas máquinas 0, 1 e 2, fazendo o deploy como indicado:

``` sh
juju deploy -n 3 --to lxd:0,lxd:1,lxd:2 --channel 8.0/stable mysql-innodb-cluster
```

### 6 - Vault

O Vault gerencia os certificados TLS que permitem a comunicação criptografada entre aplicativos em nuvem, e ele é conteinerizado na máquina 2:

``` sh
juju deploy --to lxd:0 --channel 1.8/stable vault
```

??? "Nova Versão"
    ``` sh
    juju deploy --to lxd:2 vault --channel 1.8/stable
    ```

Os passos seguintes são de acordo com as intruções abaixo, com os seus respectivos comandos:

- criar instância específica do mysql-router com o charme subordinado mysql-router;
- adicionar relação entre instância mysql-router e o banco de dados;
- adicionar relação entre instância mysql-router e o aplicativo.

``` sh
juju deploy --channel 8.0/stable mysql-router vault-mysql-router
juju integrate vault-mysql-router:db-router mysql-innodb-cluster:db-router
juju integrate vault-mysql-router:shared-db vault:shared-db
```

Para o Vault ser inicializado, desbloqueado e autorizado, são executados os comandos a seguir com os seguintes propósitos:

- Instalando o cli do Vault;
- Configurando o cli;
- Gerando-os;
- Remover o selo de 3 teclas (usar esse comando 3 vezes);
- Confirgurando o token;
- Gerando um token (tempo dos próximos passos de 10 minutos);
- Autorizando.

``` sh
sudo snap install vault
export VAULT_ADDR="http://<IP of vault>:8200"
vault operator init -key-shares=5 -key-threshold=3
vault operator unseal <Unseal Key>
export VAULT_TOKEN=<Initial Root Token>
vault token create -ttl=10m
juju run vault/0 authorize-charm token=<Token generated in the last command>
```

??? "Nova Versão"
    ``` sh
    juju run vault/leader authorize-charm token=<Token generated in the last command>
    ```

Para o Vault, é preciso um certificado de CA autoassinado para que ele possa emitir os certificados para serviços de API em nuvem.

``` sh
juju run vault/0 generate-root-ca
```

??? "Nova Versão"
    ``` sh
    juju run vault/leader generate-root-ca
    ```

Os aplicativos em nuvem são habilitados para TLS por meio da relação vault:certificates.

``` sh
juju integrate mysql-innodb-cluster:certificates vault:certificates
```

### 7 - Neutron Networking

A rede de nêutrons é implementada com 4 aplicações, com o arquivo **neutron.yaml** configurando apenas as aplicações neutron-api e ovn-chassis, e com isso são feitos os deploys:

- **ovn-central**: no mínimo 3 unidades -> conteinerizadas nas máquinas 0, 1 e 2.
- **neutron-api**: conteinerizado na máquina 1.
- **neutron-api-plugin-ovn (subordinado)**: aplicação charm subordinada.
- **ovn-chassis (subordinado)**: aplicação charm subordinada.

``` sh
juju deploy -n 3 --to lxd:0,lxd:1,lxd:2 --channel 23.09/stable ovn-central
juju deploy --to lxd:1 --channel 2023.2/stable --config neutron.yaml neutron-api
juju deploy --channel 2023.2/stable neutron-api-plugin-ovn
juju deploy --channel 23.09/stable --config neutron.yaml ovn-chassis
```

??? "Nova Versão"
    ``` sh
    juju deploy -n 3 --to lxd:0,lxd:1,lxd:2 --channel 22.03/stable ovn-central
    juju deploy --to lxd:1 --channel yoga/stable --config neutron.yaml neutron-api
    juju deploy --channel yoga/stable neutron-api-plugin-ovn
    juju deploy --channel 22.03/stable --config neutron.yaml ovn-chassis
    ```

São adicionadas as relações necessárias:

``` sh 
juju integrate neutron-api-plugin-ovn:neutron-plugin neutron-api:neutron-plugin-api-subordinate
juju integrate neutron-api-plugin-ovn:ovsdb-cms ovn-central:ovsdb-cms
juju integrate ovn-chassis:ovsdb ovn-central:ovsdb
juju integrate ovn-chassis:nova-compute nova-compute:neutron-plugin
juju integrate neutron-api:certificates vault:certificates
juju integrate neutron-api-plugin-ovn:certificates vault:certificates
juju integrate ovn-central:certificates vault:certificates
juju integrate ovn-chassis:certificates vault:certificates
```

Conectando a neutron-api ao banco de dados na nuvem:

``` sh
juju deploy --channel 8.0/stable mysql-router neutron-api-mysql-router
juju integrate neutron-api-mysql-router:db-router mysql-innodb-cluster:db-router
juju integrate neutron-api-mysql-router:shared-db neutron-api:shared-db
```

### 8 - Keystone

Aplicação Keystone conteinerizada na máquina 0, com deploy: 

``` sh
juju deploy --to lxd:0 --channel 2023.2/stable keystone
```

??? "Nova Versão"
    ``` sh
    juju deploy --to lxd:0 --channel yoga/stable keystone
    ```

Conectando o Keystone ao banco de dados na nuvem:

``` sh
juju deploy --channel 8.0/stable mysql-router keystone-mysql-router
juju integrate keystone-mysql-router:db-router mysql-innodb-cluster:db-router
juju integrate keystone-mysql-router:shared-db keystone:shared-db
```

Adicionando relações:

``` sh
juju integrate keystone:identity-service neutron-api:identity-service
juju integrate keystone:certificates vault:certificates
```

### 9 - RabbitMQ

O aplicativo rabbitmq-server é conteinerizado na máquina 2 com o charm rabbitmq-server, adicionando em seguida duas relações.

``` sh
juju deploy --to lxd:2 --channel 3.9/stable rabbitmq-server
juju integrate rabbitmq-server:amqp neutron-api:amqp
juju integrate rabbitmq-server:amqp nova-compute:amqp
```

### 10 - Nova Cloud Controller

Esse aplicativo é conteinerizado na máquina 2 com o charm nova-cloud-controller. É feito o arquivo **ncc.yaml**, que contém as configurações, e depois é feito o deploy da aplicação:

``` sh
juju deploy --to lxd:2 --channel 2023.2/stable --config ncc.yaml nova-cloud-controller
```

??? "Nova Versão"
    ``` sh
    juju deploy --to lxd:2 --channel yoga/stable --config ncc.yaml nova-cloud-controller
    ```

Conectando o Nova Cloud Controller ao banco de dados na nuvem:

``` sh
juju deploy --channel 8.0/stable mysql-router ncc-mysql-router
juju integrate ncc-mysql-router:db-router mysql-innodb-cluster:db-router
juju integrate ncc-mysql-router:shared-db nova-cloud-controller:shared-db
```

Adicionando relações:

``` sh
juju integrate nova-cloud-controller:identity-service keystone:identity-service
juju integrate nova-cloud-controller:amqp rabbitmq-server:amqp
juju integrate nova-cloud-controller:neutron-api neutron-api:neutron-api
juju integrate nova-cloud-controller:cloud-compute nova-compute:cloud-compute
juju integrate nova-cloud-controller:certificates vault:certificates
```

### 11 - Placement

A aplicação Placement é conteinerizada na máquina 2 com o charm placement.

``` sh
juju deploy --to lxd:1 --channel 2023.2/stable placement
```

??? "Nova Versão"
    ``` sh
    juju deploy --to lxd:2 --channel yoga/stable placement
    ```

Conectando o Placement ao banco de dados na nuvem:

``` sh
juju deploy --channel 8.0/stable mysql-router placement-mysql-router
juju integrate placement-mysql-router:db-router mysql-innodb-cluster:db-router
juju integrate placement-mysql-router:shared-db placement:shared-db
```

Adicionando relações:

``` sh
juju integrate placement:identity-service keystone:identity-service
juju integrate placement:placement nova-cloud-controller:placement
juju integrate placement:certificates vault:certificates
```

### 12 - Horizon - OpenStack Dashboard

O Horizon é conteinerizado na máquina 2 com o charm openstack-dashboard.

``` sh
juju deploy --to lxd:2 --channel 2023.2/stable openstack-dashboard
```

??? "Nova Versão"
    ``` sh
    juju deploy --to lxd:2 --channel yoga/stable openstack-dashboard
    ```

Conectando o Horizon ao banco de dados na nuvem:

``` sh
juju deploy --channel 8.0/stable mysql-router dashboard-mysql-router
juju integrate dashboard-mysql-router:db-router mysql-innodb-cluster:db-router
juju integrate dashboard-mysql-router:shared-db openstack-dashboard:shared-db
```

Adicionando relações:

``` sh
juju integrate openstack-dashboard:identity-service keystone:identity-service
juju integrate openstack-dashboard:certificates vault:certificates
```

### 13 - Glance

A aplicação Glance é conteinerizada na máquina 2 com o charm glance.

``` sh
juju deploy --to lxd:2 --channel 2023.2/stable glance
```

??? "Nova Versão"
    ``` sh
    juju deploy --to lxd:2 --channel yoga/stable glance
    ```

Conectando o Glance ao banco de dados na nuvem:

``` sh
juju deploy --channel 8.0/stable mysql-router glance-mysql-router
juju integrate glance-mysql-router:db-router mysql-innodb-cluster:db-router
juju integrate glance-mysql-router:shared-db glance:shared-db
```

Adicionando relações:

``` sh
juju integrate glance:image-service nova-cloud-controller:image-service
juju integrate glance:image-service nova-compute:image-service
juju integrate glance:identity-service keystone:identity-service
juju integrate glance:certificates vault:certificates
```

### 14 - Ceph Monitor

O Ceph Monitor é conteinerizado nas máquinas 0, 1 e 2 com o charm ceph-mon. Primeiro é feito o arquivo **ceph-mon.yaml**, que contém as configurações, e depois é feito o deploy da aplicação:

``` sh
juju deploy -n 3 --to lxd:0,lxd:1,lxd:2 --channel reef/stable --config ceph-mon.yaml ceph-mon
```

??? "Nova Versão"
    ``` sh
    juju deploy -n 3 --to lxd:0,lxd:1,lxd:2 --channel quincy/stable --config ceph-mon.yaml ceph-mon
    ```

Adicionando relações:

``` sh
juju integrate ceph-mon:osd ceph-osd:mon
juju integrate ceph-mon:client nova-compute:ceph
juju integrate ceph-mon:client glance:ceph
```

### 15 - Cinder

O Cinder é conteinerizado na máquina 1 com o charm cinder. É criado o arquivo de configuração **cinder.yaml**, e depois é feito o deploy da aplicação:

``` sh
juju deploy --to lxd:1 --channel 2023.2/stable --config cinder.yaml cinder
```

??? "Nova Versão"
    ``` sh
    juju deploy --to lxd:1 --channel yoga/stable --config cinder.yaml cinder
    ```

Conectando o Cinder ao banco de dados na nuvem:

``` sh
juju deploy --channel 8.0/stable mysql-router cinder-mysql-router
juju integrate cinder-mysql-router:db-router mysql-innodb-cluster:db-router
juju integrate cinder-mysql-router:shared-db cinder:shared-db
```

Adicionando relações:

``` sh
juju integrate cinder:cinder-volume-service nova-cloud-controller:cinder-volume-service
juju integrate cinder:identity-service keystone:identity-service
juju integrate cinder:amqp rabbitmq-server:amqp
juju integrate cinder:image-service glance:image-service
juju integrate cinder:certificates vault:certificates
```

A relação glance:image-service permite que o Cinder consuma a API do Glance e, como o Glance, o Cinder usa o ceph como backend de armazenamento, implementado com o comando subordinado a seguir:


``` sh
juju deploy --channel 2023.2/stable cinder-ceph
```

??? "Nova Versão"
    ``` sh
    juju deploy --channel yoga/stable cinder-ceph
    ```

Adicionando relações:

``` sh
juju integrate cinder-ceph:storage-backend cinder:storage-backend
juju integrate cinder-ceph:ceph ceph-mon:client
juju integrate cinder-ceph:ceph-access nova-compute:ceph-access
```

### 16 - Ceph RADOS Gateway

O Ceph RADOS Gateway, implantado para oferecer um gateway HTTP compatível com S3 e Swift, é conteinerizado na máquina 0 com o charm ceph-radosgw.

``` sh
juju deploy --to lxd:0 --channel reef/stable ceph-radosgw
```

??? "Nova Versão"
    ``` sh
    juju deploy --to lxd:0 --channel quincy/stable ceph-radosgw
    ```

Adicionando relação:

``` sh
juju integrate ceph-radosgw:mon ceph-mon:radosgw
```

### 17 - Ceph-OSD Integration

Com todos os comandos anteriores corretamente realizados, é preciso executar a configuração do charm ceph-osd para usar o dispositivo de bloco ```/dev/sdb``` como armazenamento para os OSDs do Ceph.

``` sh
juju config ceph-osd osd-devices='/dev/sdb'
```

## Configurando o Openstack

- Documentação da referência oficial: [https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/configure-openstack.html](https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/configure-openstack.html){:target=_blank}

Serão configurados os serviços que controlam as VMs (Nova), os volumes de disco (Cinder), e a estrutura de rede virtual (Neutron). E para isso, serão seguidos os seguintes passos:

### Passo 1: Autenticação

Criando o arquivo ```openrc``` com as credenciais de acesso ao OpenStack.

- Obtendo o endereço IP:
``` sh
juju status --format=yaml openstack-dashboard | grep public-address | awk '{print $2}' | head -1
```

- Obtendo a senha:
``` sh
juju run keystone/0 get-admin-password
```

### Passo 2: Horizon

Acessando o dashboard Horizon como administrador e mantendo ele aberto durante todo o setup do openstack para visualizar as mudanças que estão sendo feitas.

- Horizon_URL: https://#IP_obtido#/horizon
- User_Name: admin
- Password: #Senha_obtida#
- Domain_Name: admin_domain

??? Tarefa-1

    De um print das Telas abaixo:

        1. Do Status do JUJU
        2. Do Dashboard do MAAS com as máquinas.
        3. Da aba compute overview no OpenStack Dashboard.
        4. Da aba compute instances no OpenStack Dashboard.
        5. Da aba network topology no OpenStack Dashboard.

    ![Juju status](imgs/roteiro3-000.png)
    /// caption
    Figura 2 - Comando “juju status” no terminal.
    ///

    ![Dashboard do MAAS](imgs/roteiro3-001.png)
    /// caption
    Figura 3 - Dashboard do MAAS com as máquinas.
    ///

    ![Openstack overview](imgs/roteiro3-002.png)
    /// caption
    Figura 4 - Aba Compute Overview no OpenStack.
    ///

    ![Openstack instances](imgs/roteiro3-003.png)
    /// caption
    Figura 5 - Aba Compute Instances no OpenStack.
    ///

    ![Openstack network topology](imgs/roteiro3-004.png)
    /// caption
    Figura 6 - Aba Network Topoplogy no OpenStack.
    ///

### Passo 3: Imagens

- Instalando o cliente do Openstack no main via snap:
``` sh
sudo snap install openstackclients
```

- Carregando as credenciais em openrc:
``` sh
source ~/openstack-bundles/stable/openstack-base/openrc
```

- Verificando os serviços disponíveis no Openstack:
``` sh
openstack service list
```

- Fazendo ajustes na rede:
``` sh
juju config neutron-api enable-ml2-dns="true"
juju config neutron-api-plugin-ovn dns-servers="172.20.129.131"
```

- Procurando e importando a imagem do Ubuntu Jammy para o Glance:
```
curl http://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-
amd64.img \
    --output ~/cloud-images/jammy-amd64.img

openstack image create --public --container-format bare \
    --disk-format qcow2 --file ~/cloud-images/jammy-amd64.img \
    jammy-amd64
```

!!! warning "Não esquecer!"

    Fazer o clone do repositório ```openstack-bundles``` caso ainda não tenha feito.
    ``` sh
    git clone https://github.com/openstack-charmers/openstack-bundles
    ```

### Passo 4: Rede Externa

Configurando uma rede externa para conectar as VMs à rede física, usando uma faixa de alocação entre 172.16.7.0 e 172.16.8.255:

```
source ~/openstack-bundles/stable/openstack-base/openrc

openstack network create --external \
    --provider-network-type flat --provider-physical-network physnet1 \
    ext_net

openstack subnet create --network ext_net --no-dhcp \
    --gateway 172.16.0.1 --subnet-range 172.16.0.0/20 \
    --allocation-pool start=172.16.7.0,end=172.16.8.255 \
    ext_subnet
```

### Passo 5: Rede Interna e Roteador

Configurando uma rede interna para conectar as VMs à rede externa, usando a subnet 192.169.0.0/24 e sem DNS:

```
openstack network create int_net

openstack subnet create --network int_net \
   --gateway 192.169.0.1 --subnet-range 192.169.0.0/24 \
   --allocation-pool start=192.169.0.10,end=192.169.0.200 \
   int_subnet
```

Configurando o roteador:

```
openstack router create provider-router
openstack router set --external-gateway ext_net provider-router
openstack router add subnet provider-router int_subnet
```

### Passo 6: Flavors

- Criando os flavors (instance type) - **SEM ephemeral disk** para as VMs:

| Flavor Name | vCPUs | RAM (GB) | Disk |
| :---------: | :---: | :------: | :--: |
| `m1.tiny`   | 1     | 1        | 20   |
| `m1.small`  | 1     | 2        | 20   |
| `m1.medium` | 2     | 4        | 20   |
| `m1.large`  | 4     | 8        | 20   |
/// caption
Tabela 1 - Flavors criados e suas respectivas configurações. 
///

Comandos:
``` sh
openstack flavor create --ram 1000 --vcpus 1 --disk 20 m1.tiny
openstack flavor create --ram 2000 --vcpus 1 --disk 20 m1.small
openstack flavor create --ram 4000 --vcpus 2 --disk 20 m1.medium
openstack flavor create --ram 8000 --vcpus 4 --disk 20 m1.large
```

### Passo 7: Conexão

O key-pair da máquina onde está o MaaS já existe e é importado:

``` sh
openstack keypair create --public-key ~/.ssh/id_rsa.pub open_key
```

- Via o dashboard Horizon, como administrador, adicionando a liberação do SSH e ALL ICMP no security group default:

```
for i in $(openstack security group list | awk '/default/{ print $2 }'); do
   openstack security group rule create $i --protocol icmp --remote-ip 
0.0.0.0/0;    
   openstack security group rule create $i --protocol tcp --remote-ip 
0.0.0.0/0 --dst-port 22; 
done
```

### Passo 8: Instância

- Disparando uma instância m1.tiny com o nome client e sem Novo Volume:
```
openstack server create --image jammy-amd64 --flavor m1.tiny \
   --key-name open_key --network int_net \
   client
```

- Alocando um floating IP para a instância:
```
FLOATING_IP=$(openstack floating ip create -f value -c floating_ip_address ext_net)

openstack server add floating ip client $FLOATING_IP
```

- Testando a conexão SSH:
```
ssh ubuntu@$FLOATING_IP
```

??? Tarefa-2

    De um print das Telas abaixo:
        
        1. Do Dashboard do MAAS com as máquinas.
        2. Da aba compute overview no OpenStack.
        3. Da aba compute instances no OpenStack.
        4. Da aba network topology no OpenStack.

    Enumere as diferencas encontradas entre os prints das telas na Tarefa 1 e na Tarefa 2.

    Explique como cada recurso foi criado.

    ![Juju status](imgs/roteiro3-005.png)
    /// caption
    Figura 7 - Dashboard do MAAS com as máquinas.
    ///

    ![Dashboard do MAAS](imgs/roteiro3-006.png)
    /// caption
    Figura 8 - Aba Compute Overview no OpenStack.
    ///

    ![Openstack overview](imgs/roteiro3-007.png)
    /// caption
    Figura 9 - Aba Compute Instances no OpenStack.
    ///

    ![Openstack instances](imgs/roteiro3-008.png)
    /// caption
    Figura 10 - Aba Network Topoplogy no OpenStack.
    ///

    **Enumere as diferencas encontradas entre os prints das telas na Tarefa 1 e na Tarefa 2.**

    -  Compute overview: percebe-se que o número de Instâncias, VCPUs, Floating IPs, security groups e Routers atualizou para 1, pois eles foram criados. Além disso, a memória RAM agora está em 1GB, temos 6 rules dos security groups e a networks e a port atualizaram os valores, pois agora estamos utilizando instâncias; 
    -  Compute instances: criada a instância “cliente”; 
    -  Network topology: tem network, uma vez que foi criada a subnet e o roteador.

    **Explique como cada recurso foi criado.**

    Após importar as chaves de validação e imagem, e configurado a rede externa: 

    -  Rede interna e roteador: usar uma série de comandos para criar rede interna, subrede e roteador, colocando os devidos valores.
    ``` sh
    openstack network create int_net 
    openstack subnet create --network int_net --gateway 192.168.0.1 --subnet-range 192.168.0.0/24 --allocation-pool start=192.168.0.10,end=192.168.0.200 int_subnet 
    openstack router create provider-router 
    openstack router set --external-gateway ext_net provider-router 
    openstack router add subnet provider-router int_subnet
    ```

    -  Security Groups Rules: usar um comando 
    ``` sh
    openstack security group rule create
    ```

    - Instância: usar o comando a seguir passando a devida imagem, chave de validação e rede. 
    ``` sh
    openstack server create --image jammy-amd64 --flavor m1.small --key-name mykey --network int_net client
    ``` 

    -  Endereço de IP Flutuante: comandos abaixo, passando o nome do servidor criado anteriormente.  
    ``` sh
    FLOATING_IP=$(openstack floating ip create -f value -c floating_ip_address ext_net) openstack server add floating ip client $FLOATING_IP
    ```

### Escalando os nós

No OpenStack, escalar os nós de configuração é essencial para melhorar diversos aspectos dos serviços em um ambiente de nuvem. 

Alguns benefícios da escala dos nós de configuração:

- Aumento de capacidade de processamento;
- Alta disponibilidade e tolerância a falhas;
- Melhoria de desempenho e latência reduzida;
- Escalabilidade horizontal.

??? info

    Para mais detalhes sobre o escalonamento de nós no OpenStack, visitar o site da disciplina: [escalonando os nós](https://insper.github.io/computacao-nuvem/roteiros/3.setup/#escalando-os-nos){target=_blank}


Adicionando o nó reserva ao openstack no cluster como nó de computing e block storage.

- Fazer o release da máquina que está ALLOCATED no Dashboard do MaaS.

- Instalando o hypervisor, realizando o deploy na máquina:
``` sh
juju add-unit nova-compute
```

- Ao anotar o número da máquina adicionada no status, é instalado o block storage:
``` sh
juju add-unit --to <machine-id> ceph-osd
```

??? Tarefa-3

    Faça um desenho de como é a sua arquitetura de rede, desde a sua conexão com o Insper até a instância alocada.

    ![Desenho da arquitetura](imgs/roteiro3-009.png)
    /// caption
    Figura 11 - Desenho da arquitetura de rede, da conexão com o Insper até a instância alocada.
    ///


## App - Uso da infraestrutura

Levantar 4 instâncias em máquinas virtuais do OpenStack: 2 instâncias com a API do projeto, 1 instância com banco de dados e 1 instância com LoadBalancer (Nginx). Configurando-as como indicado a seguir:

![Topologia](imgs/roteiro3-topologia.png)
/// caption
Figura 12 - Topologia a ser configurada. Fonte: instruções da disciplina.
///

???+ Tarefa-4

    1. Escreva um relatório dos passos utilizados. 
    2. Anexe fotos e/ou diagramas contendo: arquitetura de rede da sua infraestrutura dentro do Dashboard do Openstack.
    3. Lista de VMs utilizadas com nome e IPs alocados 
    4. Print do Dashboard do Wordpress conectado via máquina Nginx/LB. 
    5. 4 Prints, cada um demonstrando em qual server  (máquina física) cada instancia foi alocado pelo OpenStack.

    **O relatório está presente na aba [Relatório - Tarefa 4.](relatorio.md)**

## Conclusão

Este roteiro apresentou, de forma prática e detalhada, o processo de implantação de uma nuvem privada utilizando o OpenStack em conjunto com ferramentas como MAAS, Juju e charms específicos. 

Foram abordadas todas as etapas necessárias para configurar a infraestrutura, desde a criação do controller até a implantação dos principais serviços OpenStack — incluindo redes (Neutron com SDN/OVN), armazenamento (Ceph e Cinder), autenticação (Keystone), dashboard (Horizon), imagens (Glance), computação (Nova) e banco de dados (MySQL). Além disso, destacou-se a importância do Vault na gestão de certificados e segurança. 

Ao longo do roteiro, também foram apresentadas atualizações dos comandos usados, refletindo versões mais recentes dos componentes. O conteúdo permite não apenas a compreensão dos conceitos fundamentais de nuvem privada, mas também oferece uma visão aplicada sobre redes virtuais baseadas em SDN.