# VID-06: Prova de conceito de infraestrutura em Serverless GPU (Modal / RunPod / OCI)

> **Type:** ops  
> **Status:** resolved  
> **Effort:** extracao-video-jogadas  
> **Repo(s):** flag_vision_service, flag_platform_docs, flag_platform_infra  
> **Blocks:** —  
> **Blocked by:** VID-05  
> **Origin:** Implementação da Fase 0 (Pós-Jogo / Batch)  

---

## Contexto

Para processar vídeos de partidas gravadas de 2h consumindo streams do YouTube a 10 FPS / 720p HD com pico de até 4 vídeos simultâneos, foi realizada a modelagem financeira e técnica de três abordagens de infraestrutura:
1. **Serverless GPU sob demanda (Modal.com e RunPod)**.
2. **VM com GPU dedicada na OCI mantida ligada 24/7**.
3. **VM com GPU na OCI acionada e desligada via script/SDK sob demanda**.

---

## Critérios de Aceite Atendidos

1. **Documento de Análise FinOps Publicado:**
   - Criação de [`architecture/analise-custos-infra-gpu-serverless.md`](../../architecture/analise-custos-infra-gpu-serverless.md).
   - Cálculo do tempo de inferência real para 72.000 frames (8 a 10 minutos por partida de 2h em GPU NVIDIA T4/L4).
   - Simulação de custos mensais para cenários de 10, 40 e 100 jogos/mês.
2. **Script de Deploy Serverless Implementado:**
   - Criação de [`modal_app.py`](../../../flag_vision_service/modal_app.py) no repositório `flag_vision_service`.
   - Suporte a GPU NVIDIA T4 com `scaledown_window=60` (auto-shutdown após 60 segundos ocioso com custo zero).
   - Limite de concorrência configurado para 4 instâncias paralelas (`concurrency_limit=4`).
3. **Decisão Consolidada:**
   - **Modal.com** aprovado como infraestrutura padrão da Fase 0 pelo custo aproximado de **R$ 0,45 por jogo completo de 2h** (menos de R$ 20,00/mês para 40 jogos), eliminando custos fixos ociosos de segunda a sexta-feira.
