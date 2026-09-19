# Flag Referee App — Mudanças no Modelo de Dados

> **Projeto:** Flag Referee App (mobile)  
> **Data:** 2026-09-16  
> **Backend:** Spring Boot 4.1.0  
> **API:** `/api/v1/`

---

## 1. Resumo das Mudanças

| # | Mudança | Impacto no App |
|---|---------|----------------|
| 1 | Novo endpoint `game_participants` | App pode listar árbitros/delegados de um jogo |
| 2 | Campo `player_id` em `plays` | Lance pode vincular ao cadastro do atleta |
| 3 | Funções de jogo (`GameParticipantFunction`) | App pode exibir função de cada participante |

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

### Uso no App

- **Tela de detalhe do jogo**: mostrar lista de participantes com função
- **Tela de check-in**: mostrar quem validou ( validated_by → person )
- **Dashboard**: mostrar jogos onde o usuário é participante

---

## 3. Mudanças em Plays

### Campo `player_id` (novo, opcional)

Ao criar um lance, agora é possível enviar `player_id` para vincular ao cadastro:

**Request (novo):**
```json
{
  "teamId": "uuid",
  "playerId": "uuid",          // ← NOVO (opcional)
  "playerName": "João Silva",  // ← continua obrigatório
  "playType": "TOUCHDOWN",
  "quarter": "Q1",
  "time": "10:30"
}
```

### Comportamento esperado

| Cenário | `playerId` | `playerName` | Resultado |
|---------|-----------|--------------|-----------|
| Atleta do cadastro | `"uuid"` | `"João Silva"` | Salva com vinculo |
| Atleta sem cadastro | `null` | `"João"` | Salva como texto livre |

### Uso no App

- **Tela de lance**: autocomplete de atletas do time (opção rápida)
- Se selecionar atleta do cadastro: `playerId` + `playerName` preenchidos automaticamente
- Se digitar nome manualmente: apenas `playerName` enviado

---

## 4. Tela de Jogo — Fluxo no App

### Tela de Detalhe do Jogo

```
┌─────────────────────────────────────────────┐
│  Jogo: Time A vs Time B                     │
│  Status: IN_PROGRESS                        │
│  Placar: 14 - 7                             │
├─────────────────────────────────────────────┤
│  👥 Participantes                           │
│  ├── 🟡 João Silva — Árbitro Principal      │
│  └── 🟡 Maria Santos — Árbitro Auxiliar     │
│                                             │
│  📋 Check-in: 18/20 atletas presentes       │
│                                             │
│  🏈 Lances (últimos 5)                      │
│  ├── Q1 10:30 — TOUCHDOWN — João Silva      │
│  ├── Q1 08:15 — FIRST DOWN — Pedro Santos   │
│  └── Q1 06:00 — INTERCEPTION — Lucas Costa  │
│                                             │
│  [Registrar Lance] [Check-in] [Finalizar]   │
└─────────────────────────────────────────────┘
```

### Tela de Registrar Lance

```
┌─────────────────────────────────────────────┐
│  Registrar Lance                            │
├─────────────────────────────────────────────┤
│  Time: [Time A ▼]                           │
│                                             │
│  Jogador: [João Silva        🔍]  ← autocomplete│
│  Tipo:    [TOUCHDOWN ▼]                     │
│  Quarter: [Q1 ▼]                            │
│  Tempo:   [10:30]                           │
│  Jardas:  [0]                               │
│                                             │
│  ☑ Touchdown                                │
│  ☐ First Down                               │
│  ☐ Turnover                                 │
│                                             │
│  [Cancelar]              [Salvar]           │
└─────────────────────────────────────────────┘
```

---

## 5. Tela de Participantes

### Listar Participantes

```
┌─────────────────────────────────────────────┐
│  Participantes — Jogo #123                  │
├─────────────────────────────────────────────┤
│                                             │
│  🟡 Árbitro Principal                       │
│  João Silva                                 │
│  CPF: ***.***.***-**                        │
│                                             │
│  🟡 Árbitro Auxiliar                        │
│  Maria Santos                               │
│  CPF: ***.***.***-**                        │
│                                             │
│  🔵 Delegado                                │
│  Pedro Costa                                │
│  CPF: ***.***.***-**                        │
│                                             │
│  ⚪ Marcador                                │
│  Ana Oliveira                               │
│  CPF: ***.***.***-**                        │
└─────────────────────────────────────────────┘
```

### Funcionalidade

- **Somente leitura** no app do árbitro (participantes são definidos pelo admin)
- **Filtro por jogo**: mostrar apenas participantes do jogo selecionado
- **Ícones por função**: facilitar identificação visual

---

## 6. Dropdown de Funções de Jogo

| Código | Label | Ícone | Cor |
|--------|-------|-------|-----|
| `MAIN_REFEREE` | Árbitro Principal | 🟡 | Amarelo |
| `ASSISTANT_REFEREE` | Árbitro Auxiliar | 🟡 | Amarelo |
| `DELEGATE` | Delegado | 🔵 | Azul |
| `COMMISSIONER` | Comissário | 🔵 | Azul |
| `SCORER` | Marcador | ⚪ | Cinza |
| `TIMEKEEPER` | Cronometrista | ⚪ | Cinza |

---

## 7. Endpoints Relacionados

| Endpoint | Método | Descrição | Uso no App |
|----------|--------|-----------|------------|
| `/api/v1/games/{gameId}/participants` | GET | Listar participantes | Tela de detalhe do jogo |
| `/api/v1/games/{gameId}/plays` | GET | Listar lances | Tela de lances |
| `/api/v1/games/{gameId}/plays` | POST | Registrar lance | Tela de lance |
| `/api/v1/games/{gameId}/checkin` | GET | Listar check-in | Tela de check-in |
| `/api/v1/teams/{teamId}/roster` | GET | Listar elenco do time | Autocomplete de jogadores |

---

## 8. Autocomplete de Jogadores

Ao registrar um lance, o app deve:

1. Buscar elenco do time (`GET /api/v1/teams/{teamId}/roster`)
2. Mostrar lista de atletas com nome e número
3. Ao selecionar: preencher `playerId` + `playerName` automaticamente
4. Se atleta não está no cadastro: permitir digitação manual (apenas `playerName`)

### Response do Elenco

```json
[
  {
    "id": "uuid",
    "athleteId": "uuid",
    "athleteName": "João Silva",
    "nickname": "Jota",
    "number": 12,
    "positions": "QB,WR",
    "photoUrl": "https://...",
    "status": "ACTIVE"
  }
]
```

---

## 9. Checklist de Implementação

- [ ] Tela de participantes do jogo (somente leitura)
- [ ] Ícones e cores por função
- [ ] Autocomplete de jogadores ao registrar lance
- [ ] Auto-fill `playerId` + `playerName` ao selecionar jogador
- [ ] Filtro de participantes por jogo
- [ ] Integração com endpoint de elenco para autocomplete
