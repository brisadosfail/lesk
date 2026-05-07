# 📧 Sistema de Envio de E-mails em Massa

## Requisitos

Antes de tudo, instale no seu Windows:

- [Node.js LTS](https://nodejs.org/en/download) — durante a instalação, marque a opção **"Add to PATH"**
- [Git Bash](https://git-scm.com/download/win) *(opcional, mas recomendado)*

---

## Estrutura de Arquivos

```
/
├── config.env              # Configurações de envio
├── config.js               # Provisiona os servidores via SSH
├── start.js                # Inicia o disparo
├── send.js                 # Enviador (chamado automaticamente)
├── template.html           # Template HTML do e-mail
├── database-email.txt      # Lista de e-mails dos destinatários
├── database-servers.txt    # Lista de servidores VPS
├── master                  # Chave SSH privada (sem extensão)
└── databases/              # Criado automaticamente pelo start.js
```

---

## Configuração dos Arquivos

### `database-email.txt`
Um e-mail por linha:
```
cliente1@gmail.com
cliente2@hotmail.com
cliente3@yahoo.com
```

### `database-servers.txt`
Formato `IP;hostname` — um servidor por linha:
```
123.456.789.001;mail1.seudominio.com
123.456.789.002;mail2.seudominio.com
```

### `template.html`
HTML do e-mail. Use `#NUMBER#` onde quiser inserir um número aleatório:
```html
<html>
  <body>
    <p>Olá! Código de referência: #NUMBER#</p>
  </body>
</html>
```

### `config.env`
```env
# Configuração dos Servidores
SETUP_SSH_FILENAME=master
SETUP_CLOUDFLARE_API=sua_api_key_aqui
SETUP_CLOUDFLARE_EMAIL=seu@email.com
SETUP_THEADS=10
SETUP_USE_DOMAINS=true
SETUP_LOGIN_SSH=root

# Configuração do Envio
MAIL_NAME=Seu Nome
MAIL_SUBJECT=Assunto do E-mail (<NUMBER>)
MAIL_QUANTITY_FROM_REQUEST=25
MAIL_TEMPLATE_QUANTITY_GENERATED_NUMBERS=5
MAIL_THEADS=10
MAIL_SEND_ATTACHMENT=false
MAIL_ATTACHMENT_FILENAME=example.pdf
```

---

## Instalação

Abra o **CMD** ou **PowerShell** na pasta do projeto e rode:

```bash
npm install axios dotenv rimraf
```

---

## Passo a Passo para Enviar

### 1. Provisionar os servidores *(somente na primeira vez ou em novos servidores)*

Coloque a URL do seu `script_dominio.sh` no `config.js` no campo `TARGET`:

```js
const TARGET = "https://sua-url.com/script_dominio.sh";
```

Depois rode:

```bash
node config.js
```

> Isso vai acessar cada VPS via SSH e instalar automaticamente o Postfix, DKIM, SSL e o serviço de envio.  
> Aguarde todos os terminais finalizarem antes de continuar.

---

### 2. Iniciar o envio

```bash
node start.js
```

> O `start.js` vai:
> 1. Ler todos os e-mails do `database-email.txt`
> 2. Dividir igualmente entre os servidores do `database-servers.txt`
> 3. Abrir uma janela CMD separada por servidor
> 4. Cada janela roda o `send.js` e exibe o progresso em tempo real

---

## Acompanhando o Envio

Cada janela CMD aberta mostra o progresso do seu respectivo servidor:

```
E-mail(s) Restante(s): 4750 | Servidor: mail1.seudominio.com | IP: 123.456.789.001
E-mail(s) Restante(s): 4725 | Servidor: mail1.seudominio.com | IP: 123.456.789.001
```

---

## Entendendo as Configurações de Velocidade

| Variável | O que faz |
|---|---|
| `MAIL_THEADS=10` | Requisições simultâneas por servidor |
| `MAIL_QUANTITY_FROM_REQUEST=25` | E-mails por requisição |
| Emails por ciclo (por servidor) | `THEADS × QUANTITY` = 250 |
| Emails por ciclo (total com 20 VPS) | 250 × 20 = **5.000** |

---

## Problemas Comuns

**"node não é reconhecido como comando"**
→ Reinstale o Node.js e marque **"Add to PATH"** durante a instalação. Reinicie o CMD após instalar.

**"Cannot find module 'axios'"**
→ Rode `npm install axios dotenv rimraf` na pasta do projeto.

**"Permission denied" na chave SSH**
→ Verifique se o arquivo `master` (chave privada) está na raiz do projeto e sem extensão.

**Janelas CMD fecham imediatamente**
→ Erro no `send.js`. Rode manualmente para ver o erro:
```bash
node send.js databases/database-0.txt 123.456.789.001-mail1.seudominio.com
```

**Servidores não recebem as requisições**
→ Verifique se o `script_dominio.sh` foi executado com sucesso e se a porta 4235 está aberta nas VPS.

---

## Resumo Rápido

```bash
# 1. Instalar dependências (só na primeira vez)
npm install axios dotenv rimraf

# 2. Provisionar servidores (só na primeira vez ou novos servidores)
node config.js

# 3. Enviar
node start.js
```
