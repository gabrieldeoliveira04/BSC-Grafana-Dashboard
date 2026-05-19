# BSC Grafana Dashboard

Dashboard Grafana desenvolvido para monitoramento de wallets públicas da Binance Smart Chain utilizando dados coletados e armazenados pelo Projeto 2.

Este projeto consome diretamente dados do PostgreSQL e apresenta visualizações em tempo real utilizando Grafana Cloud.

---

## Live Architecture

```txt
BSC RPC
   ↓
FastAPI Collector
   ↓
PostgreSQL (Render)
   ↓
Grafana Cloud
   ↓
Dashboard
```

---

## Dashboard Preview

![alt text](image.png)

---

## Overview

O dashboard foi desenvolvido para visualizar a evolução dos saldos monitorados em wallets públicas da Binance.

As consultas são feitas diretamente no PostgreSQL utilizando SQL customizado.

Sem uso de dashboards importados prontos.

Todas as queries foram desenvolvidas manualmente.

---

## Technologies

- Grafana Cloud
- PostgreSQL
- SQL
- Render
- FastAPI
- Docker
- Python

---

## Data Source

Datasource utilizado:

```txt
PostgreSQL
```

Origem:

```txt
Render PostgreSQL Cloud Database
```

Configuração:

```txt
SSL: required
Version: PostgreSQL 16
```

---

## Dashboard Panels

### 1 — Wallet Balances Over Time

Tipo:

```txt
Time Series
```

Objetivo:

Visualizar a evolução temporal dos saldos das wallets monitoradas.

SQL:

```sql
SELECT
    collected_at AS time,
    balance,
    wallet_address
FROM balances
ORDER BY collected_at
```

Explicação:

- collected_at → eixo temporal
- balance → valor exibido
- wallet_address → separa as linhas

---

### 2 — Current Total Balance

Tipo:

```txt
Stat
```

Objetivo:

Somar o saldo atual das wallets monitoradas.

SQL:

```sql
SELECT
    SUM(balance) AS total_balance
FROM (
    SELECT DISTINCT ON (wallet_address)
        wallet_address,
        balance
    FROM balances
    ORDER BY wallet_address, collected_at DESC
) latest
```

Explicação:

Seleciona apenas a coleta mais recente de cada wallet e soma os valores.

---

### 3 — Latest Wallet Collection

Tipo:

```txt
Table
```

Objetivo:

Exibir a última coleta realizada por wallet.

SQL:

```sql
SELECT DISTINCT ON (wallet_address)

    wallet_address,

    balance,

    collected_at

FROM balances

ORDER BY
    wallet_address,
    collected_at DESC
```

Explicação:

Seleciona somente o registro mais recente de cada carteira.

---

### 4 — Wallet Balance Variation 24h

Tipo:

```txt
Bar Chart
```

Objetivo:

Calcular variação percentual das últimas 24h.

SQL:

```sql
WITH latest AS (

    SELECT DISTINCT ON (wallet_address)

        wallet_address,
        balance AS latest_balance

    FROM balances

    ORDER BY
        wallet_address,
        collected_at DESC
),

oldest AS (

    SELECT DISTINCT ON (wallet_address)

        wallet_address,
        balance AS oldest_balance

    FROM balances

    WHERE collected_at >= NOW() - INTERVAL '24 hours'

    ORDER BY
        wallet_address,
        collected_at ASC
)

SELECT

    latest.wallet_address,

    ROUND(

        (
            (
                (
                    latest.latest_balance
                    -
                    oldest.oldest_balance
                )

                /

                NULLIF(
                    oldest.oldest_balance,
                    0
                )

            ) * 100
        )::numeric,

        2

    ) AS variation_percent

FROM latest

JOIN oldest

ON latest.wallet_address =
oldest.wallet_address
```

Explicação:

Calcula:

```txt
((saldo_atual - saldo_antigo) / saldo_antigo) * 100
```

---

## Dashboard Export

Dashboard exportado:

```txt
grafana/dashboard.json
```

---

## Technical Decision

Grafana foi conectado diretamente ao PostgreSQL em vez da API FastAPI.

Motivos:

- menos camadas intermediárias
- consultas históricas mais eficientes
- SQL nativo
- melhor performance
- reduz carga sobre a API

---