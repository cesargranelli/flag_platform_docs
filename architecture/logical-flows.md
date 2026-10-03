# Flag Platform — Fluxos Lógicos da Solução

> Fluxos operacionais ponta a ponta do ecossistema, desde o ciclo de identidade até a operação de partida e projeção em tempo real.

---

## 1. Ciclo de Vida de Identidade & Autenticação (Firebase-First — ADR-010)

O sistema utiliza o **Firebase Auth** como Provedor de Identidade (IdP) e o backend como validador *stateless* com base em **Custom Claims**.

### Cadastro e Primeiro Acesso:

```mermaid
sequenceDiagram
    actor U as Usuário
    participant App as Cliente Flutter (Kickster)
    participant FB as Firebase Auth (IdP)
    participant API as Flag Backend (/api/v1)
    participant DB as PostgreSQL

    U->>App: Preenche cadastro (E-mail + Senha)
    App->>FB: createUserWithEmailAndPassword()
    FB-->>App: Retorna UserCredential + ID Token (JWT)

    App->>API: GET /auth/me (Bearer <ID_TOKEN>)
    Note over API: JwtAuthenticationFilter:<br/>Valida ID Token em memória com chaves públicas do Google
    API->>DB: Provisiona ou busca registro em platform.users
    DB-->>API: Dados do usuário + Role
    API-->>App: HTTP 200 { user, permissions }
    App-->>U: Redireciona para Dashboard correspondente
```

### Recuperação de Senha:

```mermaid
sequenceDiagram
    actor U as Usuário
    participant App as Cliente Flutter
    participant FB as Firebase Auth

    U->>App: Solicita recuperação ("Esqueci a senha")
    App->>FB: sendPasswordResetEmail(email)
    FB-->>U: Envia e-mail oficial com link de redefinição
    U->>FB: Usuário redefine senha com segurança no Firebase
    App-->>U: Notifica instrução de sucesso na tela
```

---

## 2. Fluxo Institucional: Organização, Agremiação e Filiação

A Flag Platform separa estritamente quem organiza campeonatos (`Organization`) de quem possui as equipes esportivas (`Institution`):

```mermaid
sequenceDiagram
    actor Org as Diretor da Federação
    actor Club as Gestor da Agremiação
    participant Web as Flag Admin Web
    participant API as Flag Backend (/api/v1)
    participant DB as PostgreSQL

    Note over Org,DB: 1. Organização cria o campeonato e abre janela de filiação
    Org->>Web: Cadastra Competição e Janela de Filiação
    Web->>API: POST /competitions e POST /affiliation-windows
    API->>DB: Grava competição e janela

    Note over Club,DB: 2. Agremiação filia-se à Organização
    Club->>Web: Solicita Filiação Institucional
    Web->>API: POST /institutions/{id}/affiliations
    API->>DB: Registra InstitutionAffiliation status=PENDING
    Org->>Web: Aprova filiação
    Web->>API: PATCH /affiliations/{id}/approve
    API->>DB: Status → ACTIVE

    Note over Club,DB: 3. Inscrição de Equipes e Elencos
    Club->>Web: Inscreve Time da Agremiação na Competição
    Web->>API: POST /competitions/{id}/teams
    API->>DB: Cria competition_team
    Club->>Web: Registra Elenco da Temporada com Atletas (Persons)
    Web->>API: POST /rosters/{rosterId}/athletes
    API->>DB: Vincula PersonId em team_roster
```

---

## 3. Fluxo de Operação de Partida ao Vivo (Mesa → Público — ADR-002)

O `flag_referee_app` opera o confronto de forma tolerante a oscilações de rede. O resultado é consolidado no PostgreSQL e espelhado no Firestore para os torcedores:

```mermaid
sequenceDiagram
    actor Ref as Árbitro / Mesário
    participant RefApp as Flag Referee App
    participant API as Flag Backend (/api/v1)
    participant PG as PostgreSQL (ACID)
    participant FS as Cloud Firestore (CQRS Mirror)
    participant PubApp as Flag Public App (Torcedor)

    Note over Ref,RefApp: 1. Check-In de Atletas
    Ref->>RefApp: Confere foto, documento e camisa de cada atleta
    RefApp->>API: POST /games/{id}/checkins
    API->>PG: Grava presença confirmada do atleta no jogo

    Note over Ref,RefApp: 2. Início do Confronto e Registro de Lances
    Ref->>RefApp: Registra Touchdown / Falta / Pontuação Extra
    RefApp->>API: POST /games/{id}/plays e POST /games/{id}/score-events
    API->>PG: Grava transação de lance e atualiza placar oficial

    Note over PG,FS: 3. Projeção CQRS Light para Tempo Real
    PG-->>FS: Projeta estado atualizado do jogo (Live Score)
    FS-->>PubApp: Realtime Listener dispara atualização na tela do torcedor em milissegundos

    Note over Ref,PG: 4. Finalização da Súmula
    Ref->>RefApp: Finaliza Partida (Assinatura digital da mesa)
    RefApp->>API: POST /games/{id}/finish
    API->>PG: Atualiza status=FINISHED e recalcula Standings (Classificação)
    PG-->>FS: Projeta nova tabela de classificação no Firestore
```

---

## 4. Matriz de Autorização por Endpoint (SpEL & SecurityExpressions)

A segurança da API é garantida stateless em tempo de execução via `@PreAuthorize`:

| Recurso | Ação | Mínimo Papel Exigido | Método / Endpoint |
|---|---|:---:|---|
| **Organizações** | Criar nova Organização | `SUPER_ADMIN` | `POST /api/v1/organizations` |
| **Agremiações** | Criar Agremiação | `ORG_ADMIN` / `SUPER_ADMIN` | `POST /api/v1/institutions` |
| **Filiações** | Aprovar Filiação de Agremiação | `ORG_ADMIN` da Organização | `PATCH /api/v1/affiliations/{id}/approve` |
| **Competições** | Criar Campeonato e Categorias | `ORG_ADMIN` da Organização | `POST /api/v1/competitions` |
| **Elencos** | Inscrever Atletas no Elenco | `MANAGER` / `ORG_ADMIN` | `POST /api/v1/rosters/{id}/athletes` |
| **Partidas** | Realizar Check-in de Atleta | `REFEREE` / `MANAGER` / `ORG_ADMIN` | `POST /api/v1/games/{id}/checkins` |
| **Súmula** | Registrar Pontuação de Lance | `REFEREE` da Partida | `POST /api/v1/games/{id}/score-events` |
| **Público** | Consultar Classificação e Jogos | *Acesso Público (Sem Auth)* | `GET /api/v1/competitions/{id}/standings` |
