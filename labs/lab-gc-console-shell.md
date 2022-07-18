# Lab: Google Cloud Console e Shell

Duração: 1h

## Introdução

Neste laboratório, você se familiarizará com a interface *web* do Google Cloud. Existem dois ambientes integrados: um ambiente com GUI (interface gráfica) chamado **Cloud Console** e uma CLI (interface de linha de comando) chamada **Cloud Shell**. Neste laboratório, você usará os dois ambientes.

## Objetivos

O objetivo deste laboratório é tornar o ambiente do Google Cloud Console e do Google Cloud Shell mais familiar ao usuário.

## Tarefas

### **Tarefa 1.** Use o Cloud Console para criar um *bucket*

Nesta tarefa, você cria um *bucket*. Não se importe tanto na definição do *bucket*, pois nosso propósito aqui é se familiarizar com a interface do Cloud Console.

1. No Cloud Console, no **menu de navegação** (três barras horizontais), clique em **Cloud Storage > Navegador**.

2. Clique em **Criar bucket**.

3. Nomeie o seu *bucket* com um nome globalmente exclusivo; deixe todos os outros valores como seus padrões.

4. Clique em **Criar**.

O menu do Google Cloud contém um ícone de **Notificações** (sino). Às vezes, o *feedback* dos comandos "por baixo dos panos" é fornecido lá. Se você não tiver certeza do que está acontecendo, verifique Notificações para obter informações adicionais e também o histórico.

Fique a vontade para explorar ainda mais o Google Cloud Console! Independente do serviço utilizado (GCP, AWS, Azure), todos eles oferecem recursos muito similares.

### **Tarefa 2.** Acesse o Cloud Shell

Nesta tarefa, você explorará o Cloud Shell e alguns de seus recursos. 

Você pode usar o Cloud Shell para gerenciar projetos e recursos por meio da linha de comando sem precisar instalar o Cloud SDK e outras ferramentas em seu computador.

O Cloud Shell oferece o seguinte:

- VM temporária do Compute Engine;
- Acesso a linha de comando da instância por meio de um navegador;
- 5 GB de armazenamento persistente em disco (diretório `$HOME`);
- SDK do Cloud pré-instalado e outras ferramentas;
- `gcloud`: para trabalhar com o Compute Engine e muitos serviços do Google Cloud;
- `gsutil`: para trabalhar com o Cloud Storage;
- `kubectl`: para trabalhar com o Google Kubernetes Engine e o Kubernetes;
- `bq`: para trabalhar com o BigQuery;
- Suporte de linguagem para Java, Go, Python, Node.js, PHP e Ruby;
- Autorização integrada para acesso a recursos e instâncias.

Saiba mais sobre o Cloud Shell na documentação do [Google Cloud Shell](https://cloud.google.com/shell/docs).

> **Observação:** após 1 hora de inatividade, a instância do Cloud Shell é reciclada. Apenas o diretório `/home` persiste. Quaisquer alterações feitas na configuração do sistema, incluindo variáveis de ambiente, são perdidas entre as sessões.

#### Abra o Cloud Shell e explore os recursos

1. No menu do Google Cloud, clique em **Ativar o Cloud Shell** (canto superior direito, ao lado de Notificações). O Cloud Shell é aberto na parte inferior da janela do Cloud Console.

Há três botões na extremidade direita da barra de ferramentas do Cloud Shell:

- Minimizar/restaurar: o primeiro minimiza ou restaura a janela, dando acesso total ao Cloud Console sem fechar o Cloud Shell.
- Abrir em uma nova janela: ter o Cloud Shell na parte inferior do Console do Cloud é útil quando você está emitindo comandos individuais. No entanto, às vezes você estará editando arquivos ou deseja ver a saída completa de um comando. Para esses usos, esse botão exibe o Cloud Shell em uma janela de terminal de tamanho normal.
- Fechar terminal: este botão fecha o Cloud Shell. Sempre que você fecha o Cloud Shell, a máquina virtual é resetada e todo o contexto da máquina é perdido.

2. Feche o Cloud Shell.

### **Tarefa 3.** Use o Cloud Shell para criar um *bucket* do Cloud Storage

1. Abra o **Cloud Shell** novamente.

2. Use o comando `gsutil` para criar outro *bucket*. Substitua `<BUCKET_NAME>` por um nome globalmente exclusivo (você pode concatenar um 2 ao nome do *bucket* globalmente exclusivo usado anteriormente):

```bash
gsutil mb gs://<BUCKET_NAME>
```

3. No Cloud Console, no menu Navegação, clique em **Cloud Storage > Navegador** ou clique em **Atualizar** se você já estiver no navegador. O segundo *bucket* deve ser exibido na lista de *buckets*.

> **Observação:** você executou ações equivalentes usando o **Cloud Console** e o **Cloud Shell**. Você criou um *bucket* usando o Cloud Console e outro *bucket* usando o Cloud Shell.

### **Tarefa 4.** Explore mais recursos do Cloud Shell

#### Realize o *upload* de um arquivo

1. Abra o **Cloud Shell**.

2. Clique no botão **Mais** na barra de ferramentas do Cloud Shell para exibir mais opções.

3. Clique em **Fazer upload**. Faça o *upload* de qualquer arquivo da sua máquina local para a VM do Cloud Shell. Este arquivo será referenciado neste roteiro como `[MY_FILE]`.

4. No Cloud Shell, digite `ls` para confirmar que o arquivo foi enviado.

5. Copie o arquivo em um dos buckets que você criou anteriormente no laboratório. Substitua `[MY_FILE]` pelo arquivo que você enviou e `[BUCKET_NAME]` por um dos nomes de seus *buckets*:

```
gsutil cp [MY_FILE] gs://[BUCKET_NAME]
```

> **Observação:** você fez *upload* de um arquivo para a VM do Cloud Shell e depois o copiou para um *bucket*.

6. Explore as opções disponíveis no Cloud Shell clicando em diferentes ícones na barra de ferramentas.

7. No Cloud Console, em **Cloud Storage > Navegador**, clique no *bucket* em que foi enviado o arquivo e confirme que de fato o arquivo está armazenado.

8. Feche todas as sessões do Cloud Shell, pois existe um limite de horas semanais que pode ser utilizado gratuitamente.

### **Tarefa 5.** Crie um estado persistente no Cloud Shell

Nesta seção, você aprenderá uma prática recomendada para usar o Cloud Shell. O comando `gcloud` geralmente exige que você especifique valores como **região**, **zona** ou **ID do projeto**. Digitá-los repetidamente aumenta a chance de cometer erros. Se você usa o Cloud Shell com frequência, convém definir valores comuns em variáveis de ambiente e usá-los em vez de digitar os valores reais.

#### Identifique as regiões disponíveis

1. Abra o Cloud Shell. Observe que isso aloca uma nova VM para você.

2. Para listar as regiões disponíveis, execute o seguinte comando:

```
gcloud compute regions list
```

3. Selecione uma região da lista e anote o valor em qualquer editor de texto. Esta região agora será chamada de `[YOUR_REGION]` no restante deste roteiro de laboratório.

#### Criar e ler uma variável de ambiente

1. Crie uma variável de ambiente e substitua `[YOUR_REGION]` pela região selecionada na etapa anterior:

```bash
INFRACLASS_REGION=[YOUR_REGION]
```

2. Leia desta variável com o comando `echo`:

```bash
echo $INFRACLASS_REGION
```

3. Você pode usar variáveis de ambiente como esta em comandos `gcloud` para reduzir as possibilidades de erros de digitação e para que você não precise se lembrar de muitas informações detalhadas.

> **Observação:** sempre que você fecha o Cloud Shell e o reabre, uma nova VM é alocada e a variável de ambiente que você acabou de definir desaparece. Nas próximas etapas, você irá criar um arquivo para definir o valor para que não seja necessário inserir o comando sempre que o Cloud Shell for reiniciado.

#### Incluir a variável de ambiente através de um arquivo

1. Crie um diretório para os materiais usados neste laboratório:

```bash
mkdir infraclass
```

2. Crie um arquivo chamado `config` no diretório `infraclass`:

```bash
touch infraclass/config
```

3. Concatene o valor da sua variável de ambiente `$INFRACLASS_REGION` ao arquivo de configuração:

```bash
echo INFRACLASS_REGION=$INFRACLASS_REGION >> ~/infraclass/config
```

4. Crie uma segunda variável de ambiente para o ID do projeto, substituindo `[YOUR_PROJECT_ID]` pelo ID do projeto. Você pode encontrar o ID do projeto na página inicial do Cloud Console.

```bash
INFRACLASS_PROJECT_ID=[YOUR_PROJECT_ID]
```

Concatene o valor da variável de ambiente do ID do projeto ao arquivo de configuração:

```bash
echo INFRACLASS_PROJECT_ID=$INFRACLASS_PROJECT_ID >> ~/infraclass/config
```

Use o comando `source` para definir as variáveis de ambiente e use o comando `echo` para verificar se a variável do projeto foi definida:

```bash
source infraclass/config
echo $INFRACLASS_PROJECT_ID
```

> **Observação:** isso fornece um método para criar variáveis de ambiente e  facilmente recriá-las se o Cloud Shell for reiniciado. No entanto, você ainda precisará se lembrar de executar o comando `source` sempre que o Cloud Shell for aberto. Na próxima etapa, você modificará o arquivo `.profile` para que o comando de origem seja emitido automaticamente sempre que um terminal para o Cloud Shell for aberto.

#### Modificar o `bash profile` para criar persistência

1. Edite o arquivo `.profile` do shell com o seguinte comando:

```bash
nano .profile
```

> Se você tiver familiaridade com o shell, fique a vontade para utilizar outro editor de texto, como por exemplo o `vi(m)`.

2. Adicione a seguinte linha ao final do arquivo:

```bash
source infraclass/config
```

3. Pressione **Ctrl+O, ENTER** para salvar o arquivo e, em seguida, pressione **Ctrl+X** para sair do `nano`.

4. Feche e reabra o Cloud Shell para reiniciar a VM.

5. Use o comando `echo` para verificar se a variável ainda está definida:

```bash
echo $INFRACLASS_PROJECT_ID
```

Agora você deve ver o valor esperado que você definiu no arquivo de configuração.

> **Observação:** se o ambiente do Cloud Shell estiver corrompido, as instruções sobre como reseta-lo estão na documentação do Cloud Shell intitulado [ Disabling or Resetting Cloud Shell](https://cloud.google.com/shell/docs/resetting-cloud-shell). Isso fará com que tudo no ambiente do Cloud Shell seja resetado para o estado padrão original.

### **Tarefa 6.** Revisão da interface do Google Cloud

O Cloud Shell é um excelente ambiente interativo para explorar o Google Cloud usando comandos do SDK, como `gcloud` e `gsutil`.

Você pode instalar o **Google Cloud SDK** em um computador ou em uma instância de VM no Google Cloud. Os comandos `gcloud` e `gsutil` podem ser automatizados usando uma linguagem de script como `bash` (Linux) ou `Powershell` (Windows). Você também pode explorar usando as ferramentas de linha de comando no Cloud Shell e, em seguida, usar os parâmetros como um guia de implementação no SDK usando uma das linguagens de programação compatíveis.

A interface do Google Cloud consiste em duas partes: o Cloud Console e o Cloud Shell.

O Cloud Console:

- Fornece uma maneira rápida de executar tarefas;
- Apresenta opções para você, em vez de exigir que você as conheça;
- Executa a validação nos bastidores antes de enviar os comandos.

O Cloud Shell fornece:

- Controle detalhado;
- Uma gama completa de opções e recursos;
- Um caminho para a automação por meio de scripts.

### Finalize a sua atividade de laboratório

 No Cloud Console, em **Cloud Storage > Navegador**, exclua os dois *buckets* criados para esta atividade. Feche também o Cloud Shell.
