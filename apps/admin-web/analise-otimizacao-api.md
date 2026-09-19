# Análise de Otimização de Chamadas API — Flag Admin Web

## Contexto

O frontend admin consome **97 endpoints** distribuídos em 14 domínios. Várias telas disparam **3-5 chamadas paralelas** ao carregar, mesmo quando grande parte dos dados retornados não é utilizada na UI. Isso gera:
- Latência desnecessária (chamadas sequenciais/paralelas)
- Transferência de dados excessiva (payloads 60-80% maiores que o necessário)
- Custo de infraestrutura no backend (queries desnecessárias)
- Experiência de usuário degradada (tela "travada" carregando)

---

## Diagnóstico: Telas com Maior Consumo

### 🚨 Pior caso: `/competitions/:id/games` (5 chamadas)

| # | Endpoint | Dados usados | Dados desperdiçados |
|---|----------|-------------|---------------------|
| 1 | `GET /competitions/{id}/rounds` | id, number, name | type, competitionId |
| 2 | `GET /competitions/{id}/games` | names, status, score, venue | conferenceId, divisionId |
| 3 | `GET /competitions/{id}/teams` | teamId, logoUrl | **70% do model Team** |
| 4 | `GET /venues` (TODOS) | id, name | address, mapsUrl, orgId |
| 5 | `GET /teams` (TODOS da plataforma) | id, logoUrl | **90% do model Team** |

**Problema crítico:** A chamada #5 busca **todos os times da plataforma** (potencialmente centenas) apenas para montar um mapa `teamId → logoUrl`. A chamada #4 busca **todos os venues** globalmente.

### 🔴 `/games/:id` (5 chamadas)

| # | Endpoint | Dados usados | Problema |
|---|----------|-------------|----------|
| 1 | `GET /games/{id}` | status, scores, scheduledAt | **Retorna só UUIDs** — sem nomes |
| 2 | `GET /competitions/{id}/teams` | name, logoUrl, clubName | Busca TODOS os times para resolver 2 IDs |
| 3 | `GET /competitions/{id}/rounds` | name, number | Busca TODAS as rodadas para resolver 1 ID |
| 4 | `GET /competitions/{id}` | displayName | Busca competição só para o nome |
| 5 | `GET /venues/{id}` | name, address, mapsUrl | **Redundante** — Game já tem venueName/venueAddress |

**Problema crítico:** `GameResponse` (detail) retorna **apenas UUIDs** sem resolver nomes, enquanto `GameSummaryResponse` (list) **já resolve tudo**. Isso força o frontend a fazer chamadas extras.

### 🟡 `/competitions/:id/teams` (2 chamadas, mas pesadas)

| # | Endpoint | Problema |
|---|----------|----------|
| 1 | `GET /competitions/{id}/teams` | OK — usa a maioria dos campos |
| 2 | `GET /teams` (TODOS) | Busca **todos os times** só para `logoUrl` |

**Ironia:** `CompetitionTeamResponse` **já inclui `teamLogoUrl`** resolve do backend! A chamada #2 é 100% dispensável.

### 🟡 `/venues` (2 chamadas)

| # | Endpoint | Problema |
|---|----------|----------|
| 1 | `GET /venues` | OK |
| 2 | `GET /organizations` (TODAS) | Busca **todas as orgs** só para `tradeName` por venue |

**Gap:** `VenueResponse` **não inclui `organizationName`**, ao contrário de `TeamResponse` e `CompetitionResponse` que já resolvem.

---

## Raiz do Problema: Inconsistência nos DTOs

O backend já resolve nomes em **alguns** DTOs, mas não em **todos**:

| DTO | resolve TeamName | resolve TeamLogo | resolve VenueName | resolve OrgName |
|-----|:---:|:---:|:---:|:---:|
| `GameSummaryResponse` | ✅ | ❌ | ✅ | ❌ |
| `GameResponse` (detail) | ❌ | ❌ | ❌ | ❌ |
| `LiveGameResponse` | ✅ | ❌ | ✅ | ❌ |
| `CompetitionTeamResponse` | ✅ | ✅ | — | ✅ |
| `CompetitionResponse` | — | — | — | ✅ |
| `TeamResponse` | — | — | — | ✅ |
| `VenueResponse` | — | — | — | ❌ |
| `InstitutionResponse` | — | — | — | ❌ (só IDs) |
| `RoundResponse` | — | — | — | ❌ |

**O padrão já existe** — o problema é que `GameResponse` (detail) e `VenueResponse` não seguem o padrão.

---

## Estratégia Recomendada: Otimização no Backend (NÃO BFF)

### Por que NÃO BFF?

1. **O padrão de resolução já existe** — `GameSummaryResponse`, `CompetitionTeamResponse`, etc. já fazem JOINs para resolver nomes. Não há necessidade de uma camada extra.
2. **Manutenção** — BFF adiciona mais um serviço para manter, deployar e monitorar.
3. **Complexidade** — O backend já tem a lógica de resolução nos mappers. É questão de aplicar consistentemente.

### O que fazer (3 frentes):

---

### Frente 1: Enriquecer `GameResponse` (MAIOR IMPACTO)

**Problema:** `GET /games/{id}` retorna apenas UUIDs, forçando o frontend a buscar times, rodadas, venue e competição separadamente.

**Solução:** Adicionar ao `GameResponse` os mesmos campos resolvidos que `GameSummaryResponse` já tem:

```
GameResponse atual:
{
  "id": "...",
  "homeTeamId": "...",      // ← só UUID
  "awayTeamId": "...",      // ← só UUID
  "venueId": "...",         // ← só UUID
  "roundId": "...",         // ← só UUID
  "competitionId": "...",   // ← só UUID
  ...
}

GameResponse proposto:
{
  "id": "...",
  "homeTeamId": "...",
  "homeTeamName": "Spartans Black",      // ← NOVO
  "homeTeamShortName": "SPA",            // ← NOVO
  "homeTeamLogoUrl": "https://...",      // ← NOVO
  "awayTeamId": "...",
  "awayTeamName": "Falcons A",           // ← NOVO
  "awayTeamShortName": "FAL",            // ← NOVO
  "awayTeamLogoUrl": "https://...",      // ← NOVO
  "venueId": "...",
  "venueName": "Estádio Central",        // ← NOVO
  "venueAddress": "Rua X, 123",          // ← NOVO
  "venueMapsUrl": "https://...",         // ← NOVO
  "roundId": "...",
  "roundName": "Primeira Fase",          // ← NOVO
  "roundNumber": 1,                      // ← NOVO
  "competitionId": "...",
  "competitionName": "Liga Norte 2026",  // ← NOVO
  "competitionDisplayName": "Liga Norte 2026 Masculino", // ← NOVO
  ...
}
```

**Impacto no frontend:**
- `GameDetailScreen`: 5 chamadas → **1 chamada** (elimina teams, rounds, venue, competition providers)
- `GameEditScreen`: 5 chamadas → **2 chamadas** (só precisa de teams e rounds para dropdowns)

**Esforço backend:** Baixo — os JOINs já existem no `GameSummaryResponse`. É copiar a lógica para `GameResponse`.

---

### Frente 2: Enriquecer `VenueResponse`

**Problema:** `GET /venues` não retorna `organizationName`, forçando o frontend a buscar todas as organizações.

**Solução:** Adicionar `organizationName` ao `VenueResponse`:

```
VenueResponse atual:
{
  "id": "...",
  "name": "Estádio Central",
  "organizationId": "...",    // ← só UUID
  ...
}

VenueResponse proposto:
{
  "id": "...",
  "name": "Estádio Central",
  "organizationId": "...",
  "organizationName": "Liga Norte",  // ← NOVO
  ...
}
```

**Impacto no frontend:**
- `VenueListScreen`: 2 chamadas → **1 chamada** (elimina organizationsProvider)
- `VenueDetailScreen`: 2 chamadas → **1 chamada**
- `VenueCreateScreen`: 2 chamadas → **1 chamada**

**Esforço backend:** Muito baixo — JOIN simples com Organization.

---

### Frente 3: Adicionar `logoUrl` ao `GameSummaryResponse`

**Problema:** O frontend busca **todos os times da plataforma** (`GET /teams`) apenas para montar um mapa `teamId → logoUrl`.

**Solução:** Adicionar `homeTeamLogoUrl` e `awayTeamLogoUrl` ao `GameSummaryResponse`:

```
GameSummaryResponse atual:
{
  "homeTeamName": "Spartans",
  "awayTeamName": "Falcons",
  ...
}

GameSummaryResponse proposto:
{
  "homeTeamName": "Spartans",
  "homeTeamLogoUrl": "https://...",   // ← NOVO
  "awayTeamName": "Falcons",
  "awayTeamLogoUrl": "https://...",   // ← NOVO
  ...
}
```

**Impacto no frontend:**
- `CompetitionGamesScreen`: 5 chamadas → **3 chamadas** (elimina GET /teams e GET /venues)
- `CompetitionTeamsScreen`: 2 chamadas → **1 chamada** (elimina GET /teams)

**Esforço backend:** Baixo — JOIN com Team para pegar `logoUrl`.

---

## Resumo do Impacto

### Antes vs Depois

| Tela | Antes | Depois | Redução |
|------|-------|--------|---------|
| `/competitions/:id/games` | 5 chamadas | 2 chamadas | **-60%** |
| `/games/:id` | 5 chamadas | 1 chamada | **-80%** |
| `/games/:id/edit` | 5 chamadas | 2 chamadas | **-60%** |
| `/competitions/:id/teams` | 2 chamadas | 1 chamada | **-50%** |
| `/venues` | 2 chamadas | 1 chamada | **-50%** |
| `/venues/:id` | 2 chamadas | 1 chamada | **-50%** |
| `/venues/new` | 2 chamadas | 1 chamada | **-50%** |
| `/institutions/:id` | 4 chamadas | 3 chamadas* | **-25%** |

*\*InstitutionDetail: organizationsProvider pode ser carregado lazy (só quando modal abre)*

### Economia de Payload Estimada

| Chamada eliminada | Payload economizado |
|-------------------|-------------------|
| `GET /teams` (TODOS) | ~50-200KB (depende do número de times) |
| `GET /venues` (TODOS) | ~5-20KB |
| `GET /organizations` (TODAS) | ~20-100KB |
| `GET /competitions/{id}` (só para nome) | ~2-5KB |
| `GET /competitions/{id}/teams` (só para logos) | ~10-30KB |

**Total estimado: 87-355KB economizados por carga de tela** nas telas mais pesadas.

---

## Otimização Complementar no Frontend

Além das mudanças no backend, o frontend pode otimizar:

### 1. Lazy-load de organizationsProvider
Em `InstitutionDetailScreen`, o `organizationsProvider` é carregado no `build()` para popular um dropdown de filiação que pode **nunca ser aberto**. Mudar para carregar apenas quando o modal é aberto.

### 2. Usar CompetitionTeamResponse.teamLogoUrl
Em `CompetitionTeamsScreen`, o frontend busca `GET /teams` (TODOS) para logos, mas `CompetitionTeamResponse` **já inclui `teamLogoUrl`**. Basta usar o campo que já existe.

### 3. Criar endpoints leves para dropdowns
Para telas de criação/edição que precisam de listas para dropdowns, criar variantes leves:
- `GET /teams?fields=id,name` → retorna só id + name
- `GET /venues?fields=id,name` → retorna só id + name
- `GET /rounds?fields=id,name,number` → retorna só campos essenciais

---

## Plano de Ação

### Fase 1 — Backend (alto impacto, baixo esforço)
1. Enriquecer `GameResponse` com nomes/logos de times, venue, rodada, competição
2. Adicionar `organizationName` ao `VenueResponse`
3. Adicionar `homeTeamLogoUrl`/`awayTeamLogoUrl` ao `GameSummaryResponse`

### Fase 2 — Frontend (após backend atualizado)
1. `GameDetailScreen`: remover chamadas redundantes (teams, rounds, venue, competition providers)
2. `CompetitionGamesScreen`: remover `GET /teams` e `GET /venues`
3. `CompetitionTeamsScreen`: usar `CompetitionTeamResponse.teamLogoUrl` em vez de `GET /teams`
4. `VenueListScreen`: remover `organizationsProvider`
5. `InstitutionDetailScreen`: lazy-load de `organizationsProvider`

### Fase 3 — Otimização de payloads (futuro)
1. Criar endpoints leves `?fields=id,name` para dropdowns
2. Avaliar paginação em listas grandes

---

## Decisão: Não criar BFF

**Justificativa:**
- O padrão de resolução nos DTOs já existe e funciona
- Não há necessidade de camada adicional de complexidade
- As mudanças são pontuais e de baixo risco
- O backend já tem os JOINs necessários — é só expor nos DTOs
- BFF seria overengineering para este caso de uso

**Quando BFF faria sentido:**
- Se o frontend fosse múltiplo (web + mobile + parceiros) com necessidades diferentes
- Se houvesse necessidade de agregação complexa (dashboards com dados de 5+ entidades)
- Se a latência dos JOINs no backend fosse um problema (não é o caso — são tabelas pequenas)
