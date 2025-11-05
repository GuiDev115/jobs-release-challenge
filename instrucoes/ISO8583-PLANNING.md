# ISO 8583 Integration Challenge - Planejamento Completo

**Candidato:** GuiDev115  
**Data:** 04 de Novembro de 2025  
**Challenge:** Integrate with ISO 8583 simulator  
**Empresa:** Woovi

---

##  Índice

1. [Entendimento do Desafio](#entendimento)
2. [O que é ISO 8583](#iso8583)
3. [Arquitetura Proposta](#arquitetura)
4. [Stack Tecnológica](#stack)
5. [Fluxos: Acquire vs Issuing](#fluxos)
6. [Estrutura do Projeto](#estrutura)
7. [Fases de Implementação](#fases)
8. [Decisões Técnicas e Trade-offs](#decisoes)
9. [Deliverables](#deliverables)
10. [Timeline Estimado](#timeline)

---

##  1. Entendimento do Desafio {#entendimento}

### Requisitos do Desafio

- [ ] Integrar com simulador ISO 8583 existente
- [ ] Implementar fluxo de **Acquire** (adquirência)
- [ ] Implementar fluxo de **Issuing** (emissão)
- [ ] Criar API para processar transações
- [ ] Testes automatizados
- [ ] Documentação completa

### Referências Fornecidas
- Simulador Python: https://github.com/bassrehab/ISO8583-Simulator
- Ferramentas: https://neapay.com/free-payments-iso8583-simulator-tools.html

### Objetivo
Demonstrar capacidade de:
- Entender protocolos financeiros complexos
- Integrar sistemas existentes
- Implementar fluxos de pagamento
- Documentar decisões arquiteturais

---

##  2. O que é ISO 8583 {#iso8583}

### Definição
ISO 8583 é um padrão internacional para mensagens de transações financeiras eletrônicas, usado principalmente em:
- Autorizações de cartão de crédito/débito
- ATMs (caixas eletrônicos)
- POS (Point of Sale - maquininhas)
- Transferências bancárias

### Estrutura da Mensagem

Uma mensagem ISO 8583 possui:

1. **MTI (Message Type Indicator)** - 4 dígitos
   - Versão da mensagem (0 = ISO 8583:1987, 1 = ISO 8583:1993)
   - Classe da mensagem (1 = Autorização, 2 = Financeira, etc)
   - Função (0 = Request, 1 = Response)
   - Origem (0 = Acquirer, 1 = Issuer)

2. **Bitmap** - indica quais campos estão presentes
   - Primary bitmap (campos 1-64)
   - Secondary bitmap (campos 65-128) se presente

3. **Data Elements** - até 128 campos possíveis
   - Campo 2: PAN (Primary Account Number) - número do cartão
   - Campo 3: Processing Code
   - Campo 4: Amount (valor da transação)
   - Campo 11: STAN (System Trace Audit Number)
   - Campo 37: RRN (Retrieval Reference Number)
   - Campo 39: Response Code
   - E muitos outros...

### Tipos de Mensagens Comuns

| MTI  | Descrição |
|------|-----------|
| 0100 | Authorization Request (pedido de autorização) |
| 0110 | Authorization Response (resposta de autorização) |
| 0200 | Financial Transaction Request (transação financeira) |
| 0210 | Financial Transaction Response |
| 0400 | Reversal Request (estorno) |
| 0410 | Reversal Response |
| 0800 | Network Management Request (echo test) |
| 0810 | Network Management Response |

---

##  3. Arquitetura Proposta {#arquitetura}

### Visão Geral

```
┌─────────────┐         ┌──────────────────┐         ┌─────────────┐
│   Cliente   │────────▶│  GraphQL API     │────────▶│  MongoDB    │
│  (Frontend) │◀────────│  (Koa.js)        │◀────────│             │
└─────────────┘         └──────────────────┘         └─────────────┘
                                │
                                │
                        ┌───────▼────────┐
                        │  ISO 8583      │
                        │  Message       │
                        │  Handler       │
                        └────────┬───────┘
                                 │
                ┌────────────────┼────────────────┐
                │                │                │
         ┌──────▼──────┐  ┌─────▼─────┐  ┌──────▼──────┐
         │  Acquire    │  │  Issuing  │  │  Simulator  │
         │  Service    │  │  Service  │  │  Client     │
         └─────────────┘  └───────────┘  └─────────────┘
```

### Componentes Principais

1. **GraphQL API Layer**
   - Mutations para iniciar transações
   - Queries para consultar status
   - Subscriptions para notificações em tempo real (opcional)

2. **ISO 8583 Message Handler**
   - Parser de mensagens ISO 8583
   - Builder de mensagens
   - Validação de campos obrigatórios

3. **Acquire Service**
   - Recebe requisições de venda/pagamento
   - Encaminha para o issuer
   - Processa resposta

4. **Issuing Service**
   - Valida cartão e saldo
   - Aprova ou nega transação
   - Retorna código de resposta

5. **Simulator Client**
   - Integração com simulador Python externo
   - Fallback para simulador interno (Node.js)

---

##  4. Stack Tecnológica {#stack}

### Backend Core
- **Runtime:** Node.js (v20+)
- **Framework:** Koa.js
- **Language:** TypeScript
- **API:** GraphQL (graphql-js + graphql-http)

### Database
- **Primary:** MongoDB
- **ODM:** Mongoose
- **Collections:**
  - `transactions` - histórico de transações
  - `cards` - cartões cadastrados (para issuing)
  - `merchants` - comerciantes (para acquire)
  - `audit_logs` - logs de auditoria

### ISO 8583
- **Library:** `iso-8583` (npm) ou implementação própria
- **Alternativas:** 
  - `iso_8583` (Python - via child process)
  - `jpos` (Java - via RPC se necessário)

### Testing
- **Framework:** Jest
- **Coverage:** > 80%
- **Types:** Unit, Integration, E2E

### DevOps
- **Container:** Docker + Docker Compose
- **CI/CD:** GitHub Actions
- **Deploy:** Railway / Render / Heroku (Backend)
- **Deploy:** Vercel / Netlify (Frontend)

### Frontend (Bonus)
- **Framework:** React 18
- **Build:** Vite
- **Router:** React Router v6
- **GraphQL Client:** Relay
- **UI:** Shadcn/ui
- **Styling:** Tailwind CSS

---

##  5. Fluxos: Acquire vs Issuing {#fluxos}

### 5.1 Fluxo de Acquire (Adquirência)

**Cenário:** Uma maquininha de cartão processa uma compra

```
┌─────────┐     ┌──────────┐     ┌─────────┐     ┌─────────┐
│   POS   │────▶│ Acquire  │────▶│ Network │────▶│ Issuer  │
│(Máquina)│     │(Nossa API)│     │ (Sim.)  │     │(Banco)  │
└─────────┘     └──────────┘     └─────────┘     └─────────┘
     │               │                 │               │
     │  0200 Req     │                 │               │
     │──────────────▶│  0200 Req       │               │
     │               │────────────────▶│  0100 Req     │
     │               │                 │──────────────▶│
     │               │                 │               │
     │               │                 │  0110 Resp    │
     │               │  0210 Resp      │◀──────────────│
     │  0210 Resp    │◀────────────────│               │
     │◀──────────────│                 │               │
```

**Responsabilidades do Acquire:**
1. Receber requisição de pagamento do merchant/POS
2. Validar dados básicos (formato do cartão, valor)
3. Rotear para o issuer correto (baseado no BIN)
4. Traduzir mensagens entre formatos
5. Retornar resposta ao merchant
6. Registrar transação para reconciliação

**Campos importantes:**
- Campo 2: Número do cartão
- Campo 4: Valor da transação
- Campo 18: Merchant Category Code
- Campo 42: Merchant ID

### 5.2 Fluxo de Issuing (Emissão)

**Cenário:** Banco autoriza ou nega uma compra

```
┌──────────┐     ┌─────────┐     ┌──────────┐
│ Acquire  │────▶│ Issuer  │────▶│ Database │
│          │     │(Nossa API)│     │(Cards)   │
└──────────┘     └─────────┘     └──────────┘
     │               │                 │
     │  0100 Req     │                 │
     │──────────────▶│  Query Card     │
     │               │────────────────▶│
     │               │                 │
     │               │  Card Data      │
     │               │◀────────────────│
     │               │                 │
     │               │ Check Balance   │
     │               │ Check Limits    │
     │               │ Fraud Check     │
     │               │                 │
     │  0110 Resp    │                 │
     │◀──────────────│                 │
```

**Responsabilidades do Issuer:**
1. Receber requisição de autorização
2. Validar cartão (existe? ativo? não expirado?)
3. Verificar saldo/limite disponível
4. Aplicar regras de negócio (valor mínimo/máximo)
5. Fraud detection básico (opcional)
6. Gerar código de resposta (aprovado/negado)
7. Debitar saldo (se aprovado)

**Códigos de Resposta Comuns:**
- `00` - Approved
- `05` - Do not honor
- `14` - Invalid card number
- `51` - Insufficient funds
- `54` - Expired card
- `91` - Issuer not available

---

##  6. Estrutura do Projeto {#estrutura}

```
woovi-iso8583-challenge/
│
├── README.md                          # Overview do projeto
├── PLANNING.md                        # Este documento
├── package.json
├── tsconfig.json
├── docker-compose.yml
├── .env.example
│
├── docs/
│   ├── architecture.excalidraw        # Diagrama de arquitetura
│   ├── iso8583-fields.md             # Referência de campos
│   ├── api-documentation.md           # API GraphQL docs
│   └── decisions.md                   # ADRs (Architecture Decision Records)
│
├── src/
│   ├── server/
│   │   ├── index.ts                   # Entry point
│   │   ├── app.ts                     # Koa app setup
│   │   ├── schema/                    # GraphQL schema
│   │   │   ├── index.ts
│   │   │   ├── mutations/
│   │   │   │   ├── ProcessPayment.ts
│   │   │   │   ├── AuthorizeTransaction.ts
│   │   │   │   └── ReverseTransaction.ts
│   │   │   ├── queries/
│   │   │   │   ├── GetTransaction.ts
│   │   │   │   └── ListTransactions.ts
│   │   │   └── types/
│   │   │       ├── Transaction.ts
│   │   │       ├── Card.ts
│   │   │       └── Merchant.ts
│   │   │
│   │   └── middlewares/
│   │       ├── auth.ts
│   │       └── errorHandler.ts
│   │
│   ├── iso8583/
│   │   ├── parser.ts                  # Parse ISO 8583 messages
│   │   ├── builder.ts                 # Build ISO 8583 messages
│   │   ├── validator.ts               # Validate message format
│   │   ├── fields.ts                  # Field definitions
│   │   └── types.ts                   # TypeScript types
│   │
│   ├── services/
│   │   ├── AcquireService.ts          # Lógica de adquirência
│   │   ├── IssuingService.ts          # Lógica de emissão
│   │   ├── TransactionService.ts      # Gerenciamento de transações
│   │   └── SimulatorClient.ts         # Cliente para simulador externo
│   │
│   ├── models/
│   │   ├── Transaction.ts             # Mongoose model
│   │   ├── Card.ts
│   │   ├── Merchant.ts
│   │   └── AuditLog.ts
│   │
│   ├── utils/
│   │   ├── luhn.ts                    # Validação de cartão (Luhn algorithm)
│   │   ├── logger.ts
│   │   └── constants.ts
│   │
│   └── config/
│       ├── database.ts
│       └── environment.ts
│
├── tests/
│   ├── unit/
│   │   ├── iso8583.test.ts
│   │   ├── luhn.test.ts
│   │   └── services/
│   │
│   ├── integration/
│   │   ├── acquire.test.ts
│   │   └── issuing.test.ts
│   │
│   └── e2e/
│       └── transaction-flow.test.ts
│
├── scripts/
│   ├── seed.ts                        # Popular DB com dados de teste
│   └── generate-test-cards.ts
│
└── client/                            # Frontend (Bonus)
    ├── src/
    │   ├── App.tsx
    │   ├── components/
    │   └── relay/
    └── package.json
```

---

##  7. Fases de Implementação {#fases}

### Fase 1: Setup e Fundação (Dia 1-2)
- [ ] Criar repositório no GitHub
- [ ] Setup inicial: Node.js + TypeScript + Koa
- [ ] Configurar MongoDB (local + Docker)
- [ ] Setup Jest para testes
- [ ] Criar estrutura de pastas
- [ ] Configurar ESLint + Prettier
- [ ] Docker Compose para ambiente local

### Fase 2: ISO 8583 Core (Dia 3-4)
- [ ] Estudar formato de mensagem ISO 8583
- [ ] Implementar parser de mensagens
- [ ] Implementar builder de mensagens
- [ ] Criar tipos TypeScript para campos
- [ ] Testes unitários para parser/builder
- [ ] Documentar campos principais

### Fase 3: Models e Database (Dia 4-5)
- [ ] Definir schemas Mongoose:
  - Transaction
  - Card
  - Merchant
  - AuditLog
- [ ] Implementar seeds para dados de teste
- [ ] Gerar cartões de teste válidos

### Fase 4: Issuing Service (Dia 5-7)
- [ ] Implementar lógica de autorização
- [ ] Validação de cartão
- [ ] Verificação de saldo
- [ ] Geração de response codes
- [ ] Testes de issuing
- [ ] Documentar regras de negócio

### Fase 5: Acquire Service (Dia 7-9)
- [ ] Implementar lógica de adquirência
- [ ] Roteamento de transações
- [ ] Integração acquire ↔ issuing
- [ ] Tratamento de respostas
- [ ] Testes de acquire
- [ ] Fluxo completo funcional

### Fase 6: GraphQL API (Dia 9-11)
- [ ] Definir schema GraphQL
- [ ] Implementar mutations:
  - `processPayment`
  - `authorizeTransaction`
  - `reverseTransaction`
- [ ] Implementar queries:
  - `getTransaction`
  - `listTransactions`
- [ ] Testes de integração GraphQL
- [ ] GraphQL Playground configurado

### Fase 7: Integração com Simulador (Dia 11-12)
- [ ] Analisar simulador Python
- [ ] Criar cliente para comunicação
- [ ] Testes de integração
- [ ] Fallback para simulador interno

### Fase 8: Testes e Qualidade (Dia 12-13)
- [ ] Coverage > 80%
- [ ] Testes E2E completos
- [ ] Load testing básico
- [ ] Correção de bugs

### Fase 9: Documentação (Dia 13-14)
- [ ] README completo com:
  - Instruções de setup
  - Como rodar
  - Exemplos de uso
- [ ] Diagrama de arquitetura (Excalidraw)
- [ ] Documentação de decisões técnicas
- [ ] Postman collection
- [ ] Video demo (opcional)

### Fase 10: Deploy (Dia 14-15)
- [ ] Deploy backend (Railway/Render)
- [ ] Configurar variáveis de ambiente
- [ ] MongoDB Atlas
- [ ] CI/CD com GitHub Actions
- [ ] Testes em produção

### Fase 11 (Bonus): Frontend (Dia 15-17)
- [ ] Setup React + Vite + Relay
- [ ] Tela de simulação de pagamento
- [ ] Dashboard de transações
- [ ] Deploy no Vercel

---

##  8. Decisões Técnicas e Trade-offs {#decisoes}

### 8.1 Por que Koa.js em vez de Express?
**Decisão:** Usar Koa.js  
**Razão:** É o padrão da Woovi (mencionado nos desafios)  
**Trade-off:** Menor ecossistema que Express, mas mais moderno e com async/await nativo

### 8.2 Parser ISO 8583: Lib vs Custom?
**Opções:**
1. Usar biblioteca pronta (`iso-8583` npm package)
2. Implementação própria

**Decisão:** Começar com lib, implementar custom depois se necessário  
**Razão:** Acelera desenvolvimento, mas mostra compreensão profunda se implementar próprio  
**Recomendação:** Usar lib para MVP, depois criar versão própria documentada

### 8.3 Comunicação com Simulador Python
**Opções:**
1. Socket TCP direto
2. HTTP REST
3. gRPC
4. Child process

**Decisão:** HTTP REST (se o simulador suportar) ou implementar próprio simulador em Node  
**Razão:** Mais simples, fácil de debugar  
**Trade-off:** Menos performático que TCP puro, mas suficiente para demo

### 8.4 Banco de Dados
**Decisão:** MongoDB  
**Razão:** Padrão da Woovi, flexível para prototipação  
**Alternativa considerada:** PostgreSQL (melhor para dados financeiros transacionais)  
**Trade-off:** MongoDB é menos ACID, mas suficiente para o desafio

### 8.5 Real-time vs Batch
**Decisão:** Processar transações em real-time (síncrono)  
**Razão:** Simula o mundo real (POS espera resposta imediata)  
**Consideração futura:** Fila (Bull/BullMQ) para processamento assíncrono em produção real

### 8.6 Segurança
**Para o desafio:**
- Dados de teste, não usar cartões reais
- Tokens simulados
- Básico de autenticação (Bearer token)

**Em produção real seria necessário:**
- Encryption at rest
- TLS/SSL
- PCI-DSS compliance
- Tokenização de cartões
- HSM para chaves criptográficas

---

##  9. Deliverables {#deliverables}

### Obrigatórios
- [x] Repositório GitHub público
- [ ] README.md completo
- [ ] Código-fonte documentado
- [ ] Testes (Jest)
- [ ] API GraphQL funcional
- [ ] Fluxos de Acquire e Issuing implementados
- [ ] Diagrama de arquitetura (Excalidraw)
- [ ] Documento de decisões (este arquivo)
- [ ] Deploy da API (URL acessível)

### Diferenciais
- [ ] Frontend React + Relay
- [ ] Postman Collection
- [ ] GraphQL Playground público
- [ ] CI/CD configurado
- [ ] Documentação de API (GraphQL schema)
- [ ] Video demo explicando o projeto
- [ ] Metrics/Observability (logs estruturados)
- [ ] Docker Compose para rodar localmente
- [ ] Coverage report publicado

### Extras que impressionam
- [ ] Implementação própria de parser ISO 8583
- [ ] Suporte a múltiplos MTIs
- [ ] Reversal (estorno) funcional
- [ ] Reconciliação de transações
- [ ] Rate limiting
- [ ] Webhook notifications
- [ ] Multi-tenancy (diferentes merchants)

---

## ⏱️ 10. Timeline Estimado {#timeline}

### Cenário Rápido (1 semana)
- Foco em MVP funcional
- Acquire + Issuing básicos
- GraphQL API
- Testes essenciais
- Deploy básico

### Cenário Completo (2 semanas)
- Todas as features core
- Testes abrangentes (>80% coverage)
- Documentação completa
- Frontend bonus
- Deploy com CI/CD

### Cenário Senior (3 semanas)
- Tudo do cenário completo +
- Parser ISO 8583 próprio
- Features avançadas (reversal, reconciliation)
- Observability
- Load testing
- Video demo profissional

---

##  Próximos Passos Imediatos

1.  **Criar novo repositório:** `woovi-iso8583-challenge`
2.  **Copiar este documento** para o novo repo
3.  **Estudar ISO 8583:** Ler specs, entender campos principais
4.  **Analisar simulador Python:** Ver como funciona
5.  **Setup inicial:** Node + TypeScript + Koa + MongoDB
6.  **Primeiro teste:** Conseguir fazer parse de uma mensagem 0100

---

##  Recursos Úteis

### ISO 8583
- Wikipedia: https://en.wikipedia.org/wiki/ISO_8583
- Tutorial: https://www.codeproject.com/Articles/100084/Introduction-to-ISO-8583
- Field Guide: https://www.iso8583.info/

### Libraries Node.js
- `iso-8583`: https://www.npmjs.com/package/iso-8583
- `jcard`: https://www.npmjs.com/package/jcard (validação Luhn)

### Woovi Stack
- Woovi Blog: https://dev.to/woovi
- Relay Docs: https://relay.dev/
- GraphQL Best Practices: https://graphql.org/learn/best-practices/

### Simuladores
- ISO8583-Simulator: https://github.com/bassrehab/ISO8583-Simulator
- Neapay Tools: https://neapay.com/free-payments-iso8583-simulator-tools.html
- jPOS: https://jpos.org/ (Java, mas boa referência)

---

##  Dicas Finais

1. **Documente enquanto desenvolve**, não deixe para depois
2. **Commits pequenos e frequentes** com mensagens claras
3. **Teste desde o início**, não deixe para o final
4. **Desenhe antes de codar** - vale mais 1 hora no Excalidraw que 10 horas refatorando
5. **Peça feedback cedo** - envie o repo mesmo incompleto para mostrar progresso
6. **Foco no MVP primeiro**, depois incrementa
7. **Qualidade > Quantidade** - melhor 3 features bem feitas que 10 pela metade

---

## ✉️ Contato e Envio

Quando finalizar:
- Email: sibelius@woovi.com
- X/Twitter: https://x.com/sseraphini
- Assunto: "ISO 8583 Challenge - [Seu Nome]"
- Incluir:
  - Link do repo
  - Link do deploy
  - Link do diagrama Excalidraw
  - Breve descrição (2-3 parágrafos) das decisões principais

---

**Boa sorte! **

Se tiver dúvidas durante a implementação, pergunte no Discord do Sibelius ou procure referências no [Awesome Woovi Challenge](https://github.com/entria/awesome-woovi-challenge).
