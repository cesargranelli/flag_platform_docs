# ADR-022: Separação entre Número de Camisa Oficial (Elenco) e Numeração de Partida (Check-in/Súmula)

**Status:** Aprovado  
**Data:** 2026-10-27  
**Autor:** Tech Lead (Flag Platform)  

---

## 1. Contexto

Durante as partidas oficiais de Flag Football, imprevistos operacionais são comuns: atletas podem perder suas camisas originais, sofrer rasgos durante o jogo, ou utilizar camisas reservas/emprestadas com numeração diferente daquela cadastrada no elenco homologado da equipe.

Surgiu a necessidade de garantir uma separação estrita de escopo:
1. Uma alteração de número realizada pela mesa/árbitro no dia do jogo **não pode** substituir permanentemente a camisa oficial do atleta no time.
2. A alteração via painel administrativo (`flag_admin_web`) deve alterar o número oficial do atleta no elenco da equipe.
3. A indicação no dia do jogo via aplicativo de arbitragem (`flag_referee_app`) deve alterar o número apenas para aquela partida específica.

---

## 2. Decisão Arquitetural

Adota-se uma separação estrita de escopo entre **Número Oficial de Elenco** e **Numeração da Partida (Override)**:

### 2.1 Número Oficial de Elenco (`team_roster.jersey_number`)
- **Tabela**: `platform.team_roster` (entidade `RosterEntryEntity`).
- **Escopo**: Permanente para a equipe (elenco-base ou inscrição de campeonato).
- **Origem da Alteração**: Painel Administrativo (`flag_admin_web`), tela de Elenco (`roster_screen.dart`), acionando `PUT /api/v1/teams/{teamId}/roster/{athleteId}`.
- **Autorização**: Apenas administradores e gestores autorizados do time.

### 2.2 Numeração do Dia do Jogo / Override de Partida (`checkins.match_number`)
- **Tabela**: `platform.checkins` (entidade `CheckInEntity`).
- **Escopo**: Estritamente delimitado pela tupla `(game_id, person_id)`.
- **Origem da Alteração**: Aplicativo da Mesa/Árbitro (`flag_referee_app`), diálogo `edit_match_number_dialog.dart`, acionando `PUT /api/v1/games/{gameId}/checkin/{athleteId}/match-number`.
- **Regras de Negócio**:
  - `match_number` não nulo sobrepõe a camisa oficial apenas no contexto visual e nas jogadas (`plays`) daquele jogo.
  - O envio de valor nulo/em branco restaura automaticamente a exibição da camisa oficial.
  - Bloqueio de duplicidade: o backend rejeita com erro `409 DuplicateMatchNumberException` se dois atletas do mesmo time tentarem usar o mesmo número na mesma partida.
  - **Jamais altera ou sobrescreve** o registro em `team_roster`.

---

## 3. Modelo nos Aplicativos Clientes

### 3.1 Resolução de Número Efetivo (`CheckIn`)
Tanto no `flag_referee_app` quanto no `flag_public_app`, a resolução de qual número exibir e registrar em jogadas segue a precedência:

```dart
bool get hasOverride => matchNumber != null;
int? get effectiveNumber => matchNumber ?? athleteNumber ?? number;
```

### 3.2 Feedback Visual de Override (`AthleteCheckInCard`)
- **Sem override (`hasOverride == false`)**: Badge de camisa azul oficial Kickster (`AppColors.primary`).
- **Com override (`hasOverride == true`)**: Badge de camisa destacada em cor de aviso/âmbar (`AppColors.warning`), evidenciando à mesa que o atleta está atuando sob número excepcional.

### 3.3 Contrato de API (`CheckInResponse`)
O backend expõe na resposta de check-in (`GET /api/v1/games/{id}/checkin`):
- `athleteNumber` / `number`: Número oficial cadastrado no elenco.
- `matchNumber`: Número atribuído especificamente para esta partida (ou `null` caso padrão).
- `athleteNickname` e `position`: Metadados oficiais do atleta no elenco.

---

## 4. Consequências

- **Integridade Cadastral**: Nenhum imprevisto operacional de jogo compromete a base de dados oficial e os históricos de temporadas.
- **Rastreabilidade**: É possível auditar em qual jogo um atleta usou um número irregular sem perder a identidade de seu número oficial.
- **Clareza de Interface**: A mesa de arbitragem e os torcedores visualizam com transparência quando um jogador está jogando com numeração temporária.
