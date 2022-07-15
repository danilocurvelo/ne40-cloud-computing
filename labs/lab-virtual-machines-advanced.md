# Lab: Trabalhando com Virtual Machines

Duração: 2h

## Introdução

Neste roteiro laboratório, você configura um servidor de um jogo — um servidor de [Minecraft](https://www.minecraft.net/pt-br).

O *software* do servidor de Minecraft será executado em uma instância do **Compute Engine**.

Você irá utilizar um tipo de máquina `e2-medium` que inclui um disco de inicialização de 10 GB, 2 CPUs virtuais (vCPU) e 4 GB de RAM. Este tipo de máquina executa o Debian Linux por padrão.

Para garantir que haja bastante espaço para os dados do servidor de Minecraft, você também irá anexar um SSD persistente de 50 GB de alto desempenho à instância. Este servidor dedicado de Minecraft pode suportar até 50 jogadores.

## Objetivos

Neste roteiro de laboratório, você aprenderá a realizar as seguintes tarefas:

- Personalizar um servidor real para produção;
- Instalar e configurar o *software* necessário;
- Configurar acesso à rede;
- Agendar backups regulares.

## Tarefas

### **Tarefa 1:** Crie a VM

#### Definir uma VM usando opções avançadas

1. No Cloud Console, no menu **Navegação**, clique em **Compute Engine > Instâncias de VM**.

2. Clique em **Criar instância**.

3. Especifique o seguinte e deixe as configurações restantes como padrão:

| Propriedade | Valor |
|---|---|
| Nome | mc-server |
| Região | us-central1 |
| Zona | us-central1-a |
| Disco de inicialização | Debian GNU/Linux 9 (stretch) |
| Identidade e acesso à API > Escopos de acesso | Definir o acesso para cada API |
| Armazenamento | Leitura e gravação |

4. Clique em **Gerenciamento, segurança, discos, rede, locatário único**.

5. Clique em **Discos**. Você adicionará um disco a ser usado para armazenamento de dados do jogo.

6. Clique em **Adicionar novo disco**.

7. Especifique o seguinte e deixe as configurações restantes como padrão:

| Propriedade | Valor |
|---|---|
| Nome | minecraft-disk |
| Tipo de disco | Disco permanente SSD |
| Tipo de origem do disco | Disco em branco |
| Tamanho (GB) | 50 |
| Criptografia| Chave de criptografia gerenciada pelo Google |

8. Clique em **Salvar**. Isso cria o disco e o anexa automaticamente à VM quando a VM é criada.

9. Clique em **Rede**.

10. Especifique o seguinte e deixe as configurações restantes como padrão:

| Propriedade | Valor |
|---|---|
| Tags de rede | minecraft-server |
| Interfaces de rede | Clique em *default* para editar |
| IP externo | Criar endereço IP |
| Nome | mc-server-ip |

11. Clique em **Reservar**.

12. Clique em **Concluir**.

13. Clique em **Criar**.

### **Tarefa 2:** Prepare o disco de dados

#### Formate e monte o disco em um diretório criado

O disco está anexado à instância, mas ainda não está montado ou formatado.

1. Para **mc-server**, clique em **SSH** para abrir um terminal e se conectar.

2. Para criar um diretório que serve como ponto de montagem (*mount point*) para o disco de dados, execute o seguinte comando:

```bash
sudo mkdir -p /home/minecraft
```

3. Para formatar o disco, execute o seguinte comando:

```bash
sudo mkfs.ext4 -F -E lazy_itable_init=0,\
lazy_journal_init=0,discard \
/dev/disk/by-id/google-minecraft-disk
```
Resultado (não copie; esta é uma saída de exemplo):

```
mke2fs 1.43.4 (31-Jan-2017)
Discarding device blocks: done                            
Creating filesystem with 26214400 4k blocks and 6553600 inodes
Filesystem UUID: 4e510141-a86f-4973-b8e6-caacc8b87926
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (131072 blocks): done
Writing superblocks and filesystem accounting information: done  
```

4. Para montar o disco, execute o seguinte comando:

```bash
sudo mount -o discard,defaults /dev/disk/by-id/google-minecraft-disk /home/minecraft
```

Nenhuma saída é exibida após a montagem do disco.

### **Tarefa 3:** Instale e execute a aplicação

O servidor Minecraft é executado na *Java Virtual Machine* (JVM), e portanto, requer que o *Java Runtime Environment* (JRE) seja instalado. Como o servidor não precisa de uma interface gráfica com o usuário, você usa a versão *headless* do JRE. Isso reduz o uso de recursos do JRE na máquina, o que ajuda a garantir que o servidor Minecraft tenha espaço suficiente para expandir seu próprio uso de recursos, se necessário.

#### Instale o *Java Runtime Environment* (JRE) e o servidor Minecraft

1. No terminal **SSH** para **mc-server**, atualize os repositórios Debian na VM, executando o seguinte comando:

```bash
sudo apt-get update
```

2. Depois que os repositórios forem atualizados, para instalar o JRE headless execute o seguinte comando:

```
sudo apt-get install -y default-jre-headless
```

3. Para navegar até o diretório em que o disco permanente está montado, execute o seguinte comando:

```
cd /home/minecraft
```

4. Para instalar o wget, execute o seguinte comando:

```
sudo apt-get -y install wget
```

5. Para baixar o arquivo JAR do servidor Minecraft (versão 1.11.2), execute o seguinte comando:

```
sudo wget https://launcher.mojang.com/v1/objects/d0d0fe2b1dc6ab4c65554cb734270872b72dadd6/server.jar
```

#### Inicialize o servidor Minecraft

1. Para inicializar o servidor Minecraft, execute o seguinte comando:

```
sudo java -Xmx1024M -Xms1024M -jar server.jar nogui
```

Resultado (não copie; esta é a saída de exemplo):

```
[21:01:54] [main/ERROR]: Failed to load properties from file: server.properties
[21:01:54] [main/WARN]: Failed to load eula.txt
[21:01:54] [main/INFO]: You need to agree to the EULA in order to run the server. Go to eula.txt for more info.
```

> O servidor Minecraft não será executado a menos que você aceite os termos do *End User Licensing Agreement* (EULA).

2. Para confirmar que os arquivos que foram criados na primeira inicialização do servidor Minecraft, execute o seguinte comando:

```
sudo ls -l
```

> Você pode editar o arquivo `server.properties` para alterar o comportamento padrão do servidor Minecraft.

3. Para editar o EULA, execute o seguinte comando:

```
sudo nano eula.txt
```

4. Altere a última linha do arquivo de `eula=false` para `eula=true`

5. Pressione `Ctrl+O`, `ENTER` para salvar o arquivo e pressione `Ctrl+X` para sair do `nano`.

> Não tente reiniciar o servidor Minecraft ainda. Iremos utilizar uma técnica diferente na próxima inicialização.

#### Crie um terminal virtual para inicializar o servidor Minecraft

Se você iniciar o servidor Minecraft novamente agora, ele estará vinculado à vida útil da sua sessão SSH: ou seja, se você fechar o terminal SSH, o servidor também será encerrado. Para evitar esse problema, você pode usar o `screen`, um utilitário que permite criar um terminal virtual que pode ser "desanexado", tornando-se um processo em segundo plano (*background*), ou "reanexado", tornando-se um processo em primeiro plano (*foreground*). Quando um terminal virtual é desconectado em segundo plano, ele será executado independentemente de você estar conectado ou não.

1. Para instalar o `screen`, execute o seguinte comando:

```
sudo apt-get install -y screen
```

2. Para iniciar seu servidor Minecraft em um terminal virtual `screen`, execute o seguinte comando: 

```
sudo screen -S mcs java -Xmx1024M -Xms1024M -jar server.jar nogui
```

Resultado (não copie; esta é a saída de exemplo):

```
...
[11:34:59] [Server-Worker-1/INFO]: Preparing spawn area: 87%
[11:34:59] [Server-Worker-1/INFO]: Preparing spawn area: 90%
[11:35:00] [Server-Worker-1/INFO]: Preparing spawn area: 91%
[11:35:00] [Server-Worker-1/INFO]: Preparing spawn area: 93%
[11:35:01] [Server-Worker-1/INFO]: Preparing spawn area: 95%
[11:35:01] [Server-Worker-1/INFO]: Preparing spawn area: 96%
[11:35:02] [Server-Worker-1/INFO]: Preparing spawn area: 98%
[11:35:02] [Server-Worker-1/INFO]: Preparing spawn area: 99%
[11:35:03] [Server thread/INFO]: Time elapsed: 54556 ms
[11:35:03] [Server thread/INFO]: Done (90.152s)! For help, type "help"
```

#### Desconecte-se do `screen` e feche sua sessão SSH

1. Para desanexar o terminal do `screen`, pressione `Ctrl+A`, `Ctrl+D`. O terminal virtual continua a ser executado em segundo plano. Para reconectar o terminal, execute o seguinte comando:

```
sudo screen -r mcs
```

2. Se necessário, saia do terminal `screen` pressionando `Ctrl+A`, `Ctrl+D`.

3. Para sair do terminal SSH, execute o seguinte comando:

```
exit
```

> Parabéns! Você configurou e personalizou uma VM e instalou e configurou um *software* de uma aplicação - um servidor Minecraft!

### **Tarefa 4:** Permita o tráfego de clientes

Até este ponto, o servidor tem um endereço IP estático externo, mas não pode receber tráfego porque não há uma regra de *firewall* em vigor. O servidor Minecraft usa a porta **TCP 25565** por padrão. Então você precisa configurar uma regra de *firewall* para permitir essas conexões.

#### Crie uma regra de *firewall*

1. No Cloud Console, no menu **Navegação**, clique em **Rede VPC > Firewall**.

2. Clique em **Criar regra de firewall**.

3. Especifique o seguinte e deixe as configurações restantes como padrão:

| Propriedade | Valor |
|---|---|
| Nome | minecraft-rule |
| Destinos | Tags de destino especificadas |
| Tags de destino | minecraft-server |
| Filtro de origem | Intervalos IPv4 |
| Intervalos IPv4 de origem | `0.0.0.0/0` |
| Protocolos e portas | Portas e protocolos especificados |

4. Para **tcp**, especifique a porta **25565**.

5. Clique em **Criar**. Os usuários agora podem acessar seu servidor a partir de seus clientes Minecraft.

#### Verifique a disponibilidade do servidor

1. No painel esquerdo, clique em **Endereços IP**.

2. Localize e copie o endereço IP externo para a VM `mc-server`.

3. Use o seguinte site para testar seu servidor Minecraft: [https://mcsrvstat.us/](https://mcsrvstat.us/)

### **Tarefa 5:** Agende *backups* periódicos

Fazer *backup* dos dados do sua aplicação é uma atividade comum e recomendada. Nesse caso, você irá configurar o sistema para fazer *backup* dos dados do Minecraft no **Cloud Storage**.

#### Crie um *bucket* do Cloud Storage

1. No menu **Navegação**, clique em **Compute Engine > Instâncias de VM**.

2. Na instância **mc-server**, clique em **SSH**.

3. Crie um nome globalmente exclusivo para seu *bucket* e armazene-o na variável de ambiente `YOUR_BUCKET_NAME`. Para torná-lo único, você pode usar, por exemplo, seu ID do projeto. Execute o seguinte comando:

```
export YOUR_BUCKET_NAME=<Digite o nome do seu bucket aqui>
```

4. Verifique com `echo`:

```
echo $YOUR_BUCKET_NAME
```

5. Para criar o *bucket* usando a ferramenta `gsutil`, parte do Cloud SDK, execute o seguinte comando:

```
gsutil mb gs://$YOUR_BUCKET_NAME-minecraft-backup
```

> Se esse comando falhar, talvez você não tenha criado um nome de *bucket* exclusivo. Nesse caso, escolha outro nome de *bucket*, atualize sua variável de ambiente e tente criar o *bucket* novamente.

> Para tornar esta variável de ambiente permanente, você pode adicioná-la ao arquivo `.profile` do usuário root executando este comando: `echo YOUR_BUCKET_NAME=$YOUR_BUCKET_NAME >> ~/.profile`

#### Crie um *script* de *backup*

1. No terminal **SSH** do **mc-server**, navegue até seu diretório inicial:

```
cd/home/minecraft
```

2. Para criar o *script*, execute o seguinte comando:

```
sudo nano /home/minecraft/backup.sh
```

3. Copie e cole o seguinte *script* no arquivo:

```
#!/bin/bash
screen -r mcs -X stuff '/save-all\n/save-off\n'
/usr/bin/gsutil cp -R ${BASH_SOURCE%/*}/world gs://${YOUR_BUCKET_NAME}-minecraft-backup/$(date "+%Y%m%d-%H%M%S")-world
screen -r mcs -X stuff '/save-on\n'
```

4. Pressione `Ctrl+O`, `ENTER` para salvar o arquivo e pressione `Ctrl+X` para sair do `nano`.

> O script salva o estado atual dos dados do servidor e pausa a funcionalidade de salvamento automático do servidor. Em seguida, ele faz *backup* do diretório de dados do servidor (`world`) e coloca seu conteúdo em um diretório com um rótulo de data/hora (`<timestamp>-world`) no *bucket* do Cloud Storage. Depois que o *script* terminar de fazer o *backup* dos dados, ele retoma o salvamento automático no servidor Minecraft.

5. Para tornar o *script* executável, execute o seguinte comando:

```
sudo chmod 755 /home/minecraft/backup.sh
```

#### Teste o *script* de *backup* e agende um *cron job*

1. No terminal **SSH** do **mc-server**, execute o *script* de *backup*:

```
. /home/minecraft/backup.sh
```

2. Após a conclusão do *script*, retorne ao Cloud Console.

3. Para verificar se o arquivo de *backup* foi gravado, no menu **Navegação**, clique em **Armazenamento > Navegador**.

4. Clique no nome do *bucket* de *backup*. Você deve ver uma pasta com um nome e rótulo de data e hora. Agora que você verificou se os *backups* estão funcionando, você pode agendar um *cron job* para automatizar a tarefa.

5. No terminal **SSH** do **mc-server**, abra a tabela *cron* para edição:

```
sudo crontab -e
```

6. Quando você for solicitado a selecionar um editor, digite o número correspondente ao `nano` e pressione `ENTER`.

7. Na parte inferior da tabela *cron*, cole a seguinte linha:

```
0 */4 * * * /home/minecraft/backup.sh
```

> Essa linha instrui o *cron* a executar *backups* a **cada 4 horas**.

8. Pressione `Ctrl+O`, `ENTER` para salvar a tabela cron e pressione `Ctrl+X` para sair do nano.

> Isso cria cerca de 300 *backups* por mês no Cloud Storage, portanto, você deve gerencia-los e/ou excluí-los regularmente para evitar cobranças. O Cloud Storage oferece o recurso *Object Lifecycle Management* para definir um *Time to Live* (TTL) para objetos ou para arquivar versões mais antigas de objetos.

### **Tarefa 6:** Manutenção do servidor

Para realizar a manutenção do servidor, você precisa desligar o servidor.

#### Conecte-se via SSH ao servidor, pare-o e desligue a VM

1. No terminal **SSH** do **mc-server**, execute o seguinte comando:

```
sudo screen -r -X stuff '/stop\n'
```

2. No Cloud Console, no menu **Navegação**, clique em **Compute Engine > Instâncias de VM**.

3. Clique em **mc-server**.

4. Clique em **Parar**.

5. Na caixa de diálogo de confirmação, clique em **Parar** para confirmar. Você será desconectado da sua sessão SSH.

> Para inicializar sua instância novamente, visite a página da instância e clique em **Iniciar**. Para iniciar o servidor Minecraft novamente, você pode estabelecer uma conexão SSH com a instância, remontar seu disco permanente e iniciar seu servidor Minecraft em um novo terminal do `screen`, exatamente como fez anteriormente.

#### Automatize a manutenção do servidor com *scripts* de inicialização e desligamento

Em vez de seguir o processo manual para montar o disco permanente e iniciar a aplicação do servidor em um `screen`, você pode usar *scripts* para criar rotinas de inicialização e de desligamento para fazer isso por você.

1. Clique em **mc-server**.

2. Clique em **Editar**.

3. Em **metadados**, especifique dois novos itens:

| Chave | Valor |
|---|---|
| startup-script-url | https://storage.googleapis.com/cloud-training/archinfra/mcserver/startup.sh |
| shutdown-script-url | https://storage.googleapis.com/cloud-training/archinfra/mcserver/shutdown.sh |

> Você terá que clicar em **Adicionar item** para adicionar o `shutdown-script-url`. Quando você reinicia sua instância, o *script* de inicialização monta automaticamente o disco do Minecraft no diretório apropriado, inicia seu servidor Minecraft em uma sessão do `screen` e desconecta a sessão. Quando você interrompe a instância, o *script* de desligamento encerra o servidor Minecraft antes que a instância seja encerrada. É uma prática recomendada armazenar esses *scripts* no Cloud Storage.

4. Clique em **Salvar**.

### **Tarefa 7:** Revisão

Neste roteiro de laboratório, você criou uma instância de máquina virtual personalizada instalando o *software* base (um JRE *headless*) e o *software* de uma aplicação (um servidor de jogos Minecraft). Você personalizou a VM anexando e preparando um disco de dados SSD de alta velocidade e reservou um IP externo estático para que o endereço permanecesse consistente. Então você verificou a disponibilidade do seu servidor de jogos *online*. Você configurou um sistema de *backup* para fazer *backup* dos dados do servidor em um *bucket* do Cloud Storage e testou o sistema de *backup*. Então você automatizou os *backups* usando o *cron*. Por fim, você configurou *scripts* de manutenção usando metadados para inicialização e desligamento do servidor.

## Finalize a sua atividade de laboratório

Exclua todas as instâncias de VM, discos de armazenamento, regras de *firewall* e alocação de IPs criados para este roteiro de laboratório.

- Em **Compute Engine > instâncias de VM**, exclua **mc-server**.
- Em **Compute Engine > Storage > Discos**, exclua o disco **minecraft-disk**.
- Em **Rede VPC > Firewall**, exclua a regra criada (**minecraft-rule**).
- Em **Rede VPC > Endereços IP**, libere o IP estático criado (**mc-server-ip**).
