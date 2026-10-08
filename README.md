# chatbot-telegram

Bot de clima no Telegram feito com n8n para o desafio da disciplina de Agentes e Automação.

A ideia é simples: você manda a cidade no formato `Cidade,UF,BR` e o bot responde com a temperatura atual usando a API da OpenWeather.

```
Fortaleza,CE,BR
🌤️ A temperatura em Fortaleza é de 29°C.
```

## Como funciona

O fluxo recebe a mensagem pelo Telegram Trigger e passa o texto por um Set (Normalizar entrada), que tira acentos, deixa tudo minúsculo e limpa os espaços. Esse texto vai para a variável `queue`.

Depois o HTTP Request chama a API da OpenWeather (`/data/2.5/weather`) com `units=metric` e `lang=pt_br`. A chave da API vem da variável de ambiente pelo `{{ $env.OPENWEATHER_API_KEY }}`.

Obs: a API só aceita o parâmetro com o nome `q`, então mandei o valor da `queue` dentro dele.

Um IF confere se a resposta veio com status 200 e com a temperatura. Se sim, um Code node arredonda a temperatura e monta a mensagem. Se não, o bot manda a mensagem de erro:

```
❌ Cidade não encontrada. Usar o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).
```

## Variáveis

- `OPENWEATHER_API_KEY`: chave da OpenWeather (pega em openweathermap.org)
- `TELEGRAM_BOT_TOKEN`: token do bot gerado no @BotFather

Nenhuma chave está no repositório.

## Como usar

1. Importe o `workflow-chatbot-telegram.json` no n8n (Create Workflow > menu de 3 pontos > Import from File)
2. No nó Telegram Trigger crie uma credencial Telegram API e cole o `TELEGRAM_BOT_TOKEN`. Selecione a mesma credencial nos nós Enviar temperatura e Enviar erro
3. Configure a `OPENWEATHER_API_KEY` como variável de ambiente do n8n. Deixei um `docker-compose.yml` caso queira rodar com Docker (ele já libera o uso do `$env` com `N8N_BLOCK_ENV_ACCESS_IN_NODE=false`)
4. Salve e ative o workflow

Lembrando que o Telegram precisa de uma URL https pública pro webhook funcionar, então rodando local precisa de algo tipo ngrok.

## Testes

- `Fortaleza,CE,BR`
- `São Paulo,SP,BR`
- `Belo Horizonte,MG,BR`
- `Cidadeinventada,XX,BR` (deve cair na mensagem de erro)
