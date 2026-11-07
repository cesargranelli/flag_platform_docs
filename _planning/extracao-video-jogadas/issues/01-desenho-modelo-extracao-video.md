# VID-01: Desenho do modelo de extração de dados em vídeo para registro de jogadas

> **Type:** research  
> **Status:** open  
> **Effort:** extracao-video-jogadas  
> **Repo(s):** flag_platform_docs, flag_backend, flag_referee_app  
> **Blocks:** VID-02, VID-03, VID-05  
> **Blocked by:** —  
> **Origin:** Solicitação do usuário para registro e desenho do modelo de extração de vídeo  

---

## Contexto

A Flag Platform possui uma súmula eletrônica avançada e catálogo unificado de 38 tipos de jogadas, mas depende do apontamento manual de operadores de mesa ou árbitros. Para viabilizar a automatização ou apoio assistido por IA, faz-se necessário definir formalmente a arquitetura do modelo de extração de dados a partir de gravações em vídeo de partidas de Flag Football.

O desenho deve abranger os desafios físicos da modalidade (fitas na cintura, campos sem marcação profissional padronizada em todos os torneios, ângulos de câmera variáveis) e estabelecer as camadas do pipeline: ingestão, homografia espacial, rastreamento de atletas/bola, identificação de dorsais, reconhecimento de ações e correlação com as regras esportivas.

---

## Critérios de Aceite

1. **Documento Arquitetural Publicado:** Criação de [`architecture/modelo-extracao-dados-video-jogadas.md`](../../architecture/modelo-extracao-dados-video-jogadas.md) com o detalhamento de:
   - Casos de uso (Modo Batch pós-jogo e Modo Live Assist).
   - Pipeline de visão computacional em camadas (homografia, detecção/tracking, OCR/roster matching, reconhecimento de ações atômicas, motor de regras).
   - Comparativo de abordagens tecnológicas (Especializada modular vs. VLM Multimodal vs. Híbrida).
   - Esquema JSON de saída integrado às entidades do backend (`PlayResponse`).
   - Estratégia de validação com supervisão humana (*Human-in-the-Loop*).
2. **Registro no Tracker da Plataforma:**
   - Pasta `_planning/extracao-video-jogadas/` criada com `map.md` e tickets fatiados.
   - Tabela de esforços ativos de `_planning/README.md` atualizada com o esforço e prefixo `VID`.
3. **Alinhamento com Catálogo de Jogadas:** O modelo deve suportar os tipos fundamentais de jogadas do catálogo da plataforma (Touchdown, Passe Completo/Incompleto, Corrida, Flag Pull, Sack, Interceptação, Faltas).
