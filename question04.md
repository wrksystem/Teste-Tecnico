# Questão 4 - Modelo de Banco de Dados para Cadastro de Clientes

## Estrutura das Tabelas

### Tabela: ESTADOS
| Campo | Tipo | Descrição |
|-------|------|-----------|
| **id_estado (PK)** | INT | Identificador único do estado |
| sigla | VARCHAR(2) | Sigla do estado (ex: SP, RJ, MG) |
| nome | VARCHAR(50) | Nome completo do estado |

---

### Tabela: CLIENTES
| Campo | Tipo | Descrição |
|-------|------|-----------|
| **id_cliente (PK)** | INT | Identificador único do cliente |
| razao_social | VARCHAR(200) | Razão social do cliente |
| nome_fantasia | VARCHAR(200) | Nome fantasia |
| cnpj_cpf | VARCHAR(18) | CNPJ ou CPF |
| email | VARCHAR(100) | E-mail do cliente |
| **id_estado (FK)** | INT | Referência ao estado |

---

### Tabela: TIPOS_TELEFONE
| Campo | Tipo | Descrição |
|-------|------|-----------|
| **id_tipo_telefone (PK)** | INT | Identificador único do tipo |
| descricao | VARCHAR(50) | Descrição (Comercial, Residencial, Celular, etc) |

---

### Tabela: TELEFONES
| Campo | Tipo | Descrição |
|-------|------|-----------|
| **id_telefone (PK)** | INT | Identificador único do telefone |
| **id_cliente (FK)** | INT | Referência ao cliente |
| **id_tipo_telefone (FK)** | INT | Referência ao tipo de telefone |
| ddd | VARCHAR(2) | DDD |
| numero | VARCHAR(10) | Número do telefone |

---

## Diagrama de Relacionamentos (Modelo Lógico)

```
┌─────────────────┐
│    ESTADOS      │
│─────────────────│
│ id_estado (PK)  │
│ sigla           │
│ nome            │
└─────────────────┘
         ↑
         │ 1
         │
         │ N
┌─────────────────┐
│    CLIENTES     │
│─────────────────│
│ id_cliente (PK) │
│ razao_social    │
│ nome_fantasia   │
│ cnpj_cpf        │
│ email           │
│ id_estado (FK)  │
└─────────────────┘
         │
         │ 1
         │
         │ N
         ↓
┌─────────────────┐          ┌──────────────────────┐
│   TELEFONES     │    N:1   │  TIPOS_TELEFONE      │
│─────────────────│─────────→│──────────────────────│
│ id_telefone(PK) │          │ id_tipo_telefone(PK) │
│ id_cliente (FK) │          │ descricao            │
│id_tipo_tel (FK) │          └──────────────────────┘
│ ddd             │
│ numero          │
└─────────────────┘
```

**Relacionamentos:**
- **ESTADOS (1) → (N) CLIENTES**: Um estado pode ter vários clientes
- **CLIENTES (1) → (N) TELEFONES**: Um cliente pode ter vários telefones
- **TIPOS_TELEFONE (1) → (N) TELEFONES**: Um tipo pode ser usado em vários telefones

---

## Consulta SQL - Clientes de São Paulo com seus Telefones

```sql
SELECT 
    c.id_cliente AS codigo,
    c.razao_social,
    t.ddd,
    t.numero AS telefone,
    tt.descricao AS tipo_telefone
FROM 
    CLIENTES c
    INNER JOIN ESTADOS e ON c.id_estado = e.id_estado
    LEFT JOIN TELEFONES t ON c.id_cliente = t.id_cliente
    LEFT JOIN TIPOS_TELEFONE tt ON t.id_tipo_telefone = tt.id_tipo_telefone
WHERE 
    e.sigla = 'SP'
ORDER BY 
    c.razao_social, 
    t.id_telefone;
```

### Explicação da Query:

- **SELECT**: Retorna código do cliente, razão social e telefones
- **INNER JOIN com ESTADOS**: Garante que o cliente tenha um estado vinculado
- **LEFT JOIN com TELEFONES**: Inclui clientes mesmo sem telefone cadastrado
- **LEFT JOIN com TIPOS_TELEFONE**: Traz a descrição do tipo de telefone
- **WHERE**: Filtra apenas clientes do estado de São Paulo (SP)
- **ORDER BY**: Ordena por razão social e depois por telefone

---

## Script SQL Completo (Criação das Tabelas)

```sql
-- Criar tabela de Estados
CREATE TABLE ESTADOS (
    id_estado INT PRIMARY KEY AUTO_INCREMENT,
    sigla VARCHAR(2) NOT NULL UNIQUE,
    nome VARCHAR(50) NOT NULL
);

-- Criar tabela de Clientes
CREATE TABLE CLIENTES (
    id_cliente INT PRIMARY KEY AUTO_INCREMENT,
    razao_social VARCHAR(200) NOT NULL,
    nome_fantasia VARCHAR(200),
    cnpj_cpf VARCHAR(18) NOT NULL UNIQUE,
    email VARCHAR(100),
    id_estado INT NOT NULL,
    FOREIGN KEY (id_estado) REFERENCES ESTADOS(id_estado)
);

-- Criar tabela de Tipos de Telefone
CREATE TABLE TIPOS_TELEFONE (
    id_tipo_telefone INT PRIMARY KEY AUTO_INCREMENT,
    descricao VARCHAR(50) NOT NULL UNIQUE
);

-- Criar tabela de Telefones
CREATE TABLE TELEFONES (
    id_telefone INT PRIMARY KEY AUTO_INCREMENT,
    id_cliente INT NOT NULL,
    id_tipo_telefone INT NOT NULL,
    ddd VARCHAR(2) NOT NULL,
    numero VARCHAR(10) NOT NULL,
    FOREIGN KEY (id_cliente) REFERENCES CLIENTES(id_cliente) ON DELETE CASCADE,
    FOREIGN KEY (id_tipo_telefone) REFERENCES TIPOS_TELEFONE(id_tipo_telefone)
);

-- Índices para melhor performance
CREATE INDEX idx_cliente_estado ON CLIENTES(id_estado);
CREATE INDEX idx_telefone_cliente ON TELEFONES(id_cliente);
```

---

## Dados de Exemplo para Teste

```sql
-- Inserir Estados
INSERT INTO ESTADOS (sigla, nome) VALUES 
('SP', 'São Paulo'),
('RJ', 'Rio de Janeiro'),
('MG', 'Minas Gerais');

-- Inserir Tipos de Telefone
INSERT INTO TIPOS_TELEFONE (descricao) VALUES 
('Comercial'),
('Residencial'),
('Celular'),
('WhatsApp');

-- Inserir Clientes
INSERT INTO CLIENTES (razao_social, nome_fantasia, cnpj_cpf, email, id_estado) VALUES 
('Empresa XYZ Ltda', 'XYZ Comércio', '12.345.678/0001-90', 'contato@xyz.com', 1),
('João Silva ME', 'Silva & Cia', '98.765.432/0001-10', 'joao@silva.com', 1);

-- Inserir Telefones
INSERT INTO TELEFONES (id_cliente, id_tipo_telefone, ddd, numero) VALUES 
(1, 1, '11', '3333-4444'),
(1, 3, '11', '98888-7777'),
(2, 2, '11', '2222-3333');
```

---

## Justificativa do Modelo

### Atendimento aos Requisitos:

✅ **Cliente com número ilimitado de telefones**: A tabela TELEFONES tem relação N:1 com CLIENTES, permitindo múltiplos telefones por cliente.

✅ **Tipos de telefone cadastráveis**: A tabela TIPOS_TELEFONE é independente e permite inserção de novos tipos sem alterar a estrutura.

✅ **Estados brasileiros cadastráveis**: A tabela ESTADOS permite adicionar novos estados facilmente.

### Características Técnicas:

- **Normalização**: Modelo em 3ª Forma Normal, evitando redundância
- **Integridade Referencial**: Uso de Foreign Keys garante consistência dos dados
- **Performance**: Índices nos campos de relacionamento para consultas rápidas
- **Escalabilidade**: Estrutura preparada para crescimento do sistema
- **Manutenibilidade**: Código limpo e bem documentado
