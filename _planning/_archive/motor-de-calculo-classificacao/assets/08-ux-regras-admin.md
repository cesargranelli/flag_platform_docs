# Protótipo — UX das telas de Regras de Classificação (flag_admin_web)

Artefato do ticket *UX das telas de regras no admin (Design System Kickster)*. Rascunho para reagir,
não implementação. Base: padrão de seção de `competition_grouping_section.dart` (KicksterSectionTitle +
Card `surface`/borda `line` + `SelectableCard`/`KicksterInput`/`KicksterButton`).

---

## Tela 1 — Criação/Edição da Competição (DRAFT) — nova seção "Regras de Classificação"

> Visível apenas em DRAFT (as regras congelam ao publicar — ticket 07). Vive no fluxo create/edit,
> respeitando a separação create vs edit (ADR do projeto).

```
┌─ ⚙  Regras de Classificação ──────────────────────────────────────────────────────┐
│                                                                                   │
│  Modelo de classificação                                                          │
│  ┌───────────────┐ ┌───────────────┐                                              │
│  │ % de Vitórias │ │ Pontos (V-E-D)│   ← SelectableCard (mutuamente exclusivos)    │
│  │  (W-L-T/PCT)  │ │   3-1-0       │                                              │
│  └───────────────┘ └───────────────┘                                              │
│                                                                                   │
│  ── Pontuação (habilitada só no modo "Pontos") ─────────────────────────────────  │
│  [ Pontos/Vitória: 3 ]  [ Pontos/Empate: 1 ]  [ Pontos/Derrota: 0 ]   ← KicksterInput │
│                                                                                   │
│  ── W.O. (vitória por ausência) ────────────────────────────────────────────────  │
│  [ Placar Pró: 49 ]   [ Placar Contra: 0 ]        ← KicksterInput                 │
│  ( • ) Jogos de W.O. contam nos critérios de desempate   ← KicksterSwitch/SelectableChip │
│                                                                                   │
│  ── Escopo da classificação ────────────────────────────────────────────────────  │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐                                     │
│  │ Por Grupo  │ │   Geral    │ │   Ambos    │   ← SelectableCard                 │
│  └────────────┘ └────────────┘ └────────────┘                                     │
│                                                                                   │
│  ── Critérios de desempate (ordem importa) ─────────────────────────────────────  │
│  1. [ Confronto Direto      ▾ ]  multi-way: [ Descartar ▾ ]   ⠿  ✕   ← KicksterDropdown │
│  2. [ Saldo de Pontos       ▾ ]                               ⠿  ✕                  │
│  3. [ Força de Vitória SOV  ▾ ]                               ⠿  ✕                  │
│  4. [ Sorteio               ▾ ]                               ⠿  ✕                  │
│  [ + Adicionar critério ]                                            (outline)     │
│                                                                                   │
└───────────────────────────────────────────────────────────────────────────────────┘
```

**Pontos a decidir (é o coração deste protótipo):**
- **Reordenação:** drag & drop (coluna ⠿ acima) **ou** lista numerada sem arrastar (posições fixas com
  `KicksterDropdown` em cada linha + botões ↑/↓)? Ver seção *Opções de reordenação* abaixo.
- Parâmetros por critério (ex.: `multiWay` do Confronto Direto) aparecem **inline** na linha, ou num
  menu de opções (KicksterMenuAnchor)?

---

## Tela 2 — Detalhe da Competição — leitura das regras + card ADMIN

```
┌─ ⚙  Regras de Classificação ──────────────────────────────────────────── [só leitura] ─┐
│  Modelo: % de Vitórias (W-L-T)        Escopo: Por Grupo                                │
│  W.O.: 49×0 · conta nos desempates                                                     │
│  Ordem dos desempates:                                                                 │
│   1. Confronto Direto (descartar com 3+)  2. Saldo  3. SOV  4. Sorteio                  │
└────────────────────────────────────────────────────────────────────────────────────────┘

┌─ 📊  Motor de Cálculo da Classificação ──────────────────────  [ADMIN ONLY] ───────────┐
│  Última atualização: 26/09/2026 18:30                                                  │
│  Status: [ EM DIA ]   (ou [ PENDENTE DE RECÁLCULO ])      ← KicksterStatusBadge        │
│                                                                                        │
│  [ ⟳ Disparar Motor de Cálculo ]   ← KicksterButton (primário), isAdminUser            │
│      → showKicksterConfirmDialog → POST .../standings/recalculate → 202 → "Processando" │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

- **Confirmado:** o admin **não** exibe a tabela de classificação — só as regras configuradas e o card
  de disparo. A tabela é do `flag_public_app`.
- A leitura das regras aparece para organizador dono e ADMIN; o card de disparo só para `isAdminUser`.

---

## Opções de reordenação (a decidir)

| | A) Drag & drop | B) Lista numerada (dropdown por posição) |
|---|---|---|
| Componente | `ReorderableListView` + handle ⠿ | N linhas com `KicksterDropdown` + botões ↑/↓ |
| Prós | Intuitivo, "arrasta e pronto" | Só componentes já padronizados; previsível |
| Contras | Mais código/estado de arraste | Menos natural; posições fixas |
| Aderência Kickster | ok (não proibido) | alto (usa o `KicksterDropdown` recomendado) |

## Critérios disponíveis (catálogo fechado — ticket 03)
`Confronto Direto` (com `multiWay`), `Saldo de Pontos`, `Pontos Pró`, `Menor Pontos Sofridos`,
`Aproveitamento no Grupo`, `Força de Vitória (SOV)`, `Força de Calendário (SOS)`, `Sorteio`.
