# Levantamento de Arquitetura e Produto: App Fantasy Game (Flag Football)

> **Documento de Especificação Técnica e de Produto**  
> **Referência:** Ecossistema Flag Platform (`flag_public_app`, `flag_referee_app`, `flag_backend`)  
> **Data:** Outubro de 2026  
> **Status:** Proposta de Arquitetura & Roadmap

---

## 1. Visão Geral e Oportunidade

A Flag Platform já possui a espinha dorsal de dados de campeonatos, equipes, atletas e registro de lances de jogo em tempo real (via `flag_referee_app` e `flag_public_app`).

O **App Fantasy Game** surge como a maior alavanca de **engajamento, retenção e monetização** da comunidade de Flag Football, transformando o torcedor e o atleta de espectadores passivos em gestores de seus próprios times fictícios, acompanhando cada rodada lance a lance.

```
┌────────────────────────────────────────────────────────────────────────┐
│                         ECOSSISTEMA FLAG PLATFORM                      │
│                                                                        │
│   [flag_referee_app]          [flag_backend]        [flag_public_app]  │
│    Operação de Jogo  ───►    PostgreSQL + CQS   ◄─── Acompanhamento    │
│    (Lances / Scout)         (Regras & Dados)         (Torcedor/Atleta) │
│                                    │                                   │
│                                    ▼                                   │
│                        ┌───────────────────────┐                       │
│                        │   flag_fantasy_app    │                       │
│                        │  - Escalação Tática   │                       │
│                        │  - Mercado de Atletas │                       │
│                        │  - Ligas Privadas     │                       │
│                        │  - Parciais ao Vivo   │                       │
│                        └───────────────────────┘                       │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Diagnóstico: O Que Já Temos Pronto (`flag_public_app` & Backend)

Podemos reaproveitar aproximadamente **60% a 70%** das fundações já desenvolvidas:

| Componente | Situação Atual | Aproveitamento no Fantasy |
| :--- | :--- | :--- |
| **Design System Kickster** | Maduro (`KicksterCard`, `KicksterButton`, `KicksterBadge`, `KicksterAvatar`, `KicksterScoreCard`, `KicksterStatComparison`, `AppColors`, etc.) | **100% de reuso**. Mantém a identidade visual consistente da plataforma. |
| **Padrão de Camadas Flutter** | MVVM + Repositórios com Cache/Revalidação (`stale-while-revalidate`) + Services REST com Dio | **100% de reuso** da estrutura arquitetural e de injeção. |
| **Modelos de Domínio Base** | `Athlete`, `AthletePosition` (QB, WR, RB, C, DL, LB, DB, etc.), `Team`, `Competition`, `Round`, `Game` | **90% de reuso**. Já possuem IDs, fotos, histórico e vínculos. |
| **Captação de Lances em Campo** | `flag_referee_app` grava em tempo real via `PlayEntity` com tipo de jogada (`PlayType`), atleta executor (`playerId`), jardas e pontuação. | **Pronto na origem**. A entrada de dados que alimenta o scout já está operacional. |
| **Estratégia de Banco e Real-Time** | PostgreSQL como primário + Firestore como espelho CQS leve (ADR-002) | **Arquitetura ideal** para suportar a alta concorrência de parciais ao vivo sem gargalo no banco relacional. |

---

## 3. O Que Precisamos Construir (Gap Analysis)

### 3.1. Frontend (`flag_fantasy_app`)

Diferente do `flag_public_app`, que é primariamente de leitura e dispensa autenticação, o Fantasy Game é essencialmente **centrado na conta e na interação do usuário**.

#### 1. Autenticação e Perfil do Usuário
- **Login e Onboarding:** Integração com Firebase Auth (Google Sign-In, Apple Sign-In, Email/Senha).
- **Criação do Clube Fantasy:** Nome da franquia, escolha de escudo/emblema, cores principais e apelido do técnico.
- **Perfil & Histórico:** Registro de patrimônio acumulado, pontuação geral, histórico rodada a rodada e galeria de troféus/medalhas.

#### 2. Campo Tático / Escalação ("The Pitch")
- **Layout Gráfico Interativo:** Prancheta visual representando o campo de Flag Football (5x5 ou 7x7).
- **Estrutura de Formação Sugerida (Flag 5x5):**
  - **Ataque:** 1 QB (Quarterback), 1 Center/Snapper, 2 WR/RB.
  - **Defesa:** 1 Rusher/Blitzer, 2 DB/LB.
  - **Flex:** 1 Atleta livre (qualquer posição).
  - **Capitão:** 1 Atleta titular selecionado para pontuar multiplicado (ex: 1.5x).
  - **Banco de Reservas:** 2 Atletas de posições-chave para substituição automática caso um titular não entre em campo.
- **Restrições de Escalação:**
  - Limite orçamentário (ex: C$ 100,00 cartoletas / flag coins no início).
  - Limite de atletas do mesmo time real (ex: máximo de 2 ou 3 atletas da mesma agremiação por escalação).
  - Validação de prazo: mercado aberto vs fechado.

#### 3. Mercado de Atletas (Transfer Market)
- **Catálogo de Jogadores:** Filtros rápidos por posição (QB, WR, Defensor, etc.), time real, faixa de preço, média de pontos e status de lesão/disponibilidade.
- **Card Colecionável do Atleta:**
  - Foto oficial, número da camisa e agremiação.
  - Preço atual e variação na última rodada (▲ C$ +1.20 / ▼ C$ -0.80).
  - Mínimo de pontos para valorizar na rodada seguinte.
  - Scouts detalhados dos últimos jogos (TDs, jardas, interceptações, sacks).
  - Dificuldade do próximo adversário (ex: matchup favorável vs matchup difícil).
- **Regulamento de Fechamento do Mercado:** Bloqueio automático de alterações X minutos antes do primeiro kickoff da rodada.

#### 4. Ligas e Competições Sociais
- **Liga Nacional / Geral:** Ranking global automático reunindo todos os usuários do torneio.
- **Ligas Privadas:**
  - Criação rápida com geração de código de convite ou link profundo (deep link).
  - Modelos de Liga:
    - **Liga Clássica:** Pontos corridos acumulados ao longo de todo o campeonato.
    - **Tiro Curto / Etapa:** Competição fechada válida exclusivamente para uma rodada específica.
    - **Head-to-Head (Confronto Direto):** Duelos semanais 1x1 entre os membros da liga no formato chaveamento/tabela.
- **Feed e Resenha:** Tabela de classificação com destaque para o "Mito da Rodada" e "Lanterna".

#### 5. Parciais ao Vivo (Live Match Center)
- **Tela de Acompanhamento ao Vivo:**
  - Visualização da escalação com pontuação atualizando em tempo real durante os jogos.
  - Indicadores de status do atleta no momento: *Em Campo*, *Aguardando Jogo*, *Jogo Finalizado*.
  - Feed detalhado de lances que geraram pontos para cada atleta escalado.
- **Duelo ao Vivo:** Comparador lado a lado com amigos da liga enquanto os jogos acontecem.

---

### 3.2. Regras de Pontuação para Flag Football (Scoring Engine)

Mapeamento direto a partir do `PlayType` e `PlayEntity` já existentes na plataforma:

#### Ações de Ataque
| Ação em Campo | `PlayType` Base | Pontuação Proposta |
| :--- | :--- | :--- |
| **Touchdown de Passe** (QB) | `PASS` + `isTouchdown = true` | **+4.0 pts** |
| **Touchdown Recebido** (WR/C) | `PASS` + `isTouchdown = true` | **+6.0 pts** |
| **Touchdown Corrido** (RB/QB) | `RUN` + `isTouchdown = true` | **+6.0 pts** |
| **Conversão Extra Point 1pt** | `EXTRA_POINT_1` | **+1.0 pt** |
| **Conversão Extra Point 2pts** | `EXTRA_POINT_2` | **+2.0 pts** |
| **Passe Completo** | `PASS` | **+0.2 pt** |
| **Recepção (PPR)** | `PASS` (Receiver) | **+0.5 pt** |
| **Jardas de Passe** | `yards` | **+1.0 pt a cada 25 jardas** |
| **Jardas Corridas / Recepção** | `yards` | **+1.0 pt a cada 10 jardas** |
| **First Down Conquistado** | `FIRST_DOWN` | **+0.5 pt** |
| **Passe Incompleto** | `INCOMPLETE_PASS` | **-0.2 pt** |
| **Interceptação Lançada** | `INTERCEPTION` (Contra QB) | **-3.0 pts** |
| **Sack Sofrido** | `SACK` (Contra QB) | **-1.0 pt** |
| **Fumble / Turnover** | `isTurnover = true` | **-2.0 pts** |

#### Ações de Defesa
| Ação em Campo | `PlayType` Base | Pontuação Proposta |
| :--- | :--- | :--- |
| **Flag Pull / Tackle** | `FLAG_PULL` / `TACKLE` | **+1.0 pt** |
| **Sack Realizado** | `SACK` | **+3.0 pts** |
| **Interceptação Conquistada** | `INTERCEPTION` | **+4.0 pts** |
| **Passe Desviado (Deflection)** | `DEFLECTION` | **+1.0 pt** |
| **Pick Six (Interceptação retornada para TD)** | `PICK_SIX` | **+6.0 pts** |
| **Pick Two (Retorno de conversão)** | `PICK_TWO` | **+2.0 pts** |
| **Safety Conquistado** | `SAFETY` | **+4.0 pts** |
| **Defesa sem sofrer pontos (Shutout)** | Agregação do jogo | **+5.0 pts** |

#### Disciplinar
| Ação em Campo | `PlayType` Base | Pontuação Proposta |
| :--- | :--- | :--- |
| **Falta / Penalidade Cometida** | `PENALTY` | **-1.0 pt** |

---

### 3.3. Backend (`flag_backend`): Novos Serviços & Banco de Dados

#### 1. Novo Módulo de Domínio: `br.com.flagplatform.fantasy`
Estrutura modular alinhada à arquitetura do backend:
- `fantasy.controller`: Endpoints REST para escalação, mercado, ligas e rankings.
- `fantasy.service`: `FantasyLineupService`, `FantasyScoringEngineService`, `FantasyMarketService`, `FantasyLeagueService`.
- `fantasy.entity` & `repository`: Novas entidades mapeadas via Liquibase.

#### 2. Tabelas do Banco de Dados (Schema Relacional PostgreSQL)
- `fantasy_users`: Associação da conta de usuário à franquia fantasy.
- `fantasy_teams`: Time fantasy por competição (nome, escudo, saldo de cartoletas, pontuação acumulada).
- `fantasy_lineups`: Escalação submetida para uma rodada específica (status: `OPEN`, `LOCKED`, `CALCULATED`).
- `fantasy_lineup_athletes`: Atletas escalados, posição ocupada, preço pago e pontuação calculada.
- `fantasy_leagues`: Ligas criadas (públicas, privadas, tipo clássico ou tiro curto).
- `fantasy_league_members`: Associação entre times fantasy e ligas.
- `fantasy_athlete_market`: Histórico de preços por rodada, variação e pontuação obtida.
- `fantasy_scoring_rules`: Pesos configuráveis de cada scout por torneio.

#### 3. Motor de Cálculo & Atualização (Event-Driven Pipeline)
```
[ flag_referee_app ]
        │
   (POST /plays)
        │
        ▼
[ PlayService (flag_backend) ]
        │
        ├─► Persiste PlayEntity no PostgreSQL
        │
        ├─► [ScoringEngineService]
        │     - Identifica Atleta(s) envolvidos
        │     - Aplica tabela de pontuação
        │     - Atualiza parciais do Atleta na rodada
        │
        └─► [Firestore Mirror (ADR-002)]
              - Publica atualização em /fantasy_live_scores/{roundId}/{athleteId}
              - Apps conectados escutam via Realtime Stream
```

---

## 4. Particularidades Críticas do Flag Football

1. **Atletas "Two-Way" (Ataque e Defesa):**
   - No futebol americano profissional (NFL), atletas são estritamente ofensivos ou defensivos. No Flag Football nacional/amador, é comum o mesmo atleta atuar como Wide Receiver no ataque e Defensive Back na defesa.
   - *Diretriz de Design:* Permitir que o atleta seja escalado na posição em que foi registrado no roster da rodada, ou pontue unicamente pela posição escalada pelo usuário (evitando distorções na pontuação).
2. **Múltiplos Jogos no Mesmo Dia (Etapas de Torneio):**
   - Circuitos regionais de Flag frequentemente concentram 2 a 3 jogos de uma mesma equipe em um único sábado ou domingo.
   - O Fantasy precisa permitir que a "Rodada Fantasy" seja correspondente à "Etapa do Circuito", somando ou tirando a média dos jogos daquele dia.
3. **Dependência da Qualidade da Mesa:**
   - O fantasy game exige que a mesa de arbitragem registre o número correto do atleta no `flag_referee_app`. A funcionalidade já existente de Check-in pré-jogo de atletas e roster oficial por partida é o alicerce essencial que viabiliza essa integridade.

---

## 5. Roteiro de Implementação Sugerido (Fases)

### Fase 1 — MVP: Escalação & Pontuação Estática
- Modelagem de dados no PostgreSQL (tabelas de times fantasy, escalação e mercado).
- Endpoint para listar atletas disponíveis com preços base.
- Tela de escalação tática e fechamento de mercado.
- Motor de cálculo pós-rodada (processamento em lote ao final dos jogos).
- Ranking geral de pontuação.

### Fase 2 — Social & Gamificação (Ligas e Mercado Dinâmico)
- Criação de ligas privadas entre amigos com link/código de convite.
- Algoritmo de valorização/desvalorização de atletas (economia interna de moedas).
- Ligas mata-mata e tiro curto.

### Fase 3 — Experiência ao Vivo (Live Parciais)
- Espelhamento de pontuação em tempo real via Firestore (ADR-002).
- Tela de acompanhamento lance a lance dos atletas escalados no app.
- Notificações push de pontuação (ex: "Seu capitão marcou um Touchdown! +9.0 pts").
