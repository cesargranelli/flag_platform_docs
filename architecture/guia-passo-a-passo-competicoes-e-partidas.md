# Guia de Implementação Passo a Passo (Roadmap Oficial)

Este guia estabelece as fases, tarefas e critérios de aceite para a construção do ecossistema de **Competições, Partidas (Delegado/Mesa) e App do Torcedor**.

---

## 🧭 Visão Geral dos Apps e Responsabilidades

| Componente | Repositório / Pasta | Papel Principal |
| :--- | :--- | :--- |
| **Painel Web Admin** | `flag_admin_web` | Gestão de Competições, Clubes/Agremiações, Inscrições e Tabela de Jogos |
| **Backend & APIs** | `flag_backend` | Regras de negócio, validações, persistência e WebSockets/SSE |
| **App do Delegado (Mesa)** | `flag_referee_app` | Check-in de atletas, cronômetro, registro de jogadas/lances ao vivo, conferência final e fechamento de súmula |
| **App Público (Torcedor)** | `flag_public_app` | Calendário, Match Center ao vivo, Play-by-play, placar em tempo real e classificação |

---

## 📌 FASE 1: Módulo de Competições no Admin Web (`flag_admin_web` & `flag_backend`)

### 1.1 Modelagem e Endpoints Backend
- [ ] Validar DTOs e serviços de `competitions` para suportar:
  - `name` (*nome_torneio*)
  - `modality` (*Futebol, Basquete, eSports, Flag Football, etc.*)
  - `gender` (*Masculino, Feminino, Misto*)
  - `age_group` (*Sub-20, Adulto, etc.*)
  - `grouping_type` / `format` (*Pontos Corridos, Mata-Mata, Grupos + Eliminatória*)
  - `season` (*Temporada/Ano*)
  - `status` (*DRAFT, REGISTRATION_OPEN, ONGOING, FINISHED*)
- [ ] CRUD completo de Competições com paginação e filtros.

### 1.2 Telas no Painel Admin Kickster (`flag_admin_web`)
- [ ] **Menu Lateral**: Adicionar item "Competições" com ícone de troféu.
- [ ] **Lista de Competições (`CompetitionListScreen`)**:
  - Cards no Design System Kickster com badges de status, modalidade, gênero e categoria.
  - Ações no dropdown/menu Kickster: *Editar*, *Gerenciar Inscrições*, *Tabelamento / Jogos*, *Excluir/Arquivar*.
- [ ] **Formulário de Competição (`CompetitionFormScreen`)**:
  - Seção 1: Identificação (Nome, Temporada, Descrição).
  - Seção 2: Classificação Esportiva (Modalidade, Categoria/Faixa Etária, Gênero).
  - Seção 3: Formato de Disputa & Chaveamento.
  - Seção 4: Período de Inscrição e Limite de Equipes.

---

## 📌 FASE 2: Inscrições & Homologação de Equipes (Desacoplamento)

### 2.1 Gestão de Equipes e Elencos (Clube/Agremiação)
- [ ] Cadastro e listagem de Atletas e Comissão Técnica da Agremiação.
- [ ] Cadastro do Time (`platform.teams`) vinculado à Agremiação.
- [ ] Definição do Elenco (`platform.roster` / `platform.team_roster`) associado à temporada/competição.

### 2.2 Central de Inscrições (`platform.competition_team`)
- [ ] Submissão da inscrição do Time + Elenco pelo Clube.
- [ ] Tela do Organizador para homologação das inscrições:
  - Status: *Pendente*, *Homologado*, *Rejeitado* (com justificativa).
  - Atribuição a Grupos/Conferências/Divisões se aplicável.

---

## 📌 FASE 3: Tabelamento & Agendamento de Confrontos

### 3.1 Geração e Gestão de Rodadas (`platform.rounds` & `platform.games`)
- [ ] Tela de Montagem de Tabela (Chaveamento ou Rodadas de Pontos Corridos).
- [ ] Seleção de Confronto: `Mandante` vs `Visitante`.
- [ ] Associação obrigatória de:
  - **Localidade / Praça Esportiva (`platform.venues`)**.
  - **Data e Horário de Início**.
  - **Designação do Delegado Responsável** (`delegate_id`).

---

## 📌 FASE 4: App do Delegado / Mesa de Jogo (`flag_referee_app`)

### 4.1 Pré-Jogo (Check-in de Atletas)
- [ ] Seleção da partida agendada designada ao delegado.
- [ ] Ação de abertura: transição para `PRE_MATCH_CHECKIN`.
- [ ] Lista de presença dos elencos homologados:
  - Validação presencial de identidade/documento e foto.
  - Confirmação do número da camisa de jogo (`jersey_number`).
  - Marcação de status do atleta (*Presente, Ausente, Suspenso, Lesionado*).

### 4.2 Jogo Ao Vivo (Live Match Tracker)
- [ ] Transição para `LIVE` ao iniciar cronômetro do 1º período.
- [ ] Painel de controle de mesa:
  - Cronômetro e controle de tempos/quartos/intervalos.
  - Botões rápidos Kickster de eventos de pontuação (*Gol, Touchdown, Field Goal, Ponto Extra, etc.*).
  - Registro de ocorrências disciplinares (*Faltas, Cartões, Advertências*).
  - Narrativa do lance (`description`) para envio ao feed do torcedor.
- [ ] Broadcast de eventos em tempo real para o backend.

### 4.3 Pós-Jogo & Bloqueio para Conferência Final
- [ ] Botão de encerramento do tempo regulamentar: transição para `IN_REVIEW`.
- [ ] **Modo Auditoria do Delegado**:
  - Edição/retificação de lances já computados.
  - Inserção manual de eventos ou pontos não registrados durante a correria do jogo.
  - Ajuste e validação final do placar (`home_score`, `away_score`).
- [ ] **Homologação Final**:
  - Assinatura digital/confirmação do delegado.
  - Transição para `FINISHED`.
  - Fechamento imutável da súmula e disparo do recálculo automático de classificação (`platform.standings`).

---

## 📌 FASE 5: App Público / Torcedor (`flag_public_app`)

### 5.1 Calendário & Jogos
- [ ] Listagem de partidas por data, rodada e competição.
- [ ] Card de jogo com indicador visual de status (*Agendado*, *Ao Vivo*, *Em Revisão*, *Finalizado*).

### 5.2 Match Center (Ao Vivo & Detalhes)
- [ ] Placar em tempo real atualizado instantaneamente via WebSockets/SSE.
- [ ] Feed Play-by-Play (Linha do tempo com ícones visuais Kickster de cada lance).
- [ ] Aba de Escalação com os atletas confirmados no check-in do delegado.
- [ ] Ficha técnica e resumo dos pontuadores.

### 5.3 Classificação & Estatísticas
- [ ] Tabela de classificação com pontos, vitórias, empates, derrotas, saldo e aproveitamento atualizada após cada partida homologada.
