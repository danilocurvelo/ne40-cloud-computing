# Lab: Automatizando a implantação de infraestrutura com Terraform

Duração: 1h15min

## Introdução

O Terraform permite que você crie, altere e melhore a infraestrutura com segurança e previsibilidade. É uma ferramenta de código aberto que codifica APIs em arquivos de configuração que podem ser compartilhados entre os membros da equipe, tratados como código, editados, revisados e versionados.

Neste roteiro de laboratório, você cria uma configuração do Terraform com um módulo para automatizar a implantação da infraestrutura do Google Cloud. Especificamente, você implanta uma rede de modo automático com uma regra de *firewall* e duas instâncias de VM, conforme mostrado neste diagrama:

![Projeto](imgs/terraform1.png)

## Objetivos

Neste roteiro de laboratório, você aprenderá a:

- Criar uma configuração para uma rede de modo automático
- Criar uma configuração para uma regra de *firewall*
- Criar um módulo para instâncias de VM
- Criar e implantar uma configuração Terraform
- Verificar a implantação de uma configuração

## Tarefas

### **Tarefa 1.** Configurar o Terraform e o Cloud Shell

Configure seu ambiente do Cloud Shell para usar o Terraform.

O Terraform agora está integrado ao Cloud Shell. Verifique qual versão está instalada.

1. No Cloud Console, clique em **Ativar o Cloud Shell**.

2. Para confirmar que o Terraform está instalado, execute o seguinte comando:

```
terraform --version
```

A saída deve ficar assim:

```
Terraform v1.2.3
```

> **Observação**: não se preocupe se você receber um aviso de que a versão do Terraform está desatualizada, pois as instruções do roteiro funcionarão com o Terraform v1.2.3 e posterior. Os downloads disponíveis para a versão mais recente do Terraform estão no site do Terraform. O Terraform é distribuído como um pacote binário para todas as plataformas e arquiteturas compatíveis, e o Cloud Shell usa Linux de 64 bits.

3. Para criar um diretório para sua configuração do Terraform, execute o seguinte comando:

```
mkdir tfinfra
```

4. No Cloud Shell, clique em **Abrir editor**.

5. No painel esquerdo do editor de código, expanda a pasta `tfinfra`.

O Terraform usa uma arquitetura baseada em *plug-ins* para dar suporte às inúmeras infraestruturas e provedores de serviços disponíveis. Cada "provedor" é seu próprio binário encapsulado distribuído separadamente pelo próprio Terraform. Inicialize o Terraform definindo o Google como provedor.

6. Para criar um novo arquivo dentro da pasta `tfinfra`, clique com o botão direito do mouse na pasta `tfinfra` e clique em **New File**.

7. Nomeie o novo arquivo como `provider.tf`.

8. Copie o código em `provider.tf`:

```
provider "google" {}
```

9. Salve o arquivo.

10. Para inicializar o Terraform, execute o seguinte comando:

```
cd tfinfra
terraform init
```

A saída deve parecer com isso:

```
...
Terraform has been successfully initialized!
...
```

Agora você está pronto para trabalhar com o Terraform no Cloud Shell.

### **Tarefa 2.** Criar `mynetwork` e seus recursos

Crie a rede de modo automático `mynetwork` junto com sua regra de *firewall* e duas instâncias de VM (`mynet_us_vm` e `mynet_eu_vm`).

1. Para criar um novo arquivo dentro do `tfinfra`, clique com o botão direito do mouse na pasta `tfinfra` e clique em **New File**.

2. Nomeie o novo arquivo `mynetwork.tf` e abra-o.

3. Copie o seguinte código base em `mynetwork.tf`:

```
# Create the mynetwork network
resource [RESOURCE_TYPE] "mynetwork" {
name = [RESOURCE_NAME]
#RESOURCE properties go here
}
```

Este modelo básico é um ótimo ponto de partida para qualquer recurso do Google Cloud. O campo de nome permite nomear o recurso e o campo de tipo permite especificar o recurso do Google Cloud que você deseja criar. Você também pode definir propriedades, mas elas são opcionais para alguns recursos.

4. Em `mynetwork.tf`, substitua `[RESOURCE_TYPE]` por **"google_compute_network"** (com as aspas).

> **Observação**: o recurso **google_compute_network** é uma rede VPC. Os recursos disponíveis estão na [documentação do provedor do Google Cloud](https://www.terraform.io/docs/providers/google/index.html). Saiba mais sobre esse recurso específico na [documentação do Terraform](https://www.terraform.io/docs/providers/google/r/compute_network).

5. Em `mynetwork.tf`, substitua `[RESOURCE_NAME]` por **"mynetwork"** (com as aspas).

6. Adicione a seguinte propriedade a `mynetwork.tf`:

```
auto_create_subnetworks = true
```

Por definição, uma rede de modo automático cria automaticamente uma sub-rede em cada região. Portanto, você está configurando **auto_create_subnetworks** como **true**.

7. Verifique se `mynetwork.tf` se parece com isso:

```
# Create the mynetwork network
resource "google_compute_network" "mynetwork" {
name = "mynetwork"
auto_create_subnetworks = true
}
```

8. Salve o arquivo `mynetwork.tf`.

Defina uma regra de *firewall* para permitir tráfego HTTP, SSH, RDP e ICMP em **mynetwork**.

9. Adicione o seguinte código base a `mynetwork.tf`:

```
# Add a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on mynetwork
resource [RESOURCE_TYPE] "mynetwork-allow-http-ssh-rdp-icmp" {
name = [RESOURCE_NAME]
#RESOURCE properties go here
}
```

10. Em `mynetwork.tf`, substitua `[RESOURCE_TYPE]` por **"google_compute_firewall"** (com as aspas).

> **Observação**: o recurso **google_compute_firewall** é uma regra de *firewall*. Saiba mais sobre esse recurso específico na [documentação do Terraform](https://www.terraform.io/docs/providers/google/r/compute_firewall).

11. Em `mynetwork.tf`, substitua `[RESOURCE_NAME]` por **"mynetwork-allow-http-ssh-rdp-icmp"** (com as aspas).

12. Adicione a seguinte propriedade a `mynetwork.tf`:

```
network = google_compute_network.mynetwork.self_link
```

> **Observação**: como essa regra de *firewall* depende de sua rede, você está usando a referência **google_compute_network.mynetwork.self_link** para instruir o Terraform a resolver esses recursos em uma ordem dependente. Nesse caso, a rede é criada antes da regra de *firewall*.

13. Adicione as seguintes propriedades a `mynetwork.tf`:

```
allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
    }
allow {
    protocol = "icmp"
    }
source_ranges = ["0.0.0.0/0"]
```

A lista de regras de **permissão** especifica quais protocolos e portas são permitidos.

14. Verifique se suas adições ao `mynetwork.tf` se parecem com isso:

```
# Add a firewall rule to allow HTTP, SSH, RDP, and ICMP traffic on mynetwork
resource "google_compute_firewall" "mynetwork-allow-http-ssh-rdp-icmp" {
name = "mynetwork-allow-http-ssh-rdp-icmp"
network = google_compute_network.mynetwork.self_link
allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
    }
allow {
    protocol = "icmp"
    }
source_ranges = ["0.0.0.0/0"]
}
```

15. Salve o arquivo.

### **Tarefa 3.** Criar as VMs

Defina as instâncias de VM criando um módulo de instância de VM. Um módulo é uma configuração reutilizável dentro de uma pasta. Você usará este módulo para ambas as instâncias de VM deste roteiro.

1. Para criar uma nova pasta dentro do `tfinfra`, selecione a pasta `tfinfra` e clique em **New Folder**.

2. Nomeie a nova pasta de **instance**.

3. Para criar um novo arquivo dentro de `instance`, clique com o botão direito do mouse na pasta `instance` e clique em **New File**.

4. Nomeie o novo arquivo de `main.tf` e abra-o.

Você deve ter a seguinte estrutura de diretórios e arquivos no Cloud Shell:

```
tfinfra/
├─ instance/
│  ├─ main.tf
├─ mynetwork.tf
├─ provider.tf
```

5. Copie o seguinte código base em `main.tf`:

```
resource [RESOURCE_TYPE] "vm_instance" {
  name = [RESOURCE_NAME]
  #RESOURCE properties go here
}
```

6. Em `main.tf`, substitua `[RESOURCE_TYPE]` por **"google_compute_instance"** (com as aspas).

> **Observação**: o recurso **google_compute_instance** é uma instância do Compute Engine. Saiba mais sobre esse recurso específico na [documentação do Terraform](https://www.terraform.io/docs/providers/google/r/compute_instance).

7. Em `main.tf`, substitua `[RESOURCE_NAME]` por `"${var.instance_name}"` (com as aspas).

Como você usará este módulo para ambas as instâncias de VM, você está definindo o nome da instância como uma variável de entrada. Isso permite que você controle o nome da variável de `mynetwork.tf`. Saiba mais sobre variáveis de entrada no Terraform: [Define Input Variables Guide](https://learn.hashicorp.com/terraform/getting-started/variables).

8. Adicione as seguintes propriedades a `main.tf`:

```
zone         = "${var.instance_zone}"
machine_type = "${var.instance_type}"
```

Essas propriedades definem a zona e o tipo de máquina da instância como variáveis de entrada.

9. Adicione as seguintes propriedades a `main.tf`:

```
boot_disk {
    initialize_params {
        image = "debian-cloud/debian-9"
    }
}
```

Esta propriedade define o disco de inicialização para usar a imagem do sistema operacional Debian 9. Como ambas as instâncias de VM usarão a mesma imagem, você pode codificar essa propriedade no módulo.

10. Adicione as seguintes propriedades a `main.tf`:

```
network_interface {
    network = "${var.instance_network}"
    access_config {
        # Allocate a one-to-one NAT IP to the instance
    }
}
```  

Esta propriedade define a interface de rede fornecendo o nome da rede como variável de entrada e a configuração de acesso. Deixar a configuração de acesso vazia resulta em um endereço IP externo efêmero (obrigatório neste roteiro). Para criar instâncias com apenas um endereço IP interno, remova a seção `access_config`. Para obter mais informações, consulte a [documentação do Terraform](https://www.terraform.io/docs/providers/google/r/compute_instance).

11. Defina as 4 variáveis de entrada na parte superior de `main.tf` e verifique se `main.tf` se parece com isso, incluindo colchetes `{}`:

```
variable "instance_name" {}
variable "instance_zone" {}
variable "instance_type" {
  default = "n1-standard-1"
  }
variable "instance_network" {}
resource "google_compute_instance" "vm_instance" {
  name         = "${var.instance_name}"
  zone         = "${var.instance_zone}"
  machine_type = "${var.instance_type}"
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
      }
  }
  network_interface {
    network = "${var.instance_network}"
    access_config {
      # Allocate a one-to-one NAT IP to the instance
    }
  }
}
```

Ao dar a `instance_type` um valor padrão, você torna a variável opcional. O `instance_name`, `instance_zone` e `instance_network` são obrigatórios e você os definirá em `mynetwork.tf`.

12. Salve o arquivo `main.tf`.

13. Adicione as seguintes instâncias de VM a `mynetwork.tf`:

```
# Create the mynet-us-vm instance
module "mynet-us-vm" {
  source           = "./instance"
  instance_name    = "mynet-us-vm"
  instance_zone    = "us-central1-a"
  instance_network = google_compute_network.mynetwork.self_link
}
# Create the mynet-eu-vm" instance
module "mynet-eu-vm" {
  source           = "./instance"
  instance_name    = "mynet-eu-vm"
  instance_zone    = "europe-west1-d"
  instance_network = google_compute_network.mynetwork.self_link
}
```

Esses recursos estão aproveitando o módulo na pasta `instance` e fornecem o nome, a zona e a rede como entradas. Como essas instâncias dependem de uma rede VPC, você está usando a referência **google_compute_network.mynetwork.self_link** para instruir o Terraform a resolver esses recursos em uma ordem dependente. Nesse caso, a rede é criada antes da instância.

> **Observação**: o benefício de escrever um módulo Terraform é que ele pode ser reutilizado em várias configurações. Em vez de escrever seu próprio módulo, você também pode aproveitar os módulos existentes no [repositório de módulos do Terraform](https://registry.terraform.io/browse?provider=google&verified=true).

14. Salve o arquivo `mynetwork.tf`.

### **Tarefa 4.** Aplicar as configurações com o Terraform

É hora de aplicar a configuração `mynetwork`.

1. Para reescrever os arquivos de configuração do Terraform para um formato e estilo canônicos, execute o seguinte comando:

```
terraform fmt
```

A saída deve ser:

```
mynetwork.tf
```

> **Observação**: Se você teve erros revisite os passos anteriores.

2. Para inicializar o Terraform, execute o seguinte comando:

```
terraform init
```

A saída deve parecer com isso:

```
...
Terraform has been successfully initialized!
...
```

3. Para criar um plano de execução, execute o seguinte comando:

```
terraform plan
```

A saída deve parecer com isso:

```
...
Plan: 4 to add, 0 to change, 0 to destroy.
...
```

Terraform determinou que os 4 recursos a seguir precisam ser adicionados:

| Nome | Descrição |
|---|---|
| mynetwork | Rede VPC |
| mynetwork-allow-http-ssh-rdp-icmp | Regra de firewall |
| mynet-us-vm | Instância de VM em us-central1-a |
| mynet-eu-vm | Instância de VM em europe-west1-d |

4. Para aplicar as alterações desejadas, execute o seguinte comando:

```
terraform apply
```

5. Para confirmar as ações planejadas, digite:

```
yes
```

A saída deve se parecer com:

```
...
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
```

### **Tarefa 5.** Verifique sua implantação

No Cloud Console, verifique se os recursos foram criados.

1. No Cloud Console, no menu **Navegação**, clique em **Rede VPC > Redes VPC**.

2. Visualize a rede VPC **mynetwork** com uma sub-rede em cada região.

3. No menu Navegação, clique em **Rede VPC > Firewall**.

4. Ordene as regras de *firewall* por rede.

5. Veja a regra de *firewall* **mynetwork-allow-http-ssh-rdp-icmp** para **mynetwork**.

6. No menu **Navegação**, clique em **Compute Engine > Instâncias de VM**.

7. Veja as instâncias **mynet-us-vm** e **mynet-eu-vm**.

8. Anote o endereço IP interno para **mynet-eu-vm**.

9. Para **mynet-us-vm**, clique em **SSH** para iniciar um terminal e se conectar.

10. Para testar a conectividade com o endereço IP interno de **mynet-eu-vm**, execute o seguinte comando no terminal SSH (substituindo o endereço IP interno de **mynet-eu-vm** pelo valor observado anteriormente):

```
ping -c 3 <IP interno de mynet-eu-vm aqui>
```

> **Observação**: isso deve funcionar porque ambas as instâncias de VM estão na mesma rede e a regra de *firewall* permite tráfego ICMP!

### **Tarefa 6.** Revisão

Neste roteiro de laboratório, você criou uma configuração do Terraform com um módulo para automatizar a implantação da infraestrutura do Google Cloud. Conforme sua configuração muda, o Terraform pode criar planos de execução incrementais, o que permite que você construa sua configuração geral passo a passo.

Você pode aproveitar a configuração e o módulo que criou como ponto de partida para implantações futuras.

### **Tarefa 7.** Excluindo os recursos criados pelo Terraform

1. Para remover todos os recursos criados pela configuração que criamos no Terraform, no diretório `tfinfra` digite o seguinte comando:

```
terraform destroy
```

2. Para confirmar as ações planejadas, digite:

```
yes
```

A saída deve se parecer com:

```
...
Destroy complete! Resources: 4 destroyed.
```
