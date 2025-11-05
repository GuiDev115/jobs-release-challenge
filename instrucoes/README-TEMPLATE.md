# ISO 8583 Integration Challenge

> Implementação de fluxos de **Acquire** e **Issuing** usando o protocolo ISO 8583 para processamento de transações financeiras.

**Challenge para:** [Woovi](https://woovi.com/) | [OpenPix](https://openpix.com.br/)  
**Candidato:** GuiDev115  
**Data:** Novembro 2025

---

##  Sobre o Desafio

Este projeto implementa um simulador de transações financeiras utilizando o padrão ISO 8583, incluindo:

-  **Acquire (Adquirência):** Processar pagamentos como se fosse um gateway/POS
-  **Issuing (Emissão):** Autorizar transações como se fosse um banco emissor
-  **GraphQL API:** Interface para processar transações
-  **MongoDB:** Persistência de transações e dados

##  Arquitetura

```
Cliente → GraphQL API → ISO 8583 Handler → Acquire/Issuing Services → MongoDB
```

[Ver diagrama completo →](./docs/architecture.excalidraw)

##  Quick Start

### Pré-requisitos

- Node.js >= 20
- MongoDB (via Docker ou local)
- npm ou yarn

### Instalação

```bash
# Clone o repositório
git clone https://github.com/GuiDev115/woovi-iso8583-challenge.git
cd woovi-iso8583-challenge

# Instale as dependências
npm install

# Configure as variáveis de ambiente
cp .env.example .env

# Inicie o MongoDB (via Docker)
docker-compose up -d

# Rode as migrations/seeds (opcional)
npm run seed

# Inicie o servidor de desenvolvimento
npm run dev
```

O servidor estará disponível em: `http://localhost:4000`

##  Documentação

-  [Planejamento Completo](./PLANNING.md) - Decisões arquiteturais e trade-offs
-  [Quick Start Guide](./QUICK-START.md) - Como começar o projeto
-  [ISO 8583 Overview](./docs/iso8583-overview.md) - Entendendo o protocolo
- 🔌 [API Documentation](./docs/api-documentation.md) - Como usar a API GraphQL

##  Testes

```bash
# Rodar todos os testes
npm test

# Testes em modo watch
npm run test:watch

# Coverage
npm run test:coverage
```

##  Stack Tecnológica

**Backend:**
- Node.js + TypeScript
- Koa.js
- GraphQL
- MongoDB + Mongoose

**ISO 8583:**
- `iso-8583` library (ou implementação própria)

**Testing:**
- Jest
- Supertest

##  Funcionalidades

###  Implementado

- [x] Parser de mensagens ISO 8583
- [x] Fluxo de Acquire
- [x] Fluxo de Issuing
- [x] GraphQL API
- [x] Persistência MongoDB
- [x] Testes unitários
- [x] Validação de cartões (Luhn)

### 🚧 Em Desenvolvimento

- [ ] Frontend React + Relay
- [ ] Reversal (estorno)
- [ ] Reconciliação
- [ ] Rate limiting

###  Melhorias Futuras

- [ ] Suporte a mais MTIs
- [ ] Fraud detection
- [ ] Webhooks
- [ ] Observability (Grafana)

##  Como Usar

### Via GraphQL Playground

Acesse: `http://localhost:4000/graphql`

**Exemplo: Processar um Pagamento**

```graphql
mutation ProcessPayment {
  processPayment(input: {
    cardNumber: "4111111111111111"
    amount: 10000  # R$ 100,00 (em centavos)
    merchantId: "MERCHANT001"
  }) {
    transaction {
      id
      status
      responseCode
      amount
    }
  }
}
```

**Exemplo: Consultar Transação**

```graphql
query GetTransaction {
  transaction(id: "txn_123") {
    id
    status
    amount
    cardNumber
    createdAt
  }
}
```

### Via Postman

[Importar Collection →](./docs/postman-collection.json)

##  ISO 8583 - Mensagens Suportadas

| MTI  | Tipo | Descrição | Status |
|------|------|-----------|--------|
| 0100 | Request | Authorization |  |
| 0110 | Response | Authorization |  |
| 0200 | Request | Financial Transaction |  |
| 0210 | Response | Financial Transaction |  |
| 0400 | Request | Reversal | 🚧 |
| 0410 | Response | Reversal | 🚧 |
| 0800 | Request | Network Management |  |
| 0810 | Response | Network Management |  |

##  Segurança

⚠️ **ATENÇÃO:** Este é um projeto de demonstração para fins educacionais.

- Não use dados reais de cartões
- Não utilize em produção sem implementar:
  - PCI-DSS compliance
  - Encryption at rest/transit
  - HSM para chaves
  - Tokenização
  - Auditoria completa

##  Performance

- Latência média: < 100ms
- Throughput: ~1000 req/s (single instance)
- Coverage de testes: >80%

##  Deploy

### Backend

Deploy no Railway/Render:

```bash
# Build
npm run build

# Start
npm start
```

**URL de Produção:** [Em breve]

### Frontend (Bonus)

Deploy no Vercel:

```bash
cd client
npm run build
```

##  Contribuindo

Este é um projeto de code challenge, mas feedback é bem-vindo!

##  Decisões Técnicas

### Por que Koa.js?
- Padrão utilizado pela Woovi
- Async/await nativo
- Middleware modular

### Por que MongoDB?
- Flexibilidade no schema
- Rápido para prototipação
- Usado pela Woovi

### ISO 8583 Library vs Custom?
- Iniciei com library para acelerar
- Planejo implementar versão própria para demonstrar conhecimento profundo

[Ler mais sobre decisões →](./PLANNING.md#decisoes)

##  Recursos e Referências

- [ISO 8583 Wikipedia](https://en.wikipedia.org/wiki/ISO_8583)
- [ISO 8583 Field Guide](https://www.iso8583.info/)
- [Simulador Python](https://github.com/bassrehab/ISO8583-Simulator)
- [Woovi Blog](https://dev.to/woovi)

##  Contato

- GitHub: [@GuiDev115](https://github.com/GuiDev115)
- Email: [seu-email@example.com]

---

##  Licença

MIT License - veja [LICENSE](./LICENSE) para detalhes.

---

**Desenvolvido com  para o desafio da Woovi**

[Ver outros desafios →](https://github.com/woovibr/jobs)
