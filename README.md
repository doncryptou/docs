# 🏛️ API ESPECÍFICA PARA O CLIENT - Criação de Contas Verificadas

## 📋 Resumo

Esta API foi criada especificamente para que o **CLIENT** possa criar contas PIX de forma **automática**, **validada** e **já aprovada**.

**Principais benefícios:**
- ✅ **Validação CPFCNPJ obrigatória** automática na Receita Federal
- ✅ **Contas criadas como ATIVAS** automaticamente
- ✅ **Sem necessidade de aprovação manual**
- ✅ **Bloqueio automático** de empresas com status irregular

---

## 🔑 Autenticação

### Token de Autorização
Toda requisição deve incluir o header de autorização:

```http
Authorization: Bearer SEU_TOKEN_AQUI
```

**⚠️ IMPORTANTE:** Solicite o token ao administrador do sistema.

---

## 🌐 Endpoint

### **POST** `/api/owempay/client/accounts`

**URL Completa:** `https://admin-api.owempay.com.br/api/owempay/client/accounts`

---

## 📦 Formato da Requisição

### Headers Obrigatórios
```http
Content-Type: application/json
Authorization: Bearer SEU_TOKEN_AQUI
```

### Body (JSON)
```json
{
  "operationExternalId": "OPERATION_EXTERNAL_ID_AQUI", // External ID da operação ativa (encontra em Detalhes da operação no painel)
  "name": "OWEM PAY INSTITUICAO DE PAGAMENTO LTDA", // Nome da pessoa física ou razão social da empresa
  "fantasyName": "Owempay", // (opcional para PJ)
  "cpfCnpj": "37839059000188", // CPF ou CNPJ **SEM máscara**
  "pixKeyType": 1, // 0=CPF, 1=CNPJ, 2=Email, 3=Telefone, 4=Aleatória
  "pixKey": "37839059000188", // Valor da chave PIX (para tipo aleatório, não é necessário pois a api gera uma chave aleatória automaticamente)
  "agency": "200", // Agência bancária sempre 200
  "accountNumber": "10002151400", // Começa com 100 e depois 8 dígitos
  "accountType": 0, // 0 = corrente, 1 = poupança
  "personType": 1 // 0 = Pessoa Física, 1 = Pessoa Jurídica
}
```

### Campos Obrigatórios
| Campo | Tipo | Descrição |
|-------|------|-----------|
| `operationExternalId` | string | External ID da operação (deve existir no sistema) |
| `name` | string | Nome da pessoa física ou razão social da empresa |
| `cpfCnpj` | string | CPF (11 dígitos) ou CNPJ (14 dígitos) **SEM máscara** |

### Campos Opcionais
| Campo | Tipo | Padrão | Descrição |
|-------|------|--------|-----------|
| `fantasyName` | string | - | Nome fantasia (preenchido automaticamente para CNPJ) |
| `pixKeyType` | number | 1 | Tipo da chave PIX (0=CPF, 1=CNPJ, 2=Email, 3=Telefone, 4=Aleatória) |
| `pixKey` | string | auto-gerado | Valor da chave PIX (para tipo aleatória, não é necessário pois a api gera uma chave aleatória automaticamente) |
| `agency` | string | "200" | Agência bancária |
| `accountNumber` | string | auto-gerado | Número da conta (Começa com 100 e depois 8 dígitos) |
| `accountType` | number | 0 | Tipo da conta (0 = corrente, 1 = poupança) |
| `personType` | number | 1 | Tipo de pessoa (0 = Pessoa Física, 1 = Pessoa Jurídica) |

### Tipos de Chave PIX
| Código | Tipo | Descrição |
|--------|------|-----------|
| 0 | CPF | Chave é o CPF do titular ex: 11144477735 |
| 1 | CNPJ | Chave é o CNPJ da empresa ex: 37839059000188 |
| 2 | Email | Chave é um email ex: teste@teste.com |
| 3 | Telefone | Chave é um telefone ex: 5511999999999 |
| 4 | Aleatória | Chave UUID gerada automaticamente |

---

## 🎯 Exemplo de Requisição Completa

### Para Pessoa Física (CPF)
```bash
curl -X POST "https://admin-api.owempay.com.br/api/owempay/client/accounts" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer SEU_TOKEN_AQUI" \
  -d '{
    "operationExternalId": "SLNA8N16SE71",
    "name": "Jean Carlos Monteiro",
    "cpfCnpj": "11144477735",
    "pixKeyType": 0,
    "pixKey": "11144477735",
    "agency": "200",
    "accountNumber": "10002151400",
    "accountType": 0,
    "personType": 0
}'
```

### Para Pessoa Jurídica (CNPJ)
```bash
curl -X POST "https://admin-api.owempay.com.br/api/owempay/client/accounts" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer SEU_TOKEN_AQUI" \
  -d '{
    "operationExternalId": "SLNA8N16SE71",
    "name": "Empresa Exemplo LTDA",
    "cpfCnpj": "12345678000199",
    "fantasyName": "Exemplo Cor",
    "pixKeyType": 1,
    "pixKey": "12345678000199",
    "agency": "200",
    "accountNumber": "10005586442",
    "accountType": 0,
    "personType": 1
}'
```

---

## ✅ Resposta de Sucesso

**Status Code:** `201 Created`

```json
{
  "success": true,
  "message": "Conta criada, validada e ativada com sucesso",
  "account": {
    "id": 123,
    "name": "Jean Carlos Monteiro",
    "cpfCnpj": "11144477735",
    "pixKey": "550e8400-e29b-41d4-a716-446655440000",
    "pixKeyType": 0,
    "agency": "200",
    "accountNumber": "10002151400",
    "accountType": 0,
    "personType": 1,
    "active": true,
    "status": "active",
    "operationId": 456,
    "createdAt": "2024-01-15T10:30:00.000Z"
  },
  "operation": {
    "id": 456,
    "name": "Owempay",
    "externalId": "SLNA8N16SE71"
  },
  "validation": {
    "cpfCnpjValidated": true,
    "documentType": "CNPJ",
    "validatedAt": "2024-01-15T10:30:00.000Z"
  }
}
```

---

## ❌ Códigos de Erro

### `401 Unauthorized`
```json
{
  "error": "Token de autorização necessário",
  "message": "Header Authorization com Bearer token é obrigatório"
}
```

### `403 Forbidden`
```json
{
  "error": "Token inválido",
  "message": "Token de autorização não é válido"
}
```

### `404 Not Found`
```json
{
  "error": "Operação não encontrada",
  "message": "Operação com external_id 'SLNA8N16SE71' não foi encontrada"
}
```

### `400 Bad Request` - CPF Inválido
```json
{
  "error": "CPF_INVALIDO",
  "message": "CPF não encontrado ou inativo na Receita Federal"
}
```

### `400 Bad Request` - CNPJ Inválido
```json
{
  "error": "CNPJ_INVALIDO",
  "message": "CNPJ não encontrado ou com status irregular na Receita Federal"
}
```

### `400 Bad Request` - Empresa com Status Irregular
```json
{
  "error": "EMPRESA_STATUS_IRREGULAR",
  "message": "Empresa com situação 'BAIXADA' não pode ter conta PIX criada. Apenas empresas ATIVAS são permitidas.",
  "status": "BAIXADA"
}
```

### `400 Bad Request` - Dados Inválidos
```json
{
  "error": "Dados inválidos",
  "details": [
    {
      "code": "too_small",
      "minimum": 1,
      "type": "string",
      "inclusive": true,
      "exact": false,
      "message": "Nome é obrigatório",
      "path": ["name"]
    }
  ]
}
```

---

## 🔄 Fluxo Automático da API

1. **Recebimento da Requisição**
   - Validação do token de autorização
   - Validação dos dados de entrada

2. **Verificação da Operação**
   - Busca operação pelo `operationExternalId`
   - Verifica se operação está ativa

3. **Validação CPFCNPJ Obrigatória**
   - **CPF:** Consulta na Receita Federal + auto-preenchimento do nome
   - **CNPJ:** Consulta na Receita Federal + verificação de status + auto-preenchimento

4. **Bloqueio de Irregulares**
   - Empresas com status irregular são automaticamente rejeitadas
   - Status bloqueados: INATIVO, BAIXADO, SUSPENSO, CANCELADO, etc.

5. **Criação da Conta**
   - Chama API para criar chave PIX
   - Salva conta no banco como **ATIVA**
   - Retorna dados completos da conta criada

---

## 🔧 Configuração de Ambiente

### Variáveis de Ambiente Necessárias

```env
# Token de autenticação do CLIENT
CLIENT_SYSTEM_TOKEN=Bearer SEU_TOKEN_AQUI

# URL da API interna para validação CPF-CNPJ
ADMIN_API_URL=https://admin-api.owempay.com.br
```

---

## 📝 Notas Importantes

### ✅ **Vantagens desta API:**
1. **Sem aprovação manual:** Contas são criadas já ativas
2. **Validação garantida:** Todo CPF/CNPJ é verificado na Receita Federal
3. **Segurança:** Empresas irregulares são automaticamente bloqueadas
4. **Auto-preenchimento:** Dados oficiais são preenchidos automaticamente
5. **Logs completos:** Todas as operações são registradas

### ⚠️ **Pontos de Atenção:**
1. **Token obrigatório:** Sem token válido, a requisição é rejeitada
2. **Operação deve existir:** O `operationExternalId` deve estar cadastrado no sistema
3. **CPF/CNPJ obrigatório:** Não é possível criar conta sem validar o documento
4. **Limite de requisições:** Use de forma responsável para evitar bloqueios

### 🚫 **Casos que serão Rejeitados:**
- CPF inexistente ou irregular na Receita Federal
- CNPJ inexistente ou com status irregular
- Operação inexistente ou inativa
- Token inválido ou ausente
- Dados obrigatórios em branco

---

## 🆘 Suporte

Em caso de dúvidas ou problemas:

1. **Verifique os logs** no sistema para detalhes do erro
2. **Confirme as variáveis de ambiente** estão configuradas
3. **Teste com dados válidos** antes de usar em produção
4. **Entre em contato** com o administrador do sistema

---

## 📊 Exemplo de Integração Completa

```javascript
// Exemplo em JavaScript/Node.js
async function criarContaPIX(dadosConta) {
  try {
    const response = await fetch('https://admin-api.owempay.com.br/api/owempay/client/accounts', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer SEU_TOKEN_AQUI'
      },
      body: JSON.stringify({
        operationExternalId: dadosConta.operationId, // External ID da operação ativa (encontra em Detalhes da operação no painel)
        name: dadosConta.nome, // Nome da pessoa física ou razão social da empresa
        cpfCnpj: dadosConta.documento, // CPF ou CNPJ **SEM máscara**
        pixKeyType: 1, // 0=CPF, 1=CNPJ, 2=Email, 3=Telefone, 4=Aleatória
        pixKey: '37839059000188', // Valor da chave PIX (para tipo aleatório, não é necessário pois a api gera uma chave aleatória automaticamente)
        agency: '200', // Agência bancária sempre 200
        accountNumber: '10002151400', // Começa com 100 e depois 8 dígitos
        accountType: 0, // 0 = corrente, 1 = poupança
        personType: 1 // 0 = Pessoa Física, 1 = Pessoa Jurídica
      })
    });

    if (!response.ok) {
      const erro = await response.json();
      throw new Error(`${erro.error}: ${erro.message}`);
    }

    const resultado = await response.json();

    console.log('✅ Conta PIX criada com sucesso!');
    console.log('ID da conta:', resultado.account.id);
    console.log('Chave PIX:', resultado.account.pixKey);
    console.log('Status:', resultado.account.status);

    return resultado;

  } catch (error) {
    console.error('❌ Erro ao criar conta PIX:', error.message);
    throw error;
  }
}

// Uso da função
criarContaPIX({
  operationId: 'SLNA8N16SE71', // External ID da operação ativa (encontra em Detalhes da operação no painel)
  nome: 'Jean Carlos Monteiro', // Nome da pessoa física ou razão social da empresa
  documento: '11144477735', // CPF ou CNPJ **SEM máscara**
  pixKeyType: 0, // 0=CPF, 1=CNPJ, 2=Email, 3=Telefone, 4=Aleatória
  pixKey: '11144477735', // Valor da chave PIX (para tipo aleatório, não é necessário pois a api gera uma chave aleatória automaticamente)
  agency: '200', // Agência bancária sempre 200
  accountNumber: '10002151400', // Começa com 100 e depois 8 dígitos
  accountType: 0, // 0 = corrente, 1 = poupança
  personType: 0 // 0 = Pessoa Física, 1 = Pessoa Jurídica
});
```

---

## 🎉 Pronto para Usar!

A API está **100% funcional** e pronta para receber requisições do CLIENT. Todas as validações, verificações e criações são feitas automaticamente.

**Resultado final:** Contas PIX **válidas**, **verificadas** e **ativas** criadas automaticamente! 🚀
