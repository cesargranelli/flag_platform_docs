# Mapa — Extração de Dados em Vídeo para Registro de Jogadas (`extracao-video-jogadas`)

Esforço de arquitetura, pesquisa e implementação da solução de Visão Computacional para detecção, rastreamento e extração automática de jogadas a partir de filmagens de partidas de Flag Football gravadas ou transmitidas no YouTube.

Especificação detalhada consolidada em [`modelo-extracao-dados-video-jogadas.md`](../../architecture/modelo-extracao-dados-video-jogadas.md).

> **Regra:** As issues deste mapa são trabalhadas em ordem de maturidade técnica e validação experimental. As detalhadas têm arquivo em `issues/`; as demais ficam na tabela até serem fatiadas.

---

## Contexto da Arquitetura

- **Backend Java (OCI):** Gerencia regras de negócio, agenda as análises e recebe o resultado via Webhook.
- **Microsserviço de Visão (Python FastAPI):** Processa o stream de vídeo do YouTube sob demanda (sem download de arquivos completos em disco) no repositório [`flag_vision_service`](../../../flag_vision_service).
- **Engine Visual:** Ultralytics YOLOv8 (Nano/Small) com rastreador ByteTrack ou BoT-SORT nativo.
- **Espelhamento em Tempo Real:** O backend Java persiste no Oracle ADB e espelha no Firestore, alimentando os aplicativos móveis Flutter (`flag_referee_app` e `flag_public_app`) reativamente.
- **Otimização Operacional:** Amostragem travada em 10 FPS e resolução em 720p HD; capacidade de gerenciar fila com pico de até 4 vídeos simultâneos.
- **Infraestrutura:** Avaliação entre Serverless GPU (Modal.com ou RunPod) e instância com GPU dedicada na OCI sob demanda.

---

## Ordem de Execução (Fase 0 - Pós-Jogo / Batch)

1. **VID-01 — Desenho da arquitetura, contratos de API e modelo de dados** (consolidado em `architecture/`). Detalhada em [`issues/01-desenho-modelo-extracao-video.md`](issues/01-desenho-modelo-extracao-video.md).
2. **VID-02 — Setup do microsserviço Python (`flag_vision_service`)** com FastAPI e pipeline de captura em stream (`yt-dlp` + `OpenCV`). Detalhada em [`issues/02-setup-microsservico-coletor-youtube.md`](issues/02-setup-microsservico-coletor-youtube.md).
3. **VID-03 — Integração com YOLOv8 e ByteTrack para detecção e tracking** com amostragem a 10 FPS / 720p.
4. **VID-04 — Implementação da Homografia de Campo (2D Mapping)** para conversão métrica de pixels em jardas reais.
5. **VID-05 — Implementação do Webhook de retorno e agregação de lances para o Backend Java**.
6. **VID-06 — Benchmarking de infraestrutura de execução (Serverless GPU vs. OCI GPU sob demanda)**.

---

## Tabela de Issues

| ID | Tipo | Título | Critérios de Aceitação | Status |
|:---:|:---:|---|---|:---:|
| **VID-01** | research | Desenho da arquitetura e contratos do microsserviço de visão | Especificação técnica completa consolidada; contratos HTTP (disparo + webhook) validados; topologia de integração definida. | resolved |
| **VID-02** | task | Setup do microsserviço FastAPI e ingestão de stream do YouTube | Aplicação FastAPI com endpoint `/process-video`; captura direta da URL do YouTube com `yt-dlp` + `OpenCV` em 720p sem salvar no disco; fila assíncrona para até 4 vídeos. | resolved |
| **VID-03** | task | Pipeline YOLOv8 Nano/Small + ByteTrack para rastreamento de atletas e bola | Executar inferência a 10 FPS com IDs consistentes ao longo do lance; métricas de FPS e uso de memória documentadas. | open |
| **VID-04** | research | Calibração de campo e mapeamento homográfico (pixel -> jarda) | Detecção de linhas demarcatórias e projeção métrica das posições dos atletas e da linha de scrimmage em jardas. | open |
| **VID-05** | feature | Agregação de jogadas e disparo do Webhook para o backend Java | Transformar trajetórias e ações atômicas em eventos compatíveis com `PlayResponse` e disparar POST no callback da partida. | open |
| **VID-06** | ops | Prova de conceito de infraestrutura em Serverless GPU (Modal / RunPod / OCI) | Testar custos e tempos de inferência de uma partida completa de 2h nas opções de GPU sob demanda. | open |

---

## Fora de Escopo

- Substituição mandatória e 100% autônoma do árbitro sem supervisão humana na Fase 0.
- Download de arquivos brutos de vídeo em disco local na VM Java.
- Hardware proprietário para gravação em campo (o modelo utiliza filmagens convencionais de YouTube).
