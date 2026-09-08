# Arquitetura do Ecossistema de Partidas: Mesa do Delegado & App Público

Este documento expande o módulo de competições integrando os dois aplicativos operacionais:
1. **App do Delegado (Mesa / Súmula Eletrônica & Gestão de Partida)**
2. **App do Público / Torcedor (Live Scores, Confrontos, Classificação & Estatísticas)**

---

## 1. Visão Geral dos 3 Componentes do Ecossistema

```mermaid
flowchart TD
    subgraph BACKOFFICE["1. Web Admin (Federação / Organizador & Clubes)"]
        A1[Cadastro de Competição & Tabelamento] --> A2[Inscrição & Homologação de Times/Elencos]
        A2 --> A3[Agenda de Confrontos & Escalação de Delegados]
    end

    subgraph DELEGATE_APP["2. App do Delegado (Mesa / Operacional de Jogo)"]
        D1[Pré-Jogo: Check-in de Atletas & Validação Documental]
        D2[Início de Partida: Cronômetro & Períodos/Quarters]
        D3[Ao Vivo: Registro de Lances, Pontuações, Faltas e Eventos]
        D4[Revisão / Bloqueio: Conferência de Súmula & Ajuste de Placar]
        D5[Assinatura / Encerramento Oficial da Partida]
        D1 --> D2 --> D3 --> D4 --> D5
    end

    subgraph FAN_APP["3. App do Público / Torcedor (Kickster Live)"]
        F1[Calendário de Jogos & Notificações]
        F2[Play-by-Play em Tempo Real / Narrador de Lances]
        F3[Placar Ao Vivo, Estatísticas & Destaques]
        F4[Tabela de Classificação Atualizada Instantaneamente]
    end

    A3 -.->|Partida Agendada| D1
    D3 ==>|Websockets / Server-Sent Events| F2
    D3 ==>|Atualização de Placar| F3
    D5 ==>|Consolidação de Resultados| F4
```

---

## 2. Ciclo de Vida da Partida (Máquina de Estados)

A partida transita por estados bem definidos para garantir integridade esportiva e auditoria:

```mermaid
stateDiagram-v2
    [*] --> SCHEDULED: Criada no Tabelamento
    SCHEDULED --> PRE_MATCH_CHECKIN: Delegado abre para Check-in
    
    state PRE_MATCH_CHECKIN {
        [*] --> ValidandoDocumentos
        ValidandoDocumentos --> EscalacaoConfirmada
    }
    
    PRE_MATCH_CHECKIN --> LIVE: Delegado inicia o cronômetro
    
    state LIVE {
        [*] --> PeriodoAtivo: Registro de Lances (Touchdown, Gols, Faltas)
        PeriodoAtivo --> Intervalo: Fim de Quarto / Tempo
        Intervalo --> PeriodoAtivo: Reinício
    }
    
    LIVE --> IN_REVIEW: Bloqueio para Conferência Final
    
    state IN_REVIEW {
        [*] --> AuditoriaDeLances: Conferência de súmula
        AuditoriaDeLances --> AjustePlacar: Retificação de eventos/pontos não computados
        AjustePlacar --> ValidacaoArbitragem: De acordo de árbitros/capitães
    }
    
    IN_REVIEW --> FINISHED: Encerramento Oficial & Assinatura
    IN_REVIEW --> LIVE: Desbloqueio emergencial (se retificado ainda em jogo)
    FINISHED --> [*]
```

### Detalhamento dos Estados:
1. **`SCHEDULED`**: Jogo agendado com data, hora, local e times.
2. **`PRE_MATCH_CHECKIN`**: Aberto pelo delegado ~1h antes do jogo. Lista os atletas do Roster oficial inscrito para validação presencial (foto, RG/documento, número da camisa).
3. **`LIVE`**: Partida em andamento. Eventos transmitidos em broadcast em tempo real para os torcedores.
4. **`IN_REVIEW` (Bloqueio para Conferência)**: O cronômetro parou. O sistema trava a edição pública e coloca o jogo em modo "Em Revisão/Aguardando Súmula Oficial". O delegado confere todas as faltas, pontos, cartões e ajusta qualquer inconsistência.
5. **`FINISHED`**: Partida encerrada em definitivo. Dispara o recálculo imediato da tabela de classificação (`platform.standings`).

---

## 3. Modelo de Dados para Operação & Eventos

O banco de dados já possui fundação pronta (`platform.games`, `platform.checkins`, `platform.plays`, `platform.score_events`):

```mermaid
erDiagram
    GAMES ||--o{ CHECKINS : "possui presença confirmada"
    GAMES ||--o{ PLAYS : "possui lances registrados"
    GAMES ||--o{ GAME_AUDIT_LOG : "registra ajustes do delegado"
    
    ATHLETES ||--o{ CHECKINS : "é checado no jogo"
    TEAMS ||--o{ CHECKINS : "equipe do atleta"
    TEAMS ||--o{ PLAYS : "equipe autora da jogada"

    GAMES {
        uuid id PK
        string status "SCHEDULED, PRE_MATCH_CHECKIN, LIVE, IN_REVIEW, FINISHED"
        int home_score
        int away_score
        uuid delegate_id FK "delegado responsável"
        timestamp match_started_at
        timestamp match_ended_at
        timestamp review_started_at
    }

    CHECKINS {
        uuid id PK
        uuid game_id FK
        uuid team_id FK
        uuid athlete_id FK
        string status "PRESENT, ABSENT, SUSPENDED, INJURED"
        int jersey_number "número na partida"
        uuid validated_by FK "delegado"
        timestamp validated_at
    }

    PLAYS {
        uuid id PK
        uuid game_id FK
        uuid team_id FK
        string player_name
        string receiver_name
        string play_type "TOUCHDOWN, FIELD_GOAL, SAFETY, FOUL, INTERCEPTION, YELLOW_CARD"
        string description "Texto narrativo do lance"
        int yards
        string quarter "1Q, 2Q, HT, 3Q, 4Q, OT"
        string game_time "ex: 02:45"
        boolean is_score
        int points_added
        boolean is_verified "auditado na conferência"
    }

    GAME_AUDIT_LOG {
        uuid id PK
        uuid game_id FK
        uuid modified_by FK
        string action "SCORE_ADJUSTMENT, EVENT_CORRECTION"
        string details "ex: Corrigido Field Goal de 3 pts para time visitante"
        timestamp created_at
    }
```

---

## 4. Jornada do Delegado (App da Partida)

| Etapa | Ação na Tela | Impacto no Sistema |
| :--- | :--- | :--- |
| **1. Abertura do Jogo** | Clica em **"Abrir Check-in de Atletas"** | `status -> PRE_MATCH_CHECKIN`. Libera a lista de presença. |
| **2. Check-in** | Marca os atletas presentes, confere documento e numeração da camisa | Registra registros em `platform.checkins` com foto/validação. |
| **3. Iniciar Jogo** | Clica em **"Iniciar 1º Quarto/Tempo"** | `status -> LIVE`. O torcedor recebe notificação push de jogo iniciado. |
| **4. Durante o Jogo** | Botões rápidos Kickster: `+ Ponto`, `Falta`, `Substituição`, `Tempo Técnico` | Alimenta `platform.plays` e sobe eventos via WebSocket. |
| **5. Fim do Tempo Regulamentar** | Clica em **"Bloquear para Conferência"** | `status -> IN_REVIEW`. Tela entra em modo de auditoria. |
| **6. Conferência & Retificação** | Lista de eventos da partida: Delegado pode editar pontuação, alterar autor do lance ou adicionar evento esquecido | Registra log de auditoria e atualiza `home_score` e `away_score`. |
| **7. Finalização Oficial** | Assinatura digital do delegado e clique em **"Encerrar Partida"** | `status -> FINISHED`. Gera súmula PDF e consolida a tabela de classificação. |

---

## 5. Experiência do Torcedor (App Público Kickster)

- **Aba "Jogos / Ao Vivo":**
  - Cards de partidas com badge animado `LIVE` e cronômetro sincronizado.
  - Placar atualizado sem necessidade de recarregar a página/tela (via WebSocket/SSE).
- **Tela de Detalhes da Partida (Match Center):**
  - **Feed de Lances (Play-by-Play):** Linha do tempo visual com ícones dos eventos (bola, cartão, apito, pontuação).
  - **Escalações:** Visualização dos atletas confirmados no check-in pelo delegado.
  - **Estatísticas:** Comparativo de rendimento entre as equipes.
- **Aba "Classificação" (Standings):**
  - Tabela atualizada automaticamente assim que o delegado encerra a partida no status `FINISHED`.

---

> [!TIP]
> A separação entre o estado **`LIVE`**, o estado de auditoria **`IN_REVIEW`** e o estado definitivo **`FINISHED`** protege o sistema contra reclamações e inconsistências de súmula, garantindo que o placar final oficial seja homologado com tranquilidade antes do fechamento das estatísticas da competição.
