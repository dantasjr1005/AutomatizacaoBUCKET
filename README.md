# AutomatizacaoBUCKET
Automatizar uma criação de bucket gcp 

#Este é o número da versão da configuração do CircleCI. A versão 2.1 é a mais recente
version: 2.1  


#Aqui, você define o executor, que é o ambiente  o código será executado.
executors:
  google-executor:
    docker:
      - image: gcr.io/google.com/cloudsdktool/google-cloud-cli:alpine
    working_directory: ~/repo


#Na configuração do job chamado build, você especifica o executor (já definido como google-executor) e as etapas que o job deve executar
    jobs:
  build:
    executor: google-executor
    steps:
      - checkout

 #Essa etapa realiza o checkout do código-fonte, ou seja, ela faz o download do código do repositório do GitHub ou do repositório configurado, para que o pipeline possa trabalhar com ele
 - checkout


##Essa etapa realiza a autenticação no Google Cloud. Explicando cada comando:

Verificação da variável MY_GOOGLE_CREDENTIALS: Essa variável de ambiente deve conter as credenciais do Google Cloud em formato base64 (geralmente um arquivo JSON com as credenciais da conta de serviço). O código verifica se essa variável está vazia. Se estiver, o pipeline falha (com o comando exit 1).

Decodificação da variável base64: O valor da variável MY_GOOGLE_CREDENTIALS é decodificado de base64 e armazenado no arquivo /tmp/gcp-credentials.json.

Verificação do arquivo JSON: O comando cat é utilizado para garantir que o arquivo de credenciais foi criado corretamente.

Autenticação: A conta de serviço do Google Cloud é ativada usando o comando gcloud auth activate-service-account, passando o arquivo de chave JSON decodificado. Depois, a configuração do projeto do Google Cloud é definida com gcloud config set project.



###### Passo 1: Autenticação do Google Cloud ######

   - run:
    name: Authenticate Google Cloud
    command: |
      # Verificando o conteúdo da variável MY_GOOGLE_CREDENTIALS
      echo "Conteúdo da variável MY_GOOGLE_CREDENTIALS:"
      echo "$MY_GOOGLE_CREDENTIALS"  

      # Garantir que a variável não está vazia
      if [ -z "$MY_GOOGLE_CREDENTIALS" ]; then
        echo "A variável MY_GOOGLE_CREDENTIALS está vazia!"
        exit 1
      fi

      # Criar o arquivo de chave JSON temporário
      echo "$MY_GOOGLE_CREDENTIALS" | base64 -d > /tmp/gcp-credentials.json

      # Verificar se o arquivo foi criado corretamente
      cat /tmp/gcp-credentials.json

      # Ativar a conta de serviço usando a chave temporária
      gcloud auth activate-service-account --key-file=/tmp/gcp-credentials.json
      gcloud config set project original-nation-453118-a7  # Defina o seu ID do projeto


 ##### Passo 2: Criação do Bucket no Google Cloud Storage (GCS) ######

 Essa etapa cria o bucket no Google Cloud Storage (GCS) e faz o upload de arquivos para ele. Vamos ver o que cada comando faz:

Definição do nome do bucket: O nome do bucket é atribuído à variável BUCKET_NAME. Aqui, o nome do bucket é bucket10057564jr65190, mas você pode alterá-lo conforme necessário.

Criação do bucket: O comando gsutil mb cria um novo bucket no Google Cloud Storage com o nome definido na variável $BUCKET_NAME.

Cópia de arquivos: O comando gsutil -m cp -r ../* "gs://$BUCKET_NAME/" copia todos os arquivos e diretórios da pasta acima (../*) para o bucket recém-criado. O -m permite a cópia paralela, acelerando o processo


- run:
    name: Create Bucket on GCS
    command: |
      BUCKET_NAME=bucket10057564jr65190
      gsutil mb gs://$BUCKET_NAME  # Cria o bucket no GCS
      gsutil -m cp -r ../* "gs://$BUCKET_NAME/"


###### workflows  #####

#No workflow, você define a sequência de jobs que o CircleCI deve executar. No caso, o workflow se chama deploy, e nele você está chamando o job build.

A linha jobs: - build significa que o job build será o único job executado no workflow deploy#

workflows:
  version: 2
  deploy:
    jobs:
      - build


oberservação : Você deve prestar bastante atenção na criação da variavel e na escolher do nome do Bucket pois uma vez ja criado já tera o nome que você escolher , caso houver erro depois da criação !

     
