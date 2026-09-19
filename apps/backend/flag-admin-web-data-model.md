# Flag Admin Web — Mudanças no Modelo de Dados

> **Projeto:** Flag Admin Web (frontend)  
> **Data:** 2026-09-16  
> **Backend:** Spring Boot 4.1.0  
> **API:** `/api/v1/`

---

## 1. Resumo das Mudanças

| # | Mudança | Impacto no Frontend |
|---|---------|---------------------|
| 1 | Novo endpoint `game_participants` | Tela de confronto: atribuir árbitros/delegados |
| 2 | Funções de jogo (`GameParticipantFunction`) | Dropdown de funções ao atribuir participante |

> **Nota:** Lances (plays), check-in e fluxo de jogo ficam em `flag_referee_app`.

---

## 2. Novo Endpoint: Game Participants

### Listar participantes de um jogo

```
GET /api/v1/games/{gameId}/participants
```

**Response:**
```json
[
  {
    "id": "uuid",
    "gameId": "uuid",
    "personId": "uuid",
    "personName": "João Silva",
    "role": "REFEREE",
    "function": "MAIN_REFEREE",
    "functionLabel": "Árbitro Principal",
    "createdAt": "2026-09-16T10:00:00"
  }
]
```

### Atribuir participante a um jogo

```
POST /api/v1/games/{gameId}/participants
```

**Request:**
```json
{
  "personId": "uuid",
  "role": "REFEREE",
  "function": "MAIN_REFEREE"
}
```

**Response:** 201 Created

### Remover participante de um jogo

```
DELETE /api/v1/games/{gameId}/participants/{participantId}
```

**Response:** 204 No Content

---

## 3. Tela de Confronto — Participantes

### Layout da Tela

```
┌─────────────────────────────────────────────────────┐
│  Confronto: Time A vs Time B                        │
│  Campeonato: Liga Nacional 2026                     │
│  Data: 20/09/2026 15:00                             │
├─────────────────────────────────────────────────────┤
│  Participantes                                      │
│  [+ Adicionar]                                      │
│                                                     │
│  🏷️ Árbitro Principal                              │
│     João Silva (CPF: ***.***.***-**)                │
│     [Remover]                                       │
│                                                     │
│  🏷️ Árbitro Auxiliar                               │
│     Maria Santos (CPF: ***.***.***-**)              │
│     [Remover]                                       │
│                                                     │
│  🏷️ Delegado                                       │
│     Pedro Costa (CPF: ***.***.***-**)               │
│     [Remover]                                       │
└─────────────────────────────────────────────────────┘
```

### Funcionalidades

- **CRUD completo**: adicionar, listar e remover participantes
- **Autocomplete**: buscar pessoa por nome ou CPF
- **Filtro por role**: mostrar apenas árbitros e delegados (não atletas)
- **Validação de duplicata**: impedir mesma pessoa + mesmo jogo

---

## 4. Dropdown de Funções de Jogo

| Código | Label | Ícone Sugerido |
|--------|-------|----------------|
| `MAIN_REFEREE` | Árbitro Principal | 🟡 |
| `ASSISTANT_REFEREE` | Árbitro Auxiliar | 🟡 |
| `DELEGATE` | Delegado | 🔵 |
| `COMMISSIONER` | Comissário | 🔵 |
| `SCORER` | Marcador | ⚪ |
| `TIMEKEEPER` | Cronometrista | ⚪ |

---

## 5. Validações

| Campo | Regra | Mensagem |
|-------|-------|----------|
| `personId` | Obrigatório, UUID válido, pessoa ACTIVE | "Pessoa não encontrada ou inativa" |
| `role` | Obrigatório, PersonRole válido (REFEREE, DELEGATE, COMMISSIONER) | "Papel inválido" |
| `function` | Opcional, GameParticipantFunction válido | "Função inválida" |
| Duplicata | Mesma pessoa + mesmo jogo | "Participante já atribuído a este jogo" |

---

## 6. Endpoints Relacionados

| Endpoint | Método | Descrição |
|----------|--------|-----------|
| `/api/v1/games/{gameId}/participants` | GET | Listar participantes |
| `/api/v1/games/{gameId}/participants` | POST | Atribuir participante |
| `/api/v1/games/{gameId}/participants/{id}` | DELETE | Remover participante |
| `/api/v1/persons` | GET | Listar pessoas (para autocomplete) |
| `/api/v1/persons?role=REFEREE` | GET | Filtrar por papel (árbitros) |
| `/api/v1/persons?role=DELEGATE` | GET | Filtrar por papel (delegados) |

---

## 7. Checklist de Implementação

- [ ] Tela de participantes do confronto (CRUD)
- [ ] Dropdown de funções de jogo
- [ ] Autocomplete de pessoas (com filtro por role)
- [ ] Validação de duplicata de participante
- [ ] Validação de role ao selecionar pessoa
