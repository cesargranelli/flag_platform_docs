# ADR-021: Modelagem de Posições e Gestão de Elenco de Atletas

**Status:** Aprovado  
**Data:** 2026-10-25  
**Autor:** Tech Lead (Flag Platform)  

---

## 1. Contexto

Durante a revisão dos fluxos de cadastros da plataforma, identificou-se um gap conceitual e técnico na atribuição de **posições** e **números de camisa** dos atletas:
1. No cadastro global de pessoas (`Person`), o formulário de criação no frontend (`PersonCreateScreen`) continha um seletor visual de posições, porém a tabela `persons` no banco de dados e os DTOs do backend descartavam essa informação silenciosamente. A tela de edição (`PersonEditScreen`) sequer possuía o campo.
2. Na tabela `team_roster`, as colunas `positions` e `jersey_number` já existiam, mas o DTO `AddRosterEntryRequest` não aceitava `positions`, e o modal de inclusão de atleta no elenco no frontend (`_RosterDetailsDialog`) solicitava apenas apelido e número de camisa.
3. Não havia endpoints de `PUT` nem interface para editar o número da camisa, apelido ou posições de um atleta já incluído no elenco sem removê-lo e recriá-lo.

---

## 2. Decisão Arquitetural (Abordagem A — Duplo Nível)

Adotou-se a separação de responsabilidades em dois níveis:

### 2.1 Perfil Global da Pessoa/Atleta (`persons` / `Person`)
- **Posições Primárias/Preferenciais**: Armazenadas na coluna `positions` (VARCHAR 255) da tabela `persons` como lista/string separada por vírgula (ex.: `"QB, WR"`). Representa as posições habituais do atleta como indivíduo esportivo (usado para cards trunfo, busca e figurinhas).
- **Número da Camisa**: **NÃO pertence à pessoa**, visto que o número varia por equipe, por competição ou por temporada.

### 2.2 Vínculo no Elenco do Time (`team_roster` / `RosterEntry`)
- **Número da Camisa (`jersey_number`)**: Definido no momento em que o atleta é vinculado a um elenco (elenco-base do time ou elenco de competição).
- **Posições no Elenco (`positions`)**: Posições específicas exercidas pelo atleta naquele time/competição. Ao adicionar um atleta ao elenco, o sistema pré-carrega as posições primárias do perfil da pessoa como sugestão inicial, permitindo que o gestor ajuste até 3 posições.
- **Apelido no Elenco (`nickname`)**: Apelido pelo qual o atleta é conhecido naquele time/campeonato.

---

## 3. Implementação

### 3.1 Banco de Dados (Liquibase)
- Changeset `015-add-positions-to-persons.yaml` adicionando a coluna `positions VARCHAR(255)` na tabela `persons`.
- Registrado em `db.changelog-master.yaml` e `db.changelog-runtime.yaml`.

### 3.2 Backend (`flag_backend`)
- **`PersonEntity`**: Adicionada propriedade `positions`.
- **`CreatePersonRequest` / `UpdatePersonRequest`**: Suporte ao campo `List<String> positions`.
- **`PersonResponse`**: Retorna `List<String> positions`.
- **`PersonMapper`**: Converte `List<String>` para `String` e vice-versa.
- **`AddRosterEntryRequest`**: Adicionado campo `String positions`.
- **`UpdateRosterEntryRequest`**: Novo DTO para atualização de vínculo no elenco (`status`, `nickname`, `positions`, `number`).
- **`RosterBatchItem`**: Adicionados campos opcionais `nickname`, `positions` e `number`.
- **Endpoints de Edição**:
  - `PUT /api/v1/teams/{teamId}/roster/{athleteId}`: Atualiza atleta no elenco-base.
  - `PUT /api/v1/teams/{teamId}/competitions/{competitionId}/roster/{athleteId}`: Atualiza atleta no elenco da competição.

### 3.3 Frontend Admin (`flag_admin_web`)
- **`Person`**: `fromJson` resiliente aceitando tanto `List<dynamic>` quanto `String` separada por vírgulas.
- **`PersonEditViewModel` & `PersonEditScreen`**:
  - Inclusão do seletor de posições (`SelectableChip`, máx. 3) quando o perfil for `ATHLETE`.
  - Correção de conformidade com o Design System Kickster (substituição do `showDatePicker` pelo `showAppCalendarDialog`).
- **`RosterService` & `RosterRepository`**:
  - Métodos `add` (com posições) e `update` (com posições, número, apelido, status).
- **`RosterViewModel`**:
  - Adicionados `updatePerson` e reatividade de carregamento/mutação.
- **`RosterScreen`**:
  - `_RosterDetailsDialog`: Suporte a título dinâmico, edição, número de camisa e seletor de posições (`SelectableChip`).
  - Cards de elenco (`PersonTrunfoCard`): Botão de **Editar dados no elenco** ao lado do botão de **Remover**, além de abertura ao tocar na figurinha.

---

## 4. Consequências

- **Consistência de Domínio**: Fim do descarte silencioso de dados; o atleta possui suas posições salvas tanto no perfil pessoal quanto no contexto do time.
- **Experiência do Usuário (UX)**: Gestores de time agora podem editar camisas e posições diretamente na tela de elenco sem precisar excluir e readicionar atletas.
- **Compatibilidade**: Totalmente retrocompatível com dados e endpoints legados.
