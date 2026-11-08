# VID-04: Calibração de campo e mapeamento homográfico (pixel -> jarda)

> **Type:** research  
> **Status:** resolved  
> **Effort:** extracao-video-jogadas  
> **Repo(s):** flag_vision_service, flag_platform_docs  
> **Blocks:** VID-05  
> **Blocked by:** VID-03  
> **Origin:** Implementação da Fase 0 (Pós-Jogo / Batch)  

---

## Contexto

Para que os dados táticos extraídos por visão computacional possam ser registrados no catálogo oficial da Flag Platform, é indispensável traduzir a posição visual dos atletas na tela (coordenadas $(u, v)$ em pixels) em termos esportivos oficiais: **posição no campo em jardas (0 a 50 yd)**, ganho líquido de jardas da jogada, linha de scrimmage e avanço de primeira descida.

Como as partidas de Flag Football são gravadas sob ângulos oblíquos (câmeras laterais ou elevadas na linha de 50 jardas), desenvolveu-se a calibração por **Homografia 2D Projetiva**, mapeando o plano trapezoidal da câmera para o plano métrico retangular do campo.

---

## Critérios de Aceite Atendidos

1. **Modelagem de Dimensões Oficiais por Modalidade:**
   - Suporte para **Flag Football 5x5** (50 jardas de campo + 25 jardas de largura).
   - Suporte para **Flag Football 8x8** (80 jardas de campo + 40 jardas de largura).
   - Suporte para **Futebol Americano 11x11** (100 jardas de campo + 53.3 jardas de largura).
2. **Módulo Calibrador ([`field_calibrator.py`](../../../flag_vision_service/app/vision/field_calibrator.py)):**
   - Cálculo da matriz de homografia $H$ via `cv2.findHomography` e sua inversa $H^{-1}$.
   - Projeção de pontos de contato dos pés (`feet_x`, `feet_y`) para coordenadas métricas de campo (`FieldCoordinates`: `yard_line`, `lateral_yard`, `is_in_bounds`, `is_in_endzone`).
   - Resolução robusta contra imprecisões de ponto flutuante com tolerância epsilon.
3. **Cálculos Esportivos Integrados:**
   - `estimate_line_of_scrimmage`: Estima a linha de saída antes do snap a partir do alinhamento dos atletas.
   - `calculate_play_yards`: Determina ganho positivo ou perda de jardas (ex: sacks e tackles atrás da linha).
   - `check_first_down_conquered`: Avalia se o avanço cruzou a linha de primeira descida (ex: meio de campo).
   - `check_touchdown`: Avalia se o portador da bola cruzou a linha de gol adversária (jarda 50 no 5x5).
4. **Integração com Rastreamento e Trajetórias:**
   - [`yolo_tracker.py`](../../../flag_vision_service/app/vision/yolo_tracker.py) projeta em tempo real a jarda de cada atleta detectado.
   - [`trajectory_manager.py`](../../../flag_vision_service/app/vision/trajectory_manager.py) calcula a velocidade em jardas/segundo e o deslocamento total em jardas.
5. **Testes Automatizados:**
   - Suíte de testes unitários ([`tests/test_field_calibrator.py`](../../../flag_vision_service/tests/test_field_calibrator.py)) cobrindo precisão analítica de projeção, detecção de LOS e cálculos de jardas.
