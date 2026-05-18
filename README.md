# tecsus-workers

Workers de ingestão e sincronização de dados meteorológicos para o projeto API Meteorológica (Tecsus / Fatec).

## Serviços

| Serviço | Função |
|---|---|
| **servidor-a** | Subscriber MQTT → salva payloads brutos no MongoDB Atlas |
| **servidor-b** | Sync em loop: lê MongoDB → converte → insere no Supabase (PostgreSQL) |

## Pré-requisitos

- Docker e Docker Compose instalados
- Acesso ao broker EMQX Cloud
- Acesso ao MongoDB Atlas
- Acesso ao Supabase (service role key)

## Configuração

```bash
cp .env.example .env
# Preencha todas as variáveis no .env
```

## Subir os serviços

```bash
# Build e start em background
docker compose up -d --build

# Ver logs em tempo real
docker compose logs -f

# Ver logs de um serviço específico
docker compose logs -f servidor-a
docker compose logs -f servidor-b

# Parar tudo
docker compose down
```

## Estrutura

```
tecsus-workers/
├── docker-compose.yml
├── .env.example
├── servidor-a/
│   ├── Dockerfile
│   ├── package.json
│   ├── tsconfig.json
│   └── src/
│       ├── mqttClient.ts   ← conexão com EMQX
│       ├── subscriber.ts   ← ponto de entrada, escuta tópico MQTT
│       └── mongodb.ts      ← salva no MongoDB Atlas
└── servidor-b/
    ├── Dockerfile
    ├── package.json
    ├── tsconfig.json
    └── src/
        └── sync-mongo-postgre.ts  ← lê MongoDB, insere no Supabase
```

## Variáveis de ambiente

| Variável | Descrição |
|---|---|
| `MQTT_BROKER_URL` | URL do broker EMQX Cloud (ex: `mqtts://xxx.emqx.cloud:8883`) |
| `MQTT_USERNAME` | Usuário MQTT criado no EMQX |
| `MQTT_PASSWORD` | Senha do usuário MQTT |
| `MQTT_TOPIC` | Tópico subscrito (padrão: `estacoes/+/dados`) |
| `MONGODB_URI` | Connection string do MongoDB Atlas |
| `MONGODB_DB_NAME` | Nome do banco (padrão: `api4`) |
| `MONGODB_COLLECTION_NAME` | Coleção de payloads brutos (padrão: `raw_payloads`) |
| `NEXT_PUBLIC_SUPABASE_URL` | URL do projeto Supabase |
| `SUPABASE_SERVICE_ROLE_KEY` | Service role key do Supabase |
| `BATCH_SIZE` | Documentos por ciclo de sync (padrão: `100`) |
| `SLEEP_SECONDS` | Intervalo entre ciclos sem dados (padrão: `60`) |
| `CACHE_RELOAD_INTERVAL` | Intervalo de recarga de cache em segundos (padrão: `300`) |

## Deploy no EC2

```bash
# Na máquina EC2 (após clonar o repo e configurar o .env)
docker compose up -d --build

# Verificar status
docker compose ps
```
