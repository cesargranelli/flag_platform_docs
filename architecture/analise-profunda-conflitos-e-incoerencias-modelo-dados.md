# AnÃ¡lise Profunda: ResoluÃ§Ã£o de Conflitos, IncoerÃªncias e Fronteiras de Todas as Tabelas do Modelo

> **Projeto:** Flag Platform (Ecossistema America)  
> **Data:** 30 de Outubro de 2026  
> **Autoridade Arquitetural:** ADR-001, Domain-Driven Design (DDD) & Modulith  
> **Status:** Fase 1 ConcluÃ­da e Publicada em Staging (Liquibase 020 Ativo)  

---

## 1. VisÃ£o Executiva e DiagnÃ³stico Fundacional

O modelo de dados atual da Flag Platform reflete o crescimento orgÃ¢nico da plataforma: iniciou com um schema simplificado (onde entidades de federaÃ§Ã£o gerenciavam tudo diretamente) e evoluiu com a introduÃ§Ã£o do conceito formal de **AgremiaÃ§Ã£o (`institutions`)**, **SÃºmula EletrÃ´nica (`plays`)** e **FiliaÃ§Ã£o Anual (`institution_affiliations`)**.

Entretanto, esse crescimento acumulou **conflitos conceituais graves**, atributos alocados em tabelas incorretas, violaÃ§Ãµes de escopo e duplicaÃ§Ãµes estruturais que violam as fronteiras dos *Bounded Contexts* do esporte:

```mermaid
flowchart TD
    subgraph Erros["Anomalias Estruturais CrÃ­ticas"]
        E1["`persons.nickname` â€” Apelido esportivo forÃ§ado na certidÃ£o de pessoa civil"]
        E2["`teams.organization_id` â€” Time vinculado Ã  FederaÃ§Ã£o em vez de pertencer Ã  AgremiaÃ§Ã£o"]
        E3["NÃºmero e PosiÃ§Ãµes do Atleta â€” Risco de poluiÃ§Ã£o da identidade civil em vez de isolamento no Roster"]
        E4["Duplicidade N:N: `institution_organizations` vs `institution_affiliations`"]
        E5["Duplicidade de Timeline: `score_events` vs `plays`"]
        E6["`persons.role` â€” Papel esportivo rÃ­gido engessando uma pessoa fÃ­sica com mÃºltiplos papÃ©is"]
        E7["`venues.organization_id` â€” PraÃ§as esportivas restritas a FederaÃ§Ãµes, ignorando CTs de Clubes"]
        E8["`competition_team` â€” ConferÃªncia e DivisÃ£o gravadas como strings soltas desconectadas de conferences e divisions"]
    end

    subgraph Contextos["Fronteiras CanÃ´nicas do DomÃ­nio (DDD)"]
        C1["Identidade Civil: persons"]
        C2["GovernanÃ§a Federativa: organizations & institution_affiliations"]
        C3["Estrutura ClubÃ­stica: institutions, teams & venues"]
        C4["CompetiÃ§Ãµes: competitions, rounds, conferences, divisions & competition_team"]
        C5["Elencos e Atletas: rosters & team_roster (roster_entries)"]
        C6["OperaÃ§Ã£o de Jogo: games, checkins, game_participants & plays"]
    end

    E1 -.->|Violou| C1
    E2 -.->|Violou| C3
    E3 -.->|Violou| C5
    E4 -.->|Violou| C2
    E5 -.->|Violou| C6
    E6 -.->|Violou| C1
    E7 -.->|Violou| C3
    E8 -.->|Violou| C4
```

---

## 2. AnÃ¡lise Detalhada dos Conflitos por Bounded Context

### 2.1 Contexto de Identidade Civil vs Contexto de Elenco Esportivo

#### Conflito A: `nickname` em `persons` (Atributo Esportivo ForÃ§ado na Pessoa Civil)
- **O Problema:** A tabela `persons` representa a **pessoa fÃ­sica civil** (portadora de CPF Ãºnico, nome civil completo, data de nascimento e foto de documento).
- **A IncoerÃªncia:** Nem toda pessoa fÃ­sica no ecossistema Ã© atleta em campo: existem **Ãrbitros (`REFEREE`)**, **ComissÃ¡rios (`COMMISSIONER`)**, **Delegados de Partida (`DELEGATE`)**, **MesÃ¡rios (`SCORER`/`TIMEKEEPER`)**, **Presidentes de Clube** e **Treinadores (`COACH`)**. Nenhum Ã¡rbitro ou delegado possui ou deve possuir "apelido de jogo" no seu cadastro civil.
- **SoluÃ§Ã£o CanÃ´nica:** O apelido esportivo pertence exclusivamente a **`team_roster.nickname`**. A tabela `persons` deve conter apenas dados civis: `name`, `cpf`, `birth_date`, `gender`, `photo_url`, `status`.

#### Conflito B: NÃºmero da Camisa (`jersey_number`) Isolado Exclusivamente no Roster
- **O Problema:** O nÃºmero da camisa que o atleta veste em campo (`jersey_number` / `number`) Ã© uma atribuiÃ§Ã£o **100% esportiva e temporal**, associada Ã  inscriÃ§Ã£o do jogador em uma determinada equipe para uma temporada.
- **Regras Fundamentais de DomÃ­nio:**
  1. **Pertence estritamente ao Roster:** O nÃºmero padrÃ£o da camisa pertence Ã  tabela **`team_roster.jersey_number`**. NUNCA deve existir em `persons`.
  2. **Multi-Equipe e Troca de NÃºmero:** Um mesmo atleta civil (CPF X) pode jogar com a camisa `#10` no time A e, se for transferido ou jogar em outra modalidade, vestir a camisa `#7` no time B.
  3. **Override TemporÃ¡rio de Partida (`checkins.match_number`):** No dia de uma partida especÃ­fica, se o atleta precisar utilizar uma camisa reserva (ex.: uniforme rasgado ou duplicidade acidental no jogo), o campo `checkins.match_number` funciona estritamente como *game override* temporÃ¡rio, **sem alterar o nÃºmero oficial cadastrado no elenco** (`team_roster.jersey_number`).

#### Conflito C: PosiÃ§Ãµes do Atleta (`positions`)
- **O Problema:** PosiÃ§Ãµes em campo (ex.: `QB`, `WR`, `DB`, `C`, `RUSH`) sÃ£o especÃ­ficas de cada equipe e tÃ¡tica.
- **HistÃ³rico e CorreÃ§Ã£o:** No passado, a migration `015-add-positions-to-persons.yaml` tentou adicionar `positions` Ã  tabela `persons`, o que se provou um erro conceitual e foi revertido na migration `016-drop-positions-from-persons.yaml`. A autoridade exclusiva de posiÃ§Ãµes Ã© **`team_roster.positions`**.

#### Conflito D: `persons.role` RÃ­gido em Cadastro de CPF Ãšnico
- **O Problema:** A tabela `persons` possui uma coluna `role NOT NULL` (`ATHLETE`, `COACH`, `TECHNICAL_STAFF`, `REFEREE`, `DELEGATE`, `COMMISSIONER`).
- **A IncoerÃªncia:** No desporto amador e universitÃ¡rio brasileiro, **o papel de uma pessoa fÃ­sica Ã© contextual**:
  - No sÃ¡bado de manhÃ£, a pessoa atua como Ãrbitro Principal (`game_participants`).
  - No domingo Ã  tarde, a mesma pessoa joga como Atleta (`team_roster`) por um clube em outra divisÃ£o.
  - Sendo o `cpf` Ãºnico, o `role` fixo em `persons` bloqueia a pessoa de exercer mÃºltiplos papÃ©is no ecossistema.
- **SoluÃ§Ã£o CanÃ´nica:** O papel em `persons` deve ser um papel padrÃ£o/preferencial de cadastro (`default_role`), enquanto o papel efetivo Ã© sempre determinado pelo contexto de atuaÃ§Ã£o:
  - Na partida: `game_participants.role` e `game_participants.function`.
  - No elenco: `team_roster`.

---

### 2.2 Contexto de Estrutura ClubÃ­stica vs GovernanÃ§a Federativa

#### Conflito E: `teams.organization_id` (Quebra da Hierarquia de Propriedade do Clube)
- **O Problema:** Na governanÃ§a esportiva:
  - **FederaÃ§Ã£o / Liga (`organizations`):** Organiza competiÃ§Ãµes e homologa filiaÃ§Ãµes. NÃ£o possui equipes esportivas.
  - **AgremiaÃ§Ã£o / Clube (`institutions`):** Entidade de prÃ¡tica desportiva detentora dos direitos e mantenedora das suas equipes.
  - **Equipe (`teams`):** Pertence **100% Ã  sua AgremiaÃ§Ã£o (`institution_id`)**.
- **A IncoerÃªncia:** A tabela `teams` ainda mantÃ©m a coluna legada `organization_id` (quase sempre `NULL`), gerando queries duplicadas (`findAllByOrganizationId`) e rotas ambÃ­guas.
- **SoluÃ§Ã£o CanÃ´nica:** Dropar a coluna `organization_id` da tabela `teams`. O vÃ­nculo de um time com uma FederaÃ§Ã£o se dÃ¡ pela participaÃ§Ã£o em campeonatos via **`competition_team`** (`competition_id` $\rightarrow$ `competitions.organization_id`).

#### Conflito F: Duplicidade entre `institution_organizations` e `institution_affiliations`
- **O Problema:** Existem duas tabelas ligando Clubes a FederaÃ§Ãµes:
  1. `institution_organizations`: tabela N:N estÃ¡tica e sem auditoria (`institution_id`, `organization_id`).
  2. `institution_affiliations`: modelo rico que governa a filiaÃ§Ã£o oficial anual (`season`, `status`: `PENDING`/`APPROVED`/`REJECTED`, `requested_at`, `reviewed_at`, `rejection_reason`).
- **SoluÃ§Ã£o CanÃ´nica:** Dropar a tabela legada `institution_organizations` e unificar toda relaÃ§Ã£o FederaÃ§Ã£o $\leftrightarrow$ Clube exclusivamente em `institution_affiliations`.

#### Conflito G: PraÃ§as Esportivas (`venues.organization_id` vs Centros de Treinamento de Clubes)
- **O Problema:** A tabela `venues` possui apenas a coluna `organization_id`.
- **A IncoerÃªncia:** Um estÃ¡dio ou centro de treinamento pode pertencer a um Clube (`institution_id`), ser administrado pela FederaÃ§Ã£o (`organization_id`), ou ser uma praÃ§a pÃºblica neutra alugada para rodadas duplas. A amarraÃ§Ã£o rÃ­gida atual impede agremiaÃ§Ãµes de cadastrarem seus prÃ³prios campos mandantes.
- **SoluÃ§Ã£o CanÃ´nica:** Permitir que `venues` possua opcionalmente `institution_id` e/ou `organization_id`, ou tornÃ¡-la uma praÃ§a esportiva compartilhÃ¡vel no ecossistema.

---

### 2.3 Contexto de CompetiÃ§Ãµes e Agrupamentos

#### Conflito H: `competition_team` (Strings Desnormalizadas vs Tabelas Estruturadas)
- **O Problema:** A tabela `competition_team` grava a alocaÃ§Ã£o do time com colunas de texto soltas:
  - `group_name VARCHAR`
  - `conference_name VARCHAR`
  - `division_name VARCHAR`
- **A IncoerÃªncia:** Existem tabelas formais no banco: `conferences` e `divisions`. Se o administrador alterar o nome de uma conferÃªncia na tabela mestre, os registros em `competition_team` ficam dessincronizados.
- **SoluÃ§Ã£o CanÃ´nica:** Padronizar as chaves estrangeiras (`conference_id`, `division_id`) e manter `group_name` apenas para agrupamentos simples baseados em letras/grupos (`Grupo A`, `Grupo B`).

#### Conflito I: Nomenclatura das Tabelas de Elenco (`rosters` vs `team_roster`)
- **O Problema:**
  - `rosters`: Tabela de cabeÃ§alho do elenco (`team_id`, `competition_id`, `season`, `status`).
  - `team_roster`: Tabela de itens/linhas com os atletas inscritos (`roster_id`, `person_id`, `nickname`, `jersey_number`, `positions`, `status`).
- **A IncoerÃªncia:** A tabela de cabeÃ§alho Ã© plural (`rosters`), mas a tabela de itens chama-se `team_roster` (que parece um cabeÃ§alho singular de time).
- **SemÃ¢ntica CanÃ´nica:** No nÃ­vel de classes Java e documentaÃ§Ã£o, o cabeÃ§alho Ã© `Roster` e as entradas sÃ£o `RosterEntry`.

---

### 2.4 Contexto de Partidas e Linha do Tempo

#### Conflito J: Duplicidade entre `score_events` e `plays`
- **O Problema:**
  - `score_events`: Criado na V1 da plataforma para gravar apenas os eventos que pontuam.
  - `plays`: Tabela completa de sÃºmula eletrÃ´nica da partida (jardas, quartos, tempos, faltas, turnovers, conversÃµes de 1 e 2 pontos, touchdowns e safeties).
- **A IncoerÃªncia:** Toda vez que um lance Ã© inserido em `plays`, o backend grava uma cÃ³pia duplicada resumida em `score_events` via ApplicationEvent.
- **SoluÃ§Ã£o CanÃ´nica:** Descontinuar e dropar `score_events`. A tabela canÃ´nica para todos os eventos da partida Ã© **`plays`**.

#### Conflito K: `checkins` vs `game_participants`
- **Clareza de PapÃ©is:**
  - `checkins`: Atletas do time escalados na partida (com `team_id`, `person_id`, `match_number` de camisa e status `PRESENT`).
  - `game_participants`: Oficiais neutros da partida (com `game_id`, `person_id`, `role = REFEREE/DELEGATE/COMMISSIONER` e `function = MAIN_REFEREE/SCORER/TIMEKEEPER`).
- **DiagnÃ³stico:** A separaÃ§Ã£o conceitual aqui Ã© correta (atletas pertencem a um time no jogo; oficiais pertencem ao jogo como neutros). Apenas a nomenclatura `checkins` ganha maior clareza quando entendida como `game_roster` / `match_roster`.

---

## 3. Matriz Completa das 25 Tabelas do Ecossistema

A tabela a seguir apresenta a revisÃ£o exaustiva de cada entidade do modelo, seu status de coerÃªncia e a aÃ§Ã£o de normalizaÃ§Ã£o necessÃ¡ria:

| # | Tabela | Bounded Context | Responsabilidade de DomÃ­nio | IncoerÃªncia / Conflito Atual | AÃ§Ã£o de NormalizaÃ§Ã£o |
|---|---|---|---|---|---|
| 1 | **`persons`** | Identidade Civil | Cadastro de pessoa fÃ­sica (CPF, nome civil, nascimento, foto) | Possui `nickname` (que Ã© de atleta) e `role` rÃ­gido | **Remover `nickname`**; flexibilizar `role` para atuaÃ§Ãµes contextuais |
| 2 | **`team_roster`** | Elencos | Atletas inscritos no elenco com apelido de jogo, nÃºmero e posiÃ§Ãµes | Nome da tabela confuso com `rosters` | **Isolar aqui com exclusividade: `nickname`, `jersey_number` e `positions`** |
| 3 | **`rosters`** | Elencos | CabeÃ§alho do elenco do time (base ou competiÃ§Ã£o) | - | Coerente: vincula `team_id` e opcionalmente `competition_id` |
| 4 | **`teams`** | Estrutura ClubÃ­stica | Equipe esportiva mantida pelo clube | **Possui `organization_id`** | **Dropar `teams.organization_id`**; time pertence 100% a `institution_id` |
| 5 | **`institutions`** | Estrutura ClubÃ­stica | AgremiaÃ§Ã£o / Clube (CNPJ, presidente, sede, cores) | - | Coerente: entidade de prÃ¡tica desportiva dona dos times |
| 6 | **`organizations`**| GovernanÃ§a | FederaÃ§Ã£o / Liga esportiva que organiza torneios | - | Coerente: administra campeonatos e homologa filiaÃ§Ãµes |
| 7 | **`institution_affiliations`** | GovernanÃ§a | FiliaÃ§Ã£o anual formal do clube na federaÃ§Ã£o | - | Coerente: possui `season`, `status`, datas de homologaÃ§Ã£o |
| 8 | **`affiliation_windows`** | GovernanÃ§a | Janela temporal para abertura de pedidos de filiaÃ§Ã£o | - | Coerente: governa prazos federativos |
| 9 | **`institution_organizations`** | GovernanÃ§a | VÃ­nculo N:N legado entre clube e federaÃ§Ã£o | **Duplica `institution_affiliations`** sem auditoria ou status | **Dropar tabela `institution_organizations`** |
| 10 | **`competitions`** | CompetiÃ§Ãµes | Torneio esportivo gerido pela federaÃ§Ã£o | - | Coerente: modalidade, gÃªnero, formato, temporadas |
| 11 | **`competition_enrollment_windows`** | CompetiÃ§Ãµes | Janela de inscriÃ§Ã£o de equipes no torneio | - | Coerente: prazos de homologaÃ§Ã£o de times |
| 12 | **`competition_team`** | CompetiÃ§Ãµes | InscriÃ§Ã£o e chaveamento de equipe no torneio | Strings soltas de conferÃªncia e divisÃ£o | Substituir por FKs `conference_id` e `division_id` |
| 13 | **`conferences`** | CompetiÃ§Ãµes | DivisÃ£o de conferÃªncia no torneio (ex: Metropolis, Caipira) | - | Coerente: estrutura formal de agrupamento |
| 14 | **`divisions`** | CompetiÃ§Ãµes | SubdivisÃ£o da conferÃªncia no torneio | - | Coerente: agrupamento de times |
| 15 | **`rounds`** | CompetiÃ§Ãµes | Rodadas regulares e fases de playoffs | - | Coerente: sequenciamento de jogos |
| 16 | **`games`** | OperaÃ§Ã£o de Jogo | Partida entre mandante e visitante | - | Coerente: status, placar, data/hora e local |
| 17 | **`venues`** | Estrutura / Jogos | PraÃ§a esportiva / Campo de jogo | **Restrito a `organization_id`**, sem suporte a CT de clube | Adicionar suporte a `institution_id` |
| 18 | **`checkins`** | OperaÃ§Ã£o de Jogo | PresenÃ§a e escalaÃ§Ã£o de atletas na partida | - | Coerente: `match_number` Ã© override de camisa para o jogo |
| 19 | **`game_participants`**| OperaÃ§Ã£o de Jogo | Equipe neutra de arbitragem e mesa da partida | - | Coerente: isola oficiais com `role` e `function` |
| 20 | **`plays`** | OperaÃ§Ã£o de Jogo | Linha do tempo canÃ´nica de todos os lances | - | Coerente: fonte Ãºnica da verdade para estatÃ­sticas e placar |
| 21 | **`score_events`** | OperaÃ§Ã£o de Jogo | Resumo legado de pontuaÃ§Ãµes | **Duplica lances de `plays`** | **Descontinuar e dropar `score_events`** |
| 22 | **`standings`** | ClassificaÃ§Ã£o | Tabela consolidada de pontuaÃ§Ã£o e aproveitamento | - | Coerente: terminologia corrigida para `points_for/against` |
| 23 | **`standings_rules`** | ClassificaÃ§Ã£o | CritÃ©rios de desempate e ranking do torneio | - | Coerente: motor de cÃ¡lculo de classificaÃ§Ã£o |
| 24 | **`playoff_configurations`** | ClassificaÃ§Ã£o | Regras e chaveamento da fase final de playoffs | - | Coerente: estrutura eliminatÃ³ria |
| 25 | **`users`** | Acesso e SeguranÃ§a | Contas de login e credenciais na plataforma | - | Coerente: escopo via `organization_id` ou `institution_id` |

---

## 4. O Modelo de Elenco e Atleta Perfeito (Canonical Roster Architecture)

Para responder com precisÃ£o mÃ¡xima ao questionamento sobre os atributos do atleta:

```mermaid
classDiagram
    class Person {
        +UUID id
        +String name
        +String cpf
        +LocalDate birthDate
        +Gender gender
        +String photoUrl
        +PersonStatus status
    }

    class Roster {
        +UUID id
        +UUID teamId
        +UUID competitionId
        +String name
        +String season
        +RosterStatus status
    }

    class RosterEntry {
        +UUID id
        +UUID rosterId
        +UUID personId
        +String nickname
        +Integer jerseyNumber
        +String positions
        +RosterStatus status
    }

    class MatchCheckIn {
        +UUID id
        +UUID gameId
        +UUID teamId
        +UUID personId
        +Integer matchNumberOverride
        +CheckInStatus status
        +UUID validatedBy
        +LocalDateTime validatedAt
    }

    Person "1" <-- "0..*" RosterEntry : "identifica o atleta"
    Roster "1" *-- "0..*" RosterEntry : "contÃ©m atletas do elenco"
    Person "1" <-- "0..*" MatchCheckIn : "registra presenÃ§a em jogo"
    RosterEntry ..> MatchCheckIn : "fornece nÃºmero padrÃ£o se override for nulo"
```

### Por que esta separaÃ§Ã£o Ã© perfeita:
1. **`Person` Ã© universal:** Registra qualquer ser humano (atleta, tÃ©cnico, Ã¡rbitro, dirigente) com seus dados civis protegidos.
2. **`RosterEntry` Ã© o avatar esportivo do atleta:** Guarda o **apelido de jogo (`nickname`)**, o **nÃºmero oficial da camisa (`jerseyNumber`)** e as **posiÃ§Ãµes em campo (`positions`)** especÃ­ficos daquele time.
3. **`MatchCheckIn` Ã© a sÃºmula em tempo real:** Herda o nÃºmero padrÃ£o do `RosterEntry`, permitindo `matchNumberOverride` caso no dia do jogo ocorra troca temporÃ¡ria de camisa.

---

## 5. Roteiro Fatorado de ExecuÃ§Ã£o

```mermaid
timeline
    title Roteiro de NormalizaÃ§Ã£o do Schema e DomÃ­nio
    Fase 1 : Limpeza Imediata de Identidade : Remover persons.nickname : Dropar teams.organization_id : Isolar nickname e jersey_number em team_roster
    Fase 2 : ExtinÃ§Ã£o de Tabelas Duplicadas : Dropar institution_organizations (migrando filiaÃ§Ãµes ativas) : Dropar score_events (plays como fonte canÃ´nica)
    Fase 3 : Enriquecimento de Estrutura : Permitir venues vinculadas a institutions (CTs de Clubes) : FKs de conferences e divisions em competition_team
```

### ExecuÃ§Ã£o ConcluÃ­da (Fase 1 â€” NormalizaÃ§Ã£o CanÃ´nica):
1. **Migration Liquibase 020 (`020-normalize-schema-persons-teams.yaml`):**
   - Backfill idempotente: qualquer `persons.nickname` residual foi migrado para `team_roster.nickname`.
   - `ALTER TABLE persons DROP COLUMN nickname`.
   - `ALTER TABLE teams DROP CONSTRAINT fk_team_organization` e `DROP COLUMN organization_id`.
   - Aplicado e validado em staging contra o Oracle ADB 23ai.
2. **RefatoraÃ§Ã£o no Backend Java (`flag_backend`):**
   - Removido `nickname` de `PersonEntity`, `PersonInfo`, `PersonResponse`, `CreatePersonRequest`, `UpdatePersonRequest` e `CreatePersonBatchItem`.
   - Removido `organizationId` e `organizationName` de `TeamEntity`, `TeamResponse`, `CreateTeamRequest` e `UpdateTeamRequest`.
   - `TeamRepository` saneado: removidas queries diretas por `organizationId`. A rota legada consulta atravÃ©s das agremiaÃ§Ãµes filiadas da organizaÃ§Ã£o (`institutionOrgRepo`).
   - `RosterService` e `CheckInService` operam com `nickname`, `positions` e `jersey_number` exclusivamente em `RosterEntryEntity`.
3. **Alinhamento nos Clientes:**
   - `flag_admin_web`, `flag_public_app` e `flag_referee_app` atualizados e validados com `flutter analyze` reportando **0 erros e 0 avisos**.
   - Endpoint ao vivo validado: `GET https://api.america.app.br/api/v1/teams` retorna os times 100% associados Ã  sua agremiaÃ§Ã£o (`institutionId` / `institutionName`).

---

## 6. PrÃ³ximos Passos (Fase 2 e Fase 3)
- **Fase 2:** ExtinÃ§Ã£o de tabelas duplicadas (`institution_organizations` unificada em `institution_affiliations`, e descontinuaÃ§Ã£o de `score_events` em favor de `plays`).
- **Fase 3:** PraÃ§as esportivas (`venues`) com suporte a CTs de clubes (`institution_id`), e estruturaÃ§Ã£o de chaves estrangeiras em `competition_team` (`conference_id`, `division_id`).
