Tabela de conteúdos
=================
<!--ts-->
   * [Aula Introdutoria](#aula-introdutoria)
   * [Aula 1](#aula-1)
   * [Ferramentas](#tecnologias)
     * [O que são](#descricao-tecnologias)
<!--te-->

# Aula Introdutoria
### Alguns setores para datascience:
- saúde
- finanças
- big data
- controle de trânsito
### Desafio: Muita gente está se demitindo, ajude o RH a descobrir coisas relevantes

# Aula 1
OBS: Dados na maioria das empresas são caóticos
### 1o passo: entender o problemas
### 2o passo: modelar/arrumar/organizar os dados que serão/devem ser usados
- Primeiro entendemos quais dados fazem sentido usar, como por exemplo, o salário, então procuramos eles na empresa
## Passo a passo da aula:
- ### 1o criamos um container com mysql server (para consumirmos o banco de dados, formato de parte dos dados escolhidos)
  - Comando: ```docker run --name mysqlbd1 -e MYSQL_ROOT_PASSWORD=bootcamp -p "3307:3306" -d mysql```
  - Baixe a extensão para conectar um mysql: sql server client > cria um ícone do lado esquerdo, nesse caso as informações são:
    - host = 127.0.0.1 > o ip local
    - porta = 3307
    - username: root > definido no comando
    - senha: bootcamp > definido no comando
- ### 2o criamos um datalake em um container com minio > os dados não/semi estruturados localmente, usa mo mesmo protocolo da amazon/o S3, como: .json
  - Crie no diretório do projeto um diretório datalake
  - Comando: docker run --name minio -d -p 9000:9000 -p 9001:9001 -v "$PWD/datalake:/data" minio/minio server /data --console-address":9001"
    - --name minio > define o nome do container como minio
    - -p xxxx:yyyy > indica um mapeamento da porta xxxx local com a yyyy do container
    - -v "$PWD/datalake:/data" > PWD = pasta local, será o lugar onde os dados persistirão localmente, mesmo quando o container for desligado
    - minio/minio > chama a imagem do minio para docker
    - -d > para rodar localmente
    - server /data --console-address":9001" > servirá a pasta /data pelos dados da porta 9001
  - Acessar a parte de GUI do minio: localhost:9001
    - usuário: minioadmin
    - senha: minioadmin
    - OBS: ip do server = ip do docker
- ### 3o subimos o airflow
  - Crie um diretório do projeto, o diretório airflow/dags
  - Comando: ```docker run -d -p 8080:8080 -v "$PWD/airflow/dags:/opt/airflow/dags/" --entrypoint=/bin/bash --name airflow apache/airflow:2.1.1-python3.8 -c '(airflow
db init && airflow users create --username admin --password stack --firstname
Felipe --lastname Lastname --role Admin --email admin@example.org); airflow
webserver & airflow scheduler'```
  - Conectar com esse container: ```docker container exec -it airflow bash```
    - Baixa via pip: pymysql(driver de conexão com o sql), minio(conectar o minio ao airflow), xlrd e openpyxl(trabalhar com dados .parquet)
    - Sair: ```exit```
  - GUI airflow: localhost:8080
    - senha e usuário passados no comando de criação, no caso: admin e stack
    - Agora criamos algumas variáveis de ambiente que serão consumidas nas DAGs, tornando o código mais flexível
      - data_lake_server = 172.17.0.4:9000
      - data_lake_login = minioadmin
      - data_lake_password = minioadmin
      - database_server = 172.17.0.3 ( Use o comando inspect para descobrir o ip do
container: docker container inspect mysqlbd1 - localizar o atributo IPAddress)
      - database_login = root
      - database_password = bootcamp
      - database_name = employees
  - O que airflow irá fazer:
    - 1o - extrair dados
    - 2o - processá-los
    - 3o - escrever no  datalake
### OBSs:
- listar continares rodando: ```docker container ps```
- listar todos: ```docker container ls -a```
- desligar um container: ```docker container rm {id ou nome do container}```
- padrão de dados muito usado: .parquet, da IBM, otimizados para muitos dados
- sempre rode os comandos na pasta raiz do projeto

# Tecnologias

As seguintes ferramentas foram usadas na construção dos projetos:

- [Anaconda](https://www.anaconda.com/)
- [Docker](https://www.docker.com/)
- [Git](https://git-scm.com/)
- [Github](https://github.com/)
- [Minio](https://min.io/)
- [Apache airflow](https://airflow.apache.org/)
- [Mysql](https://www.mysql.com/)
- [Python](https://www.python.org/)

As seguintes bibliotecas python foram usadas na construção dos projetos:

- [Scikit](https://pypi.org/project/scikit-learn/)
- [Pycaret](https://pypi.org/project/pycaret/)
- [Pandas](https://pypi.org/project/pandas/)
- [Streamlit](https://pypi.org/project/streamlit/)

# Descricao tecnologias
## Anaconda:  
Conjunto de ferramentas para data science, como o python, bibliotecas...
### Docker:
Um sistema capaz de gerar containers > tipo uma máquina virtual para rodar programas, compartilhando apenas o kernel
## Git:
Versionamento de código
## Github:
Para publicar seu projeto ou armazená-lo na nuvem
## Minio:
Implementar um data lake(um repositório que armazena muito e variados dados de forma nativa/bruta)
## Apache airflow: 
Orquestrador de pipelines(transforma dados em insights)
## Mysql:
Banco de dados
## Python:
Linguagem de programação, muita usada em data science
## Bibliotecas:
- ## Scikit-learn:
Para machine learning
- ## Pycaret:
Mecher com machine learning, com maior produtividade
- ## Pandas:
Manipular dataframes/para análise de dados
- ## Streamlit:
Para criar microaplicações para mostrar os dados