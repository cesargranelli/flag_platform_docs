# VID-03: Pipeline YOLOv8 Nano/Small + ByteTrack para rastreamento de atletas e bola

> **Type:** task  
> **Status:** resolved  
> **Effort:** extracao-video-jogadas  
> **Repo(s):** flag_vision_service, flag_platform_docs  
> **Blocks:** VID-04, VID-05  
> **Blocked by:** VID-02  
> **Origin:** Implementação da Fase 0 (Pós-Jogo / Batch)  

---

## Contexto

Com o coletor de stream do YouTube e a fila assíncrona estabelecidos em [`VID-02`](02-setup-microsservico-coletor-youtube.md), o próximo passo da Fase 0 foi integrar o motor de inferência visual para detecção e rastreamento multi-objeto de alta performance a 10 FPS e 720p HD.

O sistema precisa identificar continuamente os atletas (`person` / classe 0) e a bola (`sports ball` / classe 32), atribuindo IDs persistentes frame a frame com o algoritmo **ByteTrack** para que as trajetórias espaciais (deslocamento, velocidade e posição de contato no solo) possam ser medidas com precisão métrica.

---

## Critérios de Aceite Atendidos

1. **Dependências e Engine Visual:**
   - Adicionado `ultralytics` (YOLOv8 + ByteTrack / BoT-SORT nativos) e `torch` ao [`flag_vision_service`](../../../flag_vision_service).
   - Suporte aos modelos leves `yolov8n.pt` (Nano) e `yolov8s.pt` (Small) configuráveis via `settings.YOLO_MODEL_NAME`.
2. **Wrapper de Rastreamento ([`yolo_tracker.py`](../../../flag_vision_service/app/vision/yolo_tracker.py)):**
   - Execução de `model.track(..., persist=True, tracker='bytetrack.yaml', classes=[0, 32])`.
   - Extração do ponto métrico inferior `feet_x` e `feet_y` (ponto de contato do atleta com o solo do campo).
   - Detecção e coordenadas centrais da bola de futebol americano.
3. **Gerenciador de Trajetórias ([`trajectory_manager.py`](../../../flag_vision_service/app/vision/trajectory_manager.py)):**
   - Acúmulo temporal de posições por `track_id`.
   - Cálculo automático de velocidade instantânea ($px/s$) e deslocamento espacial cumulativo.
4. **Integração com a Fila e Endpoint de Benchmark:**
   - Inserido no ciclo de vida de processamento do [`job_manager.py`](../../../flag_vision_service/app/queue/job_manager.py).
   - Novo endpoint de benchmark `POST /api/v1/collector/test-tracking` para testar consumo de stream, inferência do YOLOv8 e taxa de FPS em tempo real.
5. **Testes Automatizados:**
   - Suíte de testes unitários ([`tests/test_vision.py`](../../../flag_vision_service/tests/test_vision.py)) cobrindo parsing de detecções, cálculo cinemático de trajetórias e tratamento de exceções.
