#  Guia de Estudo - ISO 8583

## O que você PRECISA saber sobre ISO 8583

###  Conceitos Fundamentais (30 minutos)

#### 1. O que é ISO 8583?
- **Protocolo de mensagens** para transações financeiras eletrônicas
- Criado nos anos 80, atualizado em 1993 e 2003
- Usado por: bancos, processadores de pagamento, ATMs, POS
- **Analogia:** É como o "HTTP das transações bancárias"

#### 2. Estrutura de uma Mensagem

```
┌────────────────────────────────────────────┐
│  ISO 8583 Message                          │
├────────────────────────────────────────────┤
│  1. MTI (4 bytes)                          │
│     Message Type Indicator                 │
│     Ex: 0100, 0110, 0200, 0210            │
├────────────────────────────────────────────┤
│  2. Bitmap (8 ou 16 bytes)                 │
│     Indica quais campos estão presentes    │
│     Primary: campos 1-64                   │
│     Secondary: campos 65-128               │
├────────────────────────────────────────────┤
│  3. Data Elements (variável)               │
│     Campos de dados (até 128 possíveis)    │
│     Campo 2: PAN (número do cartão)        │
│     Campo 4: Amount (valor)                │
│     Campo 39: Response Code                │
│     etc...                                 │
└────────────────────────────────────────────┘
```

#### 3. MTI - Message Type Indicator

4 dígitos que identificam o tipo de mensagem:

```
  M T I
  │ │ │ └─── Origin (Origem)
  │ │ └───── Function (Função)
  │ └─────── Class (Classe)
  └───────── Version (Versão)
```

**Exemplos:**
- `0100` - Authorization Request (pedido de autorização)
- `0110` - Authorization Response (resposta de autorização)
- `0200` - Financial Transaction Request
- `0210` - Financial Transaction Response

**Decodificando 0100:**
- `0` = ISO 8583:1987 version
- `1` = Authorization message
- `0` = Request
- `0` = Acquirer (origem do pedido)

**Decodificando 0110:**
- `0` = ISO 8583:1987 version
- `1` = Authorization message
- `1` = Response
- `0` = Acquirer (destino da resposta)

#### 4. Bitmap - O Mapa de Campos

**O que é:** Indica quais campos estão presentes na mensagem

**Como funciona:**
- Cada bit representa um campo (1 = presente, 0 = ausente)
- Primary bitmap: 64 bits (campos 1-64)
- Se bit 1 = 1, então há secondary bitmap (campos 65-128)

**Exemplo:**

```
Bitmap (hex): 7234000102C00000
Em binário:  0111001000110100...

Bit 2 = 1 → Campo 2 presente (PAN)
Bit 3 = 1 → Campo 3 presente (Processing Code)
Bit 4 = 1 → Campo 4 presente (Amount)
Bit 11 = 1 → Campo 11 presente (STAN)
```

---

##  Campos Mais Importantes (15 minutos)

### Campos Obrigatórios/Comuns

| Campo | Nome | Tipo | Exemplo | Uso |
|-------|------|------|---------|-----|
| 0 | MTI | N 4 | 0100 | Tipo de mensagem |
| 2 | PAN | N ..19 | 4111111111111111 | Número do cartão |
| 3 | Processing Code | N 6 | 000000 | Tipo de transação |
| 4 | Amount | N 12 | 000000001000 | Valor (R$ 10,00) |
| 11 | STAN | N 6 | 123456 | Trace number |
| 12 | Time | N 6 | 160945 | Hora da transação |
| 13 | Date | N 4 | 1104 | Data (MMDD) |
| 22 | POS Entry Mode | N 3 | 051 | Como cartão foi lido |
| 37 | RRN | AN 12 | 123456789012 | Retrieval Reference |
| 39 | Response Code | AN 2 | 00 | Código de resposta |
| 41 | Card Acceptor Terminal ID | ANS 8 | TERM0001 | ID do terminal |
| 42 | Card Acceptor ID | ANS 15 | MERCHANT0001 | ID do merchant |
| 49 | Currency Code | N 3 | 986 | BRL (Real brasileiro) |

**Legenda:**
- N = Numeric
- AN = Alphanumeric
- ANS = Alphanumeric + Special
- ..19 = Variável até 19 dígitos

### Processing Code (Campo 3)

```
  T T A A F F
  │ │ │ │ │ └─── From Account
  │ │ │ └─┘───── Reserved
  │ └─┘─────────  To Account
  └───────────── Transaction Type
```

**Exemplos:**
- `000000` - Purchase (compra)
- `010000` - Cash withdrawal (saque)
- `200000` - Refund (reembolso)

### Response Code (Campo 39)

| Código | Significado |
|--------|-------------|
| 00 | Approved |
| 05 | Do not honor (negado) |
| 14 | Invalid card number |
| 51 | Insufficient funds (sem saldo) |
| 54 | Expired card |
| 55 | Incorrect PIN |
| 91 | Issuer not available |
| 96 | System malfunction |

---

##  Fluxos de Transação (20 minutos)

### Fluxo 1: Autorização (0100/0110)

**Cenário:** Cliente passa cartão na maquininha

```
┌─────────┐         ┌──────────┐         ┌─────────┐
│   POS   │         │ Acquire  │         │ Issuer  │
└────┬────┘         └────┬─────┘         └────┬────┘
     │                   │                     │
     │ 1. Swipe Card     │                     │
     │ (cliente passa)   │                     │
     │                   │                     │
     │ 2. Send 0100      │                     │
     │─────────────────▶ │                     │
     │                   │                     │
     │                   │ 3. Route & Send     │
     │                   │────────────────────▶│
     │                   │                     │
     │                   │      4. Check:      │
     │                   │      - Card valid?  │
     │                   │      - Has balance? │
     │                   │      - Not blocked? │
     │                   │                     │
     │                   │ 5. Send 0110        │
     │                   │◀────────────────────│
     │                   │    (approved/denied)│
     │ 6. Receive 0110   │                     │
     │◀──────────────────│                     │
     │                   │                     │
     │ 7. Print Receipt  │                     │
     │                   │                     │
```

**Mensagem 0100 (Request):**
```json
{
  "0": "0100",
  "2": "4111111111111111",
  "3": "000000",
  "4": "000000001000",
  "11": "123456",
  "12": "160945",
  "13": "1104",
  "22": "051",
  "41": "TERM0001",
  "42": "MERCHANT001",
  "49": "986"
}
```

**Mensagem 0110 (Response):**
```json
{
  "0": "0110",
  "2": "4111111111111111",
  "3": "000000",
  "4": "000000001000",
  "11": "123456",
  "37": "123456789012",
  "39": "00",  // Approved!
  "41": "TERM0001",
  "42": "MERCHANT001"
}
```

### Fluxo 2: Transação Financeira (0200/0210)

**Diferença:** 0200 já debita/credita, 0100 só autoriza

```
0100/0110: "Posso cobrar?"
0200/0210: "Estou cobrando agora!"
```

### Fluxo 3: Reversal/Estorno (0400/0410)

**Cenário:** Cancelar uma transação

```json
{
  "0": "0400",
  "2": "4111111111111111",
  "3": "000000",
  "4": "000000001000",
  "11": "123457",  // Novo STAN
  "37": "123456789012",  // RRN original
  "39": "00"  // Response da transação original
}
```

---

##  Implementação Prática

### Exemplo 1: Parser Básico (Conceitual)

```typescript
interface ISO8583Message {
  mti: string;
  bitmap: string;
  fields: Record<number, string>;
}

function parseMessage(raw: string): ISO8583Message {
  // 1. Extrair MTI (primeiros 4 bytes)
  const mti = raw.substring(0, 4);
  
  // 2. Extrair bitmap (próximos 16 hex = 64 bits)
  const bitmapHex = raw.substring(4, 20);
  const bitmap = hexToBinary(bitmapHex);
  
  // 3. Para cada bit = 1, ler o campo correspondente
  const fields: Record<number, string> = {};
  let position = 20;
  
  for (let i = 0; i < 64; i++) {
    if (bitmap[i] === '1') {
      const fieldNumber = i + 1;
      const fieldLength = getFieldLength(fieldNumber);
      fields[fieldNumber] = raw.substring(position, position + fieldLength);
      position += fieldLength;
    }
  }
  
  return { mti, bitmap, fields };
}
```

### Exemplo 2: Validação de Cartão (Luhn)

```typescript
function isValidCard(cardNumber: string): boolean {
  // Remove espaços
  const digits = cardNumber.replace(/\s/g, '');
  
  // Deve ter 13-19 dígitos
  if (digits.length < 13 || digits.length > 19) return false;
  
  // Algoritmo de Luhn
  let sum = 0;
  let isEven = false;
  
  for (let i = digits.length - 1; i >= 0; i--) {
    let digit = parseInt(digits[i]);
    
    if (isEven) {
      digit *= 2;
      if (digit > 9) digit -= 9;
    }
    
    sum += digit;
    isEven = !isEven;
  }
  
  return sum % 10 === 0;
}

// Teste
console.log(isValidCard('4111111111111111')); // true (Visa test)
```

### Exemplo 3: Builder de Mensagem

```typescript
function buildAuthRequest(data: {
  cardNumber: string;
  amount: number;
  merchantId: string;
}): string {
  const message = {
    0: '0100',
    2: data.cardNumber,
    3: '000000',
    4: data.amount.toString().padStart(12, '0'),
    11: generateSTAN(),
    12: getCurrentTime(),
    13: getCurrentDate(),
    41: 'TERM0001',
    42: data.merchantId,
    49: '986'
  };
  
  return encodeISO8583(message);
}
```

---

##  Teste Seu Conhecimento

### Quiz Rápido

1. O que significa MTI 0200?
2. Para que serve o campo 39?
3. Qual a diferença entre 0100 e 0200?
4. O que o bitmap indica?
5. Como validar um número de cartão?

<details>
<summary>Respostas</summary>

1. Financial Transaction Request (transação financeira)
2. Response Code (código de aprovação/negação)
3. 0100 é autorização, 0200 é transação financeira (já debita)
4. Quais campos estão presentes na mensagem
5. Algoritmo de Luhn + verificar comprimento (13-19 dígitos)

</details>

---

##  Links Essenciais

###  Leitura Obrigatória
1. [Wikipedia - ISO 8583](https://en.wikipedia.org/wiki/ISO_8583)
2. [ISO8583.info - Field Guide](https://www.iso8583.info/)
3. [CodeProject Tutorial](https://www.codeproject.com/Articles/100084/Introduction-to-ISO-8583)

###  Ferramentas
1. [Neapay Simulator](https://neapay.com/free-payments-iso8583-simulator-tools.html)
2. [Card Number Generator](https://www.creditcardvalidator.org/generator)
3. [BIN Lookup](https://binlist.net/)

###  Libraries
1. [iso-8583 (Node.js)](https://www.npmjs.com/package/iso-8583)
2. [jPOS (Java)](https://jpos.org/)
3. [ISO8583-Simulator (Python)](https://github.com/bassrehab/ISO8583-Simulator)

---

##  Checklist de Estudo

- [ ] Entendi o que é ISO 8583
- [ ] Sei o que é MTI e como decodificar
- [ ] Entendo como funciona o bitmap
- [ ] Conheço os 10 campos principais
- [ ] Sei a diferença entre acquire e issuing
- [ ] Entendo o fluxo de autorização (0100/0110)
- [ ] Sei validar um cartão (Luhn)
- [ ] Testei um simulador online
- [ ] Li pelo menos 2 links de referência

---

##  Próximos Passos

Depois de estudar:
1.  Volte ao QUICK-START.md
2.  Comece a implementar
3.  Teste com mensagens simples primeiro
4.  Incremente aos poucos

---

**Tempo de estudo recomendado:** 1-2 horas

Depois disso você estará pronto para começar a implementação! 
