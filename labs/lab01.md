# Lab: Google Cloud Marketplace

Duração: 25min

## Introdução

Neste laboratório, você usará o Cloud Marketplace para implantar de forma rápida e fácil uma pilha LAMP em uma instância do Compute Engine. O Bitnami LAMP Stack fornece um ambiente de desenvolvimento web completo para Linux que pode ser iniciado com um clique.

A pilha LAMP consiste em:

| Componente | Função |
|---|---|
| Linux | Sistema operacional |
| Apache HTTP Server | Servidor web |
| MySQL | Banco de dados relacional |
| PHP | Framework de aplicações web |
| phpMyAdmin | Ferramenta de administração do PHP |

Saiba mais sobre a pilha Bitnami LAMP na documentação oficial da Bitnami: [Google Cloud Platform](https://docs.bitnami.com/google/)

## Objetivos

Neste laboratório, você aprenderá a lançar uma solução usando o Cloud Marketplace.

## Tarefas

### Use o Cloud Marketplace para implantar uma pilha LAMP

1. No GCP Console, no menu de navegação (três barras horizontais), clique em **Marketplace**.

2. Na barra de pesquisa, digite `LAMP`.

3. Nos resultados da pesquisa, clique em **LAMP Packaged by Bitnami**.
    
    Se você escolher outra pilha LAMP, como a opção *Google Click to Deploy*, as instruções do laboratório não funcionarão conforme o esperado.

4. Na página do LAMP, clique em **Abrir/Launch**.

    Se esta for a primeira vez que você usa o *Compute Engine*, a *Compute Engine API* deve ser inicializada antes de continuar.

5. Para **Zone**, selecione a zona `southamerica-east1-a`.

6. Deixe as configurações restantes nos valores padrões.

7. Se você for solicitado a aceitar os Termos de Serviço do GCP Marketplace, aceite.

8. Clique em **Implantar/Deploy**.

    O status da implantação aparece na janela do console: **lampstack-1 está sendo implantado**. Quando a implantação da infraestrutura estiver concluída, o status muda para **lampstack-1 foi implantado**.

    Após a instalação do *software*, é exibido um resumo dos detalhes da instância, incluindo o endereço do site.

### Verifique a sua implantação

1. Quando a implantação estiver concluída, clique no link com o endereço do site no painel direito (se o site não estiver respondendo, aguarde 30 segundos e tente novamente).

    Como alternativa, você pode clicar em *Visit the site* na seção **LAMP packaged by Bitnami: noções básicas**. Uma nova guia do navegador exibe uma mensagem de *Congratulations!*. Esta página confirma que, como parte da pilha LAMP, o Apache HTTP Server está em execução.

2. No GCP Console, em **LAMP packaged by Bitnami: noções básicas**, clique em SSH.

    Em uma nova janela, uma sessão de *shell* seguro é inicializada em sua máquina virtual.

4. Na janela SSH recém-criada, para alterar o diretório de trabalho atual para `/opt/bitnami`, execute o seguinte comando:

```bash
cd /opt/bitnami
```

5. Para copiar o script `phpinfo.php` do diretório de instalação do PHP para um local acessível publicamente na web, execute o seguinte comando:

```bash
sudo sh -c 'echo "<?php phpinfo(); ?>" > apache2/htdocs/phpinfo.php'
```

> O script `phpinfo.php` exibe sua configuração do PHP. É frequentemente usado para verificar uma nova instalação do PHP.

6. Para fechar a janela SSH, execute o seguinte comando:

```bash
exit
```

7. Abra uma nova aba do navegador.

8. Digite o seguinte URL e substitua `SITE_ADDRESS` pelo URL no campo *Site address* no painel direito da página do **lampstack**.

```
http://SITE_ADDRESS/phpinfo.php
```

> Um resumo da configuração do PHP do seu servidor é exibido.

