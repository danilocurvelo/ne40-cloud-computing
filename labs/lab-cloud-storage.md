# Lab: Cloud Storage e Cloud SQL

Duração: 1h

## Introdução

Neste roteiro de laboratório, você irá criar um *bucket* do **Cloud Storage** e armazenar um arquivo nele. Você também configurará um aplicativo em execução no **Compute Engine** para usar um banco de dados gerenciado pelo **Cloud SQL**. Para este laboratório, você configurará um servidor *web* com PHP, um ambiente de desenvolvimento que é a base para alguns sistemas *web*. Fora deste laboratório, você usará técnicas análogas para configurar esses pacotes.

Você também irá configurar o servidor *web* para fazer referência ao arquivo armazenado no *bucket* do Cloud Storage.

## Objetivos

Neste roteiro de laboratório, você aprenderá a realizar as seguintes tarefas:

- Criar um *bucket* do Cloud Storage e colocar uma imagem nele.
- Criar uma instância do Cloud SQL e configura-la.
- Conectar-se à instância do Cloud SQL de um servidor *web*.
- Usar a imagem do *bucket* do Cloud Storage em uma página *web*.

## Tarefas

### **Tarefa 1.** Implantar uma instância de VM para um servidor da *web*

1. No Cloud Console, no menu **Navegação**, clique em **Compute Engine > Instâncias de VM**.

2. Clique em **Criar instância**.

3. Na página **Criar instância**, para **Nome**, digite `bloghost`.

4. Para **Região** e **Zona**, selecione a região e a zona de sua escolha. Verifique que o preço pode mudar dependendo da região/zona escolhida.

5. Para **Tipo de máquina**, aceite o padrão.

6. Para **disco de inicialização**, selecione **Debian GNU/Linux 9 (stretch)**.

7. Deixe os **padrões de identidade e acesso à API** inalterados.

8. Para **Firewall**, clique em Permitir tráfego HTTP.

9. Clique em **Rede, discos, segurança, gerenciamento, locatário único** para abrir essa seção.

10. Clique em **Administração** para abrir essa seção.

11. Role para baixo até a seção **Automação** e insira o seguinte *script* como o valor para o **script de inicialização**:

```
apt-get update
apt-get install apache2 php php-mysql -y
service apache2 restart
```

> **Observação**: certifique-se de fornecer esse *script* como o valor do campo **script de inicialização**. Se você colocá-lo acidentalmente em outro campo, ele não será executado quando a instância de VM for iniciada.

12. Deixe as configurações restantes como seus padrões e clique em **Criar**.

> **Observação**: a instância pode levar cerca de dois minutos para ser iniciada e estar totalmente disponível para uso.

13. Na página de **instâncias de VM**, copie os endereços IP internos e externos da instância de VM do `bloghost` em um editor de texto para uso posterior neste roteiro.

### **Tarefa 2.** Crie um *bucket* do Cloud Storage usando a linha de comando `gsutil`

Todos os nomes de *bucket* do Cloud Storage precisam ser globalmente exclusivos. Para garantir que o nome do *bucket* seja exclusivo, estas instruções orientarão você a dar ao *bucket* o mesmo nome que o ID do projeto do Cloud Platform, que também é globalmente exclusivo.

Os *buckets* do Cloud Storage podem ser associados a uma região ou a um local multirregional: EUA, UE ou ÁSIA. Nesta atividade, você associa seu *bucket* à multirregião mais próxima da região e zona que você criou sua VM.

1. No menu do Google Cloud Platform, clique em **Ativar o Cloud Shell**. 

2. Por conveniência, insira o local escolhido em uma variável de ambiente chamada `LOCATION`. Digite um destes comandos:

```
export LOCATION=US
```

ou

```
export LOCATION=EU
```

ou

```
export LOCATION=ASIA
```

3. No Cloud Shell, a variável de ambiente `DEVSHELL_PROJECT_ID` contém o ID do projeto. Digite este comando para criar um *bucket* com o nome do ID do seu projeto:

```
gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID
```

4. Copie para seu *bucket* a imagem `.png` do banner que iremos utilizar de um local públic do Cloud Storage:

```
gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
```

5. Modifique a ACL do objeto que você acabou de criar para que tenha permissão de leitura por todos:

```
gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
```

### **Tarefa 4.** Crie a instância do Cloud SQL

1. No Cloud Console, no menu **Navegação**, clique em **SQL**.

2. Clique em **Criar instância**.

3. Para **mecanismo de banco de dados**, selecione **MySQL**.

4. Para **ID da instância**, digite `blog-db` e para **Senha** digite uma senha de sua escolha.

> Nota: Escolha uma senha que você se lembre. Não há necessidade de ocultar a senha porque você usará mecanismos de conexão que não são de acesso aberto a todos.

5. Selecione **Única zona** e defina a região e a zona igual ao da VM criada.

> **Observação**: O melhor desempenho é alcançado colocando o cliente e o banco de dados estão próximos um do outro.

6. Clique em **Criar instância**.

> **Observação**: aguarde a conclusão da implantação da instância. Vai demorar alguns minutos.

7. Clique no nome da instância, **blog-db**, para abrir sua página de detalhes.

8. Na página de detalhes das instâncias do SQL, copie o endereço IP público da sua instância SQL em um editor de texto para uso posterior neste roteiro.

9. Clique no menu **Usuários** no lado esquerdo e clique em **ADICIONAR CONTA DE USUÁRIO**.

10. Para Nome de usuário, digite `blogdbuser`

11. Para Senha, digite uma senha de sua escolha. Tome nota disso.

12. Clique em **ADICIONAR** para adicionar a conta de usuário no banco de dados.

> Nota: Aguarde a criação do usuário.

13. Clique na guia **Conexões** e, em seguida, clique em **Adicionar rede**.

> Nota: O botão Adicionar rede pode ficar acinzentado se a criação da conta de usuário ainda não estiver concluída.

14. Para **Nome**, digite `web front end`

15. Para **Rede**, digite o endereço IP externo de sua instância de VM do **bloghost**, seguido por `/32`

O resultado será similar a isso:

```
35.192.208.2/32
```

> Observação: certifique-se de usar o endereço IP externo de sua instância de VM seguido por /32. Não use o endereço IP interno da instância de VM. Também não use o endereço IP do nosso exemplo acima.

16. Clique em **Concluir** para concluir a definição da rede autorizada.

17. Clique em **Salvar** para salvar a alteração de configuração.

### **Tarefa 4.** Configurar uma aplicação em uma instância do Compute Engine para usar o Cloud SQL

1. No menu **Navegação**, clique em **Compute Engine > Instâncias de VM**.

2. Na lista de instâncias de VM, clique em **SSH** na linha da sua instância de VM para **bloghost**.

3. Em sua sessão **ssh** no **bloghost**, altere seu diretório de trabalho para a raiz de documentos do servidor *web*:

```
cd /var/www/html
```

4. Use o editor de texto **nano** (ou outro de sua preferência) para editar o arquivo `index.php`:

```
sudo nano index.php
```

5. Copie o seguinte conteúdo para este arquivo:

```
<html>
<head><title>Welcome to my excellent blog</title></head>
<body>
<h1>Welcome to my excellent blog</h1>
<?php
 $dbserver = "CLOUDSQLIP";
$dbuser = "blogdbuser";
$dbpassword = "DBPASSWORD";
// In a production blog, we would not store the MySQL
// password in the document root. Instead, we would store it in a
// configuration file elsewhere on the web server VM instance.
$conn = new mysqli($dbserver, $dbuser, $dbpassword);
if (mysqli_connect_error()) {
        echo ("Database connection failed: " . mysqli_connect_error());
} else {
        echo ("Database connection succeeded.");
}
?>
</body></html>
```

> **Observação**: em uma etapa posterior, você inserirá o endereço IP da instância do Cloud SQL e a senha do banco de dados nesse arquivo. Por enquanto, deixe o arquivo inalterado.

6.. Pressione **Ctrl+O** e, em seguida, pressione **Enter** para salvar o arquivo editado.

7. Pressione **Ctrl+X** para sair do editor de texto **nano**.

8. Reinicie o servidor *web*:

```
sudo service apache2 restart
```

9. Abra uma nova guia do navegador e cole na barra de endereço o IP externo da instância de VM do seu bloghost seguido por `/index.php`. A URL ficará assim:

```
35.192.208.2/index.php
```

> **Observação**: certifique-se de usar o endereço IP externo de sua instância de VM seguido por `/index.php`. Não use o endereço IP interno da instância de VM. Também não use o endereço IP do exemplo acima.

Ao carregar a página, você verá que seu conteúdo inclui uma mensagem de erro começando com as palavras:

```
Database connection failed: ...
```

10. Retorne à sua sessão **ssh** no **bloghost**. Use o editor de texto **nano** para editar `index.php` novamente.

```
sudo nano index.php
```

11. No editor de texto **nano**, substitua `CLOUDSQLIP` pelo endereço IP público da instância do Cloud SQL que você anotou acima.

12. No editor de texto **nano**, substitua `DBPASSWORD` pela senha do banco de dados do Cloud SQL que você definiu acima. 

13. Pressione **Ctrl+O** e, em seguida, pressione **Enter** para salvar o arquivo editado.

14. Pressione **Ctrl+X** para sair do editor de texto **nano**.

15. Reinicie o servidor *web*:

```
sudo service apache2 restart
```

16. Retorne à guia do navegador na qual você abriu o endereço IP externo da instância de VM do **bloghost**. Ao carregar a página, aparece a seguinte mensagem:

```
Database connection succeeded.
```

> **Observação**: em um blog real, o status da conexão com o banco de dados não seria visível para os visitantes do blog. Em vez disso, a conexão com o banco de dados seria gerenciada exclusivamente pelo administrador.

### **Tarefa 5.** Configure uma aplicação em uma instância do Compute Engine para usar um objeto do Cloud Storage

1. No Cloud Console, clique em **Cloud Storage > Navegador**.

2. Clique no *bucket* com o nome do seu projeto do GCP.

3. Nesse *bucket*, há um objeto chamado `my-excellent-blog.png`. Na linha respectiva, clique no link de **Copiar URL**.

> Observação: se você não vir um ícone de link nem um "Link público", tente atualizar o navegador. Se você ainda não vir um ícone de link, retorne ao Cloud Shell e confirme se sua tentativa de alterar a lista de controle de acesso do objeto com o comando `gsutil acl ch` foi bem-sucedida.

4. Retorne à sua sessão **ssh** na instância de VM do **bloghost**.

5. Digite este comando para definir seu diretório de trabalho para a raiz dos documentos do servidor *web*:

```
cd /var/www/html
```

6. Use o editor de texto **nano** para editar `index.php`:

```
sudo nano index.php
```

7. Use as setas para mover o cursor para a linha que contém o elemento **h1**. Pressione **Enter** para inserir uma nova linha e cole a URL que você copiou anteriormente.

8. Cole esta tag HTML imediatamente antes do URL:

```html
<img src='
```

9. Coloque uma aspa simples de fechamento e um `>` no final da URL:

```html
'>
```

A linha resultante ficará similar a essa:

```html
<img src='https://storage.googleapis.com/qwiklabs-gcp-0005e186fa559a09/my-excellent-blog.png'>
```

O efeito dessas etapas é colocar a linha contendo `<img src='...'>` imediatamente antes da linha contendo `<h1>...</h1>`.

> Nota: Não copie o URL mostrado aqui. Em vez disso, copie o URL mostrado pelo navegador do Storage em seu próprio projeto do Cloud Platform.

10. Pressione **Ctrl+O** e, em seguida, pressione **Enter** para salvar o arquivo editado.

11. Pressione **Ctrl+X** para sair do editor de texto **nano**.

12. Reinicie o servidor *web*:

```
sudo service apache2 restart
```

13. Retorne à guia do navegador na qual você abriu o endereço IP externo da instância de VM do **bloghost**. Quando você carrega a página, seu conteúdo agora inclui uma imagem de *banner*.

## Finalize a sua atividade de laboratório

Exclua todas as instâncias de VM, instâncias de Cloud SQL e buckets do Cloud Storage utilizadas neste roteiro de laboratório.

- Em **Compute Engine > instâncias de VM**, exclua **bloghost**.
- Em **SQL**, exclua a instância **blog-db**.
- Em **Cloud Storage > Navegador**, exclua o *bucket* criado.
