# Flag Platform — Papéis, Contexto e Autorização

Como cada ator usa a plataforma, quais papéis existem e o que cada um pode fazer na API.

## Estrutura da Plataforma

| App | Responsabilidade | Público-alvo |
|-----|-----------------|--------------|
| **Flag Backend** | API REST, regras de negócio, persistência | Todos os apps |
| **Flag Admin Web** | Gestão administrativa, cadastros | Staff, organizadores |
| **Flag Referee App** | Operação de jogos, check-in, lances | Árbitros, mesa |
| **Flag Public App** | Torcedores, atletas, público | Todos |
| **Flag Tester e2e** | Testes end-to-end | QA, Dev |

## Contexto de uso (por que a plataforma existe)

Comunidade de Flag Football no Brasil. As informações estão hoje espalhadas em PDF, Instagram, WhatsApp e planilhas; o app oficial tem baixa qualidade e o site é inoperante. A Flag Platform reúne tudo em um lugar: **calendário, resultados, classificação, elenco, mapa do campo e operação ao vivo**, com padrão único entre organizações.

## Hierarquia Organizacional

```
Organization (Federação/Liga/Associação)
    └── Institution (Clube/Universidade)
         └── Team (Equipe Esportiva)
              └── Roster (Elenco por Temporada)
                   └── Athlete (Atleta)
```

## Papéis de sistema

### Papéis de Sistema (enum `UserRole`)

Define o nível de privilégio e escopo de acesso às APIs e interfaces:

| Role | Descrição | Criação / Atribuição | App Principal |
|------|-----------|----------------------|--------------|
| **ADMIN** | Administrador geral da plataforma: gerencia organizações, parâmetros globais e usuários | Seed / Admin existente | Admin Web |
| **ORGANIZER** | Gestor de federação ou liga esportiva: gerencia competições, categorias e aprova filiações | ADMIN | Admin Web |
| **COMMISSIONER** | Comissário/delegado oficial das partidas: validação e encerramento de súmulas | ORGANIZER | Referee App |
| **REFEREE** | Árbitro de campo: check-in presencial e registro de lances ao vivo | ORGANIZER / ADMIN | Referee App |
| **MANAGER** | Gestor de agremiação/clube: gerencia equipes e inscreve elencos por temporada | Auto-cadastro / ORGANIZER | Admin Web |
| **FAN** | Torcedor / Atleta: visualização pública e acompanhamento em tempo real | Auto-cadastro no Firebase | Public App |

### Papéis Esportivos da Pessoa Física (enum `PersonRole`)

Diferente do acesso ao sistema, o `PersonRole` define a atuação desportiva unificada da pessoa física (`Person` com CPF único):

| PersonRole | Descrição | Atuação Principal |
|------------|-----------|-------------------|
| **ATHLETE** | Atleta competidor | Inscrito em elencos (`team_roster`), passa por check-in presencial |
| **COACH** | Treinador / Head Coach | Comissão técnica inscrita na equipe |
| **TECHNICAL_STAFF** | Auxiliar / Preparador | Comissão técnica estendida |
| **REFEREE** | Árbitro de campo | Escalado como participante oficial (`game_participants`) |
| **DELEGATE** | Delegado da partida | Mesa de arbitragem e conferência oficial |
| **COMMISSIONER** | Comissário da liga | Auditoria de rodada e partidas |

### Custom Claims no Firebase Auth

O Firebase Auth armazena no token JWT as claims decodificadas pelo backend:

```json
{
  "role": "ORGANIZER",
  "org_id": "uuid-da-organizacao",
  "institution_id": "uuid-da-agremiacao-se-manager"
}
```

### Status de conta (enum `UserStatus`)

```
PENDING ─► ACTIVE
   │
   └──► REJECTED
```

- Registro público → **PENDING** até aprovação de um SUPER_ADMIN.
- Conta SUPER_ADMIN/ORG_ADMIN criada por SUPER_ADMIN → já **ACTIVE**.
- `login` exige `ACTIVE` (senão 403).

## Matriz de permissões (API)

Expressões centralizadas em `SecurityExpressions` (SpEL) e aplicadas via `@PreAuthorize`.

| Operação | SUPER_ADMIN | ORG_ADMIN | MANAGER | USER | Público |
|----------|:-----------:|:---------:|:-------:|:----:|:-------:|
| **CRUD de conteúdo** (organização, campeonato, categoria, campo, time, rodada, jogo, atleta, roster) | ✅ | ✅ | ❌ | ❌ | ❌ |
| **Leitura pública** (listas/detalhes de todos os domínios) | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Operação ao vivo** (status do jogo, resultado, pontos, corrigir placar) | ✅ | ✅ | ✅ | ❌ | ❌ |
| **Check-in e validação de atletas** | ✅ | ✅ | ✅ | ❌ | ❌ |
| **Gestão de usuários** (criar, listar, aprovar, rejeitar) | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Login / registro / recuperação de senha** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **`GET /games/{id}/checkin`** (regra especial) | ✅ | ✅ | ✅ | ❌ | ❌ |

**Aplicação por tela:**

- Admin Web mostra **Aprovações** e **Usuários** apenas para `SUPER_ADMIN` (cards da home variam por role).
- Referee App opera jogos com role `ORG_ADMIN` ou `MANAGER`.
- Public App é público, mas usuários logados podem ver stats personalizados.

## Jornadas por papel

### Super Admin (Admin Web)

```mermaid
flowchart LR
    A[Login com<br/>SUPER_ADMIN] --> B[Home: dashboard]
    B --> C[Gerencia organizações]
    B --> D[Aprova/rejeita usuários]
    B --> E[Configura sistema]
```

**Resultado esperado**: visão completa do sistema, controle total sobre organizações e usuários.

### Org Admin (Admin Web / Referee App)

```mermaid
flowchart LR
    A[Cadastra conta<br/>aguarda aprovação] --> B[Login]
    B --> C[Home: cards]
    C --> D[Cria organização]
    D --> E[Publica campeonato]
    E --> F[Estrutura: categorias,<br/>campos, times, rodadas, jogos]
    F --> G[Cadastra atletas e elencos]
```

**Resultado esperado**: manter o campeonato atualizado em poucos cliques, sem planilhas/PDF/WhatsApp.

### Manager (Referee App)

```mermaid
flowchart LR
    A[Conta criada pelo ORG_ADMIN<br/>role MANAGER] --> B[Login no celular]
    B --> C[Escolhe jogo]
    C --> D[Pré-jogo: check-in dos atletas]
    D --> E[Inicia a partida]
    E --> F[Atualiza placar ao vivo]
    E --> G[Registra lances/jogadas]
    F --> H[Finaliza partida]
```

**Resultado esperado**: operar a partida em menos de 30 segundos — iniciar, placar, validar atletas, finalizar.

### Atleta / Torcedor (Public App — sem login)

```mermaid
flowchart LR
    A[Abre o app] --> B[Lista de campeonatos]
    B --> C[Calendário de jogos]
    B --> D[Resultados recentes]
    B --> E[Classificação]
    C --> F[Detalhe do jogo:<br/>placar ao vivo, local, mapa]
    E --> G[Stats de atletas]
```

**Resultado esperado**: saber tudo sobre o campeonato — próximo jogo, campo, adversário, placar e classificação.

## Segurança transversal

- **JWT HS256** (stateless) injetado por `JwtAuthenticationFilter`; expiração configurável (`app.jwt.expiration-seconds`, default 3600s).
- **Rate limit** no login: `LoginRateLimitFilter` (10 tentativas / 300s por IP → 429).
- **Token de reset de senha**: armazenado com hash SHA-256, expira em 60min, uso único (`usedAt`); anti-enumeração (não revela se o e-mail existe).
- **Senhas**: BCrypt.
- **Erros de API**: `ApiException` → RFC-7807 (`ProblemDetail`) com status HTTP coerente (400/401/403/404/409/429).
- **CORS**: liberado para `localhost:*` em dev (Flutter Web) + origens configuráveis.

## Modelo de domínio (visão geral)

| Entidade | Tabela | FK |
|----------|--------|-----|
| Organization | `organizations` | — |
| Club | `clubs` | organization_id |
| Competition | `competitions` | organization_id |
| Category | `categories` | competition_id |
| Venue | `venues` | organization_id |
| Team | `teams` | category_id |
| Round | `rounds` | category_id |
| Game | `games` | round_id, home_team_id, away_team_id, venue_id |
| ScoreEvent | `score_events` | game_id, team_id |
| Standing | `standings` | category_id, team_id |
| Athlete | `athletes` | — |
| RosterEntry | `team_roster` | team_id, athlete_id |
| CheckIn | `checkins` | game_id, team_id, athlete_id |
| User | `users` | — |
| PasswordResetToken | `password_reset_tokens` | user_id |
