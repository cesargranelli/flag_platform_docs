# Mapa — Pendências e Melhorias do `flag_referee_app`

> **Esforço:** `repos/referee_app` · **Prefixo:** `REF` · **Status:** done
> **Repo afetado:** [`cesargranelli/flag_referee_app`](https://github.com/cesargranelli/flag_referee_app), [`cesargranelli/flag_backend`](https://github.com/cesargranelli/flag_backend)
> **Origem:** levantamento operacional em campo e alinhamento de produto em 2026-10-11.

Planejamento e tracking estruturado dos ajustes de ergonomia, correção de bugs (check-in, performance, scroll), novas regras de jogo (downs, kickoff, faltas duplas, turnovers on downs) e suporte a rollback/estorno no Referee App.

## Issues

| ID | Tipo | Título | Repo(s) | Status |
|----|------|--------|---------|--------|
| [REF-01](issues/REF-01-escudo-avatar-atletas-remover-container-icones.md) | design | Substituição de foto por escudo nos avatares & desencapsulamento de ícones | flag_referee_app | done |
| [REF-02](issues/REF-02-catalogo-icones-semanticos-jogadas.md) | design | Catálogo e revisão semântica de ícones específicos para jogadas | flag_referee_app | done |
| [REF-03](issues/REF-03-priorizacao-jogos-home-referee.md) | feature | Priorização e ordenação estrita dos confrontos na Home por esteira operacional | flag_referee_app | done |
| [REF-04](issues/REF-04-checkin-otimista-scroll-numero-elenco.md) | bug / perf | Check-in otimista imediato, resolução de loop de scroll, número cadastrado e filtro de elenco | flag_referee_app, flag_backend | done |
| [REF-05](issues/REF-05-controle-downs-turnover-auto-reset.md) | regra | Gestão sequencial de descidas (downs), turnover on downs automático e auto-reset | flag_referee_app | done |
| [REF-06](issues/REF-06-rollback-jogadas-backend-e-painel.md) | feature | Rollback confiável de lances no backend (estorno de pontos e score events) e botão no painel | flag_referee_app, flag_backend | done |
| [REF-07](issues/REF-07-novos-tipos-jogadas-catalogo-regras.md) | regra / api | Novos tipos de jogada: Kickoff/Retorno, FG Errados/Bloqueados, Toss, Faltas Duplas, Ejeção | flag_referee_app, flag_backend | done |
| [REF-08](issues/REF-08-registro-tempo-periodos-revisao-ordenacao.md) | bug / feature | Registro de tempo, períodos (Início/Fim), revisão e estabilização da ordenação | flag_referee_app, flag_backend | done |

## Notas do esforço
- Documento de especificação e arquitetura técnica: [plano_ajustes_referee_app.md](plano_ajustes_referee_app.md).
- O prefixo `REF` cobre o ciclo operacional de campo da arbitragem e mesa.
