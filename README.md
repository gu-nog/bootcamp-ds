Tabela de conteúdos
=================
<!--ts-->
   * [Aula Introdutoria](#aula-introdutoria)
   * [Aula 1](#aula-1)
   * [Aula 2](#aula-2)
   * [Aula 3](#aula-3)
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

# Aula 2
### Trabalharemos agora muito com a criação do dataset de input > modelagem de dados
### OBS: Containers usam uma memória relativamente alta
## Passos da modelagem dos dados:
- ### 1o: pegar nossos dados, até agora: bd(o .sql) e do datalake(os .json e .xlsx) para centralizá-los em um único datalake
- ### 2o: criar os buckets do nosso datalake(menu bucktes do minio console):
  - 1o: landing - dados brutos, em diferentes formatos e com pouca gestão(versionamento e backups)
    - No caso, crie 2 pastas nele:
      - performance-evaluation - faça upload do arquivo .json
      - working-hours - faça upload dos 6 arquivos .xlsx
  - 2o: processing - dados processados, com formato padrão(o .parquet) e mais gestão
  - 3o: curated - dados prontos, já refinados para relatórios, com mais gestão e dependendo do caso, criptografados
  - OBS: no bucket landing, para maior organização, criamos de preferência, uma pasta para cada formato
- ### 3o: colocar os dados do .sql no nosso banco de dados:
  - conexão no vs code > botão direito > import sql > nesse caso, selecione o arquivo 'employees_db.sql'
  - OBS: Essa etapa geralmente não é necessária, porque em geral as empresas já tem o banco de dados criado
- ### 4o: criamos uma pasta 'notebooks' e abre o jupyter(no anaconda) e nela adiciona o arquivo do notebook modelagem dados:
  - É de uma parte muito importante da ciência de dados: usa o pensamento analítico e o de negócios
  - Explicações do notebook:
    - Bibliotecas: pandas, datetime, math e glob(usar arquivos no sistema operacional) e via pip baixa: pymysql
    - Primeiro: conecta no banco de dados que criamos
      - Importa o método create_engine de sqlachemy.engine
      - Chama o create_engine passando a string: ```'mysql+pymysql://root:stack@127.0.0.1:3307/employees'```, onde:
        - mysql = formato do nosso banco de dados
        - pymysql = driver que fará a conexão, que acabamos de instalar
        - root = usuário para acesso
        - stack = senha de acesso
        - 127.0.0.1:3307 = o ip, no caso o local, em que a conexão será feita e a porta
        - employees = o banco de dados que será acessado, que criamos com o .sql
    - Segundo: lê o json da pasta do datalake com pandas > nesse caso, como estamos localmente, podemos pelo path, mas normalmente faríamos uma conexão com o datalake via algum protocolo, como o S3A, assim como faremos nas DAGs do airflow em breve
    - Como gerar um pandas.df rodando uma query: ```pd.read_sql_query(query_em_string_conexão_feita_via_create_engine)```
    - Terceiro: rodar uma query que conte o números de projetos por trabalhador
      - OBS: essa inteligência que um cientista de dados precisa ter. Ex: o nome do projeto não importa, já o número deles dá ideia da carga de trabalho, o que importa
    - Lista com path dos arquivos de um formato: ```glob.glob("path/*.formato")```
    - Atributos usados:
      - carga de trabalho dos últimos 3 meses(aquelas planilhas no datalake)
      - tempo do funcionário na empresa
      - se ele já sofreu um acidente de trabalho
      - outros: departamento, salário e se já saiu
- ### Para orquestrar todo esse processo agora usaremos o airflow
  - Transformaremos o código do notebook em DAGs em python que serão colocadas no airflow e colocaremos os arquivos .py dentro da pasta airflow>dags, que criamos.
    - OBS: no nosso caso os arquivos das DAGs já estão prontos, basta colocá-los
  - DAGs são as automatizações dos parâmetros definidos por você e pelo time de negócios que irão fazer parte da nossa pipeline
    - OBS: nas funções principais das DAGs(que fazem a nossa automatização) é possível fazer qualquer coisa que possa ser feita em python
  - #### Funcionamento das DAGs:
    - Bibliotecas não vistas ainda: 
    ~~~python
    from io import BytesIO
    from airflow import DAG
    from airflow.operators.python_operator import PythonOperator  # Essa e a próxima são meio que opcionais, você verá
    from airflow.operators.bash import BashOperator
    from airflow.models import Variable  # Para ler aquelas variáveis que salvamos no airflow
    from minio import Minio
    ~~~
    - Agora definimos um dicionário com alguns argumentos padrões para as DAGs:
    ~~~python
    DEFAULT_ARGS = {
      'owner': "Airflow",  # é o dono da dag, no caso, o usuário padrão do airflow
      # 2 itens da questão de agendamento do airflow, que você entenderá mais em breve:
      'depends_on_past': False,  # Se depende de algo para rodar
      'start_date': datetime(2021, 1, 13)  # Se a data já tiver passado quando você ativar a execução de DAGs no airflow, ela irá rodar imediatamente, pois será considerada 'atrasada'
    }
    ~~~
    - Instansciar um objeto de DAG: ```dag = DAG('nome_dela', default_args = DEFAULT_ARGS, schedule_interval='@once')```  >  O @once faz ela rodar apenas uma vez
    - Pegar as variáveis do airflow: ```Variable.get('nome_da_variável')```
    - Conectar com o minio por uma conexão, sem ser pela pasta dele: 
    ~~~python
    client = Minio(
      data_lake_server, # capturado da variável do airflow no caso, o ip e a porta dele
      acess_key=usuario_do_minio,  # no caso, pego em uma variável
      secret_key=senha_minio,
      secure=False  # por enquanto não precisamos estabelecer uma conexão segura/criptografada...
    )
    ~~~
    - Agora criamos as funções que rodarão o que queremos, no caso:
      - extract: lê o banco de dados com uma query e salva em csv
      - load: lê o csv > salva em .parquet > carrega os dados no 2o bucket, com:
        - ```client.fput_object('nomedobucket', 'nome_que_salvará.formato', 'path_onde_está_o_arquivo')```
    - Gerar as tarefas que rodarão no airflow: tem o conceito de operadores, permitindo executar códigos em vários sistemas, além de python, exs:
      - Tarefa em python: ```extract_task = PythonOperator(task_id='nome_task_no_airflow', provide_context=True, dag=dag, python_callable=funçãoempythonquerodará)```
      - Tarefa em bash(terminal linux): ```BashOperator = BashOperator(task_id='...', dag=dag, bash_command='cmd_str')```
    - No fim definimos a ordem para rodar as tasks, ex: ```primeira_task >> extract_task >> outra_task``` > pode ser quantas você quiser
  - OBS: listar objetos/arquivos de um bucket: ```client.list_objects('bucket', prefix='partedonomequeéigualnosarquivos', recursive=True)```  >  como baixar, iterar..., olhe na dag etl_mean_work_last_9...
  - Agora você vai no GUI do airflow:
    - assim que você clica no botão das dags, o airflow 'ativa' elas e verifica se está na hora de rodá-las
    - Dica: verifique as variáveis antes de rodar
    - Clicando em uma dag vocÊ entra nas informações dela:
      - ver status das tasks: graph view
      - se der erro na execução vai ter um quadradinho vermelho em uma task da árvore
    - Se você for em actions > delete dag, você apaga só o histórico de execução > log

# Aula 3
## Tema principal: análise exploratória dos dados - explorando bastante a parte de negócios
  - Objetivo: extrair insights > inteligência e conhecimento
### OBSs:
  - 1o starte os containers
  - O employees_dataset é a junção de todos atributos, só
## Passo a passo:
- ### 1o: coloque o arquivo do notebook 'analise' na pasta notebooks
  - Nele faremos muitas análises, extraindo vários insights
  - Funcionamento:
    - bibliotecas para gráficos: ```seaborn as sns``` e ```matplotlib.pyplot as plt```
    - 1o - conecta ao minio
    - 2o - lê o employees_dataset
    - 3o - reorganiza a ordem das colunas
    - 4o - tiramos dados nulos
      - OBS: ver as colunas nulas por coluna: ```df.isnull().sum()```
    - 5o -  arrumamos as tipagens
      - converter valores de uma coluna: ```df['col'] = df['col'].astype(tipo)```
    - 6o - renomeamos as colunas
- ## Análise estatística: no caso, com a taxa de turnover
  - Ver No de registros: ```.shape()```
  - % de turnover: ```.value_counts() / len(df)```
    - Como vemos, as amostras nesse caso estão desbalanceadas, o que irá interferir na parte de ML
  - visão geral do df: ```.describe()```
    - OBS: std é o desvio padrão dos dados
  - Uma análise já mais inteligente, ver atributos por diferentes turnouvers:
    ```python
    v = df.groupby('turnover')
    v.mean()
    ```
    - Apenas suposições para testes
  - Correlação entre os atributos
    - matriz de correlação: ```corr = (df.corr())```
    - mostrar: ```sns.heatmap(corr, xticklabels=corr.columns.values, yticktalbels=corr.columns.values)```
    - analisar valores:
      - positivo: 2 sobem juntos
      - negativo: um sobre, o outro desce
      - mais longe de 0: maior relação
    - OBS: cada anormalidade... deve ser analisada e discutida com a equipe de negócios, então sempre anote o que percebe e procure motivos para esses comportamentos
  - Veja a distribuição dos valores de cada atributo(padrões, picos...)
  - Aprofunde na relação (com o parâmetro de output) e na distribuição de atributos que você imagina que possa ter relação com o output
  - OBS: muitas vezes é comum ir tendo no processo ideias de novos atributos..., então volte etapas... se necessário
  - Use gráficos com pontos ou quadrados para reconhecer grupos/clusters de relação, então você poderá por exemplo tentar prever em qual grupo a pessoa está
    - #### Nada impede, por exemplo você faça um modelo de classificação desse também, além do preditivo 'principal', ABRA A CABEÇA!!!
      - exemplo: dar também um modelo para prever com regressão a carga horária futura
      - Depois coloque essas análises/dados nos buckets
    - Nesse caso usaremos o KMeans do sklearn.cluster
      - Instanciá-lo: 
        ```python
        KMeans(
          n_clusters = 3,  # a quantidade de clusters reconhecidos no gráfico
          random_state = 2
        )
        ```
      - Treinar: ```.fit(x)```, x é filtrado apenas com as colunas que usamos na classificação, no caso, satisfação e nota e, no caso, só as linhas de quem saiu
      - Iterável com grupo que ficou cada um: kmeans.labels_
      - sobe pro bucket curated já
  - Um método adicional para algumas dessas análises automáticas: via pip, sweetvz
    - importa e dá ```.analyze(df, 'parâmetro de output')``` e ```.show_html()```
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