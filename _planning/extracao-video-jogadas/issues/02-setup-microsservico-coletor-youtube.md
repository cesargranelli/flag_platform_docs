# VID-02: Setup do microsserviço FastAPI e ingestão de stream do YouTube

> **Type:** task  
> **Status:** resolved  
> **Effort:** extracao-video-jogadas  
> **Repo(s):** flag_vision_service, flag_platform_docs  
> **Blocks:** VID-03, VID-04, VID-05  
> **Blocked by:** VID-01  
> **Origin:** Implementação da Fase 0 (Pós-Jogo / Batch)  

---

## Contexto

Para viabilizar a extração de dados e táticas a partir de vídeos gravados de partidas de Flag Football disponíveis no YouTube, foi criado o microsserviço desacoplado `flag_vision_service` em Python/FastAPI.

O serviço opera consumindo diretamente o stream de vídeo da URL informada sem a necessidade de baixar previamente arquivos de 2h (~2GB a 6GB) para o disco da máquina. Além disso, gerencia uma fila assíncrona com semáforo com capacidade para até 4 vídeos simultâneos, reduz a amostragem temporal para 10 FPS e padroniza a resolução em 720p HD.

---

## Critérios de Aceite Atendidos

1. **Repositório e Ambiente:**
   - Criação do repositório [`flag_vision_service`](../../../flag_vision_service).
   - Gerenciamento de dependências via `uv` com Python 3.11 (`fastapi`, `uvicorn`, `yt-dlp`, `opencv-python-headless`, `pydantic-settings`, `httpx`).
2. **Módulo Coletor de Vídeo:**
   - [`app/collector/youtube_stream.py`](../../../flag_vision_service/app/collector/youtube_stream.py) implementado com `yt-dlp` e `cv2.VideoCapture`.
   - Amostragem decrescida para **10 FPS** com timestamps exatos em milissegundos.
   - Redimensionamento automático mantendo o teto de 720p HD (1280x720).
3. **Fila Assíncrona:**
   - [`app/queue/job_manager.py`](../../../flag_vision_service/app/queue/job_manager.py) com `asyncio.Semaphore(4)`.
   - Rastreamento de progresso de jobs e cliente de disparo de Webhook com retries para o backend Java.
4. **Endpoints FastAPI & Testes:**
   - `POST /api/v1/jobs/process-video` (aceitação 202 com posicionamento na fila).
   - `GET /api/v1/jobs/{id}/status` e `GET /api/v1/jobs/queue/stats`.
   - `POST /api/v1/collector/test-stream` e `GET /api/v1/collector/metadata`.
   - Suíte de testes unitários com pytest passando com 100% de sucesso.
