# Lab: Virtual Machines (VM)

Duração: 1h30min

## Introdução

Neste roteiro de laboratório, você explorará as opções de instâncias de máquina virtual e criará várias VMs com características diferentes.

## Objetivos

Neste roteiro de laboratório, você explorará as opções disponíveis para VMs e verá as diferenças entre elas.

Neste laboratório, você aprenderá a realizar as seguintes tarefas:

- Criar várias VMs padrão
- Criar VMs avançadas

## Tarefas

### **Tarefa 1:** Criar uma máquina virtual

#### Crie a VM

1. No Cloud Console, no menu **Navegação**, clique em **Compute Engine > Instâncias de VM**.

2. Clique em **Criar instância**.

3. Em **Nome**, digite um nome para sua instância. Passe o mouse sobre o ícone de ponto de interrogação para obter conselhos sobre o que constitui um nome formado corretamente.

4. Para **Região** e **Zona**, selecione **us-central1** e **us-central1-c**, respectivamente.

5. Para **configuração da máquina**, selecione **Série** como **N1**.

6. Para **Tipo de máquina**, examine as opções.

> Observe que o menu lista o número de vCPUs, a quantidade de memória e um nome simbólico, como `n1-standard-1`. O nome simbólico é o parâmetro que você usaria para selecionar o tipo de máquina se estivesse criando uma VM através do comando `gcloud`. Observe à direita da zona que há um custo estimado por mês.

7. Clique em **Detalhes** à direita de **Zona** para ver o detalhamento dos custos estimados.

8. Para **tipo de máquina**, selecione **n1-standard-4 (4 vCPU, 15 GB de memória)**. Como o custo mudou?

9. Para **tipo de máquina**, selecione **n1-standard-1 (1 vCPU, 3,75 GB de memória)**.

10. Para **disco de inicialização**, clique em **Mudar**.

11. Clique em **Versão** e selecione **Debian GNU/Linux 10 (buster)**.

12. Clique em **Selecionar**.

13. Clique em **Rede, discos, segurança, gerenciamento, locatário único**.

14. Clique em **Rede**.

15. Para **interfaces de rede**, clique na seta para abrir as configurações.

16. Selecione **Nenhum** para **IP externo**.

17. Clique em **Concluir**.

18. Deixe as configurações restantes como seus padrões e clique em **Criar**. Aguarde até que a nova VM seja criada.

#### Explore os detalhes da VM

1. Na página **instâncias de VM**, clique no nome de sua VM.

2. Localize **Plataforma da CPU** e anote o valor. Clique em **Editar**.

Observe que você não pode alterar o tipo de máquina, a plataforma da CPU ou a zona.

Você pode adicionar tags de rede e permitir tráfego de rede específico da Internet por meio de *firewalls*.

Algumas propriedades de uma VM são parte integrante da VM, são estabelecidas na sua criação e não podem ser alteradas. Outras propriedades podem ser editadas. Você pode adicionar discos adicionais e também determinar se o disco de inicialização será excluído quando a instância for excluída. Normalmente, o disco de inicialização é excluído automaticamente quando a instância é excluída. Mas às vezes você vai querer substituir esse comportamento. Esse recurso é muito importante porque você não pode criar uma imagem de um disco de inicialização quando ele estiver anexado a uma instância em execução. Portanto, você precisaria desabilitar **Excluir disco de inicialização quando a instância for excluída** para permitir a criação de uma imagem do sistema a partir do disco de inicialização.

3. Examine as políticas de disponibilidade.

Você não pode converter uma instância não preemptiva em uma preemptiva. Essa escolha deve ser feita na criação da VM. Uma instância preemptiva pode ser interrompida a qualquer momento e está disponível a um custo menor.

Se uma VM for interrompida por qualquer motivo (por exemplo, uma interrupção ou uma falha de *hardware*), o recurso de reinicialização automática a iniciará novamente. É este o comportamento que você deseja? Seus aplicativos são idempotentes (escritos para lidar com uma segunda inicialização corretamente)?

Durante a manutenção do *host*, a VM é definida para migração. No entanto, você pode encerrar a VM em vez de migrar.

Se você fizer alterações, às vezes elas podem levar vários minutos para serem implementadas, especialmente se envolverem alterações de rede, como adicionar *firewalls* ou alterar o IP externo.

4. Clique em **Cancelar**.

#### Explore os *logs* de uma VM

1. Na página de **detalhes da instância de VM** para sua VM, clique em **Cloud Logging**.

Observe que agora você navegou para a página do **Cloud Logging**.

Esta é uma visualização de *log* estruturada. Na parte superior, você pode filtrar usando os menus suspensos e há uma caixa de pesquisa para pesquisar com base em rótulos (*labels*) ou texto.

2. Clique na pequena seta à esquerda de uma das linhas para ver o tipo de informação que contém.

### **Tarefa 2.** Cria uma máquina virtual Windows

#### Crie a VM

1. No Cloud Console, no menu **Navegação**, clique em **Compute Engine > Instâncias de VM**.

2. Clique em **Criar instância**.

3. Especifique o seguinte e deixe as configurações restantes como padrão:

| Propriedade | Valor |
|---|---|
| Nome | Escolha um nome para a sua VM |
| Região | europe-west1 |
| Zona | europe-west1-c |
| Série | N1 |
| Tipo de márquina | n1-standard-2 (2 vCPUs, 7,5 GB de memória) |
| Disco de inicialização | Mudar |
| Imagens públicas > Sistema operacional | Windows Server |
| Versão | Windows Server 2016 Datacenter Core |
| Tipo de disco de inicialização | Disco permanente SSD |
| Tamanho (GB) | 100 |

4. Clique em **Selecionar**.

5. Para **Firewall**, habilite **Permitir tráfego HTTP** e **Permitir tráfegos HTTPS**.

6. Clique em **Criar**.

Quando a VM estiver em execução, observe que a opção de conexão na coluna da direita é RDP, não SSH. RDP é o *Remote Desktop Protocol*. Você precisaria do cliente RDP instalado em sua máquina local para se conectar à área de trabalho do Windows.

> **Observação:** a instalação de um cliente RDP em sua máquina local está fora do escopo deste laboratório e da aula. Se você tem o Windows na sua máquina local, você já tem o cliente nativamente. Em outros SO, você deve baixar algum cliente de terceiros. Existe um cliente RDP para o Chrome [aqui](https://chrome.google.com/webstore/detail/chrome-rdp-for-google-clo/mpbbnannobiobpnfblimoapbephgifkm?hl=en-US).


#### Defina a senha para a VM

1. Clique no nome da sua VM do Windows para acessar os detalhes da instância da VM.

2. Você não tem uma senha válida para esta VM do Windows: você não consegue fazer o login nessa VM sem uma senha. Clique em **Configurar a senha do Windows**.

3. Clique em **Definir**.

4. Copie a senha fornecida e clique em **Fechar**.

Na página de instâncias de VM, você pode clicar em RDP para baixar um arquivo `.rdp` e importar no seu cliente.

### **Tarefa 3.** Cria uma máquina virtual personalizada

#### Crie a VM

1. No Cloud Console, no menu **Navegação**, clique em **Compute Engine > Instâncias de VM**.

2. Clique em **Criar instância**.

3. Especifique o seguinte e deixe as configurações restantes como padrão:

| Propriedade | Valor |
|---|---|
| Nome | Escolha um nome para a sua VM |
| Região | us-west1 |
| Zona | us-west1-b |
| Série | N1 |
| Tipo de márquina | Personalizado |
| Núcleos | 6 vCPU |
| Memória | 32 GB |
| Disco de inicialização| Debian GNU/Linux 11 (bullseye) |

4. Clique em **Criar**.

#### Conecte-se via SSH à sua VM personalizada

1. Para a VM personalizada que você acabou de criar, clique em **SSH**.

2. Para ver informações sobre memória usada e disponível e espaço de *swap* em sua VM personalizada, execute o seguinte comando:

```bash
free
```

3. Para ver detalhes sobre a RAM instalada em sua VM, execute o seguinte comando:

```
sudo dmidecode -t 17
```

4. Para verificar o número de processadores, execute o seguinte comando:

```
nproc
```

5. Para ver detalhes sobre as CPUs instaladas em sua VM, execute o seguinte comando:

```
lscpu
```

6. Para sair do terminal SSH, execute o seguinte comando:

```
exit
```

### **Tarefa 4:** Revisão

Neste roteiro de laboratório, você criou várias instâncias de máquina virtual de tipos e características diferentes. Uma era uma pequena VM utilitária para fins de administração. Você também criou uma VM padrão e uma VM personalizada. Você também iniciou VMs com SO Windows e Linux.

## Finalize a sua atividade de laboratório

Exclua todas as instâncias de VM criadas para este roteiro de laboratório.