# 📊 Crypto Data Pipeline

**CoinGecko API v3 → Google BigQuery → Looker Studio**

------------------------------------------------------------------------

# 🎯 Visão Estratégica

Este projeto demonstra a construção de um pipeline de dados resiliente,
escalável e orientado a boas práticas de Engenharia de Dados.

Mais do que apenas consumir uma API, o objetivo foi evidenciar:

-   ✔️ Resiliência a Rate Limit (429)
-   ✔️ Retry com Backoff Exponencial
-   ✔️ Throttle Global Compartilhado
-   ✔️ Arquitetura em Camadas (RAW → FACT)
-   ✔️ Modelagem SQL sobre JSON
-   ✔️ Preparação para ambiente produtivo

O pipeline foi projetado para funcionar com a API pública da CoinGecko,
respeitando suas limitações, e preparado para evoluir para um cenário
PRO ou produtivo com billing habilitado.

------------------------------------------------------------------------

# 🏗️ Arquitetura do Pipeline

CoinGecko API\
↓\
Snapshot Markets (/coins/markets)\
↓\
BigQuery RAW (JSON versionado)\
↓\
Query SQL → Derivação Top N\
↓\
Market Chart Range (/coins/{id}/market_chart/range)\
↓\
Transformação diária (1 linha por dia)\
↓\
BigQuery FACT (Histórico estruturado)\
↓\
Looker Studio

------------------------------------------------------------------------

# 🧠 Decisões Técnicas Relevantes

## 1️⃣ Retry + Backoff Exponencial

A função `http_get_json()` implementa:

-   Retry automático para 429 e 5xx\
-   Backoff exponencial (2\^tentativa)\
-   Tratamento explícito para 401/403

Isso garante resiliência mesmo em ambiente com limitações da API
pública.

------------------------------------------------------------------------

## 2️⃣ Throttle Compartilhado entre Moedas

Na coleta histórica:

-   O timestamp da última chamada é compartilhado globalmente\
-   Evita rajadas de requisições\
-   Reduz drasticamente erros 429

Essa abordagem é mais robusta do que retry isolado por request.

------------------------------------------------------------------------

## 3️⃣ Camada RAW (Data Lake Pattern)

O snapshot é salvo completo como JSON:

-   ingestion_timestamp (TIMESTAMP)\
-   source (STRING)\
-   payload (JSON)

Benefícios:

-   Reprocessamento possível\
-   Auditoria\
-   Versionamento por execução

------------------------------------------------------------------------

## 4️⃣ Derivação do Top N

A partir do último snapshot:

-   Explosão do JSON via UNNEST\
-   Uso de JSON_VALUE\
-   SAFE_CAST para robustez\
-   Ordenação por indicador escolhido\
-   Seleção dinâmica de TOP_N

Demonstra modelagem SQL sobre JSON bruto.

------------------------------------------------------------------------

## 5️⃣ Feature Engineering Diário

A API retorna múltiplos pontos por dia.

O pipeline:

-   Ordena por timestamp\
-   Agrupa por date\
-   Seleciona o último ponto do dia

Resultado: 1 linha por ativo por dia, pronta para BI.

------------------------------------------------------------------------

# 🧰 Tecnologias Utilizadas

-   Python 3.x\
-   Requests\
-   Pandas\
-   Google Cloud BigQuery\
-   CoinGecko API v3\
-   Looker Studio

------------------------------------------------------------------------

# ⚙️ Tutorial de Execução

## 1️⃣ Instalação

``` bash
pip install google-cloud-bigquery requests python-dotenv pandas
```

------------------------------------------------------------------------

## 2️⃣ Variáveis de Ambiente

``` bash
export COINGECKO_API_KEY="..."   
export COINGECKO_IS_PRO="0"      
export GCP_PROJECT_ID="..."  
export BQ_DATASET_ID="..."  
```

------------------------------------------------------------------------

## 3️⃣ Execução

1.  Execute o notebook célula a célula\
2.  Valide a criação da tabela RAW\
3.  Verifique a derivação do Top N\
4.  Confirme a carga da tabela FACT\
5.  Conecte ao Looker Studio

------------------------------------------------------------------------

# 📊 Estrutura das Tabelas

## RAW

-   ingestion_timestamp (TIMESTAMP)\
-   source (STRING)\
-   payload (JSON)

## FACT

-   crypto_id (STRING)\
-   date (DATE)\
-   price_usd (FLOAT)\
-   market_cap_usd (FLOAT)\
-   volume_usd (FLOAT)

------------------------------------------------------------------------

# 🚀 Evoluções Futuras

-   Stage + MERGE (UPSERT) em produção\
-   Particionamento por date\
-   Clusterização por crypto_id\
-   Orquestração (Airflow / Composer)\
-   Incremental load por watermark\
-   Monitoramento e alertas\
-   Camadas SILVER e GOLD

------------------------------------------------------------------------

# 📌 Resultado Final

-   Snapshot versionado na RAW\
-   Histórico estruturado na FACT\
-   Pipeline resiliente a rate limit\
-   Base pronta para análises e dashboards\
-   Estrutura preparada para evolução produtiva

------------------------------------------------------------------------

Desenvolvido com foco em boas práticas de Engenharia de Dados.
