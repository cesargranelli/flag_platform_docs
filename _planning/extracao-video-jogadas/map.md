# Mapa — Extração de Dados em Vídeo para Registro de Jogadas (`extracao-video-jogadas`)

Esforço de pesquisa, arquitetura e implementação da solução de Visão Computacional e Inteligência Multimodal para detecção, rastreamento e extração automática de jogadas a partir de filmagens de partidas de Flag Football.

Especificação detalhada consolidada em [`modelo-extracao-dados-video-jogadas.md`](../../architecture/modelo-extracao-dados-video-jogadas.md).

> **Regra:** As issues deste mapa são trabalhadas em ordem de maturidade técnica e validação experimental. As detalhadas têm arquivo em `issues/`; as demais ficam na tabela até serem fatiadas.

---

## Contexto

- **Demanda:** A anotação em tempo real de jogadas durante partidas de Flag Football exige atenção contínua e operadores experientes na mesa de arbitragem. Ao mesmo tempo, partidas são filmadas ou transmitidas ao vivo, gerando um acervo rico em vídeo que pode ser explorado por IA.
- **Objetivo:** Criar um pipeline capaz de analisar vídeos de partidas, identificar campo, atletas, movimentações e momentos críticos (snap, passe, flag pull, touchdown), mapeando-os diretamente para o catálogo unificado de 38 jogadas da Flag Platform.
- **Modos de Operação:**
  1. *Pós-jogo (Batch)*: Geração da súmula e play-by-play a partir de gravações completas com interface de revisão ágil.
  2. *Live Assist*: Notificação quase em tempo real no `flag_referee_app` com sugestão pré-preenchida do último lance para aprovação com 1 toque.

---

## Ordem de Execução

1. **VID-01 — Desenho do modelo e pipeline de extração de jogadas em vídeo** (arquitetura, esquema de dados e definição técnica). Detalhada em [`issues/01-desenho-modelo-extracao-video.md`](issues/01-desenho-modelo-extracao-video.md).
2. **VID-02 — Coleta de dataset de referência e etiquetagem inicial** (vídeos com ângulos variados de Flag 5x5 e 8x8).
3. **VID-03 — PoC de calibração espacial de campo e homografia** (conversão de pixels do vídeo em jardas 2D do campo).
4. **VID-04 — PoC de detecção de atletas, uniformes e dorsais** (matching visual com os elencos da súmula).
5. **VID-05 — PoC de inferência de jogada com modelo multimodal** (clipe curto de jogada gerando JSON de `PlayResponse`).
6. **VID-06 — Interface de conferência e revisão humana** (player de vídeo com marcação de jogadas e confirmação rápida).

---

## Tabela de Issues

| ID | Tipo | Título | Critérios de Aceitação | Status |
|:---:|:---:|---|---|:---:|
| **VID-01** | research | Desenho do modelo de extração de dados em vídeo para registro de jogadas | Especificação técnica completa consolidada em `architecture/`; pipeline de visão e multimodal detalhado; modelo de dados alinhado ao backend; mapa do esforço criado em `_planning/`. | open |
| **VID-02** | task | Coleta e catalogação de dataset de referência de vídeos de Flag Football | Repositório de amostras de vídeo (táticas, laterais e transmissões ao vivo) com trechos de lances conhecidos mapeados para teste de modelos. | open |
| **VID-03** | research | PoC de detecção de campo e homografia espacial (pixel -> jarda) | Script/serviço que identifica linhas do campo e converte posições de jogadores em coordenadas de jardas reais (0 a 50/100 yd). | open |
| **VID-04** | research | PoC de clustering de cores de time e OCR de dorsais | Identificação automática do time em ataque/defesa por cor de uniforme e leitura dos números das camisas com cruzamento no roster da partida. | open |
| **VID-05** | feature | Pipeline multimodal de inferência de jogadas (Video-to-Play JSON) | Receber clipe de vídeo (ex.: 5 a 10s) e gerar payload compatível com `PlayResponse` (tipo de jogada, jardas, passador, recebedor, fita). | open |
| **VID-06** | design | Desenho da interface de conferência humana (Human-in-the-Loop) | Fluxo de validação com reprodução de vídeo sincronizada aos eventos do jogo no painel admin / referee app. | open |

---

## Fora de Escopo

- Substituição mandatória e 100% autônoma do árbitro sem supervisão humana na primeira fase.
- Transmissão com hardware dedicado proprietário (o sistema deve funcionar com vídeos padrão de smartphones ou câmeras convencionais).
