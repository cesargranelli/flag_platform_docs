# PLY-02: Admin Web — configuração de fases e card "Chave de Playoffs" no detalhe

> **Type:** feature
> **Status:** open
> **Effort:** playoffs-configuracao
> **Repo(s):** flag_admin_web
> **Blocks:** —
> **Blocked by:** PLY-01
> **Origin:** [spec-fases-e-geracao.md](../spec-fases-e-geracao.md) · [ADR-021](../../../adr/ADR-021-geracao-fase-a-fase-playoffs.md)

## Objetivo

Expor no `flag_admin_web` a configuração das **fases** dos playoffs e um comando manual de
**geração da chave** na **tela de detalhe da competição** (hoje só existe em Criar/Editar).

## Escopo

### 1. Configuração das fases (Create/Edit)
- Ao habilitar playoffs, calcular a **árvore** a partir dos classificados (por grupo + wildcards) —
  potência de 2 com byes.
- Permitir **escolher a fase** e **preencher os confrontos** escolhendo as posições/vencedores
  (`STANDING`, `WINNER`, `BEST_OVERALL`, `LOSER`), no padrão Kickster.
- Definir em que fase os **cabeças-de-chave** entram.

### 2. Card "Chave de Playoffs" no detalhe (`competition_detail_screen`)
- Visível para **criador** ou **ADMIN**.
- Mostra o estado da fase atual (gerada/concluída) e a ação **"Gerar próxima fase"**.
- Chama `POST /competitions/{id}/playoffs/generate`; trata 409 (fase não concluída / jogo iniciado).
- Link para ver a chave (`.../playoffs/bracket`).

## Critérios de aceite

- [ ] Configurar a lista de fases e os confrontos de cada fase.
- [ ] Card no detalhe com a ação de gerar (criador/ADMIN), a qualquer status.
- [ ] Feedback de sucesso/erro (409 etc.) no padrão da tela (SnackBar + card de resultado).
- [ ] `flutter analyze` com **0 erros e 0 avisos**.
- [ ] Padrão de tela conforme `AGENTS.md` (Create/Edit separados; design system Kickster).

## Notas

- Seguir a skill `flutter-architecture` (MVVM) e o ADR-011.
- Reutilizar `CompetitionPlayoffViewModel`/`CompetitionPlayoffSection` quando fizer sentido.
