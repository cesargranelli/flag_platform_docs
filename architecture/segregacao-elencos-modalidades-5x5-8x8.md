# Segregação de Elencos por Modalidade (Flag 5x5 vs Flag 8x8)

## 1. Contexto e Motivação
Na Flag Platform, agremiações e times disputam competições em modalidades distintas:
- **Flag 5x5**: Modalidade ágil e sem contato de linha, com elenco reduzido (idealmente 15 a 30 atletas por time).
- **Flag 8x8**: Modalidade com linha de bloqueio (OL/DL), exigindo elencos significativamente maiores (geralmente entre 50 e 80 atletas).

### O Problema Diagnosticado
Anteriormente, a tabela `rosters` possuía apenas vínculo com `team_id`, `competition_id` e `season`. Durante a carga e importação inicial dos elencos, atletas de agremiações tradicionais (como Lizards, Silver Knights e Unicamp Eucalyptus) foram associados em massa a todos os elencos do time sem distinção da modalidade de atuação.
Isso causava:
1. **Poluição de mesa no Referee App**: No check-in de partidas de Flag 5x5, a mesa e o árbitro visualizavam listas de 84 atletas para selecionar 15–20 presentes, a maioria dos quais são jogadores exclusivos de linha de 8x8.
2. **Inconsistência nos Apps**: O público e gestores viam atletas de 8x8 em elencos de 5x5 e vice-versa.
3. **Falta de granularidade no Modelo**: Elencos-base (`competition_id IS NULL`) não especificavam a modalidade esportiva correspondente.

---

## 2. Decisão Arquitetural e Modelagem

### 2.1. Alterações no Banco de Dados (Liquibase Migration 023)
Arquivo: `src/main/resources/db/changelog/changesets/023-roster-modality-and-clean-segregation.yaml`
1. **Adição da coluna `modality`** na tabela `rosters`:
   - `modality VARCHAR(20) DEFAULT 'FLAG_5X5' NOT NULL`
2. **Backfill automático**:
   - Para elencos atrelados a uma competição (`competition_id IS NOT NULL`), o campo `modality` herda diretamente de `competitions.modality`.
   - Para elencos-base (`competition_id IS NULL`), foi inferida a modalidade predominante com base nas competições disputadas pelo time na temporada.
3. **Saneamento e desduplicação de vínculos (`team_roster`)**:
   - Remoção de atletas espelhados indevidamente entre 5x5 e 8x8:
     - **Lizards**: 52 atletas removidos do 5x5 (exclusivos de 8x8) e 30 removidos do 8x8.
       - Elenco 5x5 final: 30 atletas.
       - Elenco 8x8 final: 52 atletas.
     - **Silver Knights**: 51 removidos do 5x5, 17 removidos do 8x8 e 16 mantidos em ambos.
       - Elenco 5x5 final: 33 atletas.
       - Elenco 8x8 final: 67 atletas.
     - **Unicamp Eucalyptus**: 39 removidos do 5x5, 16 removidos do 8x8 e 15 mantidos em ambos.
       - Elenco 5x5 final: 31 atletas.
       - Elenco 8x8 final: 54 atletas.

---

## 3. Impacto e Implementação no Backend (`flag_backend`)

### 3.1. Entidade e Repositório
- `RosterEntity`:
  - Campo `Modality modality` com `@Enumerated(EnumType.STRING)`.
  - Método `@PrePersist` / `inferDefaults()` que garante padrão `Modality.FLAG_5X5`.
- `RosterRepository`:
  - `Optional<RosterEntity> findByTeamIdAndCompetitionIdIsNullAndModality(UUID teamId, Modality modality)`
  - `List<RosterEntity> findAllByTeamIdAndCompetitionIdIsNull(UUID teamId)`

### 3.2. Serviço de Resolução de Elenco (`RosterService`)
- **Herança na criação**: Ao criar elenco para competição (`getOrCreateRoster`), a modalidade é herdada obrigatoriamente de `comp.modality()`.
- **Fallback estrito para Elenco-Base**:
  - `findRosterAthletesByTeamIdAndCompetitionId` e `findAthleteIdsByTeamIdAndCompetitionId` buscam o elenco da competição.
  - Caso não haja atletas vinculados, busca o elenco-base filtrando **exclusivamente pela modalidade da competição**:
    ```java
    rosterRepository.findByTeamIdAndCompetitionIdIsNullAndModality(teamId, compModality)
    ```
  - Isso impede que atletas de 8x8 apareçam como fallback em competições de 5x5.

### 3.3. Endpoint REST (`RosterApi` / `RosterController`)
- `GET /api/v1/teams/{teamId}/roster?modality=FLAG_5X5`: Permite consultar o elenco do time filtrado por modalidade.
- `RosterResponse` agora retorna o campo `modality` para consumo dos clientes.

---

## 4. Impacto e Implementação no Frontend (`flag_admin_web`)

### 4.1. Repositório e Service
- `RosterService.getRosterPage` e `RosterRepository.getRosterPage` recebem parâmetro opcional `String? modality` e repassam como query parameter `?modality=...`.

### 4.2. ViewModel (`RosterViewModel`)
- Adicionadas propriedades:
  - `String? get modalityFilter`
  - `void setModalityFilter(String? value)`
- `hasFilters` agora considera se o filtro de modalidade está ativo.

### 4.3. Interface Kickster (`RosterScreen`)
- Adicionado seletor com `KicksterDropdown<String?>` na barra de busca e filtros do elenco:
  - Todas as modalidades (nulo)
  - Flag 5x5 (`FLAG_5X5`)
  - Flag 8x8 (`FLAG_8X8`)
- Atualização em tempo real da listagem ao selecionar a modalidade.

---

## 5. Verificação e Conformidade
- Backend compilado com sucesso (`mvn clean compile -DskipTests`).
- `flag_admin_web`: `flutter analyze` com 0 erros e 0 avisos.
- `flag_referee_app`: `flutter analyze` com 0 erros e 0 avisos.
- `flag_public_app`: `flutter analyze` com 0 erros e 0 avisos.
