# OTEL Weather

Sistema distribuído com dois microsserviços para consulta de clima por CEP, com tracing distribuído via OpenTelemetry + Zipkin.

## Pré-requisitos

- Docker e Docker Compose
- Chave de API do [WeatherAPI](https://www.weatherapi.com/) (gratuito)

## Configuração

Copie o arquivo de exemplo e preencha sua chave:

```bash
cp .env.example .env
# edite .env e defina WEATHER_API_KEY=sua_chave
```

## Como rodar

```bash
docker compose --env-file .env up --build
```

## Como fazer a requisição

Envie um POST para o Serviço A na porta 8080:

```bash
curl -X POST http://localhost:8080 \
  -H "Content-Type: application/json" \
  -d '{"cep": "01310100"}'
```

Resposta de sucesso (200):

```json
{"city":"São Paulo","temp_C":25.0,"temp_F":77.0,"temp_K":298}
```

Resposta para CEP inválido (422): `invalid zipcode`

Resposta para CEP não encontrado (404): `can not find zipcode`

## Como visualizar os traces no Zipkin

Acesse http://localhost:9411 após subir o ambiente.

Clique em **Run Query** para listar os traces. Cada requisição ao Serviço A gera um trace com spans cobrindo:
- Requisição recebida pelo Serviço A
- Chamada ao Serviço B (propagação de contexto)
- Lookup no ViaCEP (`viacep-lookup`)
- Lookup no WeatherAPI (`weatherapi-lookup`)

## Arquitetura

```
POST / (porta 8080)
    └─ Serviço A
           └─ GET /{cep} (porta 8081)
                  └─ Serviço B
                         ├─ ViaCEP (localidade)
                         └─ WeatherAPI (temperatura)

OTEL Collector (4317/4318) ──► Zipkin (9411)
```
