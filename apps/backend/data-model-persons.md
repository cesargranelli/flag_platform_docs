# Modelo de Dados — Persons, Athletes, Participants

> **Projeto:** Flag Platform Backend  
> **Data:** 2026-09-16  
> **Status:** Implementado (V22-V26, com correção de schema em V26)

---

## 1. Visão Geral

O modelo de dados atual cobre três conceitos-chave:

| Conceito | Descrição | Tabela Principal |
|----------|-----------|------------------|
| **Person** | Pessoa física (qualquer papel: atleta, técnico, árbitro, delegado) | `persons` |
| **Athlete** | Pessoa com vínculo a um time/competição (jogador ou comissão técnica) | `team_roster` |
| **Participant** | Pessoa com participação em um jogo (árbitro, delegado, comissário) | `game_participants` |

---

## 2. Diagrama de Entidades

```
┌─────────────────────────────┐
│          persons             │
│─────────────────────────────│
│ id (UUID PK)                │
│ name (VARCHAR 150)          │
│ cpf (VARCHAR 14) UNIQUE     │
│ photo_url (VARCHAR 500)     │
│ status (ACTIVE/INACTIVE)    │
│ birth_date (DATE)           │
│ gender (MALE/FEMALE/MIXED)  │
│ role (PersonRole enum)      │
│ created_at / updated_at     │
└──────────┬──────────────────┘
           │
           │ person.id
           │
     ┌─────┴──────────────────────────────────┐
     │                                        │
     ▼                                        ▼
┌─────────────────────────┐     ┌──────────────────────────┐
│      team_roster        │     │   game_participants      │
│─────────────────────────│     │──────────────────────────│
│ id (PK)                 │     │ id (PK)                  │
│ roster_id (FK→roster)   │     │ game_id (FK→games)       │
│ athlete_id (FK→persons) │     │ person_id (FK→persons)   │
│ nickname (VARCHAR 100)  │     │ role (PersonRole)        │
│ number (INTEGER)        │     │ function (VARCHAR 100)   │
│ positions (VARCHAR 255) │     │ created_at               │
│ status (ACTIVE/INACTIVE)│     └──────────────────────────┘
└─────────────────────────┘
           │
           ▼
┌─────────────────────────┐
│        roster           │
│─────────────────────────│
│ id (PK)                 │
│ team_id (FK→teams)      │
│ competition_id (FK,     │
│   nullable = elenco-base│
│ season                  │
│ status                  │
└─────────────────────────┘
```

---

## 3. Tabela `persons`

Universal — qualquer pessoa física no sistema.

| Coluna | Tipo | Nullable | Descrição |
|--------|------|----------|-----------|
| `id` | UUID | NOT NULL | PK |
| `name` | VARCHAR(150) | NOT NULL | Nome completo |
| `cpf` | VARCHAR(14) | NOT NULL | CPF (único) |
| `photo_url` | VARCHAR(500) | SIM | URL da foto |
| `status` | VARCHAR(20) | NOT NULL | `ACTIVE` ou `INACTIVE` |
| `birth_date` | DATE | SIM | Data de nascimento |
| `gender` | VARCHAR(20) | SIM | `MALE`, `FEMALE` ou `MIXED` |
| `role` | VARCHAR(30) | NOT NULL | Papel da pessoa (ver PersonRole) |
| `created_at` | TIMESTAMP | NOT NULL | Auto |
| `updated_at` | TIMESTAMP | SIM | Auto |

### PersonRole enum

| Valor | Label | Uso |
|-------|-------|-----|
| `ATHLETE` | Atleta | Jogadores |
| `COACH` | Técnico | Treinadores |
| `TECHNICAL_STAFF` | Comissão Técnica | Staff auxiliar |
| `REFEREE` | Árbitro | Árbitros de partida |
| `DELEGATE` | Delegado | Delegados de federação |
| `COMMISSIONER` | Comissário | Comissários delegados |

---

## 4. Tabela `team_roster`

Vincula uma pessoa a um time em uma competição. Dados **por time/competição**.

| Coluna | Tipo | Nullable | Descrição |
|--------|------|----------|-----------|
| `id` | UUID | NOT NULL | PK |
| `roster_id` | UUID | NOT NULL | FK → `roster` |
| `athlete_id` | UUID | NOT NULL | FK → `persons` |
| `status` | VARCHAR(20) | NOT NULL | `ACTIVE` ou `INACTIVE` |
| `nickname` | VARCHAR(100) | SIM | Apelido no time |
| `number` | INTEGER | SIM | Número da camisa |
| `positions` | VARCHAR(255) | SIM | Posições (comma-separated: `"QB,WR"`) |

### Posições válidas (AthletePosition enum)

| Código | Nome |
|--------|------|
| `QB` | Quarterback |
| `RB` | Running Back |
| `WR` | Wide Receiver |
| `TE` | Tight End |
| `C` | Center |
| `DL` | Defensive Lineman |
| `LB` | Linebacker |
| `DB` | Defensive Back |
| `K` | Kicker |
| `P` | Punter |

---

## 5. Tabela `game_participants`

Participação de não-atletas em jogos (árbitros, delegados, etc.).

| Coluna | Tipo | Nullable | Descrição |
|--------|------|----------|-----------|
| `id` | UUID | NOT NULL | PK |
| `game_id` | UUID | NOT NULL | FK → `games` |
| `person_id` | UUID | NOT NULL | FK → `persons` |
| `role` | VARCHAR(30) | NOT NULL | Papel (PersonRole) |
| `function` | VARCHAR(100) | SIM | Função específica |

### Funções de jogo (GameParticipantFunction)

| Código | Label |
|--------|-------|
| `MAIN_REFEREE` | Árbitro Principal |
| `ASSISTANT_REFEREE` | Árbitro Auxiliar |
| `DELEGATE` | Delegado |
| `COMMISSIONER` | Comissário |
| `SCORER` | Marcador |
| `TIMEKEEPER` | Cronometrista |

---

## 6. Tabela `plays`

Lances de um jogo. Referencia jogador por **texto livre** (player_name) com vinculo opcional ao cadastro.

| Coluna | Tipo | Nullable | Descrição |
|--------|------|----------|-----------|
| `id` | UUID | NOT NULL | PK |
| `game_id` | UUID | NOT NULL | FK → `games` |
| `team_id` | UUID | NOT NULL | FK → teams (time que fez o lance) |
| `player_id` | UUID | SIM | FK → `persons` (vinculo opcional) |
| `player_name` | VARCHAR(100) | NOT NULL | Nome do jogador (texto) |
| `receiver_name` | VARCHAR(100) | SIM | Nome do receptor |
| `play_type` | VARCHAR(30) | NOT NULL | Tipo do lance |
| `description` | TEXT | SIM | Descrição |
| `yards` | INTEGER | SIM | Jardas |
| `quarter` | VARCHAR(5) | SIM | Quarter (ex: "Q1") |
| `time` | VARCHAR(10) | SIM | Tempo |
| `points_to_add` | INTEGER | SIM | Pontos |
| `is_first_down` | BOOLEAN | SIM | First down |
| `is_touchdown` | BOOLEAN | SIM | Touchdown |
| `is_turnover` | BOOLEAN | SIM | Turnover |

---

## 7. Tabela `checkins`

Check-in de atletas em jogos.

| Coluna | Tipo | Nullable | Descrição |
|--------|------|----------|-----------|
| `id` | UUID | NOT NULL | PK |
| `game_id` | UUID | NOT NULL | FK → `games` |
| `team_id` | UUID | NOT NULL | FK → teams |
| `athlete_id` | UUID | NOT NULL | FK → `persons` |
| `status` | VARCHAR(20) | NOT NULL | `PRESENT`, `NO_SHOW`, `NOT_REGISTERED` |
| `match_number` | INTEGER | SIM | Número override para esta partida |
| `validated_by` | UUID | SIM | Quem validou |
| `validated_at` | TIMESTAMP | SIM | Quando validou |

---

## 8. Resumo: Onde ficam os dados

| Dado | Onde fica | Observação |
|------|-----------|------------|
| Nome, CPF, foto, nascimento, gênero | `persons` | Dados pessoais universais |
| Papel (atleta, técnico, árbitro) | `persons.role` | Um pessoa = um papel |
| Apelido no time | `team_roster.nickname` | Por time/competição |
| Número da camisa | `team_roster.number` | Por time/competição |
| Posições | `team_roster.positions` | Comma-separated, por time |
| Participação em jogo (não-atleta) | `game_participants` | Árbitros, delegados |
| Check-in | `checkins` | Presença no jogo |
| Lance no jogo | `plays` | player_name + player_id |
