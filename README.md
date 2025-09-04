## 🎯 Visão Geral
Este projeto demonstra como **containerizar um website estático** (HTML, CSS e JavaScript) usando **Docker**, preparando-o para deploy manual em **AWS EC2** e gerenciando imagens no **Amazon ECR**.  

**Problema resolvido:** alterações de código quebram o site em servidores por causa de dependências diferentes. Com Docker, criamos um ambiente portátil e consistente.

## 🔧 Tecnologias e Ferramentas
- **Docker**: Containerização de aplicações  
- **Dockerfile**: Configuração para construir imagens  
- **HTML, CSS, JavaScript**: Website estático  
- **AWS CLI, ECR, EC2**: Deploy opcional na nuvem  
- **Security Groups e IAM Roles**: Controle de acesso na AWS

## 🏗️ Arquitetura do Projeto

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Código Local   │────▶│   Docker Image  │────▶│    Amazon ECR   │
│  (HTML/CSS/JS)  │     │   (Container)   │     │   (Registry)    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                                          │
                                                          ▼
                        ┌─────────────────┐     ┌─────────────────┐
                        │    Browser      │◀────│    Amazon EC2   │
                        │  (User Access)  │     │   (Container)   │
                        └─────────────────┘     └─────────────────┘
```

---

## 🐳 Fase 2: Containerização com Docker

### Passo 2.1: Criar o Dockerfile

Na raiz do projeto (mesmo nível da pasta `website/`), crie um arquivo chamado `Dockerfile`:

```bash
touch Dockerfile
```

### Passo 2.2: Escrever o Dockerfile

Abra o Dockerfile no seu editor e adicione:

```dockerfile
# Imagem base - Nginx Alpine (leve e eficiente)
FROM nginx:alpine

# Copia os arquivos do website para o diretório do Nginx
COPY website/ /usr/share/nginx/html/

# Expõe a porta 80 (documentação - não abre a porta realmente)
EXPOSE 80

# Comando padrão quando o container iniciar
CMD ["nginx", "-g", "daemon off;"]
```

#### 🎓 Entendendo cada linha:

- **FROM nginx:alpine**: Define a imagem base. Alpine é uma versão Linux super leve
- **COPY**: Copia arquivos do host para dentro da imagem
- **EXPOSE**: Documenta qual porta o container usa
- **CMD**: Define o comando padrão ao iniciar o container


### Passo 2.3: Construir a imagem Docker

No terminal, na raiz do projeto, execute:

```bash
docker build -t meu-website:v1.0 .
```

#### 🎓 Entendendo o comando:
- **docker build**: Comando para construir uma imagem
- **-t meu-website:v1.0**: Tag (nome:versão) da imagem
- **.**: Contexto de build (diretório atual)

Você verá a saída do processo de build:
```
[+] Building 10.5s (8/8) FINISHED
 => [internal] load build definition from Dockerfile
 => transferring dockerfile: 370B
 => [internal] load .dockerignore
 => [internal] load metadata for docker.io/library/nginx:alpine
 => [1/3] FROM docker.io/library/nginx:alpine
 => [2/3] RUN rm -rf /usr/share/nginx/html/*
 => [3/3] COPY website/ /usr/share/nginx/html/
 => exporting to image
 => naming to docker.io/library/meu-website:v1.0
```


### Passo 2.4: Verificar a imagem criada

```bash
docker images
```

Você deve ver sua imagem listada:
```
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
meu-website    v1.0      abc123def456   30 seconds ago   23.5MB
```



---

## 🧪 Fase 3: Teste Local do Container

### Passo 3.1: Executar o container localmente

```bash
docker run -d -p 8080:80 --name meu-website-container meu-website:v1.0
```

#### 🎓 Entendendo o comando:
- **docker run**: Cria e executa um container
- **-d**: Executa em background (detached)
- **-p 8080:80**: Mapeia porta 8080 do host para porta 80 do container
- **--name**: Nome do container
- **meu-website:v1.0**: Imagem a ser usada

### Passo 3.2: Verificar se o container está rodando

```bash
docker ps
```

Você verá algo como:
```
CONTAINER ID   IMAGE              COMMAND                  CREATED         STATUS         PORTS                  NAMES
xyz789abc123   meu-website:v1.0   "nginx -g 'daemon..."   10 seconds ago  Up 9 seconds   0.0.0.0:8080->80/tcp   meu-website-container
```


### Passo 3.3: Testar no navegador

Abra seu navegador e acesse:
```
http://localhost:8080
```

Você deve ver seu website funcionando! 🎉


### Passo 3.4: Verificar logs do container (opcional)

```bash
docker logs meu-website-container
```

### Passo 3.5: Parar e remover o container de teste

```bash
# Parar o container
docker stop meu-website-container

# Remover o container
docker rm meu-website-container
```

---

## ☁️ Fase 4: Configuração do Amazon ECR

### Passo 4.1: Acessar o Console AWS

1. Acesse [console.aws.amazon.com](https://console.aws.amazon.com)
2. Faça login com suas credenciais



### Passo 4.2: Navegar para o ECR

1. Na barra de busca superior, digite "ECR"
2. Clique em "Elastic Container Registry"



### Passo 4.3: Criar um repositório

1. Clique em "Create repository"
2. Configure:
   - **Visibility settings**: Private
   - **Repository name**: `meu-website`
   - **Tag immutability**: Disabled (padrão)
   - **Scan on push**: Enabled (recomendado para segurança)
3. Clique em "Create repository"



### Passo 4.4: Anotar a URI do repositório

Após criar, você verá algo como:
```
123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website
```

⚠️ **Importante**: Copie e guarde esta URI, você precisará dela!


---

## 📤 Fase 5: Push da Imagem para o ECR

### Passo 5.1: Configurar AWS CLI

Se ainda não configurou, execute:
```bash
aws configure
```

Você precisará fornecer:
- **AWS Access Key ID**: Obtida no IAM
- **AWS Secret Access Key**: Obtida no IAM
- **Default region**: ex: us-east-1
- **Default output format**: json

### Passo 5.2: Autenticar Docker com ECR

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
```

⚠️ **Substitua**: 
- `us-east-1` pela sua região
- `123456789012` pelo seu Account ID

Você deve ver:
```
Login Succeeded
```



### Passo 5.3: Tagar a imagem para o ECR

```bash
docker tag meu-website:v1.0 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
```

### Passo 5.4: Push da imagem

```bash
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
```

Você verá o progresso do upload:
```
The push refers to repository [123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website]
abc123: Pushed
def456: Pushed
v1.0: digest: sha256:xyz789... size: 1234
```



### Passo 5.5: Verificar no Console AWS

1. Volte ao ECR no console AWS
2. Clique no seu repositório
3. Você deve ver a imagem com a tag v1.0



---

## 🖥️ Fase 6: Provisionamento da Instância EC2

### Passo 6.1: Navegar para EC2

1. No console AWS, busque por "EC2"
2. Clique em "EC2"

### Passo 6.2: Lançar instância

1. Clique em "Launch Instance"
2. Configure:

#### Nome e tags
- **Name**: `meu-website-server`

#### Imagem de aplicação e sistema operacional
- **AMI**: Amazon Linux 2023 (Free tier eligible)



#### Tipo de instância
- **Instance type**: t2.micro (Free tier eligible)



#### Par de chaves
- Clique em "Create new key pair"
- **Key pair name**: `meu-website-key`
- **Key pair type**: RSA
- **Private key file format**: .pem (Linux/Mac) ou .ppk (Windows/PuTTY)
- Clique em "Create key pair" e salve o arquivo

⚠️ **IMPORTANTE**: Guarde este arquivo com segurança! Você precisará dele para acessar a EC2.



#### Configurações de rede
- **VPC**: Default
- **Subnet**: No preference
- **Auto-assign public IP**: Enable
- **Firewall (security groups)**: Create security group
  - **Security group name**: `meu-website-sg`
  - **Description**: Security group for website

#### Regras do Security Group
Adicione as seguintes regras:

| Type | Protocol | Port Range | Source |
|------|----------|------------|--------|
| SSH  | TCP      | 22         | My IP  |
| HTTP | TCP      | 80         | 0.0.0.0/0 |



#### Configurar armazenamento
- **Volume**: 8 GiB gp3 (padrão)

### Passo 6.3: Configurar IAM Role (Permissões para ECR)

#### Criar IAM Role
1. Em "Advanced details", encontre "IAM instance profile"
2. Clique em "Create new IAM profile"
3. Ou vá para IAM Console e:
   - Clique em "Roles" → "Create role"
   - **Trusted entity**: AWS service
   - **Use case**: EC2
   - **Permissions**: Adicione `AmazonEC2ContainerRegistryReadOnly`
   - **Role name**: `EC2-ECR-Role`


4. Volte para a configuração da EC2 e selecione o role criado

### Passo 6.4: Revisar e lançar

1. Revise todas as configurações
2. Clique em "Launch instance"
3. Aguarde a instância inicializar (status: running)



### Passo 6.5: Anotar informações importantes

Anote:
- **Public IP**: Ex: 54.123.45.67
- **Instance ID**: Ex: i-0abc123def456789

---

## 🚀 Fase 7: Deploy na EC2

### Passo 7.1: Conectar à instância EC2

#### No Linux/Mac:
```bash
# Ajustar permissões da chave
chmod 400 meu-website-key.pem

# Conectar via SSH
ssh -i meu-website-key.pem ec2-user@54.123.45.67
```

#### No Windows (usando PuTTY):
1. Converta a chave .pem para .ppk usando PuTTYgen
2. Use PuTTY para conectar com a chave .ppk

Você verá:
```
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
[ec2-user@ip-172-31-xx-xx ~]$
```



### Passo 7.2: Instalar Docker na EC2

```bash
# Atualizar pacotes
sudo yum update -y

# Instalar Docker
sudo yum install docker -y

# Iniciar serviço Docker
sudo systemctl start docker

# Habilitar Docker no boot
sudo systemctl enable docker

# Adicionar ec2-user ao grupo docker
sudo usermod -a -G docker ec2-user

# Verificar instalação
docker --version
```

### Passo 7.3: Fazer logout e login novamente

```bash
# Sair
exit

# Conectar novamente
ssh -i meu-website-key.pem ec2-user@54.123.45.67
```

### Passo 7.4: Autenticar Docker com ECR na EC2

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
```



### Passo 7.5: Pull da imagem do ECR

```bash
docker pull 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
```

Você verá:
```
v1.0: Pulling from meu-website
Status: Downloaded newer image for 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
```



### Passo 7.6: Executar o container

```bash
docker run -d -p 80:80 --name meu-website-prod --restart always 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
```

#### 🎓 Parâmetros importantes:
- **--restart always**: Reinicia o container se a EC2 reiniciar
- **-p 80:80**: Mapeia porta 80 (padrão HTTP)

### Passo 7.7: Verificar se está rodando

```bash
# Verificar container
docker ps

# Verificar logs
docker logs meu-website-prod
```


---

## ✅ Verificação e Testes

### Teste 1: Acessar pelo navegador

1. Abra seu navegador
2. Digite o IP público da EC2: `http://54.123.45.67`
3. Seu website deve aparecer! 🎉
