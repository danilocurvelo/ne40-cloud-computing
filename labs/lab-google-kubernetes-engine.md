# Lab: Implantando o Google Kubernetes Engine

Duração: 1h

## Introdução

Neste roteiro de laboratório, você usará o Cloud Console para criar *clusters* do GKE e implantar um *pod* de exemplo.

## Objetivos

Neste roteiro de laboratório, você aprenderá a realizar as seguintes tarefas:

- Use o Cloud Console para criar e manipular *clusters* do GKE;
- Use o Cloud Console para implantar um *pod*;
- Use o Cloud Console para examinar o *cluster* e os *pods*.

## Tarefas

### **Tarefa 1.** Implantar *cluster* do GKE

Nesta tarefa, você usa o Cloud Console e o Cloud Shell para implantar *clusters* do GKE.

1. No Cloud Console, no menu **Navegação**, clique em **Kubernetes Engine > Clusters**.

2. Clique em **Criar** para começar a criar um *cluster* do GKE. Na opção de *cluster* **GKE Standard**, selecione **configurar** na próxima tela.

3. Examine essa tela e os controles para alterar o nome do *cluster*, o local do *cluster*, a versão do Kubernetes, o número de nós e os recursos do nó, como o tipo de máquina no *pool* de nós padrão.

Os *clusters* podem ser criados em uma região ou em uma única zona. Uma única zona é o padrão. Quando você implanta em uma região, os nós são implantados em três zonas separadas e o número total de nós implantados será três vezes maior.

3. Altere o nome do *cluster* para **standard-cluster-1** e a zona para **us-central1-a**. Deixe todos os valores em seus padrões e clique em **Criar**.

O *cluster* começa o provisionamento.

> **Observação**: você precisa aguardar alguns minutos para que a implantação do *cluster* seja concluída.

5. Clique no *cluster* **standard-cluster-1** para ver os detalhes do *cluster*.

6. Você pode rolar a página para baixo para ver mais detalhes.

7. Clique nas guias **Armazenamento** e **Nós** sob o nome do *cluster* (**standard-cluster-1**) para ver mais detalhes do *cluster*.

### **Tarefa 2.** Implantar *cluster* do GKE

É fácil modificar muitos dos parâmetros dos *clusters* existentes usando o Cloud Console ou o Cloud Shell. Nesta tarefa, vamos usar o Cloud Console para modificar o tamanho dos *clusters* do GKE.

1. No Cloud Console, clique em **Nós** na parte superior da página de detalhes do *cluster* **standard-cluster-1**.

2. Na seção **Pools de nós**, clique em **default-pool**.

3. No Cloud Console, clique em **REDIMENSIONAR** na parte superior da página de **detalhes do pool de nós**.

4. Altere o número de nós de 3 para 4 e clique em **REDIMENSIONAR**.

5. No Cloud Console, no menu **Navegação**, clique em **Kubernetes Engine > Clusters**.

Quando a operação for concluída, a página **Kubernetes Engine > Clusters** deverá mostrar que o **standard-cluster-1** agora tem quatro nós.

### **Tarefa 3.** Implante um exemplo de carga de trabalho

Nesta tarefa, usando o Cloud Console, você implantará um *pod* para executar um servidor proxy conhecido como **nginx** como um exemplo de carga de trabalho.

1. No Cloud Console, no menu **Navegação**, clique em **Kubernetes Engine > Cargas de trabalho**.

2. Clique em **Implantar** para mostrar o assistente de criação.

3. Clique em **Continuar** para aceitar a imagem de contêiner padrão, `nginx:latest`, que implanta 3 *pods*, cada um com um único contêiner executando a versão mais recente do **nginx**.

4. Role até a parte inferior da janela e clique no botão **Implantar** deixando os detalhes de **configuração** nos padrões.

5. Quando a implantação for concluída, sua tela será atualizada para mostrar os detalhes de sua nova implantação do nginx.

### **Tarefa 4.** Veja detalhes sobre as cargas de trabalho no Cloud Console

Nesta tarefa, você visualizará os detalhes das cargas de trabalho do GKE diretamente no Cloud Console.

1. No Cloud Console, no menu **Navegação**, clique em **Kubernetes Engine > Cargas de trabalho**.

2. Clique em **nginx-1**.

> **Observação**: você pode ver os pods (3/3), pois a implantação padrão começará com três *pods*, mas será reduzida para 1 após alguns minutos. Você pode continuar com o roteiro de laboratório.

Isso exibe as informações gerais da carga de trabalho mostrando detalhes como gráficos de utilização de recursos, links para *logs* e detalhes dos *pods* associados a essa carga de trabalho.

3. No Cloud Console, clique na guia **Detalhes** da carga de trabalho **nginx-1**. A guia **Detalhes** mostra mais detalhes sobre a carga de trabalho, incluindo a especificação do *pod*, o número e o *status* das réplicas do *pod* e detalhes sobre o escalonador automático horizontal de *pods*.

4. Clique na guia Histórico de revisões. Isso exibe uma lista das revisões que foram feitas nessa carga de trabalho.

5. Clique na guia **Eventos**. Esta guia lista os eventos associados a esta carga de trabalho.

6. E, em seguida, a guia **YAML**. Essa guia fornece o arquivo YAML completo que define esses componentes e a configuração completa desse exemplo de carga de trabalho.

7. Ainda na guia **Detalhes** do Cloud Console para a carga de trabalho **nginx-1**, clique na guia **Visão geral**, role para baixo até a seção **Pods gerenciados** e clique no nome de um dos *pods* para visualizar a página de detalhes desse *pod*.

> Observação: a implantação padrão começará com três *pods*, mas será reduzida para 1 após alguns minutos, portanto, talvez seja necessário atualizar a página Visão geral para garantir que você tenha um *pod* válido para inspecionar.

8. A página de detalhes do *pod* fornece informações sobre a configuração do *pod*, a utilização de recursos e o nó em que o *pod* está em execução.

9. Na página de detalhes do *pod*, você pode clicar nas guias **Eventos** e **Registros** para visualizar detalhes do evento e links para registros de contêiner no *Cloud Operations*.

10. Clique na guia **YAML** para ver o arquivo YAML detalhado para a configuração do *pod*.

## Finalize a sua atividade de laboratório

Exclua o *cluster* criado para essa atividade.

- Em **Kubernetes Engine > Clusters**, exclua **standard-cluster-1**.
