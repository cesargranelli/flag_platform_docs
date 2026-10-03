# Spec: Configuracao de Playoffs

> **Status:** rascunho
> **Escopo:** Backend (flag_backend), Admin Web, Referee App, App Publico

---

## 1. Contexto

Hoje o `TournamentFormat` aceita `PLAYOFFS` e `GROUPS_AND_PLAYOFFS`, e o `RoundType` possui
`PLAYOFFS`, `WILDCARD`, `SEMIFINAL` e `FINAL`. No entanto, **nao ha como configurar** como a
chave de playoffs e montada: quantos times se classificam por grupo/conferencia/divisao, qual o
tipo de eliminatoria (simple/double), se ha wildcards, etc.

## 2. Objetivo

Permitir que o organizador configure os playoffs **antes de publicar o campeonato** (status
`DRAFT`), com validacoes e congelamento das regras apos a publicacao.

## 3. Escopo

### 3.1 Backend (flag_backend)

- Nova entidade `PlayoffConfiguration` (1:1 com `Competition`)
- Novo endpoint CRUD `/api/v1/competitions/{id}/playoffs`
- Geracao automatica de jogos de playoffs a partir da classificacao (endpoint
  `/api/v1/competitions/{id}/playoffs/generate`)
- Critérios de desempate configuraveis separadamente para playoffs
- Restricao: configuracao so editavel em `DRAFT`

### 3.2 Admin Web

- Tela de configuracao de playoffs (integracao com o formulario de criacao de campeonato)
- Visualizacao da chave gerada
- Botao "Gerar Chave"

### 3.3 Referee App

- Exibicao de rodadas de playoffs na tela de jogo (ja existe via `RoundType`)
- Nenhuma alteracao de fluxo necessaria

### 3.4 App Publico

- Exibicao dos jogos de playoffs na tabela/rodadas
- Destaque visual para fases eliminatorias