# Sistema de Gestão de Advertências - Uso de Celular em Ambiente Escolar

> Sistema automatizado para registro de advertências por uso indevido de celular, em conformidade com a **Lei nº 15.100/2025** do Estado de São Paulo.

[![n8n](https://img.shields.io/badge/n8n-Latest-orange)](https://n8n.io/)
[![Docker](https://img.shields.io/badge/Docker-Compose-blue)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## 📋 Índice

- [Sobre o Projeto](#sobre-o-projeto)
- [Funcionalidades](#funcionalidades)
- [Arquitetura do Sistema](#arquitetura-do-sistema)
- [Pré-requisitos](#pré-requisitos)
- [Instalação](#instalação)
- [Configuração](#configuração)
- [Como Usar](#como-usar)
- [Estrutura do Workflow](#estrutura-do-workflow)
- [Endpoints da API](#endpoints-da-api)
- [Troubleshooting](#troubleshooting)
- [Contribuindo](#contribuindo)
- [Licença](#licença)

---

## 🎯 Sobre o Projeto

Este sistema foi desenvolvido para automatizar o processo de registro de advertências por uso indevido de celular em ambiente escolar, conforme estabelecido pela **Lei nº 15.100/2025** do Estado de São Paulo.

### Contexto Legal

A lei determina que o uso de dispositivos eletrônicos por alunos só é permitido em:
- **Fins pedagógicos** orientados pelo professor
- **Situações de emergência, perigo ou necessidade**

Quando um aluno é flagrado usando o celular indevidamente, o sistema registra automaticamente a ocorrência e notifica os responsáveis.

### Fluxo do Sistema

```
┌─────────────────┐
│  Inspetor vê    │
│  aluno com      │
│  celular        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  TI acessa      │
│  formulário     │
│  web            │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Busca aluno    │
│  na planilha    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Gera QR Code   │
│  temporário     │
│  (5 minutos)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Inspetor       │
│  escaneia QR    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Sistema        │
│  registra       │
│  advertência    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Emails         │
│  automáticos    │
│  • Responsáveis │
│  • Coordenação  │
└─────────────────┘
```

---

## ✨ Funcionalidades

### 🔍 Busca Inteligente de Alunos
- Busca por nome, turma ou e-mail
- Normalização de texto (remove acentos)
- Resultados em tempo real
- Interface responsiva

### 🎫 Geração de QR Code Temporário
- Token único para cada advertência
- Validade de 5 minutos
- Proteção contra uso duplicado
- Rastreamento completo

### 📊 Sistema de Escalação
- **Estado 0**: Nenhuma advertência
- **Estado 1**: 1-2 advertências
- **Estado 2**: 3+ advertências (acionamento de medidas adicionais)
- Contador automático: ao atingir 3 advertências, reseta para 0 e aumenta o estado

### 📧 Notificações Automáticas
- **E-mail para responsáveis**: Notificação formal da advertência
- **E-mail para coordenação**: Relatório detalhado com status atual do aluno
- Templates HTML responsivos e profissionais

### 📱 Interface Responsiva
- Design moderno e clean
- Totalmente adaptável para mobile
- Modo escuro (dark mode)
- Feedback visual em tempo real

---

## 🏗️ Arquitetura do Sistema

### Componentes Principais

```
┌─────────────────────────────────────────────────────────┐
│                        n8n                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │   Webhook    │  │   Workflow   │  │  Google API  │ │
│  │   Endpoints  │→ │   Engine     │→ │  Integration │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                  Google Sheets                          │
│  ┌──────────────┐              ┌──────────────┐        │
│  │    Dados     │              │  QR Tokens   │        │
│  │   (Alunos)   │              │   (Tokens)   │        │
│  └──────────────┘              └──────────────┘        │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                      Gmail API                          │
│         Envio automático de e-mails                     │
└─────────────────────────────────────────────────────────┘
```

### Fluxos de Trabalho

O sistema possui 3 workflows principais:

#### 1. **Workflow: Formulário de Busca** (`/webhook/qr`)
- Renderiza interface de busca
- Permite busca de alunos em tempo real

#### 2. **Workflow: API de Busca** (`/webhook/api/alunos`)
- Processa queries de busca
- Retorna resultados filtrados em JSON

#### 3. **Workflow: Geração de QR** (`/webhook/qr/new`)
- Gera token único
- Cria QR Code temporário
- Registra na planilha "QR Tokens"

#### 4. **Workflow: Scan e Registro** (`/webhook/qr/scan`)
- Valida token (validade, uso único)
- Busca dados do aluno
- Atualiza advertências e estado
- Envia e-mails automáticos
- Marca token como usado

---

## 📦 Pré-requisitos

### Software Necessário

- **Docker** 20.10+
- **Docker Compose** 2.0+
- **Git**

### Contas Necessárias

1. **Conta Google** com acesso a:
   - Google Sheets API
   - Gmail API

2. **Google Cloud Console** (para criar credenciais OAuth2)

---

## 🚀 Instalação

### 1. Clone o Repositório

```bash
git clone https://github.com/seu-usuario/n8n-planilha-celulares.git
cd n8n-planilha-celulares
```

### 2. Instale o Docker

#### Linux (Ubuntu/Debian)

```bash
# Atualize os pacotes
sudo apt update

# Instale dependências
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Adicione a chave GPG do Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Adicione o repositório do Docker
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instale o Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Adicione seu usuário ao grupo docker
sudo usermod -aG docker $USER
newgrp docker

# Verifique a instalação
docker --version
docker compose version
```

#### Windows

1. Baixe o [Docker Desktop para Windows](https://www.docker.com/products/docker-desktop/)
2. Execute o instalador
3. Reinicie o computador
4. Abra o Docker Desktop

#### macOS

1. Baixe o [Docker Desktop para Mac](https://www.docker.com/products/docker-desktop/)
2. Arraste para a pasta Applications
3. Abra o Docker Desktop

### 3. Configure as Variáveis de Ambiente

```bash
# Copie o arquivo de exemplo
cp .env.example .env

# Edite o arquivo .env
nano .env
```

Conteúdo do `.env`:

```env
# Credenciais de autenticação do n8n
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=SuaSenhaSegura123!

# Porta onde o n8n será executado
N8N_PORT=5678

# Fuso horário
GENERIC_TIMEZONE=America/Sao_Paulo

# URL base para webhooks
WEBHOOK_URL=http://localhost:5678/
```

### 4. Inicie o Container

```bash
# Inicie o n8n em modo detached
docker compose up -d

# Verifique os logs
docker compose logs -f n8n
```

Aguarde alguns segundos até ver a mensagem:
```
n8n is ready on http://localhost:5678
```

### 5. Acesse o n8n

Abra seu navegador e acesse:
```
http://localhost:5678
```

Faça login com as credenciais definidas no `.env`:
- **Usuário**: admin
- **Senha**: SuaSenhaSegura123!

---

## ⚙️ Configuração

### 1. Importe o Workflow

1. No n8n, clique em **"Workflows"** → **"Add Workflow"**
2. Clique nos **3 pontos** (⋮) → **"Import from File"**
3. Selecione o arquivo `planilha-celulares.json`
4. Clique em **"Save"**

### 2. Configure o Google Sheets

#### Crie a Planilha

1. Acesse o [Google Sheets](https://sheets.google.com/)
2. Crie uma nova planilha chamada **"Planilha de Celulares"**

#### Aba 1: "Dados"

Crie as colunas:

| Token | sa | Aluno | Email | Responsável | Turma | Estado | Advertências |
|-------|----|----|-------|-------------|-------|--------|--------------|
| | 1 | Aluno 1 | aluno1@email.com | pai1@email.com | 1º Médio A | 0 | 0 |
| | 2 | Aluno 2 | aluno2@email.com | pai2@email.com | 1º Médio A | 0 | 0 |

**Importante**: 
- A coluna "Token" pode ficar vazia
- A coluna "sa" é um ID sequencial
- "Email" é o e-mail do aluno (usado para busca)
- "Responsável" é o e-mail do responsável (recebe notificações)

#### Aba 2: "QR Tokens"

Crie as colunas:

| token | email | expiresAt | createdAt | used | usedAt |
|-------|-------|-----------|-----------|------|--------|

Esta aba será preenchida automaticamente pelo sistema.

### 3. Configure as Credenciais do Google

#### Crie as Credenciais OAuth2

1. Acesse o [Google Cloud Console](https://console.cloud.google.com/)
2. Crie um novo projeto ou selecione um existente
3. Vá em **"APIs & Services"** → **"Credentials"**
4. Clique em **"Create Credentials"** → **"OAuth Client ID"**
5. Configure:
   - **Application type**: Web application
   - **Name**: n8n Google Sheets
   - **Authorized redirect URIs**: `http://localhost:5678/rest/oauth2-credential/callback`
6. Copie o **Client ID** e **Client Secret**

#### Ative as APIs Necessárias

No Google Cloud Console, ative:
1. **Google Sheets API**
2. **Gmail API**

#### Configure no n8n

##### Google Sheets

1. No workflow, clique em qualquer nó do Google Sheets
2. Em **"Credential to connect with"**, clique em **"Create New Credential"**
3. Selecione **"Google Sheets OAuth2 API"**
4. Preencha:
   - **Client ID**: Cole o Client ID copiado
   - **Client Secret**: Cole o Client Secret copiado
5. Clique em **"Connect my account"**
6. Autorize o acesso no Google
7. Salve com o nome: **"Google Sheets account"**

##### Gmail

1. No workflow, clique em qualquer nó do Gmail
2. Em **"Credential to connect with"**, clique em **"Create New Credential"**
3. Selecione **"Gmail OAuth2 API"**
4. Preencha:
   - **Client ID**: Use o mesmo Client ID
   - **Client Secret**: Use o mesmo Client Secret
5. Clique em **"Connect my account"**
6. Autorize o acesso no Google
7. Salve com o nome: **"Gmail account"**

### 4. Atualize os IDs da Planilha

No workflow, você precisa atualizar o ID da sua planilha em todos os nós do Google Sheets.

**Como encontrar o ID da planilha:**

URL da planilha:
```
https://docs.google.com/spreadsheets/d/1sjUhQa8Qf5E9Sg2si1O8T2qcplSgruRhH1iBxvU0c0k/edit
                                      ↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
                                      Este é o ID da planilha
```

**Nós que precisam ser atualizados:**

1. `Append row in sheet` (linha 14)
2. `LKP_TOKEN` (linha 137)
3. `ATUALIZA_ALUNO` (linha 198)
4. `MARCA_TOKEN_USADO` (linha 280)
5. `GET_ALUNOS` (linha 434)
6. `GET_ALUNOS1` (linha 516)

Para cada nó:
1. Clique no nó
2. Em **"Document"**, clique no campo
3. Selecione **"From list"**
4. Escolha sua planilha
5. Salve

### 5. Configure o E-mail dos Funcionários

No nó **"EMAIL_FUNCIONARIOS"** (linha 477), altere o campo **"Send to"**:

```javascript
"sendTo": "seuemail@gmail.com"  // ← Altere para o e-mail da coordenação
```

### 6. Ative o Workflow

1. No canto superior direito, mude o switch de **"Inactive"** para **"Active"**
2. Clique em **"Save"**

---

## 📖 Como Usar

### 1. Acesse o Formulário de Busca

Abra no navegador:
```
http://localhost:5678/webhook/qr
```

Você verá uma interface moderna com:
- Campo de busca de alunos
- Sugestões em tempo real
- Design responsivo

### 2. Busque o Aluno

1. Digite o nome, turma ou e-mail do aluno
2. Os resultados aparecem automaticamente
3. Clique no aluno desejado

### 3. Gere o QR Code

Após selecionar o aluno, você será redirecionado para uma página com:
- QR Code temporário (válido por 5 minutos)
- E-mail do aluno
- Link direto (alternativa ao QR)
- Data/hora de expiração

### 4. Escaneie o QR Code

1. Use a câmera do celular para escanear o QR Code
2. Ou clique no link direto

### 5. Confirmação

Após escanear, você verá:
- Confirmação de advertência registrada
- Nome e turma do aluno
- Advertências atuais (X/3)
- Estado atual

### 6. E-mails Automáticos

O sistema enviará automaticamente:

**Para os responsáveis:**
- E-mail formal notificando a advertência
- Informações sobre a Lei nº 15.100/2025
- Status atual do aluno

**Para a coordenação:**
- E-mail detalhado com todas as informações
- Status completo do aluno
- Alerta se atingiu 3 advertências

---

## 🔧 Estrutura do Workflow

### Nodes Principais

#### 1. **Webhook** (`/webhook/qr`)
- **Tipo**: Webhook
- **Função**: Renderiza o formulário de busca
- **Resposta**: HTML com interface de busca

#### 2. **Webhook GET** (`/webhook/api/alunos`)
- **Tipo**: Webhook
- **Função**: API de busca de alunos
- **Query**: `?q=termo_de_busca`
- **Resposta**: JSON com resultados

#### 3. **GET_ALUNOS1**
- **Tipo**: Google Sheets
- **Função**: Busca todos os alunos da planilha
- **Aba**: Dados

#### 4. **Code in JavaScript**
- **Tipo**: Code
- **Função**: Filtra alunos com base na query
- **Lógica**:
  - Normaliza texto (remove acentos)
  - Divide query em termos
  - Busca em nome, turma e e-mail
  - Retorna até 20 resultados

#### 5. **Webhook** (`/webhook/qr/new`)
- **Tipo**: Webhook
- **Função**: Gera QR Code para o aluno selecionado
- **Query**: `?email=aluno@email.com`

#### 6. **CRIAR_TOKEN**
- **Tipo**: Code
- **Função**: Gera token único
- **Lógica**:
  ```javascript
  token = timestamp + random_string
  expiresAt = now + 5_minutes
  ```

#### 7. **Append row in sheet**
- **Tipo**: Google Sheets
- **Função**: Salva token na aba "QR Tokens"
- **Campos**: token, email, expiresAt, createdAt, used

#### 8. **MONTAR_URL**
- **Tipo**: Set
- **Função**: Monta URL do QR Code
- **URL**: `http://localhost:5678/webhook/qr/scan?token=XXXXX`

#### 9. **Respond to Webhook**
- **Tipo**: Respond to Webhook
- **Função**: Retorna HTML com QR Code
- **Conteúdo**:
  - QR Code (via API externa)
  - E-mail do aluno
  - Link direto
  - Data de expiração

#### 10. **SCAN_QR** (`/webhook/qr/scan`)
- **Tipo**: Webhook
- **Função**: Processa o escaneamento do QR
- **Query**: `?token=XXXXX`

#### 11. **LKP_TOKEN**
- **Tipo**: Google Sheets
- **Função**: Busca o token na planilha
- **Filtro**: `token = query.token`

#### 12. **VALIDAR_TOKEN**
- **Tipo**: Code
- **Função**: Valida o token
- **Validações**:
  - Token existe?
  - Já foi usado?
  - Está expirado?

#### 13. **KEEP_TOKEN**
- **Tipo**: Set
- **Função**: Preserva dados do token para próximos nós

#### 14. **GET_ALUNOS**
- **Tipo**: Google Sheets
- **Função**: Busca todos os alunos (novamente)

#### 15. **FIND_ALUNO**
- **Tipo**: Code
- **Função**: Encontra o aluno pelo e-mail
- **Lógica**: Normaliza e compara e-mails

#### 16. **JUNTAR**
- **Tipo**: Merge
- **Função**: Combina dados do aluno e do token

#### 17. **CALCULA**
- **Tipo**: Code
- **Função**: Calcula novas advertências e estado
- **Lógica**:
  ```javascript
  advertências_atuais = advertências + 1
  
  se advertências_atuais >= 3:
    advertências_atuais = 0
    estado = estado + 1
    escalou = true
  ```

#### 18. **ATUALIZA_ALUNO**
- **Tipo**: Google Sheets
- **Função**: Atualiza advertências e estado na planilha
- **Aba**: Dados

#### 19. **MARCA_TOKEN_USADO**
- **Tipo**: Google Sheets
- **Função**: Marca token como usado
- **Aba**: QR Tokens
- **Campos**: used=true, usedAt=now

#### 20. **EMAIL_RESP**
- **Tipo**: Gmail
- **Função**: Envia e-mail para responsáveis
- **Template**: HTML formal

#### 21. **EMAIL_FUNCIONARIOS**
- **Tipo**: Gmail
- **Função**: Envia e-mail para coordenação
- **Template**: HTML detalhado

#### 22. **HTML_OK**
- **Tipo**: Respond to Webhook
- **Função**: Exibe confirmação de sucesso

---

## 🌐 Endpoints da API

### 1. Formulário de Busca

```
GET http://localhost:5678/webhook/qr
```

**Resposta**: HTML com interface de busca

---

### 2. API de Busca de Alunos

```
GET http://localhost:5678/webhook/api/alunos?q=maria
```

**Query Parameters**:
- `q` (string): Termo de busca

**Resposta**:
```json
[
  {
    "email": "maria@escola.com",
    "label": "Maria Silva — 1º Médio A"
  },
  {
    "email": "mariana@escola.com",
    "label": "Mariana Santos — 2º Médio B"
  }
]
```

---

### 3. Gerar QR Code

```
GET http://localhost:5678/webhook/qr/new?email=aluno@escola.com
```

**Query Parameters**:
- `email` (string): E-mail do aluno

**Resposta**: HTML com QR Code temporário

---

### 4. Escanear QR Code

```
GET http://localhost:5678/webhook/qr/scan?token=abc123xyz
```

**Query Parameters**:
- `token` (string): Token único gerado

**Resposta**: HTML com confirmação de advertência

---

## 🐳 Comandos Docker

### Gerenciar o Container

```bash
# Iniciar o n8n
docker compose up -d

# Ver logs em tempo real
docker compose logs -f n8n

# Parar o n8n
docker compose down

# Reiniciar o n8n
docker compose restart

# Ver status dos containers
docker compose ps

# Parar e remover tudo (incluindo volumes)
docker compose down -v
```

### Atualizar o n8n

```bash
# Baixar versão mais recente
docker compose pull

# Reiniciar com nova versão
docker compose up -d
```

### Acessar o Terminal do Container

```bash
docker exec -it n8n sh
```

### Backup dos Dados

```bash
# Criar backup da pasta de dados
tar -czf n8n_backup_$(date +%Y%m%d).tar.gz n8n_data/

# Restaurar backup
tar -xzf n8n_backup_20241028.tar.gz
```

---

## 🔍 Troubleshooting

### Problema: Container não inicia

**Sintomas**:
```
Error: Cannot start service n8n: port is already allocated
```

**Solução**:
```bash
# Verifique se a porta 5678 está em uso
sudo lsof -i :5678

# Ou altere a porta no .env
N8N_PORT=5679
```

---

### Problema: Credenciais do Google não funcionam

**Sintomas**:
- Erro ao conectar com Google Sheets
- "Invalid credentials"

**Solução**:
1. Verifique se as APIs estão ativadas no Google Cloud Console
2. Confirme que a URL de redirecionamento está correta:
   ```
   http://localhost:5678/rest/oauth2-credential/callback
   ```
3. Revogue e reconecte as credenciais no n8n

---

### Problema: Token expirado

**Sintomas**:
- Mensagem "Token expirado" ao escanear QR

**Solução**:
- Gere um novo QR Code (o token tem validade de 5 minutos)

---

### Problema: E-mails não estão sendo enviados

**Sintomas**:
- Workflow executa sem erros, mas e-mails não chegam

**Solução**:
1. Verifique se o Gmail OAuth2 está configurado
2. Confirme que o e-mail está correto no nó `EMAIL_FUNCIONARIOS`
3. Verifique a caixa de spam
4. Teste o nó de e-mail manualmente no n8n

---

### Problema: Planilha não atualiza

**Sintomas**:
- Advertências não são registradas

**Solução**:
1. Verifique se o ID da planilha está correto em todos os nós
2. Confirme que as abas "Dados" e "QR Tokens" existem
3. Verifique as permissões da conta Google

---

### Problema: Busca não retorna resultados

**Sintomas**:
- Campo de busca não mostra sugestões

**Solução**:
1. Verifique se a aba "Dados" tem registros
2. Confirme que os campos "Aluno", "Email" e "Turma" estão preenchidos
3. Teste a API diretamente:
   ```
   http://localhost:5678/webhook/api/alunos?q=aluno1
   ```

---

### Problema: Interface não carrega

**Sintomas**:
- Página em branco ao acessar `/webhook/qr`

**Solução**:
1. Verifique os logs do container:
   ```bash
   docker compose logs -f n8n
   ```
2. Confirme que o workflow está ativo
3. Reinicie o container:
   ```bash
   docker compose restart
   ```

---

### Problema: Permissões no Linux

**Sintomas**:
- Erro de permissão ao criar pasta `n8n_data`

**Solução**:
```bash
# Ajuste as permissões da pasta
sudo chown -R 1000:1000 n8n_data/
```

---

## 🤝 Contribuindo

Contribuições são bem-vindas! Para contribuir:

1. Faça um fork do projeto
2. Crie uma branch para sua feature:
   ```bash
   git checkout -b feature/minha-feature
   ```
3. Commit suas mudanças:
   ```bash
   git commit -m 'feat: Adiciona nova funcionalidade'
   ```
4. Push para a branch:
   ```bash
   git push origin feature/minha-feature
   ```
5. Abra um Pull Request

### Convenção de Commits

Usamos [Conventional Commits](https://www.conventionalcommits.org/):

- `feat`: Nova funcionalidade
- `fix`: Correção de bug
- `docs`: Documentação
- `style`: Formatação
- `refactor`: Refatoração de código
- `test`: Testes
- `chore`: Manutenção

---

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

---

## 📞 Suporte

Se você encontrar algum problema ou tiver dúvidas:

1. Verifique a seção [Troubleshooting](#troubleshooting)
2. Abra uma [Issue](https://github.com/seu-usuario/n8n-planilha-celulares/issues)
3. Entre em contato com a equipe de TI

---

## 🙏 Agradecimentos

- [n8n.io](https://n8n.io/) - Plataforma de automação
- [Google Sheets API](https://developers.google.com/sheets/api) - Integração com planilhas
- [Gmail API](https://developers.google.com/gmail/api) - Envio de e-mails
- Comunidade n8n

---

## 📚 Recursos Adicionais

- [Documentação oficial do n8n](https://docs.n8n.io/)
- [n8n Community Forum](https://community.n8n.io/)
- [Google Sheets API Docs](https://developers.google.com/sheets/api/guides/concepts)
- [Gmail API Docs](https://developers.google.com/gmail/api/guides)
- [Docker Documentation](https://docs.docker.com/)
- [Lei nº 15.100/2025 - SP](https://www.al.sp.gov.br/)

---

<div align="center">
  <p>Desenvolvido para promover um ambiente escolar mais focado e produtivo 😁</p>
  <p>© 2024 - Sistema de Gestão de Advertências</p>
</div>
