# Troubleshooting: Carregamento de Jogos Agendados no Flag Referee App

**Data:** 31 de Outubro de 2026  
**Status:** Resolvido e Publicado  
**Escopo:** `flag_referee_app` / Mesa de Arbitragem  

---

## 1. Contexto e Sintoma
Ao acessar a tela principal de confrontos (`RefereeHomeScreen`) do aplicativo da mesa de arbitragem (`flag_referee_app`), os jogos agendados (`SCHEDULED`) deixaram de aparecer ou a tela apresentava estado de erro/carregamento infinito.

---

## 2. Diagnóstico e Causa Raiz

### 2.1. Exceção de Tipo em `scheduledAt` Nulo (Causa Primária)
No backend da plataforma, o campo `scheduled_at` da entidade `Game` é anulável (`@Column(name = "scheduled_at") private LocalDateTime scheduledAt;`). Confrontos gerados automaticamente por cruzamento de playoffs ou recém-criados nascem sem data/hora até a homologação da mesa diretora.

No entanto, no frontend do `flag_referee_app`:
- Em `lib/domain/models/game.dart`:
  ```dart
  final DateTime scheduledAt; // Não-nulo
  ...
  scheduledAt: DateTime.parse(json['scheduledAt'] as String), // Lançava TypeError se nulo
  ```
- Em `lib/data/api/models/live_game_response.dart`:
  ```dart
  final DateTime scheduledAt; // Não-nulo
  ...
  scheduledAt: DateTime.parse(json['scheduledAt'] as String), // Lançava TypeError se nulo
  ```

Quando qualquer rodada continha ao menos um jogo com data não definida (por exemplo, as semifinais de Flag 8x8 Masculino recém-geradas pelo chaveamento), o parser falhava com `TypeError: type 'Null' is not a subtype of type 'String'`, estourando erro no Riverpod Provider.

### 2.2. Sobrecarga em Cascata na Home View (Gargalo de Performance)
Na visualização padrão de todas as rodadas (`_AllRoundsGamesView`), o widget executava:
```dart
for (final round in rounds) {
  final asyncVal = ref.watch(gamesByRoundProvider(round.id));
  ...
}
```
Em campeonatos com muitas rodadas (como o Flag 5x5 com 13 rodadas), eram disparadas **13 requisições HTTP simultâneas**. Se uma única requisição falhasse devido ao erro de deserialização de data nula, a flag `anyError = true` era ativada e toda a lista de jogos ficava indisponível.

---

## 3. Correções Aplicadas

### 3.1. Null-Safety e Fallbacks em `datetime_utils.dart`
Funções de formatação agora aceitam `DateTime?` e fornecem texto substituto amigável:
```dart
String formatDateTime(DateTime? value, {String fallback = 'A definir'});
String formatTime(DateTime? value, {String fallback = '--:--'});
String formatDate(DateTime? value, {String fallback = 'A definir'});
```

### 3.2. Modelo `Game` e `LiveGameResponse` Tolerantes a Nulo
- Campo `final DateTime? scheduledAt;`
- Parsing seguro com `DateTime.tryParse(json['scheduledAt'] as String)?.toLocal()`
- Serialização condicional `if (scheduledAt != null) 'scheduledAt': scheduledAt!.toIso8601String()`

### 3.3. Endpoint em Lote `gamesByCompetitionProvider`
Criado provider dedicado no `game_providers.dart`:
```dart
final gamesByCompetitionProvider = FutureProvider.family<List<Game>, String>(
  (ref, competitionId) => ref.watch(gameApiProvider).listByCompetition(competitionId),
);
```
O endpoint `/api/v1/competitions/{id}/games?size=500` retorna todos os jogos do campeonato em **uma única requisição HTTP**, eliminando a sobrecarga de 13 chamadas paralelas.

### 3.4. Ordenação Operacional Robusta com Valores Nulos
Criados comparadores com ordenação segura (`_compareScheduledAtAsc` e `_compareScheduledAtDesc`):
- Jogos com data definida são priorizados cronologicamente.
- Jogos com data a definir são alocados com segurança no final da lista, sem lançar exceções.

---

## 4. Validação e Qualidade de Código
- **`flutter analyze`:** Executado no `flag_referee_app` com **0 erros e 0 avisos** (`No issues found!`).
- **Deploy:** Commit `243c64a` na branch `staging`, sincronizado com `develop` e acionado no Hub Central de Deploys da plataforma (`flag_platform_infra` run `41218812791`).
