#  Quick Start - ISO 8583 Challenge

## Passo a Passo para Começar AGORA

### 1. Criar Novo Repositório (5 min)

```bash
# No GitHub
# 1. Ir em: https://github.com/new
# 2. Nome: woovi-iso8583-challenge
# 3. Description: "ISO 8583 Integration Challenge for Woovi - Acquire & Issuing flows"
# 4. Public 
# 5. Add README 
# 6. Add .gitignore: Node 
# 7. License: MIT 
# 8. Create repository

# Clonar localmente
cd ~/documentos
git clone https://github.com/GuiDev115/woovi-iso8583-challenge.git
cd woovi-iso8583-challenge
```

### 2. Copiar Documentação (2 min)

```bash
# Copiar o planejamento
cp ~/documentos/jobs-release-challenge/ISO8583-PLANNING.md ./PLANNING.md
cp ~/documentos/jobs-release-challenge/QUICK-START.md ./QUICK-START.md

# Commit inicial
git add .
git commit -m "docs: add planning and quick start guide"
git push
```

### 3. Setup Projeto Node.js (10 min)

```bash
# Inicializar projeto
npm init -y

# Instalar dependências principais
npm install koa koa-router koa-bodyparser graphql graphql-http mongoose dotenv

# Instalar dependências de desenvolvimento
npm install -D typescript @types/node @types/koa @types/koa-router @types/koa-bodyparser ts-node-dev jest @types/jest ts-jest eslint prettier

# Instalar biblioteca ISO 8583
npm install iso-8583

# Criar tsconfig.json
npx tsc --init

# Commit
git add .
git commit -m "chore: setup Node.js project with dependencies"
git push
```

### 4. Estrutura de Pastas (3 min)

```bash
# Criar estrutura
mkdir -p src/{server,iso8583,services,models,utils,config}
mkdir -p tests/{unit,integration,e2e}
mkdir -p docs
mkdir -p scripts

# Criar arquivos base
touch src/server/index.ts
touch src/server/app.ts
touch .env.example
touch .gitignore
touch docker-compose.yml

# Commit
git add .
git commit -m "chore: create project structure"
git push
```

### 5. Estudar ISO 8583 (30-60 min)

**Leitura obrigatória:**
- [ ] https://en.wikipedia.org/wiki/ISO_8583
- [ ] https://www.iso8583.info/
- [ ] Olhar o simulador Python: https://github.com/bassrehab/ISO8583-Simulator

**Entender:**
- O que é MTI (Message Type Indicator)
- Como funciona o bitmap
- Campos principais: 2, 3, 4, 11, 37, 39
- Diferença entre 0100 (auth request) e 0110 (auth response)

### 6. Primeiro Código - Hello World ISO 8583 (30 min)

```typescript
// src/iso8583/hello.ts
import ISO8583 from 'iso-8583';

// Criar uma mensagem de autorização (0100)
const message = {
  0: '0100',  // MTI
  2: '4111111111111111',  // PAN (número do cartão)
  3: '000000',  // Processing code
  4: '000000001000',  // Amount (R$ 10,00)
  11: '123456',  // STAN
  37: '123456789012',  // RRN
};

// Testar parse/build
console.log('Message:', message);
```

### 7. MongoDB Local (10 min)

```bash
# Via Docker
docker run -d \
  --name mongo-iso8583 \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  mongo:latest

# Ou usar MongoDB Compass
# Download: https://www.mongodb.com/try/download/compass
```

### 8. Primeiro Teste (15 min)

```bash
# Configurar Jest
npx ts-jest config:init

# Criar primeiro teste
# tests/unit/iso8583.test.ts
# Testar parse de uma mensagem simples

# Rodar
npm test
```

##  Checklist Dia 1

- [ ] Repositório criado no GitHub
- [ ] Projeto Node.js + TypeScript configurado
- [ ] Estrutura de pastas criada
- [ ] Dependências instaladas
- [ ] MongoDB rodando (Docker)
- [ ] Leitura sobre ISO 8583 feita
- [ ] Primeiro teste passando
- [ ] README.md atualizado com instruções de setup

##  O que fazer primeiro

### Opção A: Começar pelo Core (Recomendado)
1. Implementar parser ISO 8583
2. Criar tipos TypeScript
3. Testar mensagens básicas
4. Depois partir para serviços

### Opção B: Começar pelo API
1. Setup Koa + GraphQL
2. Criar schema básico
3. Implementar mutations
4. Depois integrar ISO 8583

### Opção C: Estudar Primeiro
1. Analisar simulador Python
2. Ler RFCs e specs
3. Desenhar arquitetura no Excalidraw
4. Depois começar a codar

**Minha recomendação: Opção A** - Entender o core do ISO 8583 é fundamental.

##  Mensagem para o CEO

Quando tiver o repo pronto (mesmo com pouco código), já pode mandar mensagem:

```
Oi! Comecei o desafio do ISO 8583 que você mencionou.

Repo: https://github.com/GuiDev115/woovi-iso8583-challenge

Estou implementando os fluxos de Acquire e Issuing usando Node.js + TypeScript.
Por enquanto foquei em [o que já fez].

Vou atualizando o repo conforme avanço. Qualquer feedback é bem-vindo!
```

##  Recursos Rápidos

- Validar cartão: https://www.freeformatter.com/credit-card-number-generator-validator.html
- Testar ISO8583: https://neapay.com/free-payments-iso8583-simulator-tools.html
- Gerar cartões teste: https://www.creditcardvalidator.org/generator

##  Comandos Úteis

```bash
# Desenvolvimento
npm run dev

# Testes
npm test
npm run test:watch
npm run test:coverage

# Build
npm run build

# Lint
npm run lint
npm run lint:fix

# Docker
docker-compose up -d
docker-compose logs -f
docker-compose down
```

##  Troubleshooting

**MongoDB não conecta:**
```bash
# Ver se está rodando
docker ps | grep mongo

# Ver logs
docker logs mongo-iso8583
```

**TypeScript dando erro:**
```bash
# Limpar cache
rm -rf node_modules package-lock.json
npm install
```

**Jest não roda:**
```bash
# Verificar configuração
cat jest.config.js
```

##  Aprendizados Esperados

No final deste desafio você vai saber:
-  Como funciona o protocolo ISO 8583
-  Diferença entre acquire e issuing
-  Como processar transações financeiras
-  GraphQL + Relay
-  Arquitetura de sistemas de pagamento
-  Testes em aplicações financeiras

---

**Agora é mão na massa! **

Próximo arquivo para ler: `PLANNING.md` (o documento completo)
