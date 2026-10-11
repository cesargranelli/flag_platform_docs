# PUB-04: Calendário de jogos não carrega quando algum jogo tem `scheduledAt` nulo

> **Type:** bug
> **Status:** resolved
> **Effort:** repos/public_app
> **Repo(s):** flag_public_app, flag_backend
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-08

## Sintoma

O **calendário de jogos** do app público não carrega os jogos da competição **Flag 8x8 Masculino**
(os demais campeonatos carregam).

## Causa

`GET /api/v1/competitions/{id}/games` da 8x8 retorna **4 jogos com `scheduledAt: null`** (os jogos de
**playoffs gerados** pelo motor, criados com `scheduledAt`/`venueId` nulos). No app:

```dart
scheduledAt: DateTime.parse(json['scheduledAt'] as String).toLocal(),
```

o cast `as String` de `null` lança `TypeError` → `HttpGameRepository.listByCompetition` falha → e o
`findSchedule` (que **isola falhas por competição** com `catchError`) **descarta a lista inteira**
daquela competição → calendário vazio.

## Evidência

```
Flag 5x5 Feminino - Temporada Regular: jogos=68 | scheduledAt nulos=0
Flag 5x5 Masculino - Acesso:           jogos=12 | scheduledAt nulos=0
Flag 5x5 Masculino - Temporada Regular: jogos=90 | scheduledAt nulos=0
Flag 8x8 Masculino:                    jogos=47 | scheduledAt nulos=4
```

## Correção proposta

- **App**: `Game.fromJson` deve **tolerar `scheduledAt` nulo** (ex.: `DateTime?` e tratar na UI como
  "data a definir"), e o `findSchedule` não deve derrubar a lista inteira por um item inválido
  (parse **por item**, isolando o item ruim).
- **Backend (avaliar)**: jogos de playoffs gerados **sem data** — decidir se o motor deve exigir
  agendamento (ou preencher uma data padrão) para não propagar `null`.

## Critérios de aceite

- [ ] Calendário da 8x8 carrega (jogos sem data aparecem como "data a definir", sem quebrar a lista).
- [ ] Um jogo inválido não zera a lista da competição.
