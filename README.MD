# 🚀 Microsserviços e Orquestração Docker (Padrão 2026)

Este projeto demonstra a arquitetura de uma aplicação de e-commerce dividida em microsserviços Node.js altamente otimizados. O repositório está preparado tanto para **desenvolvimento local ágil** quanto para **produção em alta disponibilidade**, aplicando as melhores práticas globais de segurança, isolamento e performance em containers.

## 📁 Estrutura do Projeto

```text
pratica-docker/
├── .gitignore
├── README.md
├── docker-compose.yml
├── docker-swarm.yml
├── api-gateway/
│   ├── Dockerfile
│   ├── .dockerignore
│   └── ...
├── inventory-service/
│   ├── Dockerfile
│   └── ...
├── order-service/
│   ├── Dockerfile
│   └── ...
└── product-service/
    ├── Dockerfile
    └── ...
```

---

## 🛠️ Parte 1: Desenvolvimento Local (Docker Compose)

O ambiente de desenvolvimento local foi projetado para ser rápido e econômico. Ele utiliza o **Compose Watch** para sincronizar alterações de código em tempo real e a flag nativa `--watch` do Node.js 22+, eliminando a necessidade de ferramentas externas como o `nodemon`.

### 1. Iniciar o ambiente com Auto-Reload
Para subir todos os microsserviços em modo de desenvolvimento ativo, execute o comando abaixo na raiz do projeto:
```bash
docker compose watch
```
*Toda alteração que você fizer nos arquivos `.js` locais será sincronizada instantaneamente para dentro do container correspondente sem a necessidade de reiniciar o container.*

### 2. Parar os Containers Locais
Para derrubar o ambiente local e limpar as redes criadas pelo Compose:
```bash
docker compose down
```

---

## 🌐 Parte 2: Produção e Alta Disponibilidade (Docker Swarm)

Para elevar o ambiente para um ambiente produtivo resiliente, transformamos o ecossistema em um **Cluster Swarm**. Nesta etapa, os microsserviços ganham réplicas, restrições de nós, limites estritos de hardware, banco de dados persistente e criptografia de rede de nível de kernel.

### 1. Inicializar o Cluster (Nó Manager)
Transforme a sua máquina principal no líder (Manager) do ecossistema Swarm:
```bash
docker swarm init --advertise-addr <IP_DO_SEU_SERVIDOR>
```
*(Opcional) Caso queira adicionar outras máquinas (trabalhadores) ao cluster, copie o comando `docker swarm join --token...` gerado no terminal e execute-o nas outras máquinas.*

### 2. Criar a Rede Overlay Criptografada
O Swarm exige uma rede do tipo `overlay` para gerenciar a comunicação entre containers de nós distintos. Criamos uma rede isolada com criptografia IPsec ativada:
```bash
docker network create --driver overlay --opt encrypted internal-overlay
```

### 3. Preparar as Imagens de Produção
O Docker Swarm não realiza o build de imagens em tempo de execução. Antes do deploy, você deve buildar as imagens utilizando o estágio otimizado de produção e enviá-las para o seu registro de containers (ex: Docker Hub):

```bash
# Exemplo para o API Gateway (repita para os outros serviços mudando o nome)
docker build --target production -t seu-registro/api-gateway:latest ./api-gateway
docker push seu-registro/api-gateway:latest
```

### 4. Realizar o Deploy da Stack no Swarm
Com as imagens publicadas e a rede pronta, inicialize a infraestrutura inteira (Microsserviços + Banco de Dados + Réplicas) com um único comando:
```bash
docker stack deploy -c docker-swarm.yml meu-ecommerce
```

### 5. Comandos Úteis de Gerenciamento do Swarm

* **Verificar o status da Stack implantada:**
  ```bash
  docker stack ls
  ```
* **Listar todos os serviços e a quantidade de réplicas ativas:**
  ```bash
  docker stack services meu-ecommerce
  ```
* **Visualizar em quais nós cada container (tarefa) está rodando:**
  ```bash
  docker stack ps meu-ecommerce
  ```
* **Visualizar os logs unificados de um serviço específico (ex: API Gateway):**
  ```bash
  docker service logs -f meu-ecommerce_api-gateway
  ```

### 6. Remover a Stack do Servidor
Se precisar encerrar a aplicação e remover todos os serviços do cluster (os volumes persistentes do banco de dados serão preservados):
```bash
docker stack rm meu-ecommerce
```

---

## 🔒 Pilares de Boas Práticas Aplicados (2026)

* **Isolamento de Segurança:** Apenas o `api-gateway` expõe portas públicas (Porta `80`). Os microsserviços e o banco de dados estão totalmente isolados do mundo externo, comunicando-se estritamente por DNS interno na rede privada criptografada.
* **Tamanho Otimizado (< 150MB):** Dockerfiles baseados em `node:22-alpine` combinados com Multi-stage builds garantem que os arquivos de desenvolvimento e a pasta `node_modules` local fiquem de fora da imagem final de produção.
* **Segurança Non-Root:** O container de produção executa sob o usuário `node`, mitigando riscos de segurança caso o container seja violado.
* **Limitação de Recursos:** Definição de limites rígidos de CPU (máx 50-75%) e Memória (máx 256-384MB) por container, evitando que uma falha de código (como memory leak) trave o servidor host.
