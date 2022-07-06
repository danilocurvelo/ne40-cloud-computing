# Lab: Balanceadores de carga de rede

Duração: 45min

## Introdução

Neste roteiro de laboratório, você aprenderá a configurar um balanceador de carga de rede para seus aplicativos executados em máquinas virtuais (VMs) do Compute Engine.

Há várias maneiras de [balancear a carga no Google Cloud](https://cloud.google.com/load-balancing/docs/load-balancing-overview#a_closer_look_at_cloud_load_balancers). Este laboratório mostra a configuração do seguinte balanceador de carga:

- [Balanceador de carga de rede](https://cloud.google.com/compute/docs/load-balancing/network/)

Você é encorajado a digitar os comandos por conta própria, o que pode ajudá-lo a aprender os conceitos básicos. 

## Objetivos

Neste roteiro de laboratório, você aprenderá a realizar as seguintes tarefas:

- Configurar um balanceador de carga de rede.
- Obter experiência prática aprendendo o funcionamento do balanceador de carga de rede.

## Tarefas

### **Tarefa 1:** Defina a região e a zona padrão para todos os recursos

1. No Cloud Shell, defina a zona padrão:

```
gcloud config set compute/zone us-central1-a
```

2. Defina a região padrão:

```
gcloud config set compute/region us-central1
```

> Saiba mais sobre como escolher zonas e regiões [aqui](https://cloud.google.com/compute/docs/zones).

### **Tarefa 2.** Crie várias instâncias de servidores *web*

Para este cenário de balanceamento de carga, crie três instâncias de VM do Compute Engine e instale o Apache nelas. Em seguida, adicione uma regra de *firewall* que permita que o tráfego HTTP alcance as instâncias.

1. Crie três novas máquinas virtuais em sua zona padrão e dê a todas a mesma *tag* de rede. O código fornecido define a zona como `us-central1-a`. Definir o campo de *tags* permite que você faça referência a essas instâncias de uma só vez, como com uma regra de *firewall*. Esses comandos também instalam o Apache em cada instância e dão a cada instância uma página inicial exclusiva.

```
gcloud compute instances create www1 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www1</h1></body></html>' | tee /var/www/html/index.html"
```

```
gcloud compute instances create www2 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www2</h1></body></html>' | tee /var/www/html/index.html"
```

```
gcloud compute instances create www3 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www3</h1></body></html>' | tee /var/www/html/index.html"
```

2. Crie uma regra de *firewall* para permitir tráfego externo para as instâncias de VM:

```
gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80
```

Agora você precisa obter os endereços IP externos de suas instâncias e verificar se elas estão em execução.

3. Execute o seguinte comando para listar suas instâncias. Você verá seus endereços IP na coluna `EXTERNAL_IP`:

```
gcloud compute instances list
```

4. Verifique se cada instância está sendo executada usando o utilitário `curl`, substituindo `[IP_ADDRESS]` pelo endereço IP de cada uma de suas VMs:

```
curl http://[IP_ADDRESS]
```

Você verá o código HTML da página requisistada.

### **Tarefa 3.** Configure o serviço de balanceamento de carga

Ao configurar o serviço de balanceamento de carga, suas instâncias de máquina virtual receberão pacotes destinados ao endereço IP externo estático que você configurar. As instâncias feitas com uma imagem do *Compute Engine* são configuradas automaticamente para lidar com esse endereço IP.

> Para obter mais informações, consulte [Visão geral do balanceamento de carga de rede TCP/UDP externa](https://cloud.google.com/compute/docs/load-balancing/network/).

1. Crie um endereço IP externo estático para seu balanceador de carga:

```
gcloud compute addresses create network-lb-ip-1 \
 --region us-central1
```

Saída similar esperada:

```
Created [https://www.googleapis.com/compute/v1/projects/psychic-upgrade-195615/regions/us-central1/addresses/network-lb-ip-1].
```

2. Adicione um recurso de verificação de integridade:

```
gcloud compute http-health-checks create basic-check
```

> **Observação:** O Google Cloud fornece mecanismos de verificação de integridade que determinam se as instâncias de *back-end* respondem corretamente ao tráfego. Veja mais [aqui](https://cloud.google.com/load-balancing/docs/health-checks?hl=pt_br).

3. Adicione um *pool* de destino na mesma região que suas instâncias. Execute o seguinte para criar o *pool* de destino e use a verificação de integridade, necessária para que o serviço funcione:

```
gcloud compute target-pools create www-pool \
    --region us-central1 --http-health-check basic-check
```

> **Observação:** Um *pool* de destino define um grupo de instâncias que receberá o tráfego de entrada do balanceador de carga.

4. Adicione as instâncias ao *pool*:

```
gcloud compute target-pools add-instances www-pool \
    --instances www1,www2,www3
```

5. Adicione a seguinte regra de encaminhamento:

```
gcloud compute forwarding-rules create www-rule \
    --region us-central1 \
    --ports 80 \
    --address network-lb-ip-1 \
    --target-pool www-pool
```

> **Observação:** Uma regra de encaminhamento e seu respectivo endereço IP representam a configuração de *front-end* de um balanceador de carga do Google Cloud. A regra especifica para qual *pool* de destinos o tráfego será encaminhado.

### **Tarefa 4.** Enviando tráfego para suas instâncias

Agora que o serviço de balanceamento de carga está configurado, você pode começar a enviar tráfego para a regra de encaminhamento e observar o tráfego ser disperso para diferentes instâncias.

1. Digite o seguinte comando para visualizar o endereço IP externo da regra de encaminhamento `www-rule` usada pelo balanceador de carga:

```
gcloud compute forwarding-rules describe www-rule --region us-central1
```

2. Use o comando `curl` para acessar o endereço IP externo, substituindo `IP_ADDRESS` pelo endereço IP externo do comando anterior:

```
while true; do curl -m1 IP_ADDRESS; done
```

A resposta do comando `curl` alterna aleatoriamente entre as três instâncias. Se sua resposta não for bem-sucedida inicialmente, aguarde aproximadamente 30 segundos para que a configuração seja totalmente carregada e para que suas instâncias sejam marcadas como íntegras antes de tentar novamente.

3. Use **Ctrl + c** para parar de executar o comando.

## Finalize a sua atividade de laboratório

Exclua todos os recursos criados para essa atividade.

- Em **Rede > Serviços de rede > Balanceamento de carga**, exclua `www-pool`. Marque também para remoção da verificação de integridade `basic-check`.
- Em **Rede VPC > Endereços IP**, exclua `network-lb-ip-1` (liberar endereço estático).
- Em **Rede VPC > Firewall**, exclua `www-firewall-network-lb`.
- Em **Compute Engine > Instâncias de VM**, exclua todas as VMs criadas (`www1`, `www2` e `www3`).