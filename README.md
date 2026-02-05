🏗️ Microservices Order System - Hub Principal

Este repositório é o ponto central da arquitetura de processamento de pedidos. Ele orquestra o ecossistema de microsserviços, infraestrutura de mensageria, banco de dados e segurança, garantindo que todo o ambiente seja provisionado de forma automática via Docker.

🌌 Arquitetura do Sistema

A solução foi desenhada seguindo os princípios de Sistemas Distribuídos e Event-Driven Architecture:

Segurança: O IAM Service centraliza a emissão de Tokens JWT.

Contratos: O Shared Contracts garante que todos os serviços falem a mesma língua através de modelos OpenAPI.

Produção: O Pedido Service recebe requisições REST, valida o Token e posta no RabbitMQ.

Consumo: O Order Service captura as mensagens da fila e persiste no MongoDB.

📂 Componentes do Ecossistema

IAM Service (Security Hub) Responsável pela autenticação M2M e proteção dos endpoints. 🔗 Ver Repositório

Shared Contracts (Library) Contratos compartilhados e geração de código via OpenAPI/Swagger. 🔗 Ver Repositório

Pedido Service (Producer) API REST com Swagger para postagem de pedidos na fila. 🔗 Ver Repositório

Order Service (Consumer) Worker que consome a fila e expõe Swagger para consulta no banco NoSQL. 🔗 Ver Repositório

🛠️ Infraestrutura Automatizada

Para facilitar o desenvolvimento e os testes, o projeto utiliza Docker para subir todos os serviços de suporte:

RabbitMQ: Broker de mensageria para comunicação assíncrona.

MongoDB: Banco de dados NoSQL de alto desempenho para persistência de pedidos.

MongoDB Compass: Interface visual para gestão dos dados do banco.

Maven Build Automático: O processo de compilação e instalação dos JARs (incluindo o Shared Contracts) é feito dentro dos containers, eliminando a necessidade de configuração manual do Maven na máquina local.

🚀 Como Executar o Ecossistema Completo

    1.  Clonagem
        Certifique-se de que todos os repositórios acima foram clonados dentro da mesma pasta raiz onde este arquivo orquestrador se encontra.

    2. Execução via Docker
        Utilize o arquivo específico de orquestração para subir o ambiente (infraestrutura + serviços + build automático):

Bash
docker-compose -f docker-compose-order-system.yml up -d --build

Nota: O parâmetro --build é fundamental na primeira execução para que o Docker execute o mvn clean install e gere as imagens dos serviços corretamente.

📡 Documentação e Testes (Swagger)

Uma vez que o ambiente esteja de pé, você pode interagir com as APIs através das interfaces Swagger integradas:

POST (Enviar Pedidos): http://localhost:8081/swagger-ui.html

GET (Consultar Pedidos): http://localhost:8082/swagger-ui.html

RabbitMQ Console: http://localhost:15672 (Login: guest / Senha: guest)

Mongo Express (Interface Web): http://localhost:8085 (Login: admin / Senha: pass)

🏗️ Diferenciais da Arquitetura
[!IMPORTANT]

🛡️ Resiliência (Message Broker)
O sistema foi desenhado para ser tolerante a falhas. Caso o order-service pare de funcionar, o pedido-service continuará recebendo e enfileirando pedidos no RabbitMQ. Assim que o serviço de processamento retornar, ele consumirá a fila automaticamente, garantindo que nenhum dado seja perdido.

📊 Performance e Consultas (MongoDB)
Para lidar com grandes volumes de dados (como a massa de 1000 registros fornecida), implementamos estratégias de otimização no order-service:

Paginação Default: Por padrão, as consultas retornam 10 registros, garantindo baixa latência.

Filtros Dinâmicos: Consultas otimizadas por Filial, Cliente e Pedido.

Modo Unpaged: Para integrações que necessitam do dump completo, basta enviar o parâmetro unPaged=true na query string para ignorar a paginação e trazer todos os resultados de uma vez.

🚀 Guia de Teste Rápido (Fluxo Completo)

Para validar o ecossistema, siga estes passos utilizando apenas o Swagger, que já está configurado para facilitar os testes.

1️⃣ Obter Token de Acesso (IAM-Service)

Antes de enviar pedidos, você precisa se autenticar:

URL: http://localhost:8080/swagger-ui.html

Ação: Localize o endpoint POST /api/auth/login.

Credenciais (Copia e Cola):

   
    {
        "clientId": "service-integration-provider",
        "clientSecret": "7e5a8f42-c1b3-4d9a-8e2f-1a5c6b7d8e9f"
    }
Resultado: Copie o valor do token retornado no JSON.

2️⃣ Autorizar o Swagger (Pedido-Service)

Agora que você tem o token, vamos "avisar" o serviço de pedidos que você está autorizado:

Acesse o Swagger do Pedido Service: http://localhost:8081/swagger-ui.html

Clique no botão Authorize (ícone de cadeado no topo da página).

No campo de texto, cole o token obtido no passo anterior.

Clique em Authorize e depois em Close.

3️⃣ Simular Envio de Pedidos (POST)

Com o cadeado fechado (autorizado), vamos enviar a massa de dados:

Endpoint: POST /api/pedidos

Massa de Dados: Na raiz deste projeto, abra a pasta /test-data e copie o conteúdo do arquivo bulk_orders.json.

Ação: No Swagger, clique em "Try it out", cole o conteúdo do arquivo no corpo da requisição e clique em Execute.

4️⃣ Validar o Processamento (Order-Service & Banco)

O processamento é assíncrono via RabbitMQ. Você pode acompanhar o resultado por:

Logs: docker logs -f order-service (Verá o processamento em tempo real).

Banco de Dados: Acesse o Mongo Express em http://localhost:8085 (admin/pass) e veja os pedidos criados na coleção orders_db.

Fila: Verifique o tráfego de mensagens no RabbitMQ em http://localhost:15672 (guest/guest).