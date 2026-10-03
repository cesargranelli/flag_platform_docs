# ADR-017: Configuracao de Playoffs

> **Status:** aceito
> **Data:** 2026-09-30
> **Decisores:** tech-lead

---

## Contexto

O modulo de competicoes ja suporta torneios com playoffs (`TournamentFormat.PLAYOFFS`,
`GROUPS_AND_PLAYOFFS`) e rodadas do tipo `WILDCARD`, `SEMIFINAL`, `FINAL`. No entanto, nao ha
como configurar **como** a chave de playoffs e montada: quantos times se classificam por
grupo/conferencia/divisao, qual o tipo de eliminatoria, se ha wildcards, etc.

## Decisao

### 1. Entidade separada 1:1 (`PlayoffConfiguration`)

Seguindo o padrao estabelecido por `CompetitionStandingsRulesEntity` (ADR-003), a configuracao
de playoffs sera uma entidade independente com 1:1 com `Competition`. Nao acrescentamos colunas
no `CompetitionEntity` para manter a entidade de cabecalho enxuta.

**Campos principais:**
- `enabled` (boolean) — se os playoffs estao ativos
- `qualified_per_group` (int) — quantos times por grupo/conferencia/divisao se classificam
- `wildcard_slots` (int) — quantos wildcards (melhores fora dos classificados por grupo)
- `format` (enum: SINGLE_ELIMINATION, DOUBLE_ELIMINATION, ROUND_ROBIN_KNOCKOUT)
- `tiebreaker_criteria_json` (JSON) — criterios de desempate especificos para playoffs
- `auto_generate` (boolean) — se a geracao de jogos e automatica ou manual

### 2. Geracao de jogos via endpoint dedicado

O endpoint `POST /api/v1/competitions/{id}/playoffs/generate` executa o motor de geracao a
partir da classificacao ja calculada (ou recalcula). O admin pode chamar multipleas vezes ate
confirmar a chave. Apos a confirmacao, as rodadas e jogos sao criados de forma irreversivel
(mas ainda em DRAFT, ate o campeonato ser publicado).

### 3. Criterios de desempate separados

Os criterios de desempate para playoffs sao armazenados em `PlayoffConfiguration.tiebreaker_criteria_json`,
permitindo configuracao independente da fase regular (que vive em
`CompetitionStandingsRulesEntity`).

### 4. Resticao de edicao

A configuracao de playoffs so e editavel quando o campeonato esta em status `DRAFT`. Apos a
publicacao, as regras congelam (mesmo padrao das regras de classificacao).

## Consequencias

### Positivos

- Configuracao explicita e visivel na API
- Separação clara entre fase regular e playoffs
- Geracao automatica reduz erro humano na montagem da chave
- Consistencia com padrao existente de entidades 1:1

### Negativos

- Nova entidade implica nova migration Liquibase
- Novo endpoint aumenta a superficie da API
- Motor de geracao de jogos de playoffs e um acrescento de complexidade

## Alternativas consideradas

1. **Colunas direto no CompetitionEntity** — rejeitado: entidade de cabecalho ja e pesada
   (grouping_config JSON, tournament_format, etc.) e seguir o padrao de entidades auxiliares
   e mais limpo.

2. **Geracao automatica ao publicar** — rejeitado: o admin precisa revisar a chave antes de
   ela ficar visivel para jogadores e torcedores.

3. **Sem configuracao de wildcards** — rejeitado: wildcards sao comuns em torneios reais e
   o app publico precisa exibir essas informacoes.