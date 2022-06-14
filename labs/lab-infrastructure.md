# Lab: Google Cloud Marketplace

Duração: 40min

## Introdução

Neste roteiro de laboratório, você irá criar uma implantação em minutos usando o Google Cloud Marketplace. Este laboratório mostra vários serviços de infraestrutura do Google Cloud em ação e ilustra o poder desta plataforma. 

## Objetivos

Neste roteiro de laboratório, você aprenderá a realizar as seguintes tarefas:

- Usar o Marketplace para criar um ambiente de integração contínua com o Jenkins;
- Verificar se você pode gerenciar o serviço pela GUI do Jenkins;
- Administrar o serviço a partir da máquina virtual através de SSH.

### **Tarefa 1:** Use o Marketplace para criar uma implantação Jenkins

1. No Cloud Console, no menu **Navegação**, clique em **Marketplace**.

2. Localize a solução do Jenkins procurando por **Jenkins packaged by Bitnami**.

3. Clique na solução e leia sobre o serviço prestado pelo software.

O Jenkins é um ambiente de integração contínua de código aberto. Você pode definir *jobs* no Jenkins que podem executar tarefas como rodar uma compilação agendada de *software* e fazer *backup* de dados. Observe todos os pacotes instalados como parte da solução Jenkins no lado direito da descrição, em **Mais detalhes**.

O serviço que você está usando, **Marketplace**, faz parte do Google Cloud. O pacote Jenkins é desenvolvido e mantido por um parceiro do ecossistema chamado **Bitnami**. Observe no lado direito um campo que diz "Última atualização". Há quanto tempo esse modelo foi atualizado?

4. Clique em **Abrir**.

5. Verifique os detalhes da implantação, aceite os termos e serviços e clique em **Implantar**.

> **Observação:** levará um minuto ou dois para o *Deployment Manager* configurar a implantação. Você pode observar o *status* conforme as tarefas estão sendo executadas. O *Deployment Manager* está inicializando uma instância de máquina virtual (VM) e instalando e configurando o *software* para você. Você verá que o `jenkins-1` foi implantado quando o processo estiver concluído. 

> O *Deployment Manager* é um serviço do Google Cloud que utiliza modelos escritos em uma combinação de YAML, Python e Jinja2 para automatizar a alocação de recursos do Google Cloud e realizar tarefas de configuração. Nos bastidores, uma máquina virtual foi criada. Um *script* de inicialização foi usado para instalar e configurar o *software*, e as regras de *firewall* de rede foram criadas para permitir o tráfego para o serviço.

6. No painel direito, clique em **Mais sobre o software** para visualizar detalhes adicionais do *software*. Observe todos os *softwares* que foram instalados.

7. Copie os valores de usuário Admin (*Admin user*) e a respectiva senha (*Admin password*) para um editor de texto.

8. Clique em **Visit the site** para visualizar o site em outra guia do navegador. 

9. Faça login com os valores de usuário e senha previamente copiados.

> **Observação:** Se você receber o erro HTTP 404, remova a parte `/jenkins/` do endereço do site. Por exemplo: http://35.238.162.236

10. Na interface do Jenkins, no painel esquerdo, clique em **Gerenciar Jenkins**. Veja todas as ações disponíveis. Agora você está preparado para gerenciar o serviço Jenkins. O foco deste laboratório é a infraestrutura do Google Cloud, e não o gerenciamento do Jenkins, portanto, nosso objetivo é simplesmente ver que esse menu está disponível.

11. Deixe a janela do navegador aberta para o serviço Jenkins. Você o usará na próxima tarefa.

> Agora você viu que o *software* está instalado e funcionando corretamente. Na próxima tarefa, você abrirá uma sessão de terminal SSH para a VM onde o serviço está hospedado e verificará se tem controle administrativo sobre o serviço.

12. No Cloud Console, no menu **Navegação**, clique em **Deployment Manager**.

13. Clique em `jenkins-1`.

14. Clique em SSH para se conectar aa máquina do servidor Jenkins.

> A interface do console está executando várias tarefas para você de forma transparente. Por exemplo, ele transferiu chaves de autenticação para a máquina virtual que hospeda o Jenkins para que você possa se conectar com segurança à máquina usando SSH.

15. Na janela do SSH, digite o seguinte comando para encerrar todos os serviços em execução:

```bash
sudo /opt/bitnami/ctlscript.sh stop
```

16. Atualize a janela do navegador onde está aberto o Jenkins. Você não verá mais a interface do Jenkins porque o serviço foi encerrado.

17. Na janela do SSH, digite o seguinte comando para reiniciar os serviços:

```bash
sudo /opt/bitnami/ctlscript.sh restart
```

18. Retorne à janela do navegador onde está aberto o Jenkins e atualize-a. Você pode ter que fazer isso algumas vezes antes que o serviço seja acessível.

19. Na janela SSH, digite `exit` para fechar a sessão do terminal SSH.

## Finalize a sua atividade de laboratório

Clique em **Excluir**, confirmando a exclusão de **jenkins-1 e todos os recursos criados por ela**.