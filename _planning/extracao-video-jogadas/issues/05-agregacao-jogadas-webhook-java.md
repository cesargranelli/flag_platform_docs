# VID-05: Agregação de jogadas e disparo do Webhook para o backend Java

> **Type:** feature  
> **Status:** resolved  
> **Effort:** extracao-video-jogadas  
> **Repo(s):** flag_vision_service, flag_platform_docs, flag_backend  
> **Blocks:** VID-06  
> **Blocked by:** VID-04  
> **Origin:** Implementação da Fase 0 (Pós-Jogo / Batch)  

---

## Contexto

Com as camadas de ingestão direta de stream do YouTube ([`VID-02`](02-setup-microsservico-coletor-youtube.md)), inferência contínua do YOLOv8 + ByteTrack ([`VID-03`](03-pipeline-yolov8-bytetrack.md)) e calibração espacial de campo em jardas ([`VID-04`](04-homografia-campo-mapeamento-2d.md)), foi desenvolvida a camada final de agregação esportiva: transformar o fluxo ininterrupto de coordenadas métricas em **eventos esportivos discretos** (`PASS_COMPLETE`, `RUSH`, `TOUCHDOWN`, `SACK`) e transmiti-los via Webhook assíncrono para o backend Java persistir no Oracle ADB e espelhar no Firestore.

---

## Critérios de Aceite Atendidos

1. **Agregador Baseado em Máquina de Estados ([`play_aggregator.py`](../../../flag_vision_service/app/vision/play_aggregator.py)):**
   - Transições de estado: `PRE_SNAP_FORMATION` $\to$ `BALL_LIVE` $\to$ `RESOLVING`.
   - Congelamento e identificação precisa da Linha de Scrimmage (LOS) na formação estática pré-snap.
   - Detecção de snap por dispersão/aceleração de atletas (spread $> 6.0$ jardas).
   - Detecção de bola em voo e trajetórias parabólicas para discriminar passes de corridas terrestres.
   - Detecção de apito/bola morta por desaceleração e estabilização dos atletas.
2. **Classificação e Regras de Pontuação:**
   - Detecção de `TOUCHDOWN` quando o avanço cruza a jarda 50 (no Flag 5x5).
   - Detecção de `FIRST_DOWN` com meta móvel e renovação de descidas para 1ª descida ao cruzar o meio de campo.
   - Detecção de `SACK` quando a fita do QB é retirada atrás da linha de scrimmage.
3. **Disparo do Webhook Assíncrono com Retries ([`job_manager.py`](../../../flag_vision_service/app/queue/job_manager.py)):**
   - Payload JSON canônico (`BatchPlaysWebhookPayload`) contendo métricas de processamento (`ProcessingSummary`) e a lista cronológica ordenada de lances (`PlayDetail`).
   - Cliente HTTP assíncrono com política de até 3 tentativas de reenvio em caso de falha de rede ou timeout.
4. **Testes Automatizados:**
   - Suíte de testes unitários ([`tests/test_play_aggregator.py`](../../../flag_vision_service/tests/test_play_aggregator.py)) cobrindo o ciclo de vida completo de um passe completo com conquista de 1ª descida e um touchdown.
