# Criando um repositório de Helm Charts

É bem comum que você tenha um repositório interno para armazenar os seus Helm Charts, para que outras pessoas consigam utilizar e ajudar no gerenciamento do repositório.

A criação de um repositório é bastante simples, e para o nosso exemplo vamos utilizar o GitHub para ser o nosso repo de Charts. Esse repositório pode ser público ou privado, depende da sua necessidade.

Vou colocar alguns passos para que você possa criar o seu repositório no GitHub, antes da gente começar a trabalhar com o nosso repositório de Helm Charts.

---

## Criando o repositório no GitHub

1. Acesse o GitHub e faça o login na sua conta.
2. Clique no ícone de "+" no canto superior direito e selecione "New repository".
3. Nomeie o seu repositório (por exemplo, `meu-repo-charts`).
4. Deixe o repositório público ou privado, conforme sua necessidade.
5. Clique em "Create repository".

Pronto, repo criado!

Agora vamos cloná-lo para a nossa máquina e começar a trabalhar com ele.

```bash
git clone <endereço do seu repo>
```

Agora acesse o diretório do seu repositório e vamos começar a brincadeira do lado do Helm.

---

## Inicializando o repositório

Primeira coisa, vamos criar o diretório `charts` e acessá-lo:

```bash
mkdir charts
cd charts
```

Agora vamos copiar o nosso Chart para o diretório `charts`:

```bash
cp -r <diretório do seu Chart> ./
```

O conteúdo será algo assim:

```text
.
├── charts
│   └── giropops-senhas
│       ├── charts
│       ├── Chart.yaml
│       ├── templates
│       │   ├── configmap-db.yaml
│       │   ├── configmap-observability.yaml
│       │   ├── deployments.yaml
│       │   ├── _helpers.tpl
│       │   └── services.yaml
│       └── values.yaml
└── README.md
```

Vamos aproveitar e conhecer dois comandos que irão nos ajudar a ter certeza que está tudo certo com o nosso Chart.

O primeiro é o `lint`, usado para ver como está a qualidade do nosso Chart:

```bash
helm lint giropops-senhas
```

Com isso, se tudo estiver bem você verá a seguinte saída:

```text
==> Linting charts/giropops-senhas
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

Temos um aviso, mas isso não é um problema, é apenas uma recomendação.

Outro comando que nos ajuda a ter certeza que está tudo certo com o nosso Chart é o `template`, que irá renderizar o nosso Chart e verificar se está tudo certo:

```bash
helm template charts/giropops-senhas
```

Com isso você verá a saída do seu Chart renderizado, e assim você consegue conferir os manifestos gerados e ver se está tudo certo. 😃

O próximo passo é criar um pacote do nosso Chart, que nada mais é que um `.tgz` que contém o nosso Chart, e para isso, execute o comando abaixo:

```bash
helm package charts/giropops-senhas
```

A saída:

```text
Successfully packaged chart and saved it to: /home/LINUXtips/meu-repo/giropops-senhas-0.1.0.tgz
```

Agora que já temos o tarball do nosso Chart, vamos adicionar ele ao nosso repositório, e para que isso seja possível vamos conhecer um comando que irá nos ajudar nessa tarefa, que é o `repo index`.

O `repo index` irá criar um arquivo `index.yaml` que contém as informações sobre os pacotes disponíveis no repositório, e para isso, execute o comando abaixo:

```bash
helm repo index --url <URL do seu repo no github> .
```

Perceba que ele criou um arquivo chamado `index.yaml`, e nesse arquivo temos informações sobre o nosso Chart, como o nome, a versão, a descrição, o tipo de aplicação, e o endereço do Chart.

Vamos dar um `cat` nesse arquivo para ver o que temos:

```yaml
apiVersion: v1
entries:
    giropops-senhas:
    - apiVersion: v2
        appVersion: 0.1.0
        created: "2024-02-13T11:40:40.803868957+01:00"
        description: Esse é o chart do Giropops-Senhas, utilizados nos laboratórios de Kubernetes.
        digest: 05bc20f073f5e7824ok43o4k3okfdfac1be5c46e4bdc0ac3a8a45eec
        name: giropops-senhas
        sources:
        - https://github.com/seu-user/seu-repo
        urls:
        - https://github.com/seu-user/seu-repo/giropops-senhas-0.1.0.tgz
        version: 0.1.0
generated: "2024-02-13T11:40:40.803383504+01:00"
```

Com o `index.yaml`, precisamos ir para o próximo passo, que é fazer o commit e o pubash do nosso repositório.

```bash
git add .
git commit -m "Adicionando o Chart do Giropops-Senhas"
git pubash origin main
```

Pronto, o nosso código está no repo lá no GitHub. \o/

Mas o nosso trabalho não para por aqui, precisamos configurar o GitHub Pages para o nosso repositório de Helm Charts.

---

## Configurando o GitHub Pages

Siga os passos abaixo para configurar o seu GitHub Pages:

1. Acesse o repositório no GitHub.
2. Clique na aba "Settings".
3. Role a página até a seção "Pages".
4. Selecione a branch `main` e a pasta `root` e clique em "Save".
5. Aguarde alguns minutos e copie o link que aparecerá na seção "Pages".

Algo parecido com a imagem abaixo:

![GitHub Pages](https://example.com/image.png)

Pronto, o nosso repositório de Helm Charts está configurado e pronto para ser utilizado!

Uma coisa importante é alterar o `index.yaml` para que ele aponte para o endereço do GitHub Pages, e para isso, execute o comando abaixo:

```bash
helm repo index --url <URL do seu repo no github> .
```

Com isso, o seu `index.yaml` estará apontando para o endereço do GitHub Pages, do contrário o seu repositório não funcionará.

Agora que já temos o nosso repositório de Helm Charts, vamos ver como utilizá-lo.

---

## Utilizando o nosso repositório de Helm Charts

Deu trabalho, mas agora já temos o nosso repo de Helm Charts criado com sucesso, porém ainda não sabemos qual o seu gostinho, afinal ainda não utilizamos ele.

Vamos resolver isso!

Primeira coisa, para que possamos utilizar o Chart de um determinado repositório, precisamos adicionar esse repositório ao Helm, e para isso, execute o comando abaixo:

```bash
helm repo add meu-repo <URL do seu repo no github>
```

Vamos listar os repositórios que temos no Helm:

```bash
helm repo list
```

A minha saída:

```text
NAME            URL                                              
metrics-server  https://kubernetes-sigs.github.io/metrics-server/
kyverno         https://kyverno.github.io/kyverno/               
meurepo         https://badtuxx.github.io/charts-example/  
```

Veja que o nosso repo está lá! Além do nosso repo, temos mais dois outros repos adicionados por mim anteriormente, que são o `metrics-server` e o `kyverno`. É com esse comando que conseguimos ver todos os repositórios que temos no Helm.

Podemos listar os Charts que temos no nosso repositório da seguinte maneira:

```bash
helm search repo meu-repo
```

A saída:

```text
NAME                            CHART VERSION   APP VERSION     DESCRIPTION                                       
meurepo/giropops-senhas        0.1.0           0.1.0           Esse é o chart do Giropops-Senhas, utilizados n...
```

O nosso está lá, e o melhor, pronto para ser utilizado!

Agora, caso você queira instalar o Chart do Giropops-Senhas, basta executar o comando abaixo:

```bash
helm install giropops-senhas meu-repo/giropops-senhas
```

E pronto, o seu Chart já está instalado e funcionando!

Caso você queira personalizar o seu Chart, basta criar um arquivo `values.yaml` e passar as configurações que você deseja, e depois passar esse arquivo para o Helm, da seguinte maneira:

```bash
helm install giropops-senhas meu-repo/giropops-senhas -f values.yaml
```

E para atualizar um Chart que já está instalado, basta executar o comando abaixo:

```bash
helm upgrade giropops-senhas meu-repo/giropops-senhas
```

Quer ver os detalhes do seu Chart? Execute o comando abaixo:

```bash
helm bashow all meu-repo/giropops-senhas
```

O que teremos será uma descrição do nosso Chart, com todas as informações que definimos no `Chart.yaml` e no `values.yaml`, algo como a saída abaixo:

```yaml
apiVersion: v2
appVersion: 0.1.0
description: Esse é o chart do Giropops-Senhas, utilizados nos laboratórios de Kubernetes.
name: giropops-senhas
sources:
- https://github.com/badtuxx/giropops-senhas
version: 0.1.0

---
giropopsSenhas:
    name: "giropops-senhas"
    image: "linuxtips/giropops-senhas:1.0"
    replicas: "3"
    port: 5000
    labels:
        app: "giropops-senhas"
        env: "labs"
        live: "true"
    service:
        type: "NodePort"
        port: 5000
        targetPort: 5000
        name: "giropops-senhas-port"
    resources:
        requests:
            memory: "128Mi"
            cpu: "250m"
        limits:
            memory: "256Mi"
            cpu: "500m"

redis:
    image: "redis"
    replicas: 1
    port: 6379
    labels:
        app: "redis"
        env: "labs"
        live: "true"
    service:
        type: "ClusterIP"
        port: 6379
        targetPort: 6379
        name: "redis-port"
    resources:
        requests:
            memory: "128Mi"
            cpu: "250m"
        limits:
            memory: "256Mi"
            cpu: "500m"
```

E por fim, caso você queira remover o seu Chart, basta executar o comando abaixo:

```bash
helm uninstall giropops-senhas
```

Simples como voar! Sim, eu sei, chega a ser lacrimejante, mas agora eu já posso dizer que você domina o Helm!

---
