# AnÃƒÂ¡lise Profunda: ResoluÃƒÂ§ÃƒÂ£o de Conflitos, IncoerÃƒÂªncias e Fronteiras de Todas as Tabelas do Modelo

> **Projeto:** Flag Platform (Ecossistema America)  
> **Data:** 30 de Outubro de 2026  
> **Autoridade Arquitetural:** ADR-001, Domain-Driven Design (DDD) & Modulith  
> **Status:** Fase 1 ConcluÃƒÂ­da e Publicada em Staging (Liquibase 020 Ativo)  

---

## 1. VisÃƒÂ£o Executiva e DiagnÃƒÂ³stico Fundacional

O modelo de dados atual da Flag Platform reflete o crescimento orgÃƒÂ¢nico da plataforma: iniciou com um schema simplificado (onde entidades de federaÃƒÂ§ÃƒÂ£o gerenciavam tudo diretamente) e evoluiu com a introduÃƒÂ§ÃƒÂ£o do conceito formal de **AgremiaÃƒÂ§ÃƒÂ£o (`institutions`)**, **SÃƒÂºmula EletrÃƒÂ´nica (`plays`)** e **FiliaÃƒÂ§ÃƒÂ£o Anual (`institution_affiliations`)**.

Entretanto, esse crescimento acumulou **conflitos conceituais graves**, atributos alocados em tabelas incorretas, violaÃƒÂ§ÃƒÂµes de escopo e duplicaÃƒÂ§ÃƒÂµes estruturais que violam as fronteiras dos *Bounded Contexts* do esporte:

```mermaid
flowchart TD
    subgraph Erros["Anomalias Estruturais CrÃƒÂ­ticas"]
        E1["`persons.nickname` Ã¢â‚¬â€ Apelido esportivo forÃƒÂ§ado na certidÃƒÂ£o de pessoa civil"]
        E2["`teams.organization_id` Ã¢â‚¬â€ Time vinculado ÃƒÂ  FederaÃƒÂ§ÃƒÂ£o em vez de pertencer ÃƒÂ  AgremiaÃƒÂ§ÃƒÂ£o"]
        E3["NÃƒÂºmero e PosiÃƒÂ§ÃƒÂµes do Atleta Ã¢â‚¬â€ Risco de poluiÃƒÂ§ÃƒÂ£o da identidade civil em vez de isolamento no Roster"]
        E4["Duplicidade N:N: `institution_organizations` vs `institution_affiliations`"]
        E5["Duplicidade de Timeline: `score_events` vs `plays`"]
        E6["`persons.role` Ã¢â‚¬â€ Papel esportivo rÃƒÂ­gido engessando uma pessoa fÃƒÂ­sica com mÃƒÂºltiplos papÃƒÂ©is"]
        E7["`venues.organization_id` Ã¢â‚¬â€ PraÃƒÂ§as esportivas restritas a FederaÃƒÂ§ÃƒÂµes, ignorando CTs de Clubes"]
        E8["`competition_team` Ã¢â‚¬â€ ConferÃƒÂªncia e DivisÃƒÂ£o gravadas como strings soltas desconectadas de conferences e divisions"]
    end

    subgraph Contextos["Fronteiras CanÃƒÂ´nicas do DomÃƒÂ­nio (DDD)"]
        C1["Identidade Civil: persons"]
        C2["GovernanÃƒÂ§a Federativa: organizations & institution_affiliations"]
        C3["Estrutura ClubÃƒÂ­stica: institutions, teams & venues"]
        C4["CompetiÃƒÂ§ÃƒÂµes: competitions, rounds, conferences, divisions & competition_team"]
        C5["Elencos e Atletas: rosters & team_roster (roster_entries)"]
        C6["OperaÃƒÂ§ÃƒÂ£o de Jogo: games, checkins, game_participants & plays"]
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

## 2. AnÃƒÂ¡lise Detalhada dos Conflitos por Bounded Context

### 2.1 Contexto de Identidade Civil vs Contexto de Elenco Esportivo

#### Conflito A: `nickname` em `persons` (Atributo Esportivo ForÃƒÂ§ado na Pessoa Civil)
- **O Problema:** A tabela `persons` representa a **pessoa fÃƒÂ­sica civil** (portadora de CPF ÃƒÂºnico, nome civil completo, data de nascimento e foto de documento).
- **A IncoerÃƒÂªncia:** Nem toda pessoa fÃƒÂ­sica no ecossistema ÃƒÂ© atleta em campo: existem **ÃƒÂrbitros (`REFEREE`)**, **ComissÃƒÂ¡rios (`COMMISSIONER`)**, **Delegados de Partida (`DELEGATE`)**, **MesÃƒÂ¡rios (`SCORER`/`TIMEKEEPER`)**, **Presidentes de Clube** e **Treinadores (`COACH`)**. Nenhum ÃƒÂ¡rbitro ou delegado possui ou deve possuir "apelido de jogo" no seu cadastro civil.
- **SoluÃƒÂ§ÃƒÂ£o CanÃƒÂ´nica:** O apelido esportivo pertence exclusivamente a **`team_roster.nickname`**. A tabela `persons` deve conter apenas dados civis: `name`, `cpf`, `birth_date`, `gender`, `photo_url`, `status`.

#### Conflito B: NÃƒÂºmero da Camisa (`jersey_number`) Isolado Exclusivamente no Roster
- **O Problema:** O nÃƒÂºmero da camisa que o atleta veste em campo (`jersey_number` / `number`) ÃƒÂ© uma atribuiÃƒÂ§ÃƒÂ£o **100% esportiva e temporal**, associada ÃƒÂ  inscriÃƒÂ§ÃƒÂ£o do jogador em uma determinada equipe para uma temporada.
- **Regras Fundamentais de DomÃƒÂ­nio:**
  1. **Pertence estritamente ao Roster:** O nÃƒÂºmero padrÃƒÂ£o da camisa pertence ÃƒÂ  tabela **`team_roster.jersey_number`**. NUNCA deve existir em `persons`.
  2. **Multi-Equipe e Troca de NÃƒÂºmero:** Um mesmo atleta civil (CPF X) pode jogar com a camisa `#10` no time A e, se for transferido ou jogar em outra modalidade, vestir a camisa `#7` no time B.
  3. **Override TemporÃƒÂ¡rio de Partida (`checkins.match_number`):** No dia de uma partida especÃƒÂ­fica, se o atleta precisar utilizar uma camisa reserva (ex.: uniforme rasgado ou duplicidade acidental no jogo), o campo `checkins.match_number` funciona estritamente como *game override* temporÃƒÂ¡rio, **sem alterar o nÃƒÂºmero oficial cadastrado no elenco** (`team_roster.jersey_number`).

#### Conflito C: PosiÃƒÂ§ÃƒÂµes do Atleta (`positions`)
- **O Problema:** PosiÃƒÂ§ÃƒÂµes em campo (ex.: `QB`, `WR`, `DB`, `C`, `RUSH`) sÃƒÂ£o especÃƒÂ­ficas de cada equipe e tÃƒÂ¡tica.
- **HistÃƒÂ³rico e CorreÃƒÂ§ÃƒÂ£o:** No passado, a migration `015-add-positions-to-persons.yaml` tentou adicionar `positions` ÃƒÂ  tabela `persons`, o que se provou um erro conceitual e foi revertido na migration `016-drop-positions-from-persons.yaml`. A autoridade exclusiva de posiÃƒÂ§ÃƒÂµes ÃƒÂ© **`team_roster.positions`**.

#### Conflito D: `persons.role` RÃƒÂ­gido em Cadastro de CPF ÃƒÅ¡nico
- **O Problema:** A tabela `persons` possui uma coluna `role NOT NULL` (`ATHLETE`, `COACH`, `TECHNICAL_STAFF`, `REFEREE`, `DELEGATE`, `COMMISSIONER`).
- **A IncoerÃƒÂªncia:** No desporto amador e universitÃƒÂ¡rio brasileiro, **o papel de uma pessoa fÃƒÂ­sica ÃƒÂ© contextual**:
  - No sÃƒÂ¡bado de manhÃƒÂ£, a pessoa atua como ÃƒÂrbitro Principal (`game_participants`).
  - No domingo ÃƒÂ  tarde, a mesma pessoa joga como Atleta (`team_roster`) por um clube em outra divisÃƒÂ£o.
  - Sendo o `cpf` ÃƒÂºnico, o `role` fixo em `persons` bloqueia a pessoa de exercer mÃƒÂºltiplos papÃƒÂ©is no ecossistema.
- **SoluÃƒÂ§ÃƒÂ£o CanÃƒÂ´nica:** O papel em `persons` deve ser um papel padrÃƒÂ£o/preferencial de cadastro (`default_role`), enquanto o papel efetivo ÃƒÂ© sempre determinado pelo contexto de atuaÃƒÂ§ÃƒÂ£o:
  - Na partida: `game_participants.role` e `game_participants.function`.
  - No elenco: `team_roster`.

---

### 2.2 Contexto de Estrutura ClubÃƒÂ­stica vs GovernanÃƒÂ§a Federativa

#### Conflito E: `teams.organization_id` (Quebra da Hierarquia de Propriedade do Clube)
- **O Problema:** Na governanÃƒÂ§a esportiva:
  - **FederaÃƒÂ§ÃƒÂ£o / Liga (`organizations`):** Organiza competiÃƒÂ§ÃƒÂµes e homologa filiaÃƒÂ§ÃƒÂµes. NÃƒÂ£o possui equipes esportivas.
  - **AgremiaÃƒÂ§ÃƒÂ£o / Clube (`institutions`):** Entidade de prÃƒÂ¡tica desportiva detentora dos direitos e mantenedora das suas equipes.
  - **Equipe (`teams`):** Pertence **100% ÃƒÂ  sua AgremiaÃƒÂ§ÃƒÂ£o (`institution_id`)**.
- **A IncoerÃƒÂªncia:** A tabela `teams` ainda mantÃƒÂ©m a coluna legada `organization_id` (quase sempre `NULL`), gerando queries duplicadas (`findAllByOrganizationId`) e rotas ambÃƒÂ­guas.
- **SoluÃƒÂ§ÃƒÂ£o CanÃƒÂ´nica:** Dropar a coluna `organization_id` da tabela `teams`. O vÃƒÂ­nculo de um time com uma FederaÃƒÂ§ÃƒÂ£o se dÃƒÂ¡ pela participaÃƒÂ§ÃƒÂ£o em campeonatos via **`competition_team`** (`competition_id` $\rightarrow$ `competitions.organization_id`).

#### Conflito F: Duplicidade entre `institution_organizations` e `institution_affiliations`
- **O Problema:** Existem duas tabelas ligando Clubes a FederaÃƒÂ§ÃƒÂµes:
  1. `institution_organizations`: tabela N:N estÃƒÂ¡tica e sem auditoria (`institution_id`, `organization_id`).
  2. `institution_affiliations`: modelo rico que governa a filiaÃƒÂ§ÃƒÂ£o oficial anual (`season`, `status`: `PENDING`/`APPROVED`/`REJECTED`, `requested_at`, `reviewed_at`, `rejection_reason`).
- **SoluÃƒÂ§ÃƒÂ£o CanÃƒÂ´nica:** Dropar a tabela legada `institution_organizations` e unificar toda relaÃƒÂ§ÃƒÂ£o FederaÃƒÂ§ÃƒÂ£o $\leftrightarrow$ Clube exclusivamente em `institution_affiliations`.

#### Conflito G: PraÃƒÂ§as Esportivas (`venues.organization_id` vs Centros de Treinamento de Clubes)
- **O Problema:** A tabela `venues` possui apenas a coluna `organization_id`.
- **A IncoerÃƒÂªncia:** Um estÃƒÂ¡dio ou centro de treinamento pode pertencer a um Clube (`institution_id`), ser administrado pela FederaÃƒÂ§ÃƒÂ£o (`organization_id`), ou ser uma praÃƒÂ§a pÃƒÂºblica neutra alugada para rodadas duplas. A amarraÃƒÂ§ÃƒÂ£o rÃƒÂ­gida atual impede agremiaÃƒÂ§ÃƒÂµes de cadastrarem seus prÃƒÂ³prios campos mandantes.
- **SoluÃƒÂ§ÃƒÂ£o CanÃƒÂ´nica:** Permitir que `venues` possua opcionalmente `institution_id` e/ou `organization_id`, ou tornÃƒÂ¡-la uma praÃƒÂ§a esportiva compartilhÃƒÂ¡vel no ecossistema.

---

### 2.3 Contexto de CompetiÃƒÂ§ÃƒÂµes e Agrupamentos

#### Conflito H: `competition_team` (Strings Desnormalizadas vs Tabelas Estruturadas)
- **O Problema:** A tabela `competition_team` grava a alocaÃƒÂ§ÃƒÂ£o do time com colunas de texto soltas:
  - `group_name VARCHAR`
  - `conference_name VARCHAR`
  - `division_name VARCHAR`
- **A IncoerÃƒÂªncia:** Existem tabelas formais no banco: `conferences` e `divisions`. Se o administrador alterar o nome de uma conferÃƒÂªncia na tabela mestre, os registros em `competition_team` ficam dessincronizados.
- **SoluÃƒÂ§ÃƒÂ£o CanÃƒÂ´nica:** Padronizar as chaves estrangeiras (`conference_id`, `division_id`) e manter `group_name` apenas para agrupamentos simples baseados em letras/grupos (`Grupo A`, `Grupo B`).

#### Conflito I: Nomenclatura das Tabelas de Elenco (`rosters` vs `team_roster`)
- **O Problema:**
  - `rosters`: Tabela de cabeÃƒÂ§alho do elenco (`team_id`, `competition_id`, `season`, `status`).
  - `team_roster`: Tabela de itens/linhas com os atletas inscritos (`roster_id`, `person_id`, `nickname`, `jersey_number`, `positions`, `status`).
- **A IncoerÃƒÂªncia:** A tabela de cabeÃƒÂ§alho ÃƒÂ© plural (`rosters`), mas a tabela de itens chama-se `team_roster` (que parece um cabeÃƒÂ§alho singular de time).
- **SemÃƒÂ¢ntica CanÃƒÂ´nica:** No nÃƒÂ­vel de classes Java e documentaÃƒÂ§ÃƒÂ£o, o cabeÃƒÂ§alho ÃƒÂ© `Roster` e as entradas sÃƒÂ£o `RosterEntry`.

---

### 2.4 Contexto de Partidas e Linha do Tempo

#### Conflito J: Duplicidade entre `score_events` e `plays`
- **O Problema:**
  - `score_events`: Criado na V1 da plataforma para gravar apenas os eventos que pontuam.
  - `plays`: Tabela completa de sÃƒÂºmula eletrÃƒÂ´nica da partida (jardas, quartos, tempos, faltas, turnovers, conversÃƒÂµes de 1 e 2 pontos, touchdowns e safeties).
- **A IncoerÃƒÂªncia:** Toda vez que um lance ÃƒÂ© inserido em `plays`, o backend grava uma cÃƒÂ³pia duplicada resumida em `score_events` via ApplicationEvent.
- **SoluÃƒÂ§ÃƒÂ£o CanÃƒÂ´nica:** Descontinuar e dropar `score_events`. A tabela canÃƒÂ´nica para todos os eventos da partida ÃƒÂ© **`plays`**.

#### Conflito K: `checkins` vs `game_participants`
- **Clareza de PapÃƒÂ©is:**
  - `checkins`: Atletas do time escalados na partida (com `team_id`, `person_id`, `match_number` de camisa e status `PRESENT`).
  - `game_participants`: Oficiais neutros da partida (com `game_id`, `person_id`, `role = REFEREE/DELEGATE/COMMISSIONER` e `function = MAIN_REFEREE/SCORER/TIMEKEEPER`).
- **DiagnÃƒÂ³stico:** A separaÃƒÂ§ÃƒÂ£o conceitual aqui ÃƒÂ© correta (atletas pertencem a um time no jogo; oficiais pertencem ao jogo como neutros). Apenas a nomenclatura `checkins` ganha maior clareza quando entendida como `game_roster` / `match_roster`.

---

## 3. Matriz Completa das 25 Tabelas do Ecossistema

A tabela a seguir apresenta a revisÃƒÂ£o exaustiva de cada entidade do modelo, seu status de coerÃƒÂªncia e a aÃƒÂ§ÃƒÂ£o de normalizaÃƒÂ§ÃƒÂ£o necessÃƒÂ¡ria:

| # | Tabela | Bounded Context | Responsabilidade de DomÃƒÂ­nio | IncoerÃƒÂªncia / Conflito Atual | AÃƒÂ§ÃƒÂ£o de NormalizaÃƒÂ§ÃƒÂ£o |
|---|---|---|---|---|---|
| 1 | **`persons`** | Identidade Civil | Cadastro de pessoa fÃƒÂ­sica (CPF, nome civil, nascimento, foto) | Possui `nickname` (que ÃƒÂ© de atleta) e `role` rÃƒÂ­gido | **Remover `nickname`**; flexibilizar `role` para atuaÃƒÂ§ÃƒÂµes contextuais |
| 2 | **`team_roster`** | Elencos | Atletas inscritos no elenco com apelido de jogo, nÃƒÂºmero e posiÃƒÂ§ÃƒÂµes | Nome da tabela confuso com `rosters` | **Isolar aqui com exclusividade: `nickname`, `jersey_number` e `positions`** |
| 3 | **`rosters`** | Elencos | CabeÃƒÂ§alho do elenco do time (base ou competiÃƒÂ§ÃƒÂ£o) | - | Coerente: vincula `team_id` e opcionalmente `competition_id` |
| 4 | **`teams`** | Estrutura ClubÃƒÂ­stica | Equipe esportiva mantida pelo clube | **Possui `organization_id`** | **Dropar `teams.organization_id`**; time pertence 100% a `institution_id` |
| 5 | **`institutions`** | Estrutura ClubÃƒÂ­stica | AgremiaÃƒÂ§ÃƒÂ£o / Clube (CNPJ, presidente, sede, cores) | - | Coerente: entidade de prÃƒÂ¡tica desportiva dona dos times |
| 6 | **`organizations`**| GovernanÃƒÂ§a | FederaÃƒÂ§ÃƒÂ£o / Liga esportiva que organiza torneios | - | Coerente: administra campeonatos e homologa filiaÃƒÂ§ÃƒÂµes |
| 7 | **`institution_affiliations`** | GovernanÃƒÂ§a | FiliaÃƒÂ§ÃƒÂ£o anual formal do clube na federaÃƒÂ§ÃƒÂ£o | - | Coerente: possui `season`, `status`, datas de homologaÃƒÂ§ÃƒÂ£o |
| 8 | **`affiliation_windows`** | GovernanÃƒÂ§a | Janela temporal para abertura de pedidos de filiaÃƒÂ§ÃƒÂ£o | - | Coerente: governa prazos federativos |
| 9 | **`institution_organizations`** | GovernanÃƒÂ§a | VÃƒÂ­nculo N:N legado entre clube e federaÃƒÂ§ÃƒÂ£o | **Duplica `institution_affiliations`** sem auditoria ou status | **Dropar tabela `institution_organizations`** |
| 10 | **`competitions`** | CompetiÃƒÂ§ÃƒÂµes | Torneio esportivo gerido pela federaÃƒÂ§ÃƒÂ£o | - | Coerente: modalidade, gÃƒÂªnero, formato, temporadas |
| 11 | **`competition_enrollment_windows`** | CompetiÃƒÂ§ÃƒÂµes | Janela de inscriÃƒÂ§ÃƒÂ£o de equipes no torneio | - | Coerente: prazos de homologaÃƒÂ§ÃƒÂ£o de times |
| 12 | **`competition_team`** | CompetiÃƒÂ§ÃƒÂµes | InscriÃƒÂ§ÃƒÂ£o e chaveamento de equipe no torneio | Strings soltas de conferÃƒÂªncia e divisÃƒÂ£o | Substituir por FKs `conference_id` e `division_id` |
| 13 | **`conferences`** | CompetiÃƒÂ§ÃƒÂµes | DivisÃƒÂ£o de conferÃƒÂªncia no torneio (ex: Metropolis, Caipira) | - | Coerente: estrutura formal de agrupamento |
| 14 | **`divisions`** | CompetiÃƒÂ§ÃƒÂµes | SubdivisÃƒÂ£o da conferÃƒÂªncia no torneio | - | Coerente: agrupamento de times |
| 15 | **`rounds`** | CompetiÃƒÂ§ÃƒÂµes | Rodadas regulares e fases de playoffs | - | Coerente: sequenciamento de jogos |
| 16 | **`games`** | OperaÃƒÂ§ÃƒÂ£o de Jogo | Partida entre mandante e visitante | - | Coerente: status, placar, data/hora e local |
| 17 | **`venues`** | Estrutura / Jogos | PraÃƒÂ§a esportiva / Campo de jogo | **Restrito a `organization_id`**, sem suporte a CT de clube | Adicionar suporte a `institution_id` |
| 18 | **`checkins`** | OperaÃƒÂ§ÃƒÂ£o de Jogo | PresenÃƒÂ§a e escalaÃƒÂ§ÃƒÂ£o de atletas na partida | - | Coerente: `match_number` ÃƒÂ© override de camisa para o jogo |
| 19 | **`game_participants`**| OperaÃƒÂ§ÃƒÂ£o de Jogo | Equipe neutra de arbitragem e mesa da partida | - | Coerente: isola oficiais com `role` e `function` |
| 20 | **`plays`** | OperaÃƒÂ§ÃƒÂ£o de Jogo | Linha do tempo canÃƒÂ´nica de todos os lances | - | Coerente: fonte ÃƒÂºnica da verdade para estatÃƒÂ­sticas e placar |
| 21 | **`score_events`** | OperaÃƒÂ§ÃƒÂ£o de Jogo | Resumo legado de pontuaÃƒÂ§ÃƒÂµes | **Duplica lances de `plays`** | **Descontinuar e dropar `score_events`** |
| 22 | **`standings`** | ClassificaÃƒÂ§ÃƒÂ£o | Tabela consolidada de pontuaÃƒÂ§ÃƒÂ£o e aproveitamento | - | Coerente: terminologia corrigida para `points_for/against` |
| 23 | **`standings_rules`** | ClassificaÃƒÂ§ÃƒÂ£o | CritÃƒÂ©rios de desempate e ranking do torneio | - | Coerente: motor de cÃƒÂ¡lculo de classificaÃƒÂ§ÃƒÂ£o |
| 24 | **`playoff_configurations`** | ClassificaÃƒÂ§ÃƒÂ£o | Regras e chaveamento da fase final de playoffs | - | Coerente: estrutura eliminatÃƒÂ³ria |
| 25 | **`users`** | Acesso e SeguranÃƒÂ§a | Contas de login e credenciais na plataforma | - | Coerente: escopo via `organization_id` ou `institution_id` |

---

## 4. O Modelo de Elenco e Atleta Perfeito (Canonical Roster Architecture)

Para responder com precisÃƒÂ£o mÃƒÂ¡xima ao questionamento sobre os atributos do atleta:

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
    Roster "1" *-- "0..*" RosterEntry : "contÃƒÂ©m atletas do elenco"
    Person "1" <-- "0..*" MatchCheckIn : "registra presenÃƒÂ§a em jogo"
    RosterEntry ..> MatchCheckIn : "fornece nÃƒÂºmero padrÃƒÂ£o se override for nulo"
```

### Por que esta separaÃƒÂ§ÃƒÂ£o ÃƒÂ© perfeita:
1. **`Person` ÃƒÂ© universal:** Registra qualquer ser humano (atleta, tÃƒÂ©cnico, ÃƒÂ¡rbitro, dirigente) com seus dados civis protegidos.
2. **`RosterEntry` ÃƒÂ© o avatar esportivo do atleta:** Guarda o **apelido de jogo (`nickname`)**, o **nÃƒÂºmero oficial da camisa (`jerseyNumber`)** e as **posiÃƒÂ§ÃƒÂµes em campo (`positions`)** especÃƒÂ­ficos daquele time.
3. **`MatchCheckIn` ÃƒÂ© a sÃƒÂºmula em tempo real:** Herda o nÃƒÂºmero padrÃƒÂ£o do `RosterEntry`, permitindo `matchNumberOverride` caso no dia do jogo ocorra troca temporÃƒÂ¡ria de camisa.

---

## 5. Roteiro Fatorado de ExecuÃƒÂ§ÃƒÂ£o

```mermaid
timeline
    title Roteiro de NormalizaÃƒÂ§ÃƒÂ£o do Schema e DomÃƒÂ­nio
    Fase 1 : Limpeza Imediata de Identidade : Remover persons.nickname : Dropar teams.organization_id : Isolar nickname e jersey_number em team_roster
    Fase 2 : ExtinÃƒÂ§ÃƒÂ£o de Tabelas Duplicadas : Dropar institution_organizations (migrando filiaÃƒÂ§ÃƒÂµes ativas) : Dropar score_events (plays como fonte canÃƒÂ´nica)
    Fase 3 : Enriquecimento de Estrutura : Permitir venues vinculadas a institutions (CTs de Clubes) : FKs de conferences e divisions em competition_team
```

### ExecuÃƒÂ§ÃƒÂ£o ConcluÃƒÂ­da (Fase 1 Ã¢â‚¬â€ NormalizaÃƒÂ§ÃƒÂ£o CanÃƒÂ´nica):
1. **Migration Liquibase 020 (`020-normalize-schema-persons-teams.yaml`):**
   - Backfill idempotente: qualquer `persons.nickname` residual foi migrado para `team_roster.nickname`.
   - `ALTER TABLE persons DROP COLUMN nickname`.
   - `ALTER TABLE teams DROP CONSTRAINT fk_team_organization` e `DROP COLUMN organization_id`.
   - Aplicado e validado em staging contra o Oracle ADB 23ai.
2. **RefatoraÃƒÂ§ÃƒÂ£o no Backend Java (`flag_backend`):**
   - Removido `nickname` de `PersonEntity`, `PersonInfo`, `PersonResponse`, `CreatePersonRequest`, `UpdatePersonRequest` e `CreatePersonBatchItem`.
   - Removido `organizationId` e `organizationName` de `TeamEntity`, `TeamResponse`, `CreateTeamRequest` e `UpdateTeamRequest`.
   - `TeamRepository` saneado: removidas queries diretas por `organizationId`. A rota legada consulta atravÃƒÂ©s das agremiaÃƒÂ§ÃƒÂµes filiadas da organizaÃƒÂ§ÃƒÂ£o (`institutionOrgRepo`).
   - `RosterService` e `CheckInService` operam com `nickname`, `positions` e `jersey_number` exclusivamente em `RosterEntryEntity`.
3. **Alinhamento nos Clientes:**
   - `flag_admin_web`, `flag_public_app` e `flag_referee_app` atualizados e validados com `flutter analyze` reportando **0 erros e 0 avisos**.
   - Endpoint ao vivo validado: `GET https://api.america.app.br/api/v1/teams` retorna os times 100% associados ÃƒÂ  sua agremiaÃƒÂ§ÃƒÂ£o (`institutionId` / `institutionName`).

---

## 6. PrÃƒÂ³ximos Passos (Fase 2 e Fase 3)
- **Fase 2:** ExtinÃƒÂ§ÃƒÂ£o de tabelas duplicadas (`institution_organizations` unificada em `institution_affiliations`, e descontinuaÃƒÂ§ÃƒÂ£o de `score_events` em favor de `plays`).
- **Fase 3:** PraÃƒÂ§as esportivas (`venues`) com suporte a CTs de clubes (`institution_id`), e estruturaÃƒÂ§ÃƒÂ£o de chaves estrangeiras em `competition_team` (`conference_id`, `division_id`).
