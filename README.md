# Bot de Clima no Telegram com N8N

Chatbot no Telegram que informa a temperatura atual de qualquer cidade do Brasil. O usuário envia o nome da cidade e do estado, o workflow consulta a API gratuita da OpenWeather e responde com uma mensagem curta e amigável.

Exemplo de conversa:

```
Usuário: Belo Horizonte,MG,BR
Bot:     🌤️ A temperatura em Belo Horizonte é de 25°C.
```

## Como o workflow funciona

```
Telegram Trigger > Normalizar entrada > Consultar OpenWeather > Resposta valida?
                                                                  ├ sim > Formatar mensagem > Enviar temperatura
                                                                  └ não > Enviar erro
```

| Nó | Tipo | O que faz |
|---|---|---|
| Telegram Trigger | Telegram Trigger | Recebe as mensagens de texto enviadas ao bot |
| Normalizar entrada | Set | Cria a variável `queue` com o texto sem acentos, em minúsculas, sem espaços sobrando (ex.: `São Paulo , SP , BR` vira `sao paulo,sp,br`) e guarda o `chatId` |
| Consultar OpenWeather | HTTP Request | Chama `https://api.openweathermap.org/data/2.5/weather` com Send Query Parameters ativado |
| Resposta valida? | IF | Confere se o status HTTP é 200 e se o JSON tem o campo `main.temp` |
| Formatar mensagem | Code | Extrai a temperatura, arredonda e monta a frase final (lógica determinística) |
| Enviar temperatura | Telegram (Send Message) | Envia a temperatura ao usuário |
| Enviar erro | Telegram (Send Message) | Envia a mensagem de cidade não encontrada |

### Parâmetros enviados à OpenWeather

| Parâmetro | Valor | Função |
|---|---|---|
| `q` | `{{ $json.queue }}` | Cidade formatada (vem da variável `queue`) |
| `units` | `metric` | Temperatura em graus Celsius |
| `lang` | `pt_br` | Respostas em português do Brasil |
| `appid` | `{{ $env.OPENWEATHER_API_KEY }}` | Chave da API lida da variável de ambiente |

> Observação: o enunciado chama a cidade formatada de `queue`, e esse é o nome da variável criada no nó Set. Porém a OpenWeather só reconhece o parâmetro com o nome `q`. Por isso o parâmetro enviado se chama `q` e recebe o valor de `queue`. Se o nome do parâmetro fosse `queue`, a API responderia erro 400.

## Variáveis esperadas

| Variável | Onde é usada |
|---|---|
| `OPENWEATHER_API_KEY` | Variável de ambiente do n8n, lida no nó HTTP Request pela expressão `{{ $env.OPENWEATHER_API_KEY }}` |
| `TELEGRAM_BOT_TOKEN` | Token gerado pelo @BotFather, cadastrado no n8n como credencial do tipo **Telegram API** |

Nenhuma chave ou token está salvo neste repositório.

## Pré-requisitos

1. **Bot no Telegram:** no Telegram, procure @BotFather, envie `/newbot`, escolha nome e usuário (terminando em `bot`) e guarde o token.
2. **Chave da OpenWeather:** crie uma conta em https://home.openweathermap.org/users/sign_up e copie sua chave em https://home.openweathermap.org/api_keys. Chaves novas podem levar até 2 horas para ativar.
3. **Docker** instalado (para rodar o n8n localmente).
4. **URL pública HTTPS:** o Telegram só envia mensagens para endereços públicos. Localmente, use o ngrok (`ngrok http 5678`) e copie a URL gerada.

## Como rodar o n8n com Docker

1. Na pasta do projeto, crie um arquivo `.env` (ele está no `.gitignore` e não vai para o GitHub):

```
OPENWEATHER_API_KEY=sua_chave_da_openweather
WEBHOOK_URL=https://seu-endereco.ngrok-free.app/
```

2. Suba o n8n:

```bash
docker compose up -d
```

3. Acesse http://localhost:5678 e crie seu usuário.

O `docker-compose.yml` já define `N8N_BLOCK_ENV_ACCESS_IN_NODE=false`, que libera o uso de `$env` dentro dos nós.

> No n8n Cloud não é possível criar variáveis de ambiente próprias, então a expressão `$env.OPENWEATHER_API_KEY` não funciona lá. Use o n8n com Docker.

## Como importar o workflow

1. No n8n, clique em **Create Workflow**.
2. No menu de três pontos (canto superior direito), escolha **Import from File**.
3. Selecione `workflow-chatbot-telegram.json`.

## Como inserir as credenciais

**Telegram (`TELEGRAM_BOT_TOKEN`)**
1. Abra o nó **Telegram Trigger**.
2. Em Credential, clique em **Create New Credential**.
3. Cole o token do BotFather no campo **Access Token** e salve.
4. Abra os nós **Enviar temperatura** e **Enviar erro** e selecione essa mesma credencial.

**OpenWeather (`OPENWEATHER_API_KEY`)**
Não há credencial a criar no n8n: a chave é lida do arquivo `.env` pelo Docker. Basta preencher o `.env` e reiniciar o container (`docker compose restart`).

## Como executar e testar

1. Clique em **Save** e depois ative o workflow (chave **Active** no topo).
2. No Telegram, abra o seu bot e envie mensagens de teste:

| Mensagem enviada | Retorno esperado |
|---|---|
| `Fortaleza,CE,BR` | `🌤️ A temperatura em Fortaleza é de XX°C.` |
| `São Paulo,SP,BR` | `🌤️ A temperatura em São Paulo é de XX°C.` |
| `Belo Horizonte,MG,BR` | `🌤️ A temperatura em Belo Horizonte é de XX°C.` |
| `Cidadequenaoexiste,XX,BR` | `❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).` |

Também funciona sem acento ou com letras misturadas, como `sao paulo,sp,br` ou `FORTALEZA, CE, BR`.

## Problemas comuns

| Sintoma | Causa provável |
|---|---|
| Toda cidade retorna erro | Chave da OpenWeather ainda não ativada ou `.env` não carregado |
| Erro de acesso a `$env` | Falta `N8N_BLOCK_ENV_ACCESS_IN_NODE=false` no ambiente |
| Bot não responde | Workflow não está ativo ou `WEBHOOK_URL` não é uma URL HTTPS pública |

## Estrutura do repositório

```
chatbot-telegram/
├── workflow-chatbot-telegram.json
├── README.md
├── docker-compose.yml
└── .gitignore
```
