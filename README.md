![Python](https://img.shields.io/badge/Python-3.x-blue)
![BigQuery](https://img.shields.io/badge/BigQuery-GCP-orange)
![API](https://img.shields.io/badge/API-CoinGecko-green)

# 📊 Crypto Data Pipeline

**CoinGecko API v3 → Google BigQuery**

Este projeto implementa um pipeline de dados simples, reprodutível e
escalável para ingestão e modelagem de dados de criptomoedas utilizando
a API da CoinGecko (v3) e o Google BigQuery como Data Warehouse.

------------------------------------------------------------------------

## 🎯 Objetivo

Construir um pipeline que:

1.  📥 Coleta um snapshot de mercado via endpoint `/coins/markets`
2.  💾 Persiste o payload bruto em uma tabela RAW no BigQuery
3.  🏆 Deriva o Top N ativos por variação de preço nas últimas 24h (indicador pode ser variável)	
4.  📈 Coleta o histórico diário de preços via
    `/coins/{id}/market_chart/range`
5.  📊 Persiste os dados históricos em uma tabela FACT

------------------------------------------------------------------------

## 🏗️ Arquitetura do Pipeline

CoinGecko API\
↓\
Snapshot Markets (/coins/markets)\
↓\
BigQuery RAW (JSON)\
↓\
Query Top N por Market Cap\
↓\
Market Chart Range (/coins/{id}/market_chart/range)\
↓\
BigQuery FACT (Histórico Diário)
↓\
Looker Studio (data viz)

------------------------------------------------------------------------

## 🧰 Tecnologias Utilizadas

-   Python 3.x
-   Requests
-   Pandas
-   Google Cloud BigQuery
-   CoinGecko API (Public)

------------------------------------------------------------------------

## ⚙️ Configuração do Ambiente

### 1️⃣ Instalar dependências (fora do colab)

``` bash
pip install google-cloud-bigquery requests python-dotenv
```

------------------------------------------------------------------------

### 2️⃣ Variáveis de Ambiente

``` bash
export COINGECKO_API_KEY="..."      # Opcional (necessário para PRO)
export COINGECKO_IS_PRO="0"         # "1" para PRO | "0" para Public
export GCP_PROJECT_ID="..."
export BQ_DATASET_ID="..."
```

------------------------------------------------------------------------

## 🔄 Etapas do Pipeline

### 🔹 1. Setup

-   Importação de bibliotecas
-   Configuração de variáveis
-   Inicialização do cliente BigQuery

------------------------------------------------------------------------

### 🔹 2. Funções Utilitárias

Implementação de:

    -   `http_get_json()`
    -   Retry exponencial\
    -   Tratamento de erros HTTP (429, 5xx)\
    -   Controle de rate limit

------------------------------------------------------------------------

### 🔹 3. Criação da Tabela RAW

Tabela destinada a armazenar o snapshot bruto da API.

Campos principais:

-   ingestion_timestamp (TIMESTAMP)\
-   source (STRING)\
-   payload (JSON)

------------------------------------------------------------------------

### 🔹 4. Coleta do Snapshot de Mercado

Endpoint utilizado:

GET /coins/markets

Características:

-   Paginação controlada\
-   Controle de intervalo entre chamadas\
-   Compatível com API Public

------------------------------------------------------------------------

### 🔹 5. Derivação do Top N por Market Cap

A partir do último snapshot ingerido na RAW:

-   Ordenação por price_change_percentage_24h_in_currency\
-   Seleção dos Top N ativos\
-   Preparação para coleta histórica

------------------------------------------------------------------------

### 🔹 6. Coleta de Histórico Diário

Endpoint utilizado:

GET /coins/{id}/market_chart/range

Coleta:

-   Preço\
-   Market Cap\
-   Volume

------------------------------------------------------------------------

### 🔹 7. Persistência na Tabela FACT

Campos típicos:

-   crypto_id (STRING)\
-   date (DATE)\
-   price (FLOAT)\
-   market_cap (FLOAT)\
-   volume (FLOAT)\

------------------------------------------------------------------------

## 📈 Possíveis Evoluções

-   Orquestração com software desejado\
-   Incremental load\
-   Monitoramento e alertas\
-   Camadas RAW → SILVER → GOLD\

------------------------------------------------------------------------

## 🚀 Como Executar

1.  Configure as variáveis de ambiente\
2.  Execute o notebook célula a célula\
3.  Verifique as tabelas no BigQuery\
4.  Valide os dados ingeridos

------------------------------------------------------------------------

## 📌 Resultado Esperado

-   Snapshot bruto versionado na camada RAW\
-   Histórico estruturado na camada FACT\
-   Base pronta para análises e dashboards
