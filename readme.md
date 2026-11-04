# 🐾 PetSched — Sistema de Agendamento para Clínicas Veterinárias

O **PetSched** é uma solução web desenvolvida para otimizar o fluxo de agendamentos em clínicas veterinárias e pet shops. A aplicação permite a gestão de tutores, pets, profissionais veterinários e serviços, garantindo a integridade dos horários e evitando sobreposição de consultas (*overbooking*).

---

## 📌 Sumário
1. [Requisitos do Sistema](#-requisitos-do-sistema)
2. [Regras de Negócio](#-regras-de-negócio)
3. [Diagrama Entidade-Relacionamento (ER)](#-diagrama-entidade-relacionamento-er)
4. [Script do Banco de Dados (SQL)](#-script-do-banco-de-dados-sql)
5. [Como Executar o Projeto](#-como-executar-o-projeto)

---

## 📋 Requisitos do Sistema

### 🔹 Requisitos Funcionais (RF)
- **RF-001 [Cadastro de Tutores]:** O sistema deve permitir o cadastro, edição e listagem de tutores (clientes).
- **RF-002 [Cadastro de Pets]:** Cada tutor pode cadastrar um ou mais pets vinculados ao seu perfil.
- **RF-003 [Gestão de Serviços]:** O sistema deve permitir cadastrar serviços oferecidos (ex: Consulta Geral, Vacinação, Banho e Tosa), definindo tempo de duração e preço.
- **RF-004 [Gestão de Veterinários]:** Cadastro de profissionais com seus respectivos registros profissionais (CRMV) e especialidades.
- **RF-005 [Agendamento de Consultas]:** O sistema deve permitir agendar consultas selecionando pet, serviço, veterinário, data e horário.
- **RF-006 [Controle de Status]:** Os agendamentos devem possuir estados observáveis (`Agendado`, `Confirmado`, `Em Atendimento`, `Concluído`, `Cancelado`).

### 🔹 Requisitos Não-Funcionais (RNF)
- **RNF-001 [Desempenho]:** A checagem de disponibilidade de horários deve responder em menos de 500ms.
- **RNF-002 [Integridade de Dados]:** Garantir restrições de chave estrangeira (FK) e unicidade em nível de banco de dados para prevenção de agendamentos duplicados.
- **RNF-003 [Interface Responsiva]:** A interface web deve adaptar-se a dispositivos móveis, tablets e desktops (Mobile-First).

---

## ⚡ Regras de Negócio

- **RN-001 (Prevenção de Overbooking):** Um veterinário não pode ter dois agendamentos no mesmo horário. A validação deve considerar o horário de início e a duração estimada do serviço.
- **RN-002 (Horário de Funcionamento):** Agendamentos só podem ser realizados entre **08:00 e 18:00**, de segunda-feira a sábado.
- **RN-003 (Antecedência Mínima para Cancelamento):** O cancelamento de um agendamento pelo cliente/tutor só é permitido com pelo menos **2 horas de antecedência** em relação ao horário marcado.
- **RN-004 (Inativação Logica):** Veterinários e Serviços inativos não devem aparecer disponíveis para novos agendamentos, mantendo contudo o histórico de consultas passadas intacto.

---

## 📐 Diagrama Entidade-Relacionamento (ER)

```
[ TUTORES ] 1 ──── N [ PETS ] 1 ──── N [ AGENDAMENTOS ] N ──── 1 [ SERVIÇOS ]
                                                │
                                                N ──── 1 [ VETERINÁRIOS ]
```

### Relacionamentos:
- **TUTORES → PETS:** Um tutor pode ter 1 ou N pets cadastrados (1:N).
- **PETS → AGENDAMENTOS:** Um pet pode possuir histórico de N agendamentos (1:N).
- **VETERINÁRIOS → AGENDAMENTOS:** Um veterinário atende N agendamentos (1:N).
- **SERVIÇOS → AGENDAMENTOS:** Um serviço pode ser associado a N agendamentos (1:N).

---

## 🗄️ Script do Banco de Dados (SQL)

O script abaixo foi projetado para **MySQL 8.0+** / **MariaDB**.

```sql
CREATE DATABASE IF NOT EXISTS petsched_db 
CHARACTER SET utf8mb4 
COLLATE utf8mb4_unicode_ci;

USE petsched_db;

-- 1. Tabela de Tutores
CREATE TABLE tutores (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    cpf VARCHAR(14) NOT NULL UNIQUE,
    telefone VARCHAR(20) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;

-- 2. Tabela de Pets
CREATE TABLE pets (
    id INT AUTO_INCREMENT PRIMARY KEY,
    tutor_id INT NOT NULL,
    nome VARCHAR(50) NOT NULL,
    especie ENUM('Cachorro', 'Gato', 'Ave', 'Roedor', 'Outro') NOT NULL,
    raca VARCHAR(50),
    data_nascimento DATE,
    observacoes TEXT,
    FOREIGN KEY (tutor_id) REFERENCES tutores(id) ON DELETE CASCADE
) ENGINE=InnoDB;

-- 3. Tabela de Serviços
CREATE TABLE servicos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(80) NOT NULL,
    duracao_minutos INT NOT NULL DEFAULT 30,
    preco DECIMAL(10, 2) NOT NULL,
    ativo TINYINT(1) DEFAULT 1
) ENGINE=InnoDB;

-- 4. Tabela de Veterinários
CREATE TABLE veterinarios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    crmv VARCHAR(20) NOT NULL UNIQUE,
    especialidade VARCHAR(50) DEFAULT 'Clínico Geral',
    ativo TINYINT(1) DEFAULT 1
) ENGINE=InnoDB;

-- 5. Tabela de Agendamentos
CREATE TABLE agendamentos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    pet_id INT NOT NULL,
    servico_id INT NOT NULL,
    veterinario_id INT NOT NULL,
    data_hora DATETIME NOT NULL,
    status ENUM('Agendado', 'Confirmado', 'Em Atendimento', 'Concluído', 'Cancelado') DEFAULT 'Agendado',
    observacoes TEXT,
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (pet_id) REFERENCES pets(id),
    FOREIGN KEY (servico_id) REFERENCES servicos(id),
    FOREIGN KEY (veterinario_id) REFERENCES veterinarios(id),
    UNIQUE KEY uk_vet_horario (veterinario_id, data_hora)
) ENGINE=InnoDB;
```

---

## 🚀 Como Executar o Projeto

1. Clone este repositório:
   ```bash
   git clone https://github.com/seu-usuario/petsched.git
   ```
2. Importe o script SQL no seu gerenciador de banco de dados (MySQL/MariaDB).
3. Configure as credenciais de acesso ao banco no seu arquivo de ambiente (`.env` ou arquivo de configuração).
4. Inicie o servidor local PHP/Web e acesse a aplicação pelo navegador.