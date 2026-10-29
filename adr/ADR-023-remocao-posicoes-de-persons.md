# ADR-023: Remoção do Campo Positions de Persons (Posições Pertencem ao Elenco)

**Status:** Aprovado  
**Data:** 2026-10-28  
**Autor:** Tech Lead (Flag Platform)  

---

## 1. Contexto

Na revisão dos fluxos e no [ADR-021](file:///C:/Projetos/America/flag_platform_docs/adr/ADR-021-athlete-positions-and-roster-management.md), havia sido introduzida provisoriamente uma coluna `positions` na tabela `persons` (perfil global da pessoa física).

Entretanto, uma análise aprofundada de modelagem de domínio evidenciou que:
1. `Person` representa uma **Pessoa Física** (entidade cadastral e biográfica com nome, CPF, data de nascimento, gênero, foto, email).
2. Posições esportivas (`QB`, `WR`, `CB`, `S`, etc.) **não são atributos biográficos ou imutáveis de um indivíduo**.
3. Uma pessoa pode atuar como Quarterback em uma equipe de flag 5x5, como Cornerback em uma equipe de flag 7x7, ou até mesmo desempenhar papéis de comissão técnica ou arbitragem em outras competições.
4. Manter `positions` na tabela `persons` viola a separação de responsabilidades (SRP), cria duplicidade com o elenco e gera inconsistências quando um atleta atua em equipes ou modalidades distintas.

---

## 2. Decisão Arquitetural

1. **Remoção Total de `positions` de `persons`**:
   - A coluna `positions` foi removida da tabela `platform.persons` via migração Liquibase (`016-drop-positions-from-persons.yaml`).
   - Todos os DTOs (`CreatePersonRequest`, `UpdatePersonRequest`, `PersonResponse`, `CreatePersonBatchItem`), entidades e mappers de `Person` deixam de referenciar `positions`.
   - As telas de cadastro e edição de pessoas físicas (`PersonCreateScreen` e `PersonEditScreen`) não exibem seletores de posição.

2. **Posições Pertencem Estritamente ao Vínculo de Elenco (`team_roster`)**:
   - A definição de posições é realizada única e exclusivamente no momento em que a pessoa é vinculada a um time (`team_roster` / `RosterEntryEntity`).
   - Na tela de gestão de elenco (`roster_screen.dart`), ao adicionar ou editar um atleta, o gestor define suas posições de jogo específicas para aquele time/campeonato.

---

## 3. Consequências

- **Modelo de Domínio Limpo**: `Person` foca estritamente em identidade pessoal, sem misturar funções desportivas contextuais.
- **Flexibilidade Multi-equipe**: O mesmo indivíduo pode possuir inscrições com posições diferentes em times ou modalidades distintas sem conflito cadastral.
- **Consistência**: Alinhamento com a mesma regra já aplicada ao número da camisa (`jersey_number`), que também reside exclusivamente em `team_roster`.
