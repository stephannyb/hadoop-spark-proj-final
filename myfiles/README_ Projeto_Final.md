## README do Projeto Final de Engenharia de Dados

-----

Este projeto foi desenvolvido como parte da disciplina de Engenharia de Dados na Universidade Federal do Rio Grande do Norte (UFRN), Centro de Tecnologia (CT), Departamento de Engenharia de Computação e Automação (DCA), sob a orientação do Prof. Dr. Carlos Viegas.

**Aluna:** Tereza Stephany de Brito Félix - 20180155133

-----

### Visão Geral

Este `README.md` detalha os passos para configurar e executar o ambiente do projeto, incluindo a subida de containers, configuração de conectores e verificação de dados. O objetivo principal é demonstrar um pipeline de dados em tempo real utilizando **Spark**, **Kafka**, **Debezium** e **MongoDB**.

-----

### Primeiros Passos

Para iniciar o projeto, é necessário subir os containers Docker definidos nos arquivos `docker-compose.yml` e `docker-compose.aux.yml`.

#### Subir os Containers

Execute os seguintes comandos no terminal:

```bash
docker compose -f docker-compose.yml up
docker compose -f docker-compose.aux.yml up -d
```

-----

### Configuração Pós-Inicialização dos Containers

Após a subida bem-sucedida dos containers, siga os passos abaixo para configurar o ambiente.

#### Acessar o Spark Master e Instalar Dependências

Acesse o container do **Spark Master** e instale as bibliotecas necessárias:

```bash
docker exec -it spark-master bash
```

Dentro do container `spark-master`:

1.  **Instalar PyMongo:**

    ```bash
    pip install --break-system-packages pymongo
    ```

2.  **Instalar cURL:**

    ```bash
    sudo apt-get update && sudo apt-get install -y curl
    ```

#### Configurar o Conector Debezium

O **Debezium** será utilizado para capturar alterações no **MongoDB**. Siga os passos para inicializar o replica set do MongoDB e criar o conector.

1.  **Inicializar Replica Set no MongoDB:**

    Acesse o container do **MongoDB**:

    ```bash
    docker exec -it mongodb bash
    ```

    Dentro do container `mongodb`, acesse o `mongosh` e inicialize o replica set:

    ```bash
    mongosh
    rs.initiate()
    ```

2.  **Criar o Conector Debezium:**

    Retorne ao container do **Spark Master**. Certifique-se de que o arquivo `connector.json` esteja dentro da pasta `myfiles`, execute o comando para criar o conector:

    ```bash
    curl -X POST http://connect:8083/connectors -H "Content-Type: application/json" -d @myfiles/connector.json
    ```

    **Resposta Esperada:**

    ```json
    {"name":"mongo-ingressantes-connector","config":{"connector.class":"io.debezium.connector.mongodb.MongoDbConnector","tasks.max":"1","mongodb.hosts":"mongodb:27017","mongodb.connection.string":"mongodb://mongodb:27017/?replicaSet=rs0","mongodb.name":"mongo-monitor","topic.prefix":"mongo","database.include.list":"engdados","collection.include.list":"engdados.ingressantes","mongodb.ssl.enabled":"false","name":"mongo-ingressantes-connector"},"tasks":[],"type":"source"}
    ```

-----

### Verificação do Conector e Tópicos Kafka

Após a configuração do conector, verifique se o **Debezium** está funcionando corretamente e se o tópico do **Kafka** foi criado.

1.  **Acessar o Container do Debezium:**

    ```bash
    docker exec -it connect bash
    ```

2.  **Verificar o Status do Conector:**

    Dentro do container `connect`:kafka-topics.sh --bootstrap-server kafka:9092 --list

    ```bash
    curl http://connect:8083/connectors
    ```

    **Resposta Esperada:**

    ```
    ["mongo-ingressantes-connector"]
    ```

3.  **Acessar o Kafka e Listar Tópicos:**

    Acesse o container do **Kafka**:

    ```bash
    docker exec -it kafka bash
    ```

    Dentro do container `kafka`, liste os tópicos:

    ```bash
    kafka-topics.sh --bootstrap-server kafka:9092 --list
    ```

    A **resposta esperada** deve conter:

    ```
    mongo.engdados.ingressantes
    ```

-----

### Operações Adicionais

#### Modelo de Inserção no Banco (Para Simular Dados em Tempo Real)

Este é um exemplo de documento que pode ser inserido na coleção `ingressantes` para simular a chegada de novos dados:

```javascript
db.ingressantes.insertOne({
  "matricula": "999999",
  "ano_ingresso": "2025",
  "periodo_ingresso": "1",
  "id_curso": "99",
  "id_unidade": "999",
  "id_unidade_gestora": "999",
  "nome_discente": "Tereza Stephanny",
  "sexo": "F",
  "forma_ingresso": "Vestibular",
  "tipo_discente": "Regular",
  "status": "Ativo",
  "sigla_nivel_ensino": "GR",
  "nivel_ensino": "Graduação",
  "nome_curso": "Engenharia de Teste",
  "modalidade_educacao": "Presencial",
  "nome_unidade": "Unidade Teste",
  "nome_unidade_gestora": "Unidade Gestora Teste"
})
```

#### Estrutura da Collection `ingressantes`

A coleção `ingressantes` possui a seguinte estrutura de campos:

```
["matricula";"ano_ingresso";"periodo_ingresso";"id_curso";"id_unidade";"id_unidade_gestora";"nome_discente";"sexo";"forma_ingresso";"tipo_discente";"status";"sigla_nivel_ensino";"nivel_ensino";"nome_curso";"modalidade_educacao";"nome_unidade";"nome_unidade_gestora"]
```

-----

### Execução do Jupyter Notebook

Após configurar o ambiente, execute as células do Jupyter Notebook na ordem.

1.  **Execução da Célula de Streaming:**
    Após executar a célula de streaming no Jupyter, simule uma inserção de dados no banco (usando o modelo de inserção acima) para observar a impressão no console.

2.  **Execução das Células Restantes:**
    Continue executando as demais células do notebook em sequência.

3.  **Verificação no MongoDB (Após a Última Célula):**
    Após a execução da última célula do Jupyter, acesse novamente o container do **MongoDB** e execute os seguintes comandos para verificar os dados:

    ```bash
    show collections
    db.ingressantes.dataSize()
    db.reingressantes.dataSize()
    db.reingressantes.find().limit(5)
    ```