# Evo CRM — Deploy Guide para EasyPanel

## Pré-requisitos

- VPS com Ubuntu 20.04+ (recomendado: 4GB RAM, 2 CPUs, 40GB SSD)
- Docker e Docker Compose instalados
- Domínio configurado com DNS apontando para o IP da VPS
- EasyPanel instalado na VPS

## Passo 1: Preparar o Repositório

Clone o repositório em sua máquina local:

```bash
git clone --recurse-submodules https://github.com/evolution-foundation/evo-crm-community.git
cd evo-crm-community
```

Copie o arquivo de ambiente e configure:

```bash
cp .env.example .env
```

**Edite o `.env` com suas configurações de produção:**

```bash
# Database
POSTGRES_HOST=localhost
POSTGRES_USERNAME=evo_crm_user
POSTGRES_PASSWORD=<senha_forte_gerada>
POSTGRES_DATABASE=evo_community_prod

# Redis
REDIS_PASSWORD=<senha_forte_gerada>

# URLs de Produção (SUBSTITUA pelos seus domínios)
FRONTEND_URL=https://seu-dominio.com
AUTH_SERVICE_URL=https://auth.seu-dominio.com
BACKEND_URL=https://crm.seu-dominio.com

# VITE URLs para o Frontend
VITE_API_URL=https://crm.seu-dominio.com
VITE_AUTH_API_URL=https://auth.seu-dominio.com
VITE_EVOAI_API_URL=https://api.seu-dominio.com
VITE_AGENT_PROCESSOR_URL=https://processor.seu-dominio.com

# SMTP (configure com seu provedor)
SMTP_ADDRESS=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=seu-email@gmail.com
SMTP_PASSWORD=sua-senha-de-app
```

## Passo 2: Gerar Certificados SSL

Para Let's Encrypt (recomendado):

```bash
# Instale o certbot
sudo apt update
sudo apt install certbot python3-certbot-nginx

# Pare o nginx se estiver rodando
sudo systemctl stop nginx

# Gere os certificados (substitua pelos seus domínios)
sudo certbot certonly --standalone -d seu-dominio.com \
  -d auth.seu-dominio.com \
  -d crm.seu-dominio.com \
  -d api.seu-dominio.com \
  -d processor.seu-dominio.com

# Copie os certificados para o projeto
sudo cp /etc/letsencrypt/live/seu-dominio.com/fullchain.pem nginx/ssl/
sudo cp /etc/letsencrypt/live/seu-dominio.com/privkey.pem nginx/ssl/
sudo chown -R $USER:$USER nginx/ssl/
```

**Alternativa: Certificados Auto-assinados (para testes)**

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/ssl/privkey.pem \
  -out nginx/ssl/fullchain.pem \
  -subj "/C=BR/ST=SP/L=Sao Paulo/O=EvoCRM/CN=seu-dominio.com"
```

## Passo 3: Criar Repositório Git

```bash
# Inicialize o git (remova .git existente se houver)
rm -rf .git
git init

# Configure git
git config user.name "Seu Nome"
git config user.email "seu-email@exemplo.com"

# Adicione todos os arquivos
git add .

# Commit inicial
git commit -m "Initial commit - Evo CRM Community"

# Crie branch de produção
git checkout -b production

# Adicione remote (substitua pelo seu repositório)
git remote add origin https://github.com/seu-usuario/evo-crm.git

# Push
git push -u origin production
```

## Passo 4: Deploy no EasyPanel

### Opção A: Usando Docker Compose diretamente

```bash
# Na VPS, clone o repositório
git clone https://github.com/seu-usuario/evo-crm.git
cd evo-crm
git checkout production

# Copie e configure o .env
cp .env.example .env
nano .env  # Edite com suas configurações

# Inicie os serviços
docker-compose -f docker-compose.prod.yml up -d

# Verifique os logs
docker-compose -f docker-compose.prod.yml logs -f
```

### Opção B: Usando EasyPanel UI

1. **Acesse o EasyPanel** em `https://seu-servidor:3000`

2. **Crie um novo projeto:**
   - Clique em "New Project"
   - Nome: `evo-crm`
   - Type: Docker Compose

3. **Configure o repositório:**
   - Repository: `https://github.com/seu-usuario/evo-crm`
   - Branch: `production`
   - Docker Compose File: `docker-compose.prod.yml`

4. **Configure as variáveis de ambiente:**
   - Adicione todas as variáveis do `.env`
   - Marque como "Secret" as senhas

5. **Configure os domínios:**
   - Frontend: `https://seu-dominio.com`
   - CRM API: `https://crm.seu-dominio.com`
   - Auth: `https://auth.seu-dominio.com`

6. **Clique em "Deploy"**

## Passo 5: Configurar DNS

Adicione os seguintes registros DNS no seu provedor:

```
Tipo    Nome    Valor
A       @       IP_DA_VPS
A       auth    IP_DA_VPS
A       crm     IP_DA_VPS
A       api     IP_DA_VPS
A       processor IP_DA_VPS
```

## Passo 6: Verificar Instalação

Acesse:
- Frontend: `https://seu-dominio.com`
- Auth Service: `https://auth.seu-dominio.com/health`
- CRM API: `https://crm.seu-dominio.com/health/live`
- Core API: `https://api.seu-dominio.com/evo/api/v1/health`
- Processor: `https://processor.seu-dominio.com/health`

## Estrutura de Portas

| Serviço | Porta Intern | Descrição |
|---------|-------------|-----------|
| Frontend | 80 | Interface React |
| CRM API | 3000 | API Rails do CRM |
| Auth | 3001 | Serviço de autenticação |
| Core | 5555 | Serviço Core Go |
| Processor | 8000 | Processador de Agentes |
| Bot Runtime | 8080 | Runtime de Bots |

## Comandos Úteis

```bash
# Ver status dos serviços
docker-compose -f docker-compose.prod.yml ps

# Ver logs de um serviço específico
docker-compose -f docker-compose.prod.yml logs -f evo-crm

# Reiniciar um serviço
docker-compose -f docker-compose.prod.yml restart evo-crm

# Atualizar código
git pull origin production
docker-compose -f docker-compose.prod.yml up -d --build

# Backup do banco
docker-compose -f docker-compose.prod.yml exec postgres pg_dump -U postgres evo_community > backup.sql

# Limpar tudo
docker-compose -f docker-compose.prod.yml down -v
```

## Troubleshooting

### Serviço não inicia
```bash
# Ver logs detalhados
docker-compose -f docker-compose.prod.yml logs <servico>

# Verificar variáveis de ambiente
docker-compose -f docker-compose.prod.yml config
```

### Problema de conexão com banco
```bash
# Verificar se PostgreSQL está rodando
docker-compose -f docker-compose.prod.yml exec postgres pg_isready

# Recriar banco
docker-compose -f docker-compose.prod.yml exec evo-crm bundle exec rails db:drop db:create db:migrate
```

### Frontend mostra erro 502
```bash
# Verificar se todos os serviços estão healthy
docker-compose -f docker-compose.prod.yml ps

# Rebuild do frontend
docker-compose -f docker-compose.prod.yml up -d --build evo-frontend
```

## Recursos Recomendados

| Componente | Mínimo | Recomendado |
|------------|--------|-------------|
| RAM | 4GB | 8GB |
| CPU | 2 cores | 4 cores |
| Disco | 40GB SSD | 80GB SSD |
| RAM PostgreSQL | 512MB | 2GB |
| RAM Redis | 256MB | 512MB |
