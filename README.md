# Configurando um Replica Set do MongoDB com Docker

Este guia detalha o processo de configuração de um Replica Set do MongoDB utilizando Docker.

## Pré-requisitos
Antes de iniciar, certifique-se de ter os seguintes itens instalados:

1. [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. [MongoDB](https://www.mongodb.com/try/download/community)

## Passos para Configuração

### 1. Baixar e executar o MongoDB no Docker

```sh
# Baixar a imagem do MongoDB
docker pull mongo

# Criar um container MongoDB
docker run -d --name mongodb -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin mongo
```

### 2. Criar uma rede Docker

```sh
docker network create mongoCluster
```

### 3. Instanciar os containers do MongoDB

```sh
# Criar os nós do Replica Set

docker run -d --rm -p 27017:27017 --name mongo10 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo10
docker run -d --rm -p 27019:27017 --name mongo20 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo20
docker run -d --rm -p 27020:27017 --name mongo30 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo30
docker run -d --rm -p 27021:27017 --name mongo40 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo40
```

### 4. Configuração do Replica Set

```sh
# Acessar um dos containers para configuração
docker exec -it mongo10 mongosh

# Iniciar o Replica Set
rs.initiate({
  _id: "myReplicaSet",
  members: [
    {_id: 0, host: "mongo10"},
    {_id: 1, host: "mongo20"},
    {_id: 2, host: "mongo30"},
    {_id: 3, host: "mongo40"}
  ]
})
```

### 5. Testar a configuração

```sh
docker exec -it mongo10 mongosh --eval "rs.status()"
```

Se tudo estiver correto, o status do Replica Set será exibido.

### 6. Conectar ao MongoDB Compass

1. Abrir o MongoDB Compass
2. Adicionar uma nova conexão
3. Inserir a URL gerada no passo de configuração (passo 4)
4. Salvar e conectar

### 7. Testar inserções no banco

```sh
# Acessar o MongoDB Shell
docker exec -it mongo10 mongosh

# Criar um banco de dados e inserir documentos
use CorporeSystem
db.cliente.insertOne({codigo:1, nome: "Ana Maria"})
db.cliente.insertOne({codigo:2, nome: "Maria Jose"})
db.cliente.insertOne({codigo:3, nome: "Jose Silva"})
db.cliente.insertOne({codigo:4, nome: "Luis Souza"})
db.cliente.insertOne({codigo:5, nome: "Fernando Silva"})

# Verificar os dados inseridos
db.cliente.find()
```

### 8. Testar falha do nó primário

```sh
# Parar o nó primário
docker stop mongo10

# Verificar a nova eleição do nó primário
docker exec -it mongo20 mongosh --eval "rs.status()"
```


### 9. Forçar a eleição de um novo nó primário

```sh
rs.stepDown()
```


## Observações

- Se o cluster for interrompido, será necessário reconfigurar o Replica Set.
- Utilize o MongoDB Compass para uma melhor visualização e gerenciamento dos dados.
- Certifique-se de que todas as instâncias estão rodando corretamente antes de realizar testes.

---

### Conexão Padrão para o MongoDB

```
mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.4.2
```

Outras conexões possíveis:

```
docker exec -it mongo20 mongosh
docker exec -it mongo30 mongosh
docker exec -it mongo40 mongosh
```

---

Agora seu ambiente MongoDB está configurado com Docker e um Replica Set funcional!

