# Lab: Utilizando o Cloud Run

Duração: 1h

## Introdução

Neste roteiro de laboratório, você usará o Cloud Run para implantar uma aplicação em contêiner.

O Cloud Run é uma plataforma de computação gerenciada que permite executar contêineres *stateless* que são invocáveis a partir de requisições HTTP. O Cloud Run é *serverless*: ele abstrai todo o gerenciamento de infraestrutura para que você possa se concentrar no que mais importa: criar aplicativos excelentes.

## Objetivos

Neste roteiro de laboratório, você aprenderá a:

- Ativar a API Cloud Run.
- Criar um simples aplicativo *Node.js* que possa ser implantado como um contêiner *serverless* e *stateless*.
- Colocar seu aplicativo em um contêiner e subi-lo para o *Artifact Registry*.
- Implantar uma aplicação em contêiner no *Cloud Run*.
- Excluir imagens desnecessárias para evitar cobranças extras de armazenamento.

## Tarefas

### **Tarefa 1.** Ative a API Cloud Run e configure o seu ambiente do shell

1. No Cloud Shell, ative a API Cloud Run:

```
gcloud services enable run.googleapis.com
```

Se você for solicitado a autorizar o uso de suas credenciais, faça-o. Você deverá ver uma mensagem de sucesso semelhante a esta:

```
Operation "operations/acf.cc11852d-40af-47ad-9d59-477a12847c9e" finished successfully.
```

> **Observação**: você também pode ativar a API usando a seção APIs e serviços do Cloud Console.

2. Defina a região:

```
gcloud config set compute/region us-central1
```

3. Crie uma variável de ambiente `LOCATION`:

```
LOCATION="us-central1"
```

### **Tarefa 2.** Codifique seu exemplo de aplicação

Nesta tarefa, você construirá um aplicativo simples em **NodeJS** baseado em **express** que responde a requisições HTTP.

1. No Cloud Shell, crie um novo diretório chamado `helloworld` e navegue para esse diretório:

```
mkdir helloworld && cd helloworld
```

2. Em seguida, você estará criando e editando arquivos. Para editar arquivos, use qualquer editor de sua preferência (por exemplo, `vi`, `emac`, `nano`), ou use o **Cloud Shell Code Editor** clicando no botão **Abrir Editor** no Cloud Shell.

3. Crie um arquivo `package.json` e adicione o seguinte conteúdo a ele:

```json
{
  "name": "helloworld",
  "description": "Simple hello world sample in Node",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "author": "Google LLC",
  "license": "Apache-2.0",
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

Mais importante ainda, o arquivo acima contém um *script* de inicialização e uma dependência do *framework* **express**.

4. Em seguida, no mesmo diretório, crie um arquivo `index.js`.  e copie as seguintes linhas nele:

```js
const express = require('express');
const app = express();
const port = process.env.PORT || 8080;
app.get('/', (req, res) => {
  const name = process.env.NAME || 'World';
  res.send(`Hello ${name}!`);
});
app.listen(port, () => {
  console.log(`helloworld: listening on port ${port}`);
});
```

Este código cria um servidor *web* básico que escuta na porta definida pela variável de ambiente `PORT`. Seu aplicativo agora está concluído e pronto para ser colocado em um contêiner e carregado no *Container Registry*.

> Você pode usar muitas outras linguagens para começar a usar o Cloud Run. Você encontrará documentação para Go, Python, Java, PHP, Ruby, scripts Shell e outros aqui: [https://cloud.google.com/run/docs/quickstarts/build-and-deploy](https://cloud.google.com/run/docs/quickstarts/build-and-deploy)

### **Tarefa 3.** Coloque seu aplicativo em um contêiner e carregue-o no *Artifact Registry*

5. Para colocar a aplicação de exemplo em um contêiner, crie um novo arquivo chamado `Dockerfile` no mesmo diretório dos arquivos de origem e adicione o seguinte conteúdo:

```python
# Use the official lightweight Node.js 12 image.
# https://hub.docker.com/_/node
FROM node:12-slim
# Create and change to the app directory.
WORKDIR /usr/src/app
# Copy application dependency manifests to the container image.
# A wildcard is used to ensure copying both package.json AND package-lock.json (when available).
# Copying this first prevents re-running npm install on every code change.
COPY package*.json ./
# Install production dependencies.
# If you add a package-lock.json, speed your build by switching to 'npm ci'.
# RUN npm ci --only=production
RUN npm install --only=production
# Copy local code to the container image.
COPY . ./
# Run the web service on container startup.
CMD [ "npm", "start" ]
```

6. Agora, crie sua imagem de contêiner usando o **Cloud Build** executando o comando a seguir no diretório que contém o `Dockerfile`. Observe a variável de ambiente `$GOOGLE_CLOUD_PROJECT` no comando, que contém o ID do projeto do seu ambiente:

```
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld
```

O **Cloud Build** é um serviço que executa suas compilações no GCP. Ele executa uma série de etapas de compilação, em que cada etapa de compilação é executada em um contêiner do Docker para produzir seu contêiner de aplicativo (ou outros artefatos) e enviá-lo para o *Cloud Registry*, tudo em um único comando.

Uma vez enviado para o repositório, você verá uma mensagem de **SUCCESS** contendo o nome da imagem (`gcr.io/[PROJECT-ID]/helloworld`). A imagem é armazenada no *Artifact Registry* e pode ser reutilizada, se desejado.

7. Liste todas as imagens de contêiner associadas ao seu projeto atual usando este comando:

```
gcloud container images list
```

8. Para executar e testar a aplicação localmente no Cloud Shell, inicie-o usando este comando padrão do docker:

```
docker run -d -p 8080:8080 gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld
```

9. Na janela do Cloud Shell, clique em **Visualização na web** e selecione **Visualizar na porta 8080**.

Isso deve abrir uma janela do navegador mostrando a mensagem "Hello World!".

### **Tarefa 4.** Implante no Cloud Run

1. A implantação da aplicação em contêiner no Cloud Run é feita usando o comando a seguir, adicionando seu ID do projeto:

```
gcloud run deploy --image gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld --allow-unauthenticated --region=$LOCATION
```

2. Quando solicitado:

- confirme o nome do serviço pressionando **Enter**

Aguarde alguns instantes até que a implantação seja concluída. Em caso de sucesso, a linha de comando exibe a URL do serviço:

```
Service [helloworld] revision [helloworld-00001] has been deployed
and is serving traffic at https://helloworld-wdl7fdwaaa-uc.a.run.app
```

Agora você pode visitar seu contêiner implantado abrindo a URL do serviço em qualquer janela do navegador.

**Parabéns!** Você acabou de implantar um aplicativo empacotado em uma imagem de contêiner no Cloud Run. O Cloud Run dimensiona automaticamente e horizontalmente sua imagem de contêiner para lidar com as requisições recebidas e, em seguida, reduz quando a demanda diminui. Em seu próprio ambiente, você paga apenas pela CPU, memória e rede consumidas durante o tratamento de requisições.

Para este laboratório, você usou a linha de comando gcloud. O Cloud Run também está disponível por meio do Console do Cloud.

No menu **Navegação**, na seção **Sem servidor**, clique em **Cloud Run** e você verá seu serviço **helloworld** listado.

### **Tarefa 5.** Limpeza para evitar cobranças

Embora o Cloud Run não seja cobrado quando o serviço não estiver em uso, você ainda poderá ser cobrado pelo armazenamento da imagem de contêiner criada.

Você pode decidir excluir seu projeto do GCP para evitar cobranças, o que interromperá o faturamento de todos os recursos usados nesse projeto ou simplesmente excluir sua imagem **helloworld** usando este comando:

```
gcloud container images delete gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld
```

Para excluir o serviço criado no Cloud Run, use este comando:

```
gcloud beta run services delete helloworld
```