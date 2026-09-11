# ADR-011: Arquitetura MVVM Flutter — Guia de Migração e Padrões

**Status:** Aceito  
**Data:** 2026-09-09  
**Autor:** Tech Lead (Flag Platform)  

---

## Contexto

O `flag_admin_web` possui módulos em dois padrões arquiteturais distintos:

1. **Legado** (`lib/src/features/`): Screens ConsumerWidget/ConsumerStatefulWidget sem ViewModels, usando providers Riverpod diretamente
2. **Novo** (`lib/ui/`): Módulos com ViewModels ChangeNotifier, mas sem seguir estritamente os padrões oficiais do Flutter

**Problema:** A abordagem atual viola recomendações do Flutter App Architecture Guide:
- ViewModels não estendem ChangeNotifier
- State é mutável e exposto diretamente
- Operations assíncronas sem Command pattern
- Views acessam providers globais em vez de receber VM via construtor
- Data flow bidirecional em vez de unidirectional

**Referências:**
- [Flutter Guide to App Architecture](https://docs.flutter.dev/app-architecture/guide)
- [Flutter Architecture Case Study](https://docs.flutter.dev/app-architecture/case-study)
- [Flutter Recommendations](https://docs.flutter.dev/app-architecture/recommendations)
- [Flutter Design Patterns](https://docs.flutter.dev/app-architecture/design-patterns)

---

## Decisão

Adotar estritamente os padrões MVVM do Flutter App Architecture Guide para todos os módulos do `flag_admin_web`.

---

## Arquitetura MVVM — Definição Formal

### Visão Geral

```
┌─────────────────────────────────────────────────────────────┐
│                      UI LAYER                               │
│  ┌─────────────┐      ┌─────────────────────────────────┐  │
│  │    View     │ ◄─── │         ViewModel               │  │
│  │  (Widget)   │      │  (ChangeNotifier + Commands)    │  │
│  └─────────────┘      └──────────────┬──────────────────┘  │
│         ▲                             │                     │
│         │ constructor injection       │ repository          │
│         │                             ▼                     │
├─────────────────────────────────────────────────────────────┤
│                      DATA LAYER                             │
│  ┌─────────────────────────────────┐                       │
│  │        Repository               │                       │
│  │  (Cache + Domain Models)        │                       │
│  └──────────────┬──────────────────┘                       │
│                 │ service                                   │
│                 ▼                                           │
│  ┌─────────────────────────────────┐                       │
│  │         Service                 │                       │
│  │  (API Client / External)        │                       │
│  └─────────────────────────────────┘                       │
└─────────────────────────────────────────────────────────────┘
```

### Regras Fundamentais

| Regra | Descrição |
|-------|-----------|
| **1:1 View-ViewModel** | Cada View tem exatamente um ViewModel |
| **Create/Edit separados** | Formulários de criação e edição são ViewModels e Screens distintos (ex: `GameCreateViewModel`, `GameEditViewModel`) |
| **N:N ViewModel-Repository** | VMs podem usar múltiplos Repositories |
| **N:N Repository-Service** | Repositories podem usar múltiplos Services |
| **VM nunca vê View** | ViewModels são classes Dart puras |
| **Repository nunca vê outros Repos** | isolation total entre repos |
| **Service é o nível mais baixo** | Services não dependem de nada além de APIs externas |
| **Unidirectional Data Flow** | Dados: Data → VM → View. Eventos: View → VM → Data |

---

## Separação de Responsabilidades: Screen vs ViewModel

### Princípio: Single Responsibility

Cada classe tem **exatamente uma razão para mudar**.

| Classe | Responsabilidade Única | O que FAZ | O que NÃO faz |
|--------|------------------------|-----------|---------------|
| **Screen** | Renderizar UI e capturar eventos do usuário | Montar widget tree, chamar viewModel.method(), exibir estados (loading/erro/empty) | Consultar APIs, fazer cache, validar regras de negócio, gerenciar estado assíncrono |
| **ViewModel** | Gerenciar estado da UI e orquestrar lógica de apresentação | Carregar dados, filtrar, ordenar, formatar para exibição, gerenciar commands | Acessar banco de dados diretamente, renderizar widgets, navegar entre rotas |

### O que cada camada PODE fazer

```
SCREEN                                    VIEWMODEL
─────────────────────────────            ─────────────────────────────
✅ Montar widget tree                    ✅ Consultar Repository/Service
✅ Chamar viewModel.load()               ✅ Armazenar state (private)
✅ Exibir loading/error/empty            ✅ Filtrar/ordenar dados
✅ Capturar input do usuário             ✅ Formatar dados para exibição
✅ Navegar via context.go/push           ✅ Gerenciar commands (async)
✅ Mostrar SnackBar/Dialog               ✅ Chamar notifyListeners()
✅ Scroll, animation, layout             ✅ Validar regras de negócio
                                         
❌ Consultar APIs                        ❌ Montar widgets
❌ Fazer cache de dados                  ❌ Usar BuildContext
❌ Filtrar/ordenar dados                 ❌ Navegar (context.go)
❌ Validar regras de negócio             ❌ Mostrar SnackBar/Dialog
❌ Gerenciar estado assíncrono           ❌ Acessar ref (Riverpod)
❌ Acessar Repository diretamente       ❌ Ter widgets filhos
```

### Exemplo: Separação Correta

```dart
// ❌ ERRADO — Screen com lógica de negócio
class GameListScreen extends ConsumerStatefulWidget {
  @override
  Widget build(BuildContext context) {
    final games = ref.watch(gamesByRoundProvider(roundId));
    // Lógica de filtro DENTRO da screen ← VIOLA SRP
    final filtered = games.where((g) => g.status == GameStatus.scheduled).toList();
    return ListView(children: filtered.map((g) => GameCard(game: g)).toList());
  }
}

// ✅ CORRETO — Screen limpa, ViewModel gerencia lógica
class GameListScreen extends ConsumerStatefulWidget {
  @override
  Widget build(BuildContext context) {
    final vm = ref.watch(gameListViewModelProvider);
    return ListenableBuilder(
      listenable: vm,
      builder: (context, _) {
        // Screen apenas EXPOE o state que o ViewModel já processou
        return ListView(
          children: vm.filteredGames.map((g) => GameCard(game: g)).toList(),
        );
      },
    );
  }
}

// ViewModel encapsula a lógica de filtro
class GameListViewModel extends ChangeNotifier {
  List<Game> _allGames = [];
  GameStatus? _statusFilter;
  
  List<Game> get filteredGames {
    if (_statusFilter == null) return _allGames;
    return _allGames.where((g) => g.status == _statusFilter).toList();
  }
  
  void setStatusFilter(GameStatus? status) {
    _statusFilter = status;
    notifyListeners(); // UI re-renderiza automaticamente
  }
}
```

### Testabilidade

A separação permite testar cada camada isoladamente:

```dart
// Teste unitário do ViewModel (SEM Flutter framework)
test('filtra jogos por status', () {
  final repository = MockGameRepository();
  when(repository.getGamesByCompetition(any)).thenAnswer((_) async => [
    Game(id: '1', status: GameStatus.scheduled, ...),
    Game(id: '2', status: GameStatus.finished, ...),
  ]);
  
  final viewModel = GameListViewModel(repository: repository);
  viewModel.setStatusFilter(GameStatus.scheduled);
  
  expect(viewModel.filteredGames.length, 1);
  expect(viewModel.filteredGames.first.status, GameStatus.scheduled);
});

// Teste de widget (COM Flutter framework)
testWidgets('exibe lista de jogos filtrados', (tester) async {
  final viewModel = GameListViewModel(repository: mockRepository);
  await tester.pumpWidget(MaterialApp(
    home: ListenableBuilder(
      listenable: viewModel,
      builder: (context, _) => GameListScreen(viewModel: viewModel),
    ),
  ));
  
  expect(find.byType(GameCard), findsOneWidget);
});
```

### Consequências da Violação

| Violação | Consequência |
|----------|--------------|
| Screen consulta API diretamente | Difícil testar sem mock de HTTP |
| ViewModel monta widgets | Impossível testar lógica sem rodar Flutter |
| Screen gerencia estado assíncrono | Duplicação de código loading/error em todas as telas |
| ViewModel navega via context | Acoplamento com router, impossível testar unitário |

---

## Estrutura de Pastas

```
lib/
  ui/
    core/                            # Design system, widgets compartilhados
      ui/
        <shared_widgets>             # Shared widgets
      themes/
    <feature_name>/
      view_models/
        <feature>_view_model.dart    # ChangeNotifier por feature
      widgets/
        <feature>_screen.dart        # ConsumerStatefulWidget
        components/
          <other_widgets>            # Sub-widgets reutilizáveis (opcional)
  domain/
    models/
      <model_name>.dart              # Modelos puros (fromJson/toJson)
  data/
    repositories/
      <repository>.dart              # Cache + Transformação
    services/
      <service>.dart                 # Abstract + Implementação
    models/
      <api_model_class>.dart         # APIs of the modules/features
  condig/
    providers/                       # Riverpod providers (DI chain)
  utils/
  routing/                           # GoRouter config
  main_staging.dart
  main_development.dart
  main.dart
```

---

## Padrões por Camada

### 1. Domain Models

Modelos de dados imutáveis, sem lógica de negócio.

```dart
// lib/domain/models/game.dart
class Game {
  final String id;
  final String roundId;
  final String? homeTeamName;
  final GameStatus status;

  const Game({
    required this.id,
    required this.roundId,
    this.homeTeamName,
    required this.status,
  });

  factory Game.fromJson(Map<String, dynamic> json) => Game(
    id: json['id'] as String,
    roundId: json['roundId'] as String,
    homeTeamName: json['homeTeamName'] as String?,
    status: GameStatus.fromJson(json['status'] as String),
  );

  Map<String, dynamic> toJson() => {
    'id': id,
    'roundId': roundId,
    if (homeTeamName != null) 'homeTeamName': homeTeamName,
    'status': status.toJson(),
  };
}
```

**Regras:**
- Construtor `const` obrigatório
- Campos `final` (imutável)
- Factory `fromJson` para desserialização
- Método `toJson` para serialização
- Sem lógica de negócio (apenas transformação de dados)

---

### 2. Data Services

Wrapper de APIs externas. Stateless, sem side effects.

```dart
// lib/data/services/game_service.dart
abstract class GameService {
  Future<List<Game>> listByCompetition(String competitionId);
  Future<Game> getById(String id);
  Future<Game> create({
    required String roundId,
    required String homeTeamId,
    required String awayTeamId,
    String? venueId,
    required DateTime scheduledAt,
  });
}

// lib/data/services/api_game_service.dart
class ApiGameService implements GameService {
  final GameApi _api;

  ApiGameService(ApiClient client) : _api = GameApi(client);

  @override
  Future<List<Game>> listByCompetition(String competitionId) =>
      _api.listByCompetition(competitionId);

  @override
  Future<Game> getById(String id) => _api.getById(id);

  @override
  Future<Game> create({...}) => _api.create(...);
}
```

**Regras:**
- Interface abstrata (permite mock para testes)
- Implementação API concreta
- Stateless (sem estado)
- Uma Service por fonte de dados
- Níveis mais baixo da arquitetura

---

### 3. Data Repositories

Source of truth. Cache, error handling, retry, domain model transformation.

```dart
// lib/data/repositories/game_repository.dart
class GameRepository {
  final GameService _service;

  GameRepository({required GameService service}) : _service = service;

  // Cache em memória com TTL 30s
  final Map<String, List<Game>> _cacheByComp = {};
  final Map<String, DateTime> _lastFetchByComp = {};
  static const Duration _cacheTtl = Duration(seconds: 30);

  Future<List<Game>> getGamesByCompetition(
    String competitionId, {
    bool forceRefresh = false,
  }) async {
    final cached = _cacheByComp[competitionId];
    final lastFetch = _lastFetchByComp[competitionId];
    final isCacheValid = cached != null &&
        lastFetch != null &&
        DateTime.now().difference(lastFetch) < _cacheTtl;

    if (!forceRefresh && isCacheValid) {
      return cached!;
    }

    final data = await _service.listByCompetition(competitionId);
    _cacheByComp[competitionId] = List<Game>.unmodifiable(data);
    _lastFetchByComp[competitionId] = DateTime.now();
    return _cacheByComp[competitionId]!;
  }

  void clearCache(String competitionId) {
    _cacheByComp.remove(competitionId);
    _lastFetchByComp.remove(competitionId);
  }
}
```

**Regras:**
- Uma Repository por tipo de dados
- Cache com TTL (30s padrão)
- `clearCache()` após mutações (create/update/delete)
- Transforma dados da Service em modelos de domínio
- Nunca aware de outras Repositories

---

### 4. ViewModels (ChangeNotifier + Commands)

Gerencia state da UI, expõe callbacks, transforma dados para apresentação.

**Responsabilidade única:** Orquestrar o state que a Screen vai renderizar.

#### 4.1 Regras do ViewModel

| Regra | Descrição |
|-------|-----------|
| **State privado** | Backing fields `_variavel`, expostos via getters |
| **Getters públicos** | UI lê state apenas via `vm.variavel` |
| **Setters privados** | UI não seta state diretamente |
| **Commands** | Operações assíncronas (load, create, update, delete) |
| **notifyListeners()** | Chamado após QUALQUER mudança de state |
| **Sem BuildContext** | ViewModel não navega, não mostra SnackBar |
| **Sem widgets** | ViewModel não monta UI |
| **Sem ref** | ViewModel não acessa Riverpod diretamente |
| **Create ≠ Edit** | Formulários de criação e edição são ViewModels distintos com responsabilidades diferentes |

#### 4.2 ViewModel Base

```dart
// lib/ui/game/view_models/game_list_view_model.dart
class GameListViewModel extends ChangeNotifier {
  final GameRepository _repository;

  GameListViewModel({required GameRepository repository})
      : _repository = repository {
    // Commands criados no construtor
    load = Command0(_load)..execute();
    selectCompetition = Command1(_selectCompetition);
  }

  // State privado (imutável para o UI)
  List<Game> _games = [];
  String? _selectedCompetitionId;
  String? _selectedRoundId;

  // Getters públicos
  List<Game> get games => _games;
  String? get selectedCompetitionId => _selectedCompetitionId;
  String? get selectedRoundId => _selectedRoundId;

  // Commands (async operations com state management)
  late Command0 load;
  late Command1<void, String?> selectCompetition;

  // Métodos privados (implementação dos commands)
  Future<Result<List<Game>>> _load() async {
    try {
      final games = await _repository.getGamesByCompetition(
        _selectedCompetitionId ?? '',
      );
      _games = games;
      return Result.ok(games);
    } catch (e) {
      return Result.error(e);
    } finally {
      notifyListeners();
    }
  }

  Future<Result<void>> _selectCompetition(String? competitionId) async {
    _selectedCompetitionId = competitionId;
    _selectedRoundId = null;
    _games = [];
    notifyListeners();
    return Result.ok(null);
  }
}
```

#### 4.2 Command Pattern

Wrapper para operações assíncronas com state tracking.

```dart
// lib/ui/core/commands/command.dart
abstract class Command<T> extends ChangeNotifier {
  bool _running = false;
  Result<T>? _result;

  bool get running => _running;
  bool get error => _result is Error;
  bool get completed => _result is Ok;
  Result<T>? get result => _result;

  Future<void> _execute(Future<Result<T>> Function() action) async {
    if (_running) return;
    _running = true;
    _result = null;
    notifyListeners();
    try {
      _result = await action();
    } finally {
      _running = false;
      notifyListeners();
    }
  }
}

class Command0<T> extends Command<T> {
  final Future<Result<T>> Function() _action;

  Command0(this._action);

  Future<void> execute() => _execute(_action);
}

class Command1<T, A> extends Command<T> {
  final Future<Result<T>> Function(A) _action;

  Command1(this._action);

  Future<void> execute(A arg) => _execute(() => _action(arg));
}
```

#### 4.3 ViewModel de Detalhe

```dart
class GameDetailViewModel extends ChangeNotifier {
  final GameService _service;

  GameDetailViewModel({required GameService service}) : _service = service {
    load = Command1(_load);
  }

  Game? _game;
  Game? get game => _game;

  late Command1<void, String> load;

  Future<Result<Game>> _load(String gameId) async {
    try {
      _game = await _service.getById(gameId);
      return Result.ok(_game!);
    } catch (e) {
      return Result.error(e);
    } finally {
      notifyListeners();
    }
  }
}
```

#### 4.4 ViewModel de Formulário

```dart
class GameFormViewModel extends ChangeNotifier {
  final GameRepository _repository;

  GameFormViewModel({required GameRepository repository})
      : _repository = repository {
    create = Command0(_create);
    update = Command0(_update);
  }

  // Form state
  String? _roundId;
  String? _homeTeamId;
  String? _awayTeamId;
  String? _venueId;
  DateTime? _scheduledAt;

  // Getters
  String? get roundId => _roundId;
  String? get homeTeamId => _homeTeamId;
  // ... etc

  // Setters (marcam dirty)
  void setRoundId(String? value) {
    _roundId = value;
    notifyListeners();
  }

  late Command0 create;
  late Command0 update;

  Future<Result<void>> _create() async {
    try {
      await _repository.createGame(
        competitionId: _competitionId!,
        roundId: _roundId!,
        homeTeamId: _homeTeamId!,
        awayTeamId: _awayTeamId!,
        venueId: _venueId,
        scheduledAt: _scheduledAt!,
      );
      return Result.ok(null);
    } catch (e) {
      return Result.error(e);
    }
  }
}
```

**Regras dos ViewModels:**
- Extends `ChangeNotifier` obrigatório
- State privado com backing fields
- Getters públicos para leitura
- Setters privados (usados internamente)
- `notifyListeners()` após qualquer mudança de state
- Commands para operações assíncronas
- Repositories injetados via construtor (privados)
- Sem aware de Widgets/Views

---

### 5. Views (Screens)

Widgets que renderizam UI. Recebem ViewModel via construtor.

```dart
// lib/ui/game/widgets/game_list_screen.dart
class GameListScreen extends ConsumerStatefulWidget {
  const GameListScreen({super.key});

  @override
  ConsumerState<GameListScreen> createState() => _GameListScreenState();
}

class _GameListScreenState extends ConsumerState<GameListScreen> {
  @override
  Widget build(BuildContext context) {
    // ViewModel criado via provider (injeção de dependência)
    final viewModel = ref.watch(gameListViewModelProvider);

    return AppScreen(
      title: 'Jogos',
      body: ListenableBuilder(
        listenable: viewModel,
        builder: (context, _) {
          // State do ViewModel exposto via getters
          if (viewModel.load.running) {
            return const AppLoading(message: 'Carregando jogos...');
          }

          if (viewModel.load.error) {
            return AppErrorState(
              message: 'Erro ao carregar jogos',
              onRetry: () => viewModel.load.execute(),
            );
          }

          return AppEntityListScreen<Game>(
            items: viewModel.games,
            cardBuilder: (game) => _gameCard(game),
          );
        },
      ),
    );
  }

  Widget _gameCard(Game game) {
    return KicksterScoreCard(
      homeTeamName: game.homeTeamName ?? 'Casa',
      awayTeamName: game.awayTeamName ?? 'Fora',
      // ...
    );
  }
}
```

**Regras das Views:**
- ConsumerStatefulWidget ou ConsumerWidget
- Recebe ViewModel via provider (não via construtor direto)
- Usa `ListenableBuilder` para ouvir mudanças do ViewModel
- Sem lógica de negócio (apresentação + routing + animation)
- State flags do Command: `running`, `error`, `completed`
- Apenas conditionais simples, routing, e layout

---

### 6. Dependency Injection (Providers)

Cadeia: Service → Repository → ViewModel

```dart
// lib/src/providers/providers.dart

// === SERVICES ===
final gameServiceProvider = Provider<GameService>(
  (ref) => ApiGameService(ref.watch(apiClientProvider)),
);

// === REPOSITORIES ===
final gameRepositoryProvider = Provider<GameRepository>(
  (ref) => GameRepository(service: ref.watch(gameServiceProvider)),
);

// === VIEWMODELS ===
final gameListViewModelProvider =
    ChangeNotifierProvider.autoDispose<GameListViewModel>(
  (ref) => GameListViewModel(
    repository: ref.watch(gameRepositoryProvider),
  ),
);

final gameDetailViewModelProvider =
    ChangeNotifierProvider.autoDispose.family<GameDetailViewModel, String>(
  (ref, id) => GameDetailViewModel(
    service: ref.watch(gameServiceProvider),
  ),
);

final gameCreateViewModelProvider =
    ChangeNotifierProvider.autoDispose<GameCreateViewModel>(
  (ref) => GameCreateViewModel(
    repository: ref.watch(gameRepositoryProvider),
  ),
);

final gameEditViewModelProvider =
    ChangeNotifierProvider.autoDispose.family<GameEditViewModel, String>(
  (ref, gameId) => GameEditViewModel(
    repository: ref.watch(gameRepositoryProvider),
  ),
);
```

**Regras de DI:**
- Services no topo (Provider simples)
- Repositories dependem de Services
- ViewModels dependem de Repositories
- ViewModels são `autoDispose` (liberam memória quando a tela sai)
- ViewModels familiares quando precisam de ID (`.family`)
- Providers centralizados em `providers.dart`
- **Create e Edit são providers distintos**

---

### 7. Rotas (GoRouter)

ViewModels criados na configuração de rotas. Create e Edit são rotas distintas.

```dart
// lib/src/router/app_router.dart
GoRoute(
  path: '/games',
  name: 'games',
  builder: (context, state) => const GameListScreen(),
  routes: [
    GoRoute(
      path: 'new',
      name: 'gameCreate',
      builder: (context, state) {
        final args = state.extra as GameCreateArgs?;
        return GameCreateScreen(args: args);
      },
    ),
    GoRoute(
      path: ':id',
      name: 'gameDetail',
      builder: (context, state) {
        final game = state.extra as Game?;
        return GameDetailScreen(
          gameId: state.pathParameters['id']!,
          game: game,
        );
      },
    ),
    GoRoute(
      path: ':id/edit',
      name: 'gameEdit',
      builder: (context, state) {
        final args = state.extra as GameEditArgs?;
        return GameEditScreen(
          gameId: state.pathParameters['id']!,
          args: args,
        );
      },
    ),
  ],
),
```

---

## Checklist de Migração por Módulo

Para cada módulo, verificar:

### Domain
- [ ] Model em `lib/domain/models/` com `fromJson`/`toJson`
- [ ] Campos `final` e construtor `const`
- [ ] Sem lógica de negócio no model

### Data Layer
- [ ] Service abstrata em `lib/data/services/`
- [ ] Implementação API em `lib/data/services/api_*.dart`
- [ ] Repository com cache TTL em `lib/data/repositories/`
- [ ] `clearCache()` após mutações

### UI Layer
- [ ] ViewModel extende `ChangeNotifier`
- [ ] State privado com backing fields
- [ ] Getters públicos para leitura
- [ ] Commands para operações assíncronas
- [ ] `notifyListeners()` após mudanças
- [ ] Screen usa `ListenableBuilder`
- [ ] Screen não contém lógica de negócio
- [ ] **Create e Edit são ViewModels e Screens distintos**

### Providers
- [ ] Service provider
- [ ] Repository provider
- [ ] ViewModel provider (autoDispose)
- [ ] **Create e Edit são providers distintos**
- [ ] Cadeia Service → Repository → ViewModel

### Rotas
- [ ] Rotas atualizadas em `app_router.dart`
- [ ] Nomes no padrão camelCase
- [ ] **Create (`/new`) e Edit (`/:id/edit`) são rotas distintas**

---

## Módulos a Migrar

| Módulo | Telas | Status |
|--------|-------|--------|
| Games | List, Detail, Create, Edit, Import | A migrar |
| Rounds | List, Detail, Create, Edit | A migrar |
| Venues | List, Detail, Create, Edit | A migrar |
| Users | List, Create, Edit | A migrar |
| Approvals | List | A migrar |
| Rosters | List, Import | A migrar |

---

## Consequências

### Positivas
- ✅ Alinhamento com recomendações oficiais do Flutter
- ✅ Testabilidade: ViewModels isolados da UI
- ✅ Manutenibilidade: padrão único documentado
- ✅ Performance: cache TTL + unidirectional data flow
- ✅ Experiência do desenvolvedor: padrão canônico do ecossistema Flutter

### Negativas
- ❌ Refatoração de ViewModels existentes (adicionar ChangeNotifier + Commands)
- ❌ Screens precisam usar ListenableBuilder
- ❌ Curva de aprendizado para Command pattern

### Riscos
- ⚠️ Quebra de funcionalidade → mitigado por migração módulo a módulo
- ⚠️ Complexidade do Command pattern → mitigado por exemplos de referência

---

## Referências de Código

Módulo de referência (canonical): **Person** (`lib/ui/person/`)

Exemplos de cada padrão:
- ViewModel de listagem: `lib/ui/person/view_models/person_view_model.dart`
- ViewModel de criação: `lib/ui/person/view_models/person_create_view_model.dart`
- ViewModel de edição: `lib/ui/person/view_models/person_edit_view_model.dart`
- Screen com ListenableBuilder: `lib/ui/person/widgets/person_list_screen.dart`
- Screen de criação: `lib/ui/person/widgets/person_create_screen.dart`
- Screen de edição: `lib/ui/person/widgets/person_edit_screen.dart`
- Repository com cache: `lib/data/repositories/person_repository.dart`
- Service abstrata: `lib/data/services/person_service.dart`
- Providers: `lib/src/providers/providers.dart`

---

*Esta ADR substitui a ADR-011 anterior (Migração do Módulo de Atletas).*
