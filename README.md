# CDC_FINAL_V6.md
## Application Mobile Flutter — Résidence L'Amandier B
### Bouskoura, Maroc | Version 6.0 MILITAIRE GENÈSE — Mai 2026
### Architecte Principal : Claude Sonnet 4.6 | Exécutant Syntaxique : Jules (AI Coding Agent)

---

> **Directive Absolue :** Ce document annihile la V5 et toutes ses versions antérieures (V1.0 à V5.0). Il constitue la **vérité unique, immuable et opérationnelle** du projet. Toute divergence entre ce document et le code produit est une erreur de l'exécutant. Ce CDC V6 introduit quatre mutations architecturales chirurgicales appliquées au corpus V5 : **(I) Résolution Entité Topologique** — le flux Résident bifurque vers un sélecteur graphique 3D du bâtiment avant toute demande de PIN, éradiquant définitivement l'ambiguïté d'identité multi-résident ; **(II) Séquence Genèse** — le Wizard de configuration initiale est redéfini en 5 étapes capturant l'état thermodynamique et financier exact du bâtiment au T=0 du mandat ; **(III) Purisme Réactif** — l'`AppEventBus` impératif est éradiqué ; la réactivité inter-module est réalisée exclusivement via des `StreamProvider` Riverpod dérivés des Contrats, exposés dans `reactive_bridges.dart` ; **(IV) Enforcement Offline-First** — toute mutation sur un contrat (`IFinanceContract`, `IIncidentContract`, etc.) met à jour l'état Riverpod local *avant* l'appel RPC, avec encapsulation dans `OfflineQueueService` (Hive) si hors-ligne, et rollback uniquement sur échec serveur définitif.

---

# ═══════════════════════════════════════════════════════════
# MANIFESTE V6 — SEPT LOIS INVIOLABLES
# ═══════════════════════════════════════════════════════════

## LOI I — ISOLATION TOPOLOGIQUE ABSOLUE

Les modules de l'application sont des **îles logicielles hermétiques**. Un module ne connaît pas l'existence d'un autre module. La communication inter-module s'effectue exclusivement via :

1. **Contrats abstraits** (`lib/core/contracts/`) : interfaces Dart pures définissant ce qu'un module expose vers l'extérieur.
2. **Ponts Réactifs** (`lib/core/providers/reactive_bridges.dart`) : `StreamProvider` Riverpod dérivés des contrats, accessibles à tous les modules sans import cross-module. Remplace définitivement l'`AppEventBus` V5.
3. **Deep Linking GoRouter** : navigation inter-module via URI uniquement.

**Test d'amputation :** Si le répertoire `lib/modules/finance/` est physiquement supprimé du projet, l'application doit compiler et s'exécuter. Le shell UI détecte l'absence du module via le `ModuleRegistry` et masque dynamiquement les blocs manquants. Tout module doit passer ce test.

**Cas limite explicite — Incident → Tâche :** Le module Incidents ne peut **jamais** importer `lib/modules/tasks/`. La conversion s'effectue en deux temps : (a) `IncidentNotifier` appelle `incidentContractImpl.emitEscalation(payload)` ; (b) `TaskNotifier`, via `ref.listen(incidentEscalationStreamProvider, ...)`, reçoit le payload et crée la tâche. Ni l'un ni l'autre ne s'importe mutuellement.

## LOI II — AMPUTATION DU MODULE AG

Le module Assemblée Générale (AG) est **définitivement éradiqué**. Toute entité liée à l'AG est interdite : tables de données, modèles Dart (`AgSessionModel`, `AgPresenceModel`, `AgVoteModel`), écrans, providers, routes, constantes, widgets. Toute référence à l'AG dans le code est une violation de cette loi.

## LOI III — AUTHENTIFICATION TOPOLOGIQUE DÉTERMINISTE V6

L'unique flux d'authentification V6 est une **bifurcation physique** en deux chemins distincts :

**Chemin Staff** (`syndicAlpha`, `adjointLecture`, `conciergeT errain`, `menageRestreint`) :
1. Sélection du rôle Staff via `StaffRoleSelectorScreen` (4 tuiles).
2. Saisie du PIN 8 chiffres via `PinEntryScreen`.

**Chemin Résident** :
1. `ResidentBuildingSelectorScreen` : rendu graphique 3D convexe du bâtiment — **AUCUN PIN N'EST DEMANDÉ ICI**.
2. Tap sur le nœud de l'appartement (1-15) → résolution physique de l'identité.
3. Saisie du PIN 8 chiffres avec `apartmentId` en contexte cryptographique.

**Règles invariantes :**
- L'email est interdit comme mécanisme d'authentification — il n'existe que comme payload informatif dans la base de données.
- `pinLength = 8` strictement.
- Le PIN ne circule jamais en clair — hash `SHA-256(pin + salt_from_server(apartmentId))` pour les résidents, `SHA-256(pin + salt_from_server(role))` pour le staff.
- La biométrie est admise uniquement comme raccourci de déverrouillage de session active.

## LOI IV — THÈME DUAL LUXURY (INVIOLABLE)

Le mode Light V6 est **Alabaster Institutionnel** (`#F0E8DC` exclusif, `#FFFFFF` banni). Le mode Dark est **Abyssal** (`#050508`). Toute surface utilise les tokens `AppColors.*`. Le widget `BuildingTopologySelector` adopte l'effet convexe 3D via dégradé radial conforme aux deux thèmes.

- **Fond principal Light :** Alabaster chaud (`#F0E8DC`)
- **Fond principal Dark :** Noir abyssal (`#050508`)
- **Surface Light :** Parchemin (`#EDE2D4`)
- **Surface Dark :** `#0D0D14`
- **Futur impayé :** Gris chaud Light (`#9E9488`) / Gris abyssal Dark (`#504E64`) — **JAMAIS ROUGE**

## LOI V — PURISME RÉACTIF RIVERPOD (NOUVELLE V6)

**L'`AppEventBus` et le répertoire `lib/core/events/` sont définitivement éradiqués.** Aucune instance de `StreamController` à usage général n'existe en dehors des implémentations de contrats.

La réactivité inter-module est accomplie par la triade suivante :

1. **Contrat** (`lib/core/contracts/i_xxx_contract.dart`) : déclare une méthode `Stream<XxxPayload> watchXxx()`.
2. **Implémentation** (`lib/modules/xxx/application/xxx_contract_impl.dart`) : maintient un `StreamController.broadcast()` interne. Expose le stream via le contrat. Expose une méthode `emitXxx(payload)` appelée exclusivement par les Notifiers du même module.
3. **Pont Réactif** (`lib/core/providers/reactive_bridges.dart`) : `StreamProvider` Riverpod dérivés des providers de contrats. Accessibles à tous les modules via `ref.listen(xxxStreamProvider, ...)` sans aucun import cross-module.

**Règle de style :** Tout Notifier qui réagit à un événement inter-module utilise `ref.listen` dans son `build()` ou dans `onAppStart`. Jamais de subscription impérative à une source globale non-Riverpod.

## LOI VI — OFFLINE-FIRST DOCTRINAL (NOUVELLE V6)

**Toute mutation** exposée par un contrat (`IFinanceContract.fifoPayment`, `IIncidentContract.createIncident`, etc.) **DOIT** suivre le protocole Optimistic UI V6 :

```
ÉTAPE 1 : Mise à jour immédiate de l'état Riverpod local (optimiste).
ÉTAPE 2 : Vérification ConnectivityProvider.
  SI en ligne  → Exécuter l'appel RPC.
                 En cas d'échec définitif (non-réseau) → Rollback état local.
  SI hors-ligne → Encapsuler le payload RPC dans OfflineQueueService (Hive).
                  Afficher un badge "En attente de synchronisation".
ÉTAPE 3 : SyncService rejoue la queue au retour de connectivité.
          Sur succès de replay → état local déjà correct, aucune action.
          Sur échec définitif de replay → Rollback + notification utilisateur.
```

**Le rollback ne se produit QUE sur échec serveur définitif** (code HTTP 4xx non-réseau, erreur Postgres métier). Une erreur réseau transitoire produit uniquement une entrée en queue offline, jamais un rollback.

## LOI VII — PORTES DE VÉRIFICATION CHIRURGICALE

À la fin de **chaque phase**, l'exécutant (Jules) doit passer la Gate associée avant toute progression. Une Gate non passée = blocage total. Les phases ne peuvent pas être exécutées en parallèle. L'acyclicité du graphe est absolue.

---

# ═══════════════════════════════════════════════════════════
# PRÉAMBULE — INVENTAIRE & QUOTA DE PHASES V6
# ═══════════════════════════════════════════════════════════

## Inventaire des Fichiers `.dart` V6

| Couche | Description | Fichiers |
|--------|-------------|----------|
| `lib/core/design/` | Design system complet (couleurs, typo, dimensions, ombres, animations, thème, gradients) | **7** |
| `lib/core/` | Kernel : constantes, erreurs (3), extensions (4), utils (3). *Events éradiqués.* | **11** |
| `lib/core/contracts/` | Contrats modulaires abstraits V6 (10 interfaces, étendues avec streams réactifs) | **10** |
| `lib/core/router/` | Shell : guards (3), router, route names | **5** |
| `lib/core/providers/` | Providers globaux : supabase, auth_state, theme, connectivity + **reactive_bridges** (nouveau V6) | **5** |
| `lib/core/services/` | Services noyau (9) | **9** |
| `lib/` | `app.dart` + `main.dart` | **2** |
| `lib/shared/widgets/` | Widgets partagés (40 V5 + `building_topology_widget.dart` nouveau V6) | **41** |
| `lib/modules/auth/` | Auth V6 : bifurcation + sélecteur bâtiment résident + sélecteur staff (13 fichiers vs 10 en V5) | **13** |
| `lib/modules/wizard/` | Wizard Séquence Genèse 5 étapes | **10** |
| `lib/modules/dashboard/` | Shell dashboards 5 rôles | **10** |
| `lib/modules/finance/` | Finance complet (domaine + data + app + présentation) | **30** |
| `lib/modules/incidents/` | Incidents FSM complet | **10** |
| `lib/modules/tasks/` | Tâches maître + instances + legacy | **10** |
| `lib/modules/hr/` | RH : salaires, avances | **10** |
| `lib/modules/prestataires/` | Annuaire prestataires | **10** |
| `lib/modules/annonces/` | Tableau d'affichage | **10** |
| `lib/modules/notifications/` | Centre de notifications | **7** |
| `lib/modules/documents/` | Documents PDF | **7** |
| `lib/modules/profile/` | Profil utilisateur | **7** |
| `lib/shell/` | Module Registry + App Loader + Dynamic Nav | **3** |
| **TOTAL** | | **T = 227** |

```
T = 227 fichiers .dart
N = ⌈227 / 10⌉ = 23 phases
Phases P0 → P22 (P0 livre 0 fichiers dart — scaffold pur)
Quota par phase : ≤ 10 fichiers (tolérance ±3 sur phases restructurées V6)
Delta V5→V6 : −2 (events éradiqués) +1 (reactive_bridges) +1 (building_topology_widget) +3 (auth screens) = +3
```

---

# ═══════════════════════════════════════════════════════════
# PARTIE I — ARCHITECTURE MODULAIRE TOPOLOGIQUE V6
# ═══════════════════════════════════════════════════════════

## I.1 — Structure Topologique du Projet V6

```
lib/
├── main.dart
├── app.dart
├── core/
│   ├── design/
│   │   ├── app_colors.dart
│   │   ├── app_text_styles.dart
│   │   ├── app_dimensions.dart
│   │   ├── app_shadows.dart
│   │   ├── app_animations.dart
│   │   ├── app_theme.dart
│   │   └── app_gradients.dart
│   ├── constants.dart
│   ├── errors/
│   │   ├── failure.dart
│   │   ├── app_exception.dart
│   │   └── error_handler.dart
│   ├── contracts/
│   │   ├── i_module_registrar.dart
│   │   ├── i_auth_contract.dart          ← V6 : +watchLogout(), +watchSetupCompleted()
│   │   ├── i_finance_contract.dart       ← V6 : +watchPayments() → Stream<PaymentRecordedPayload>
│   │   ├── i_incident_contract.dart      ← V6 : +watchEscalations(), +watchStatusChanges()
│   │   ├── i_task_contract.dart          ← V6 : +watchOverdue() → Stream<TaskOverduePayload>
│   │   ├── i_hr_contract.dart
│   │   ├── i_annonce_contract.dart
│   │   ├── i_notification_contract.dart
│   │   ├── i_document_contract.dart
│   │   └── i_profile_contract.dart
│   ├── extensions/
│   │   ├── string_extensions.dart
│   │   ├── datetime_extensions.dart
│   │   ├── num_extensions.dart
│   │   └── context_extensions.dart
│   ├── utils/
│   │   ├── amount_formatter.dart
│   │   ├── date_formatter.dart
│   │   └── validators.dart
│   ├── router/
│   │   ├── route_names.dart
│   │   ├── auth_guard.dart
│   │   ├── role_guard.dart
│   │   ├── setup_guard.dart
│   │   └── app_router.dart
│   ├── providers/
│   │   ├── supabase_provider.dart
│   │   ├── auth_state_provider.dart
│   │   ├── theme_provider.dart
│   │   ├── connectivity_provider.dart
│   │   └── reactive_bridges.dart         ← NOUVEAU V6 — Remplace AppEventBus
│   └── services/
│       ├── supabase_service.dart
│       ├── secure_storage_service.dart
│       ├── fcm_service.dart
│       ├── pdf_generation_service.dart
│       ├── idempotency_service.dart
│       ├── image_upload_service.dart
│       ├── offline_queue_service.dart
│       ├── sync_service.dart
│       └── weather_service.dart
├── shared/
│   └── widgets/
│       ├── buttons/
│       │   ├── primary_button.dart
│       │   ├── secondary_button.dart
│       │   ├── danger_button.dart
│       │   └── icon_action_button.dart
│       ├── dialogs/
│       │   ├── confirm_dialog.dart
│       │   ├── amount_input_dialog.dart
│       │   ├── loading_dialog.dart
│       │   ├── error_dialog.dart
│       │   └── success_dialog.dart
│       ├── loaders/
│       │   ├── shimmer_loader.dart
│       │   └── full_screen_loader.dart
│       ├── forms/
│       │   ├── amount_field.dart
│       │   ├── text_field_custom.dart
│       │   ├── date_picker_field.dart
│       │   ├── dropdown_field.dart
│       │   └── photo_picker_field.dart
│       ├── layouts/
│       │   ├── app_scaffold.dart
│       │   ├── sliver_header.dart
│       │   ├── section_header.dart
│       │   └── empty_state.dart
│       ├── navigation/
│       │   ├── bottom_nav_bar.dart
│       │   ├── app_top_bar.dart
│       │   └── role_drawer.dart
│       ├── contextual/
│       │   ├── clock_widget.dart
│       │   ├── weather_widget.dart
│       │   └── contextual_header.dart
│       ├── cards/
│       │   ├── info_card.dart
│       │   ├── stat_card.dart
│       │   ├── transaction_card.dart
│       │   ├── apartment_card.dart
│       │   ├── incident_card.dart
│       │   └── task_card.dart
│       ├── lists/
│       │   ├── paginated_list.dart
│       │   └── transaction_list_tile.dart
│       ├── building_topology_widget.dart  ← NOUVEAU V6 — Sélecteur bâtiment 3D convexe
│       ├── pin_keyboard.dart
│       ├── currency_badge.dart
│       ├── status_badge.dart
│       ├── role_chip.dart
│       ├── qr_code_widget.dart
│       └── treasury_gauge.dart
├── shell/
│   ├── module_registry.dart
│   ├── app_module_loader.dart
│   └── dynamic_nav_builder.dart
└── modules/
    ├── auth/
    │   ├── auth_module.dart
    │   ├── domain/
    │   │   ├── user_model.dart
    │   │   ├── app_role.dart
    │   │   └── session_model.dart
    │   ├── data/
    │   │   ├── auth_repository.dart
    │   │   └── auth_repository_impl.dart
    │   ├── application/
    │   │   ├── auth_provider.dart
    │   │   └── pin_provider.dart
    │   └── presentation/
    │       ├── splash_screen.dart
    │       ├── auth_bifurcation_screen.dart       ← NOUVEAU V6 (remplace role_selection)
    │       ├── staff_role_selector_screen.dart    ← NOUVEAU V6
    │       ├── resident_building_selector_screen.dart ← NOUVEAU V6
    │       └── pin_entry_screen.dart              ← MODIFIÉ V6 : context-aware (role|apartmentId)
    ├── wizard/   [structure V5 maintenue, contenu 5 étapes Genèse]
    └── [autres modules — structure V5 maintenue]
```

**Répertoire `lib/core/events/` : ABOLI. Aucune trace dans le projet V6.**

## I.2 — Pattern des Contrats Modulaires V6

Les contrats V6 sont étendus avec des méthodes de streaming réactif. Chaque méthode `watchXxx()` remplace un événement qui était précédemment émis sur l'`AppEventBus`.

```dart
// lib/core/contracts/i_module_registrar.dart — INCHANGÉ V5
abstract interface class IModuleRegistrar {
  String get moduleId;
  List<RouteBase> get routes;
  List<QuickAction> get quickActions;
  void onAppStart(ProviderContainer container);
}

class QuickAction {
  final String label;
  final String iconAsset;
  final String routeUri;
  final Set<AppRole> allowedRoles;
  const QuickAction({required this.label, required this.iconAsset,
                     required this.routeUri, required this.allowedRoles});
}
```

```dart
// lib/core/contracts/i_auth_contract.dart — AMENDÉ V6
abstract interface class IAuthContract {
  AppRole? get currentRole;
  String? get currentUserId;
  String? get currentApartmentId; // NOUVEAU V6 — null pour staff
  bool get isAuthenticated;
  Stream<bool> get authStream;
  Stream<void> watchLogout();           // NOUVEAU V6 — remplace UserLoggedOutEvent
  Stream<void> watchSetupCompleted();   // NOUVEAU V6 — remplace WizardCompletedEvent
}
```

```dart
// lib/core/contracts/i_finance_contract.dart — AMENDÉ V6
abstract interface class IFinanceContract {
  Future<int> getApartmentBalanceCentimes(String apartmentId);
  Future<TreasurySnapshot> getTreasurySnapshot();
  Stream<int> watchUnpaidCount();
  Stream<PaymentRecordedPayload> watchPayments(); // NOUVEAU V6 — remplace PaymentRecordedEvent
}

// Value objects — déclarés dans lib/core/contracts/ uniquement
@immutable
class TreasurySnapshot {
  final int caisseCentimes;
  final int banqueCentimes;
  final int totalCentimes;
  final double runwayMonths;
  const TreasurySnapshot({required this.caisseCentimes, required this.banqueCentimes,
    required this.totalCentimes, required this.runwayMonths});
}

@immutable
class PaymentRecordedPayload {
  final String apartmentId;
  final int amountCentimes;
  final String periode;
  const PaymentRecordedPayload({required this.apartmentId,
    required this.amountCentimes, required this.periode});
}
```

```dart
// lib/core/contracts/i_incident_contract.dart — AMENDÉ V6
abstract interface class IIncidentContract {
  Stream<int> watchOpenIncidentCount();
  Future<void> createFromTaskPayload(IncidentCreationPayload p);
  Stream<IncidentEscalationPayload> watchEscalations();    // NOUVEAU V6
  Stream<IncidentStatusChangePayload> watchStatusChanges(); // NOUVEAU V6
}

@immutable
class IncidentEscalationPayload {
  final String incidentId;
  final String title;
  final String description;
  final String? assignedUserId;
  final String? apartmentId;
  const IncidentEscalationPayload({required this.incidentId, required this.title,
    required this.description, this.assignedUserId, this.apartmentId});
}

@immutable
class IncidentStatusChangePayload {
  final String incidentId;
  final String newStatus;
  final String? residentUserId;
  const IncidentStatusChangePayload({required this.incidentId,
    required this.newStatus, this.residentUserId});
}
```

```dart
// lib/core/contracts/i_task_contract.dart — AMENDÉ V6
abstract interface class ITaskContract {
  Stream<int> watchPendingTaskCount(String userId);
  Future<void> createFromIncidentPayload(TaskCreationPayload p);
  Stream<TaskOverduePayload> watchOverdue(); // NOUVEAU V6
}

@immutable
class TaskOverduePayload {
  final String taskInstanceId;
  final String taskTitle;
  final String? assignedUserId;
  const TaskOverduePayload({required this.taskInstanceId,
    required this.taskTitle, this.assignedUserId});
}
```

**Règle d'or des contrats V6 :** Les types de retour des contrats sont soit des **primitives Dart**, soit des **value objects `@immutable` déclarés dans `lib/core/contracts/`**. Jamais des modèles internes d'un module.

## I.3 — Ponts Réactifs V6 (`reactive_bridges.dart`)

```dart
// lib/core/providers/reactive_bridges.dart
// ═══════════════════════════════════════════════════════════
// PONT RÉACTIF V6 — L'AppEventBus est MORT. Vive Riverpod.
// Ce fichier est le SEUL point d'accès aux streams inter-module.
// Les modules y accèdent via ref.listen(...) sans aucun import cross-module.
// ═══════════════════════════════════════════════════════════

// Fournisseurs de contrats (injectés par le Shell au démarrage via ModuleRegistry)
final financeContractProvider = Provider<IFinanceContract>((ref) =>
  throw UnimplementedError('FinanceModule must be registered'));

final incidentContractProvider = Provider<IIncidentContract>((ref) =>
  throw UnimplementedError('IncidentsModule must be registered'));

final taskContractProvider = Provider<ITaskContract>((ref) =>
  throw UnimplementedError('TasksModule must be registered'));

final authContractProvider = Provider<IAuthContract>((ref) =>
  throw UnimplementedError('AuthModule must be registered'));

// ── PONT 1 : Incidents → Tasks ──────────────────────────────
final incidentEscalationStreamProvider =
    StreamProvider<IncidentEscalationPayload>((ref) {
  return ref.watch(incidentContractProvider).watchEscalations();
});

// ── PONT 2 : Finance → Notifications ───────────────────────
final paymentRecordedStreamProvider =
    StreamProvider<PaymentRecordedPayload>((ref) {
  return ref.watch(financeContractProvider).watchPayments();
});

// ── PONT 3 : Incidents → Notifications ─────────────────────
final incidentStatusStreamProvider =
    StreamProvider<IncidentStatusChangePayload>((ref) {
  return ref.watch(incidentContractProvider).watchStatusChanges();
});

// ── PONT 4 : Tasks → Notifications ─────────────────────────
final taskOverdueStreamProvider =
    StreamProvider<TaskOverduePayload>((ref) {
  return ref.watch(taskContractProvider).watchOverdue();
});

// ── PONT 5 : Auth → Global (logout) ────────────────────────
final authLogoutStreamProvider = StreamProvider<void>((ref) {
  return ref.watch(authContractProvider).watchLogout();
});

// ── PONT 6 : Auth → Shell (wizard terminé) ─────────────────
final wizardCompletedStreamProvider = StreamProvider<void>((ref) {
  return ref.watch(authContractProvider).watchSetupCompleted();
});
```

**Patron d'implémentation dans un ContractImpl :**

```dart
// lib/modules/incidents/application/incident_contract_impl.dart
class IncidentContractImpl implements IIncidentContract {
  // StreamControllers internes — durée de vie = durée de vie de l'app
  final _escalationCtrl = StreamController<IncidentEscalationPayload>.broadcast();
  final _statusCtrl     = StreamController<IncidentStatusChangePayload>.broadcast();

  @override
  Stream<IncidentEscalationPayload> watchEscalations() => _escalationCtrl.stream;

  @override
  Stream<IncidentStatusChangePayload> watchStatusChanges() => _statusCtrl.stream;

  /// Appelé UNIQUEMENT par IncidentNotifier (même module)
  void emitEscalation(IncidentEscalationPayload payload) =>
      _escalationCtrl.add(payload);

  void emitStatusChange(IncidentStatusChangePayload payload) =>
      _statusCtrl.add(payload);

  void dispose() {
    _escalationCtrl.close();
    _statusCtrl.close();
  }
}
```

**Patron d'abonnement dans un Notifier consommateur :**

```dart
// lib/modules/tasks/application/task_provider.dart
@riverpod
class TaskNotifier extends _$TaskNotifier {
  @override
  TaskState build() {
    // Abonnement réactif au pont — JAMAIS d'import du module Incidents
    ref.listen(incidentEscalationStreamProvider, (_, next) {
      next.whenData((payload) => _createTaskFromEscalation(payload));
    });
    return const TaskState.initial();
  }

  Future<void> _createTaskFromEscalation(IncidentEscalationPayload payload) async {
    // LOI VI — Optimistic UI
    state = state.addOptimisticTask(payload);
    final result = await _repo.createMaster(TaskMasterModel.fromPayload(payload));
    result.fold(
      (failure) => state = state.removeOptimisticTask(payload.incidentId),
      (_) => state = state.confirmTask(payload.incidentId),
    );
  }
}
```

## I.4 — Règles de Communication Inter-Module V6

| Scénario | Mécanisme V6 | Import autorisé |
|----------|-------------|-----------------|
| Module A lit données de Module B | Contrat `IXxxContract` | `core/contracts/` uniquement |
| Module A navigue vers Module B | URI GoRouter | Constante `RouteNames.*` |
| Module A réagit à événement de Module B | `ref.listen(xxxStreamProvider, ...)` | `core/providers/reactive_bridges.dart` |
| Dashboard affiche compteurs d'un module | Contrat `IXxxContract` | `core/contracts/` uniquement |
| Module absent | `ModuleRegistry` retourne null | L'UI masque le bloc |

**AppEventBus V5 :**

| Event V5 (ABOLI) | Pont Réactif V6 |
|------------------|-----------------|
| `IncidentEscalatedToTaskEvent` | `incidentEscalationStreamProvider` |
| `PaymentRecordedEvent` | `paymentRecordedStreamProvider` |
| `IncidentStatusChangedEvent` | `incidentStatusStreamProvider` |
| `TaskOverdueEvent` | `taskOverdueStreamProvider` |
| `UserLoggedOutEvent` | `authLogoutStreamProvider` |
| `WizardCompletedEvent` | `wizardCompletedStreamProvider` |

## I.5 — Doctrine Offline-First V6 — Patron d'Implémentation Obligatoire

```dart
// Patron Optimistic UI V6 — à appliquer sur CHAQUE méthode de mutation
// Exemple : PaymentNotifier.fifoPayment()

Future<void> fifoPayment({
  required String apartmentId,
  required int montantCentimes,
  required String methode,
}) async {
  // ÉTAPE 1 — Mise à jour optimiste immédiate
  final previousState = state;
  state = state.copyWith(
    balance: state.balance - montantCentimes,
    isOptimistic: true,
  );

  final key = IdempotencyService.generate(
    userId: ref.read(authContractProvider).currentUserId!,
    action: 'FIFO_${apartmentId}_${DateTime.now().toIso8601String().substring(0, 10)}',
  );

  // ÉTAPE 2 — Vérification connectivité
  final isOnline = ref.read(connectivityProvider) == ConnectivityStatus.online;

  if (!isOnline) {
    // HORS-LIGNE : Encapsulation dans la queue
    await ref.read(offlineQueueServiceProvider).enqueue(
      OfflineMutation(
        rpcName: 'fn_apply_fifo_payment',
        params: {
          'p_idempotency_key': key,
          'p_apartment_id': apartmentId,
          'p_montant_centimes': montantCentimes,
          'p_methode': methode,
        },
        idempotencyKey: key,
      ),
    );
    // État optimiste maintenu — SyncService confirmera plus tard
    return;
  }

  // EN LIGNE : Appel RPC
  final result = await ref.read(supabaseServiceProvider).rpcEither<PaymentResult>(
    'fn_apply_fifo_payment',
    params: {
      'p_idempotency_key': key,
      'p_apartment_id': apartmentId,
      'p_montant_centimes': montantCentimes,
      'p_methode': methode,
    },
  );

  result.fold(
    // Échec serveur définitif → rollback
    (failure) => state = previousState.copyWith(isOptimistic: false),
    // Succès → confirmer l'état optimiste
    (data) {
      state = state.copyWith(isOptimistic: false);
      // Émettre vers le pont réactif (notifications, dashboard...)
      ref.read(financeContractImplProvider).emitPayment(
        PaymentRecordedPayload(
          apartmentId: apartmentId,
          amountCentimes: montantCentimes,
          periode: data.periode,
        ),
      );
    },
  );
}
```

---

# ═══════════════════════════════════════════════════════════
# PARTIE II — SYSTÈME DE DESIGN V6 DUAL THÈME
# ═══════════════════════════════════════════════════════════

*Le système de design est inchangé par rapport à la V5. La palette `AppColors`, la typographie, les dimensions, les ombres, les animations et les gradients définis en V5 restent la référence normative. Seul le widget `BuildingTopologySelector` introduit un usage spécifique des tokens de couleur pour l'effet convexe 3D, détaillé en Partie III.3.*

**Rappel des tokens fondamentaux (non-négociables) :**

| Token | Light | Dark |
|-------|-------|------|
| `background` | `#F0E8DC` Alabaster | `#050508` Abyssal |
| `surface` | `#EDE2D4` Parchemin | `#0D0D14` |
| `surfaceVariant` | `#E3D5C2` Os calcifié | `#14141E` |
| `surfaceContainer` | `#F6F0E8` Linge chaud | `#1A1A28` |
| `surfaceHigh` | `#D8CAB4` Grès chaud | `#22223A` |
| `futureUnpaid` | `#9E9488` Gris chaud | `#504E64` Gris abyssal |
| `primary` | `#1A1A2E` Navy profond | `#7C9FFF` Bleu électrique |
| `accent` | `#C8916B` Terracotta | `#E8A882` Terracotta lumineux |

---

# ═══════════════════════════════════════════════════════════
# PARTIE III — AUTHENTIFICATION V6
# ═══════════════════════════════════════════════════════════

## III.1 — Bifurcation Initiale : Staff vs Résident

L'écran de bifurcation (`AuthBifurcationScreen`) est le **premier écran post-splash** (si aucune session n'est active). Il présente deux choix exclusifs :

```
┌────────────────────────────────────────────────────────┐
│         RÉSIDENCE L'AMANDIER B                        │
│              Bouskoura                                │
│                                                        │
│     ╔══════════════════╗  ╔══════════════════╗        │
│     ║                  ║  ║                  ║        │
│     ║   🏢  STAFF      ║  ║   🏠  RÉSIDENT   ║        │
│     ║ Syndic / Adjoint ║  ║  Appartement     ║        │
│     ║ Concierge / Ménage║  ║  1 à 15          ║        │
│     ║                  ║  ║                  ║        │
│     ╚══════════════════╝  ╚══════════════════╝        │
│                                                        │
│                 [Horloge + Météo]                      │
└────────────────────────────────────────────────────────┘
```

**Tap STAFF →** `context.go(RouteNames.staffRoleSelector)`
**Tap RÉSIDENT →** `context.go(RouteNames.residentBuildingSelector)`

## III.2 — Chemin Staff : `StaffRoleSelectorScreen`

Grille 4 tuiles (inchangée fonctionnellement par rapport à la V5 `RoleSelectionScreen`, mais désormais **limitée aux 4 rôles Staff**). Tap tuile → `AuthNotifier.selectStaffRole(role)` → `context.go(RouteNames.pinEntry)`.

```dart
// Rôles affichés dans StaffRoleSelectorScreen (JAMAIS AppRole.resident ici)
static const _staffRoles = [
  AppRole.syndicAlpha,
  AppRole.adjointLecture,
  AppRole.conceierge,
  AppRole.menageRestreint,
];
```

## III.3 — Chemin Résident : `ResidentBuildingSelectorScreen`

Cet écran est le **cœur de la Mutation I**. Il affiche une représentation graphique tactile et luxueuse de la Résidence L'Amandier B avant de demander tout identifiant.

**Composition de l'écran :**

```
┌────────────────────────────────────────────────────────┐
│  ← Retour          VOTRE APPARTEMENT         [thème]  │
│─────────────────────────────────────────────────────── │
│                                                        │
│  Niveau 3 ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐  │
│   Apt 11  │ 11  │ │ 12  │ │ 13  │ │ 14  │ │ 15  │  │
│  à 15     └─────┘ └─────┘ └─────┘ └─────┘ └─────┘  │
│           ══════════════════════════════════════════   │
│  Niveau 2 ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐  │
│   Apt 6   │  6  │ │  7  │ │  8  │ │  9  │ │ 10  │  │
│  à 10     └─────┘ └─────┘ └─────┘ └─────┘ └─────┘  │
│           ══════════════════════════════════════════   │
│  Niveau 1 ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐  │
│  (RDC)    │  1  │ │  2  │ │  3  │ │  4  │ │  5  │  │
│   Apt 1   └─────┘ └─────┘ └─────┘ └─────┘ └─────┘  │
│  à 5                                                   │
│                                                        │
│     Sélectionnez votre appartement pour continuer      │
└────────────────────────────────────────────────────────┘
```

**Tap sur un nœud d'appartement →** `AuthNotifier.selectApartment(apartmentNumber, apartmentId)` → `context.go(RouteNames.pinEntry)` avec `apartmentId` en paramètre de route.

## III.4 — Spécification du Widget `BuildingTopologyWidget`

```dart
// lib/shared/widgets/building_topology_widget.dart
// Widget réutilisable — utilisé dans ResidentBuildingSelectorScreen
// ET dans le dashboard Adjoint (BuildingMatrixPreview), en mode lecture seule

class BuildingTopologyWidget extends ConsumerWidget {
  final void Function(int apartmentNumber, String apartmentId)? onApartmentTap;
  final Map<String, Color>? apartmentColors; // null = mode auth (aucune coloration)
  final bool isInteractive; // false = lecture seule (dashboard)

  const BuildingTopologyWidget({
    this.onApartmentTap,
    this.apartmentColors,
    this.isInteractive = true,
    super.key,
  });
```

**Spécification visuelle — Effet Convexe 3D :**

Chaque nœud d'appartement est un `GestureDetector` wrappant un `AnimatedContainer` avec `BoxDecoration` à dégradé radial convexe :

```dart
// Thème Light — Effet convexe Alabaster
BoxDecoration _convexDecorationLight() => BoxDecoration(
  borderRadius: BorderRadius.circular(AppDimensions.radiusM),
  gradient: RadialGradient(
    center: Alignment(-0.3, -0.3), // Source lumineuse haut-gauche
    radius: 1.0,
    colors: [
      AppColors.lightSurfaceContainer, // #F6F0E8 — centre lumineux
      AppColors.lightSurface,          // #EDE2D4 — mi-chemin
      AppColors.lightSurfaceHigh,      // #D8CAB4 — bords en ombre
    ],
    stops: const [0.0, 0.5, 1.0],
  ),
  boxShadow: [
    // Ombre portée bas-droite — renforce l'illusion de relief
    BoxShadow(
      color: AppColors.lightShadow,
      offset: const Offset(2, 3),
      blurRadius: 6,
    ),
    // Surbrillance haut-gauche — highlight spéculaire
    BoxShadow(
      color: Colors.white.withOpacity(0.6),
      offset: const Offset(-1, -1),
      blurRadius: 2,
    ),
  ],
);

// Thème Dark — Effet convexe Abyssal avec lueur interne
BoxDecoration _convexDecorationDark() => BoxDecoration(
  borderRadius: BorderRadius.circular(AppDimensions.radiusM),
  gradient: RadialGradient(
    center: Alignment(-0.3, -0.3),
    radius: 1.0,
    colors: [
      AppColors.darkSurfaceHigh,       // #22223A — centre lumineux
      AppColors.darkSurfaceVariant,    // #14141E — mi-chemin
      AppColors.darkSurface,           // #0D0D14 — bords profonds
    ],
    stops: const [0.0, 0.5, 1.0],
  ),
  boxShadow: [
    BoxShadow(
      color: AppColors.darkShadow,
      offset: const Offset(2, 3),
      blurRadius: 8,
    ),
    // Lueur électrique subtile sur le bord haut-gauche (Dark uniquement)
    BoxShadow(
      color: AppColors.darkPrimary.withOpacity(0.15),
      offset: const Offset(-1, -1),
      blurRadius: 4,
    ),
  ],
);
```

**Animation de pression (press feedback) :**

```dart
// GestureDetector avec TapDown/TapUp pour simuler l'enfoncement
bool _isPressed = false;

// Sur tapDown : gradient s'aplatit (convexe → plat)
// Sur tapUp   : gradient reprend sa forme convexe
// Duration    : AppAnimations.fast (150ms)
// Curve       : Curves.easeOut
```

**Layout de la grille :**

```dart
// Structure : Column de 3 Row, ordre top → bottom (Niveau 3 affiché EN HAUT)
Column(
  children: [
    _buildFloorRow(levelLabel: 'Niveau 3', apartments: [11, 12, 13, 14, 15]),
    _buildFloorSeparator(), // Ligne décorative représentant le plancher
    _buildFloorRow(levelLabel: 'Niveau 2', apartments: [6, 7, 8, 9, 10]),
    _buildFloorSeparator(),
    _buildFloorRow(levelLabel: 'Niveau 1 (RDC)', apartments: [1, 2, 3, 4, 5]),
  ],
)
// Chaque nœud : largeur = (screenWidth - padding - 4*gap) / 5
// Hauteur nœud : width * 1.1 (légèrement portrait, proportionnel à une porte d'appartement)
```

**Label de chaque nœud :**
- Numéro de l'appartement : `AppTextStyles.headingH2`, `AppColors.lightPrimary / darkTextPrimary`
- Sous-label optionnel (si `apartmentColors` fourni) : solde coloré via `CurrencyBadge`

## III.5 — `PinEntryScreen` V6 — Context-Aware

```dart
// Route : /auth/pin?role=syndicAlpha  (staff)
//      OU /auth/pin?apartmentId=uuid  (résident)
// Le PinEntryScreen sait d'où il vient via les paramètres de route

class PinEntryScreen extends ConsumerStatefulWidget {
  final AppRole? role;         // Non-null si chemin Staff
  final String? apartmentId;   // Non-null si chemin Résident
  final int? apartmentNumber;  // Pour affichage uniquement (Résident)
  // ...
}
```

**Affichage contextuel :**
- Staff : "Connexion — [Label du rôle]"
- Résident : "Appartement [N° apt] — Votre PIN"
- `PinKeyboard(pinLength: 8)` — identique dans les deux chemins

## III.6 — FSM AuthState V6

```dart
// lib/modules/auth/application/auth_provider.dart
sealed class AuthState { const AuthState(); }

class AuthInitial          extends AuthState { const AuthInitial(); }
class AuthUnauthenticated  extends AuthState { const AuthUnauthenticated(); }

// Chemin Staff
class AuthStaffRoleSelected extends AuthState {
  final AppRole role;
  const AuthStaffRoleSelected(this.role);
}

// Chemin Résident
class AuthResidentBuildingDisplayed extends AuthState {
  const AuthResidentBuildingDisplayed();
}
class AuthResidentApartmentSelected extends AuthState {
  final int apartmentNumber;
  final String apartmentId;
  const AuthResidentApartmentSelected({
    required this.apartmentNumber,
    required this.apartmentId,
  });
}

// Commun — PIN en cours de saisie
class AuthPinPending extends AuthState {
  final AppRole? role;         // non-null si Staff
  final String? apartmentId;   // non-null si Résident
  const AuthPinPending({this.role, this.apartmentId})
    : assert(role != null || apartmentId != null,
        'AuthPinPending doit avoir soit role soit apartmentId');
}

class AuthAuthenticated extends AuthState {
  final AppRole role;
  final String userId;
  final String displayName;
  final String? apartmentId; // non-null pour AppRole.resident
  const AuthAuthenticated({required this.role, required this.userId,
    required this.displayName, this.apartmentId});
}

class AuthBlocked extends AuthState {
  final DateTime unblockedAt;
  const AuthBlocked(this.unblockedAt);
}
```

## III.7 — Flux d'Authentification V6 Complet

```
[SplashScreen]
     │ vérification session SecureStorage (< 500ms)
     ├── session valide ───────────────────────────► [DashboardRouter]
     │
     └── pas de session ──► [AuthBifurcationScreen]
                                     │
                    ┌────────────────┴────────────────┐
                    │ STAFF                  RÉSIDENT  │
                    ▼                                  ▼
           [StaffRoleSelectorScreen]   [ResidentBuildingSelectorScreen]
             4 tuiles de rôles           Bâtiment 3×5 — 15 nœuds 3D
                    │                                  │
                    │ role sélectionné    apt. tappé → apartmentId résolu
                    │                                  │
                    └────────────────┬────────────────┘
                                     ▼
                              [PinEntryScreen]
                          (context-aware : role | apartmentId)
                                8 chiffres saisis
                                hash SHA-256 côté client
                                     │
                          Staff : fn_validate_pin(role, pin_hash)
                          Résident : fn_validate_pin_resident(apartment_id, pin_hash)
                                     │
                    ┌────────────────┴────────────────┐
                    │ succès                  échec    │
                    ▼                                  ▼
              [DashboardRouter]              compteur tentatives
         session → SecureStorage       5 tentatives → blocage 15min
         (token + rôle + apartmentId?)
```

**Règles PIN invariantes V6 :**
- `pinLength = 8` (STRICTEMENT 8)
- Hash Résident : `SHA-256(pin + salt_from_server(apartmentId))`
- Hash Staff : `SHA-256(pin + salt_from_server(role))`
- Le PIN en clair n'existe jamais en mémoire plus de 50ms
- `pinMaxAttempts = 5`, `pinBlockDurationMinutes = 15`
- `SessionModel` V6 inclut `apartmentId (String?)` — nullable pour staff

---

# ═══════════════════════════════════════════════════════════
# PARTIE IV — SÉQUENCE GENÈSE V6 (WIZARD 5 ÉTAPES)
# ═══════════════════════════════════════════════════════════

## IV.1 — Philosophie de la Séquence Genèse

La Séquence Genèse est l'opération de **calibrage thermodynamique initial** de la résidence. Elle s'exécute une unique fois, au T=0 du mandat, exclusivement par le rôle `syndicAlpha`. Elle capture l'état exact du système — identité du mandat, charges structurelles, provisions, trésorerie initiale, et situation financière de chaque appartement — avant que le premier centime ne soit traité par l'application.

**Invariants absolus de la Séquence Genèse :**
1. Elle n'est accessible qu'au rôle `syndicAlpha` (SetupGuard).
2. Elle est idempotente : `fn_complete_wizard_setup` avec la même clé d'idempotency ne réécrit pas les données.
3. Tous les montants sont des `int` centimes — aucun `double`.
4. L'email du syndic est **informatif uniquement** — aucun mécanisme d'authentification par email n'est déclenché.
5. Les soldes initiaux des appartements (Step 5) peuvent être négatifs (dette antérieure), nuls, ou positifs (avance de paiement). Les avances positives sont logiquement déjà comptabilisées dans les soldes caisse/banque de Step 4 — le Step 5 ne fait qu'allouer cet actif par résident.

## IV.2 — Step 1 : Identité du Mandat

**Titre :** "Identité du Mandat"
**Sous-titre :** "Informations du syndic responsable et date de début de mandat"

Champs obligatoires :

| Champ | Type Flutter | Validation | Nom RPC |
|-------|-------------|------------|---------|
| Date de début de mandat | `DatePickerField` | ≤ aujourd'hui | `p_mandate_start_date` |
| Prénom du syndic | `TextFieldCustom` | 2-50 chars, non vide | `p_syndic_first_name` |
| Nom du syndic | `TextFieldCustom` | 2-50 chars, non vide | `p_syndic_last_name` |
| Téléphone | `TextFieldCustom` | `isValidMoroccanPhone()` | `p_syndic_phone` |
| Email | `TextFieldCustom` | format email valide | `p_syndic_email` |

> **Note critique :** L'email est stocké en base de données comme information de contact. Il ne déclenche aucun flux d'authentification. Aucun appel à `signInWithOtp`, `signInWithPassword`, ou équivalent ne peut apparaître dans le code de ce step. La présence de tout mécanisme d'auth email dans ce step constitue une **VIOLATION LOI III V6**.

## IV.3 — Step 2 : Charges Fixes Mensuelles

**Titre :** "Charges Fixes Mensuelles"
**Sous-titre :** "Dépenses structurelles récurrentes de la résidence"

Champs obligatoires (tous `AmountField`, valeur minimale 0 MAD) :

| Poste | Description | Nom RPC |
|-------|-------------|---------|
| Salaire Concierge | Salaire mensuel brut du concierge | `p_salaire_concierge_centimes` |
| Salaire Personnel Ménage | Salaire mensuel brut des agents de ménage | `p_salaire_menage_centimes` |
| Assurance Résidence | Cotisation mensuelle assurance bâtiment | `p_assurance_centimes` |
| Contrat Maintenance Ascenseur | Abonnement mensuel maintenance ascenseur | `p_maintenance_ascenseur_centimes` |
| Services Eau & Électricité | Part résidence eau et électricité communes | `p_services_eau_electricite_centimes` |

**Total charges fixes calculé Dart-side** et affiché en temps réel sous le dernier champ :
```dart
int get totalChargesFixesCentimes =>
  salaireConcierge + salaireMenage + assurance +
  maintenanceAscenseur + servicesEauElectricite;
```

## IV.4 — Step 3 : Provisions Variables Mensuelles

**Titre :** "Provisions Variables"
**Sous-titre :** "Provisions mensuelles pour dépenses aléatoires"

Champs obligatoires (tous `AmountField`, valeur minimale 0 MAD) :

| Poste | Description | Nom RPC |
|-------|-------------|---------|
| Provision Consommables | Budget mensuel fournitures et produits d'entretien | `p_provision_consommables_centimes` |
| Provision Imprévus | Réserve mensuelle pour dépenses non planifiées | `p_provision_imprevus_centimes` |

**BurnRate prévisionnel calculé Dart-side** et affiché à titre indicatif :
```dart
int get burnRatePrevisionnel =>
  totalChargesFixesCentimes + provisionConsommables + provisionImprevus;
// Affiché : "BurnRate mensuel estimé : X,XX MAD"
```

## IV.5 — Step 4 : État Zéro Absolu (Trésorerie T=0)

**Titre :** "État Zéro Absolu"
**Sous-titre :** "Situation de trésorerie exacte au [mandateStartDate]"

Ce step capture l'état physique exact des liquidités de la résidence au jour du démarrage du mandat.

Champs obligatoires (tous `AmountField`, valeur minimale 0 MAD) :

| Poste | Description | Nom RPC |
|-------|-------------|---------|
| Solde Caisse | Espèces physiquement disponibles | `p_solde_initial_caisse_centimes` |
| Solde Banque | Solde du compte bancaire de la résidence | `p_solde_initial_banque_centimes` |

**Trésorerie Totale T=0 calculée Dart-side** et affichée :
```dart
int get tresorerieInitialeTotale => caisseCentimes + banqueCentimes;
// Affiché via TreasuryGauge en miniature
```

> **Note logique :** Les avances de paiement positives que les résidents ont versées avant T=0 (Step 5) sont déjà physiquement incluses dans ces soldes. Le Step 5 effectue uniquement l'allocation comptable par appartement. Il n'y a pas de double comptabilisation.

## IV.6 — Step 5 : Vecteurs Résidents (Soldes Initiaux des 15 Appartements)

**Titre :** "Situation Financière des Résidents à T=0"
**Sous-titre :** "Solde de chaque appartement au [mandateStartDate] — valeur négative = dette antérieure"

Ce step présente une interface liste des 15 appartements. Chaque ligne contient :
- Numéro d'appartement (label fixe)
- Champ de saisie de solde — **peut être négatif, nul, ou positif**

**Spécification critique :** L'`AmountField` standard ne permet pas les valeurs négatives. Ce step utilise un widget spécialisé `SignedAmountField` (déclaré dans `wizard_widgets.dart`) qui :
- Affiche un toggle +/− précédant le montant
- Émet un `int` centimes (négatif si dette, positif si avance)
- Valide que la valeur absolue est < 100 000 MAD (plafond raisonnable)

```dart
// SignedAmountField
// Appartement 1 :  [−] [_______500,00 MAD]  → émet -50000 (dette)
// Appartement 2 :  [+] [_______000,00 MAD]  → émet 0 (à jour)
// Appartement 7 :  [+] [_______250,00 MAD]  → émet 25000 (avance)
```

**Affichage récapitulatif Dart-side en pied de liste :**
```dart
int get totalDettesCentimes =>
  apartments.where((a) => a.soldeCentimes < 0)
            .fold(0, (sum, a) => sum + a.soldeCentimes);

int get totalAvancesCentimes =>
  apartments.where((a) => a.soldeCentimes > 0)
            .fold(0, (sum, a) => sum + a.soldeCentimes);

// Affiché :
// "Dettes antérieures : X,XX MAD | Avances : Y,YY MAD"
// Note : totalAvancesCentimes ≤ tresorerieInitialeTotale (cohérence logique)
```

**Nom RPC du payload :** `p_resident_initial_balances (JSONB)` — array de `{apartment_id, solde_centimes}`.

## IV.7 — Récapitulatif Final (Step 6 — Confirmation)

Écran de synthèse non-éditable affichant l'intégralité des données saisies avant validation finale. Bouton "Finaliser la Configuration" → `WizardNotifier.finalize()` → appel `fn_complete_wizard_setup(p_idempotency_key, ...)` → émet `_authContractImpl.emitSetupCompleted()` → le pont réactif `wizardCompletedStreamProvider` déclenche l'unlock de l'app dans le shell.

---

# ═══════════════════════════════════════════════════════════
# PARTIE V — ROADMAP — 23 PHASES ACYCLIQUES STRICTES V6
# ═══════════════════════════════════════════════════════════

## Graphe de Dépendances Global

```
P0 ──► P1 ──► P2 ──► P3 ──► P4 ──► P5
                                    │
                    ┌───────────────┘
                    ▼
             P6 ──► P7 ──► P8 ──► P9
                                   │
                    ┌──────────────┘
                    ▼
            P10 ──► P11 ──► P12
                              │
                    ┌─────────┘
                    ▼
    P13 ──► P14 ──► P15 ──► P16 ──► P17 ──► P18
                                              │
                    ┌─────────────────────────┘
                    ▼
            P19 ──► P20 ──► P21 ──► P22
```

**Loi d'acyclicité :** Aucune phase ne peut commencer si ses prérequis ne sont pas compilés sans erreur et n'ont pas passé leur Gate.

---

## ════ PHASE 0 — Scaffold Projet (0 Fichiers Dart) ════

**Durée estimée :** 1 heure
**Prérequis :** Environnement Flutter 3.22+, SDK Dart 3.3+.

**Actions non-Dart obligatoires :**
- `flutter create amandier_b --org ma.syndic --platforms android,ios`
- Configurer `pubspec.yaml` :

```yaml
dependencies:
  flutter: { sdk: flutter }
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0
  go_router: ^14.0.0
  freezed_annotation: ^2.4.0
  json_annotation: ^4.9.0
  supabase_flutter: ^2.5.0
  flutter_secure_storage: ^9.0.0
  crypto: ^3.0.3
  firebase_core: ^3.0.0
  firebase_messaging: ^15.0.0
  flutter_local_notifications: ^17.0.0
  hive_flutter: ^1.1.0
  pdf: ^3.11.0
  printing: ^5.13.0
  image_picker: ^1.1.0
  flutter_image_compress: ^2.3.0
  connectivity_plus: ^6.0.0
  http: ^1.2.0
  fl_chart: ^0.68.0
  cached_network_image: ^3.3.0
  lottie: ^3.1.0
  qr_flutter: ^4.1.0
  timezone: ^0.9.0
  intl: ^0.19.0
  dartz: ^0.10.1

dev_dependencies:
  build_runner: ^2.4.0
  freezed: ^2.5.0
  json_serializable: ^6.8.0
  riverpod_generator: ^2.4.0
  custom_lint: ^0.6.0
  riverpod_lint: ^2.3.0
  flutter_test: { sdk: flutter }
```

- Créer `.env` (jamais commité).
- `flutterfire configure`, télécharger `google-services.json` / `GoogleService-Info.plist`.
- Créer la structure de répertoires : `lib/core/`, `lib/shared/`, `lib/modules/`, `lib/shell/`.
- **Vérification critique :** Aucun répertoire `lib/core/events/` ne doit exister. Il n'existera jamais dans ce projet.

### ⚔️ GATE P0
- [ ] `flutter pub get` → 0 erreur. **Si non → ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur sur le squelette vide. **Si non → ÉCHEC.**
- [ ] `.env` dans `.gitignore`. **Si non → ÉCHEC SÉCURITÉ CRITIQUE.**
- [ ] `google-services.json` dans `.gitignore`. **Si non → ÉCHEC SÉCURITÉ CRITIQUE.**
- [ ] Répertoire `lib/core/events/` **inexistant**. **Si créé → VIOLATION LOI V — ÉCHEC.**

---

## ════ PHASE 1 — Design System V6 Dual Thème ════

> 🔒 **RAPPEL ABSOLU PHASE 1 :**
> (1) `AppColors.lightBackground` = `Color(0xFFF0E8DC)` — `#FFFFFF` est **interdit**.
> (2) Zéro valeur hex hardcodée en dehors de `AppColors`.
> (3) Aucun répertoire events dans le projet.

**Fichiers `.dart` livrés : 7**
**Durée estimée :** 1 jour | **Prérequis :** PHASE 0

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/core/design/app_colors.dart` | Palette complète Light Alabaster + Abyssal Dark. Tous tokens typés. `lightFutureUnpaid = Color(0xFF9E9488)` gris chaud — jamais rouge. |
| 2 | `lib/core/design/app_text_styles.dart` | Inter + Playfair Display + JetBrains Mono. |
| 3 | `lib/core/design/app_dimensions.dart` | Espacement `space4`→`space64`, rayons, hauteurs composants. |
| 4 | `lib/core/design/app_shadows.dart` | Ombres sémantiques `level1`→`level4` Light ; élévations lumineuses Dark. `AppShadows.forTheme(bool isDark)`. |
| 5 | `lib/core/design/app_animations.dart` | Durées (fast:150ms, normal:250ms, slow:400ms), courbes. |
| 6 | `lib/core/design/app_gradients.dart` | `heroGradientLight`, `heroGradientDark`, `shimmerGradient`. |
| 7 | `lib/core/design/app_theme.dart` | `AppTheme.light` + `AppTheme.dark` complets. Aucune couleur laissée à Material par défaut. |

### ⚔️ GATE P1
- [ ] `AppTheme.light` et `AppTheme.dark` s'appliquent sans crash. **Si non → ÉCHEC.**
- [ ] Zéro `Color(0xFF...)` hardcodé hors `app_colors.dart`. **Si oui → ÉCHEC.**
- [ ] `AppColors.lightBackground` = `Color(0xFFF0E8DC)`. Si `Color(0xFFFFFFFF)` → **VIOLATION LOI IV — ÉCHEC.**
- [ ] `AppColors.lightFutureUnpaid` est un gris chaud (non rouge). **Si rouge → ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 2 — Kernel Core V6 (Constants, Erreurs, Extensions, Utils) ════

> 🔒 **RAPPEL ABSOLU PHASE 2 :**
> (1) **Zéro fichier `app_event_bus.dart` ou `app_events.dart`** — ces fichiers sont définitivement bannis.
> (2) Aucun `StreamController` global dans cette phase.
> (3) `AppConstants.pinLength == 8` — invariant absolu.

**Fichiers `.dart` livrés : 11**
**Durée estimée :** 1 jour | **Prérequis :** PHASE 1

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/core/constants.dart` | `AppConstants` : URLs env, coordonnées Bouskoura (`lat:33.4590/lon:-7.6564`), timezone `Africa/Casablanca`, `pinLength = 8`, `pinMaxAttempts = 5`, `burnRateMonths = 3`, `defaultPageSize = 20`, `totalApartments = 15`, `cotisationMensuelle = 25000` (250 MAD fixe), `supabaseProjectRef = 'fypbkzlfyjjchaislaia'`. |
| 2 | `lib/core/errors/failure.dart` | `sealed class Failure` : `ServerFailure`, `NetworkFailure`, `CacheFailure`, `ValidationFailure`, `PermissionFailure`, `NotFoundFailure`, `AuthFailure`, `PinBlockedFailure`. |
| 3 | `lib/core/errors/app_exception.dart` | Mapping `PostgrestException` → `Failure`. `42501` → `PermissionFailure`, `23505` → `ServerFailure`, `PGRST116` → `NotFoundFailure`. |
| 4 | `lib/core/errors/error_handler.dart` | `ErrorHandler.handle(Object, StackTrace?) → Failure`. Logging structuré sans PII. |
| 5 | `lib/core/extensions/string_extensions.dart` | `capitalize`, `toInitials`, `maskPin`, `toPeriodeDisplay`, `isValidMoroccanPhone`. |
| 6 | `lib/core/extensions/datetime_extensions.dart` | `toPeriode`, `toDisplayDate`, `isSameMonth`, `toBouskouraTime`, `isInPast`, `isInFuture`. |
| 7 | `lib/core/extensions/num_extensions.dart` | `toCentimes`, `toMad`, `toFormattedMad`, `clampPositive`, `toPercent`. |
| 8 | `lib/core/extensions/context_extensions.dart` | `context.theme`, `context.colors`, `context.isDark`, `context.textStyles`, `context.screenWidth`, `context.screenHeight`. |
| 9 | `lib/core/utils/amount_formatter.dart` | `AmountFormatter.format(int centimes) → String`. Formatage MAD localisé FR. |
| 10 | `lib/core/utils/date_formatter.dart` | `DateFormatter.toDisplay(DateTime) → String`. Localisation FR, fuseau Casablanca. |
| 11 | `lib/core/utils/validators.dart` | Validateurs métier : phone, email, pin, montant, UUID. |

### ⚔️ GATE P2
- [ ] **Zéro fichier `app_event_bus.dart` ou `app_events.dart` dans tout le projet.** Grep → 0 résultat. **Si trouvé → VIOLATION LOI V — ÉCHEC ABSOLU.**
- [ ] `Failure` est une `sealed class`. **Si non → ÉCHEC.**
- [ ] `AppConstants.pinLength == 8`. **Si 4 → VIOLATION LOI III — ÉCHEC.**
- [ ] `AppException` mappe `PGRST116` en `NotFoundFailure`. **Si non → ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 3 — Contrats Modulaires V6 ════

> 🔒 **RAPPEL ABSOLU PHASE 3 :**
> (1) Cette phase définit le **seul langage inter-module autorisé**. Les streams déclarés ici remplaceront l'AppEventBus.
> (2) Les payloads de streams (`IncidentEscalationPayload`, etc.) sont des `@immutable` value objects définis dans ce répertoire.
> (3) Aucune logique métier — interfaces pures.

**Fichiers `.dart` livrés : 10**
**Durée estimée :** 1 jour | **Prérequis :** PHASE 2

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/core/contracts/i_module_registrar.dart` | `IModuleRegistrar` + `QuickAction`. |
| 2 | `lib/core/contracts/i_auth_contract.dart` | `IAuthContract` V6 : `currentRole`, `currentUserId`, `currentApartmentId`, `isAuthenticated`, `authStream`, `watchLogout()`, `watchSetupCompleted()`. |
| 3 | `lib/core/contracts/i_finance_contract.dart` | `IFinanceContract` V6 : méthodes V5 + `watchPayments() → Stream<PaymentRecordedPayload>`. Value objects : `TreasurySnapshot`, `PaymentRecordedPayload`. |
| 4 | `lib/core/contracts/i_incident_contract.dart` | `IIncidentContract` V6 : méthodes V5 + `watchEscalations() → Stream<IncidentEscalationPayload>` + `watchStatusChanges() → Stream<IncidentStatusChangePayload>`. Value objects : `IncidentEscalationPayload`, `IncidentStatusChangePayload`, `IncidentCreationPayload`. |
| 5 | `lib/core/contracts/i_task_contract.dart` | `ITaskContract` V6 : méthodes V5 + `watchOverdue() → Stream<TaskOverduePayload>`. Value objects : `TaskCreationPayload`, `TaskOverduePayload`. |
| 6 | `lib/core/contracts/i_hr_contract.dart` | `IHrContract` : `getActiveStaff()`. Value object : `StaffSummary`. |
| 7 | `lib/core/contracts/i_annonce_contract.dart` | `IAnnonceContract` : `watchActiveAnnonceCount()`, `getLatestAnnonces(int limit)`. Value object : `AnnoncePreview`. |
| 8 | `lib/core/contracts/i_notification_contract.dart` | `INotificationContract` : `watchUnreadCount()`, `markAllRead()`. |
| 9 | `lib/core/contracts/i_document_contract.dart` | `IDocumentContract` : `generateReceiptUrl(ReceiptRequest)`, `generateExpenseVoucherUrl(ExpenseVoucherRequest)`. Value objects correspondants. |
| 10 | `lib/core/contracts/i_profile_contract.dart` | `IProfileContract` : `getProfile(String userId)`, `updateProfile(UserProfile)`. Value object : `UserProfile`. |

### ⚔️ GATE P3
- [ ] Toutes les interfaces sont des `abstract interface class`. **Si non → ÉCHEC.**
- [ ] Les méthodes `watchXxx()` retournent des `Stream<XxxPayload>` où `XxxPayload` est déclaré dans le même fichier. **Si non → ÉCHEC.**
- [ ] Aucun type de retour n'est un modèle d'un module futur. **Si oui → VIOLATION LOI I — ÉCHEC.**
- [ ] `TreasurySnapshot`, `IncidentEscalationPayload`, `TaskOverduePayload`, etc. sont des `@immutable const`-compatibles. **Si non → ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 4 — Shell : Router, Guards, Providers Globaux, Points d'Entrée ════

> 🔒 **RAPPEL ABSOLU PHASE 4 :**
> (1) `reactive_bridges.dart` est livré dans cette phase. Il expose les `StreamProvider` inter-modules.
> (2) `app_router.dart` ne peut importer aucun module.
> (3) `RouteNames` V6 inclut les nouvelles routes auth : `authBifurcation`, `staffRoleSelector`, `residentBuildingSelector`.

**Fichiers `.dart` livrés : 11** *(tolérance +1 — reactive_bridges est structurellement lié aux providers)*
**Durée estimée :** 1 jour | **Prérequis :** PHASE 3

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/core/router/route_names.dart` | `RouteNames` : toutes les URIs. Nouvelles V6 : `authBifurcation = '/auth'`, `staffRoleSelector = '/auth/staff'`, `residentBuildingSelector = '/auth/resident/building'`, `pinEntry = '/auth/pin'`. |
| 2 | `lib/core/router/auth_guard.dart` | Redirect vers `RouteNames.authBifurcation` si non authentifié. |
| 3 | `lib/core/router/role_guard.dart` | `RoleGuard(Set<AppRole> allowed)` — redirect `/unauthorized` si rôle non autorisé. |
| 4 | `lib/core/router/setup_guard.dart` | Redirect vers `RouteNames.wizard` si `setup_completed == false` ET rôle == `syndicAlpha`. |
| 5 | `lib/core/router/app_router.dart` | `GoRouter` principal. Routes statiques : splash, authBifurcation, staffRoleSelector, residentBuildingSelector, pinEntry, unauthorized, wizardShell. Routes dynamiques modules : `ref.watch(moduleRoutesProvider)`. |
| 6 | `lib/core/providers/supabase_provider.dart` | `supabaseClientProvider` singleton. |
| 7 | `lib/core/providers/auth_state_provider.dart` | `authStateProvider` — écoute `IAuthContract.authStream`. |
| 8 | `lib/core/providers/theme_provider.dart` | `themeProvider` — `ThemeMode`, persisté `SecureStorage`. |
| 9 | `lib/core/providers/connectivity_provider.dart` | `connectivityProvider` — `ConnectivityStatus` temps réel. |
| 10 | `lib/core/providers/reactive_bridges.dart` | **NOUVEAU V6.** 6 `StreamProvider` inter-modules + 6 providers de contrats (`financeContractProvider`, etc.). Voir Partie I.3 pour le code complet. |
| 11 | `lib/app.dart` + `lib/main.dart` | `AmandierApp` : `ProviderScope`, `MaterialApp.router`, init Supabase, Hive, Timezone. *(binôme compté pour 1 fichier quota)* |

### ⚔️ GATE P4
- [ ] `reactive_bridges.dart` existe dans `lib/core/providers/`. **Si absent → VIOLATON LOI V — ÉCHEC.**
- [ ] `reactive_bridges.dart` ne contient **aucun** `StreamController` — uniquement des `StreamProvider` dérivant des contrats. **Si StreamController présent ici → MAUVAISE ARCHITECTURE — ÉCHEC.**
- [ ] `AppRouter` ne contient aucun import de `lib/modules/`. **Si oui → VIOLATION LOI I — ÉCHEC.**
- [ ] `RouteNames.residentBuildingSelector` existe. **Si absent → MUTATION I NON IMPLÉMENTÉE — ÉCHEC.**
- [ ] L'app lance sur simulateur, affiche le splash sans crash. **Si non → ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 5 — Services Core ════

> 🔒 **RAPPEL ABSOLU PHASE 5 :**
> (1) `PdfGenerationService` : uniquement `Map<String, dynamic>` primitives via `SendPort`.
> (2) `OfflineQueueService` : queue FIFO Hive — aucune perte de mutation offline.
> (3) `SyncService` : rejoue la queue au retour de connectivité, invalide les providers stale.

**Fichiers `.dart` livrés : 9**
**Durée estimée :** 2 jours | **Prérequis :** PHASE 4

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/core/services/supabase_service.dart` | `rpcEither<T>()`, `selectEither<T>()`, retry 3x backoff exponentiel. |
| 2 | `lib/core/services/secure_storage_service.dart` | CRUD `FlutterSecureStorage` : PIN token, rôle session, `apartmentId` session (V6), thème, tentatives PIN, block timestamp. |
| 3 | `lib/core/services/fcm_service.dart` | Init FCM, réception push, canal `alarm_channel`. `onMessage` → `ref.read(fcmEventProvider).add(...)` (Riverpod pur, pas de bus). |
| 4 | `lib/core/services/pdf_generation_service.dart` | `compute()` isolate-safe. `Map<String, dynamic>` primitives uniquement. Retourne `Uint8List`. |
| 5 | `lib/core/services/idempotency_service.dart` | `generate({userId, action, extra?}) → String` : SHA-256 déterministe. |
| 6 | `lib/core/services/image_upload_service.dart` | Upload Supabase Storage, compression 1024px JPEG 80%, retry. `Either<Failure, String>`. |
| 7 | `lib/core/services/weather_service.dart` | OpenWeatherMap Bouskoura, cache Hive 30min. Retourne `WeatherData`. |
| 8 | `lib/core/services/offline_queue_service.dart` | Queue FIFO `OfflineMutation` (Hive box `'offline_mutations'`). `enqueue()`, `processQueue()`. `maxRetryCount = 3`. Structure `OfflineMutation` : `rpcName`, `params`, `idempotencyKey`, `retryCount`. |
| 9 | `lib/core/services/sync_service.dart` | Écoute `connectivityProvider`. Sur retour online : `OfflineQueueService.processQueue()` → invalide providers stale. |

### ⚔️ GATE P5
- [ ] `PdfGenerationService` : uniquement `Map<String, dynamic>` primitives dans `SendPort`. **Si non → CRASH ISOLATE PRODUCTION.**
- [ ] `OfflineQueueService.enqueue()` persiste la mutation. Après restart simulé, `processQueue()` la retrouve. **Si non → AMNÉSIE OFFLINE — ÉCHEC.**
- [ ] `IdempotencyService.generate()` est déterministe pour mêmes paramètres + même jour. **Si non → ÉCHEC.**
- [ ] `SecureStorage` V6 inclut méthodes `setApartmentId(String)` et `getApartmentId() → String?`. **Si absent → MUTATION I NON SUPPORTÉE — ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 6 — Shared Widgets L1 : Boutons, Dialogs, Loaders ════

*(Inchangée fonctionnellement par rapport à V5. Rappel des points critiques.)*

**Fichiers `.dart` livrés : 10**
**Durée estimée :** 1 jour | **Prérequis :** PHASE 1

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/shared/widgets/buttons/primary_button.dart` | `PrimaryButton({label, onPressed, isLoading, isDisabled})`. |
| 2 | `lib/shared/widgets/buttons/secondary_button.dart` | Outlined, adaptatif thème. |
| 3 | `lib/shared/widgets/buttons/danger_button.dart` | Destructif rouge. Dialog de confirmation interne obligatoire. |
| 4 | `lib/shared/widgets/buttons/icon_action_button.dart` | FAB-style, small/medium/large. |
| 5 | `lib/shared/widgets/dialogs/confirm_dialog.dart` | `Future<bool>`. |
| 6 | `lib/shared/widgets/dialogs/amount_input_dialog.dart` | Émet `int` centimes. Jamais `double`. |
| 7 | `lib/shared/widgets/dialogs/loading_dialog.dart` | Overlay non-dismissible. |
| 8 | `lib/shared/widgets/dialogs/error_dialog.dart` | `Failure` → message FR + retry callback. |
| 9 | `lib/shared/widgets/dialogs/success_dialog.dart` | Lottie check, auto-dismiss 2s. |
| 10 | `lib/shared/widgets/loaders/shimmer_loader.dart` | `ShimmerLoader.lines(n)`, `.card()`, `.grid(rows, cols)`. |

### ⚔️ GATE P6
- [ ] Zéro `Color(0xFF...)` hors `app_colors.dart`. **Si oui → ÉCHEC.**
- [ ] `AmountInputDialog` émet `int` centimes. **Si double → VIOLATION — ÉCHEC.**
- [ ] Aucun import de `lib/modules/`. **Si oui → VIOLATION LOI I — ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 7 — Shared Widgets L2 : Formulaires & Layouts ════

*(Inchangée par rapport à V5 + ajout `SignedAmountField` pour le Wizard Step 5.)*

**Fichiers `.dart` livrés : 10**
**Durée estimée :** 1 jour | **Prérequis :** PHASE 6

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/shared/widgets/forms/amount_field.dart` | Formatage MAD temps réel, émet `int` centimes. `AmountField({label, onChanged(int centimes), initialCentimes?})`. |
| 2 | `lib/shared/widgets/forms/text_field_custom.dart` | `TextFormField` stylisé. |
| 3 | `lib/shared/widgets/forms/date_picker_field.dart` | Date localisée FR, `DD/MM/YYYY`. |
| 4 | `lib/shared/widgets/forms/dropdown_field.dart` | **Interdit : paramètre `initialValue`.** Paramètre correct : `value`. |
| 5 | `lib/shared/widgets/forms/photo_picker_field.dart` | Galerie + caméra, max 5 photos. |
| 6 | `lib/shared/widgets/forms/signed_amount_field.dart` | **NOUVEAU V6.** `SignedAmountField({label, onChanged(int centimes), initialCentimes?})`. Toggle +/−, émet centimes signés (négatif = dette). Utilisé exclusivement dans Wizard Step 5. |
| 7 | `lib/shared/widgets/layouts/app_scaffold.dart` | Scaffold de base, `AppTopBar` optionnel, `BottomNavBar` conditionnel. |
| 8 | `lib/shared/widgets/layouts/sliver_header.dart` | En-tête collapsible. |
| 9 | `lib/shared/widgets/layouts/section_header.dart` | `SectionHeader({title, action?})`. |
| 10 | `lib/shared/widgets/layouts/empty_state.dart` | `EmptyState({lottieAsset, title, message, ctaLabel?, onCta?})`. |

> **Note quota :** `signed_amount_field.dart` remplace `full_screen_loader.dart` (déplacé en P6 sous `loaders/`). Le quota reste 10.

### ⚔️ GATE P7
- [ ] `DropdownField` n'a **aucun** paramètre `initialValue`. **Si oui → RÉGRESSION — ÉCHEC.**
- [ ] `SignedAmountField` émet bien des centimes négatifs pour une dette. Test : toggle `−` + saisie 250 → `onChanged(-25000)`. **Si non → WIZARD STEP 5 CASSÉ — ÉCHEC.**
- [ ] `AmountField` émet `int` centimes (jamais `double`). **Si non → ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 8 — Shared Widgets L3 : Navigation, Contextuels & BuildingTopology ════

> 🔒 **RAPPEL ABSOLU PHASE 8 :**
> (1) `BuildingTopologyWidget` est livré dans cette phase. Il encapsule toute la logique 3D convexe.
> (2) `ContextualHeader` reste non-optionnel dans tous les dashboards.
> (3) `PinKeyboard` affiche 8 points — `pinLength = 8` inviolable.
> (4) `ClockWidget` : `_timer?.cancel()` **obligatoire** dans `dispose()`.

**Fichiers `.dart` livrés : 11** *(tolérance +1 — BuildingTopologyWidget est architecturalement lié à cette phase)*
**Durée estimée :** 1.5 jours | **Prérequis :** PHASE 5, PHASE 7

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/shared/widgets/navigation/bottom_nav_bar.dart` | Items rôle-dépendants, badge notifications. |
| 2 | `lib/shared/widgets/navigation/app_top_bar.dart` | `AppTopBar`, `ThemeToggle`. |
| 3 | `lib/shared/widgets/navigation/role_drawer.dart` | Avatar, nom, rôle, navigation secondaire, déconnexion. |
| 4 | `lib/shared/widgets/contextual/clock_widget.dart` | `Timer.periodic(1s)`, `Africa/Casablanca`, `_timer?.cancel()` **obligatoire**. |
| 5 | `lib/shared/widgets/contextual/weather_widget.dart` | Météo Bouskoura, skeleton pendant chargement. |
| 6 | `lib/shared/widgets/contextual/contextual_header.dart` | Conteneur obligatoire, `ClockWidget` + `WeatherWidget` intégrés, hauteur 64dp. |
| 7 | `lib/shared/widgets/pin_keyboard.dart` | Clavier 3×4, **8 points indicateurs** (`pinLength = 8`), shake + vibration sur erreur. |
| 8 | `lib/shared/widgets/building_topology_widget.dart` | **NOUVEAU V6.** `BuildingTopologyWidget({onApartmentTap?, apartmentColors?, isInteractive})`. Grille 3×5 en 3 niveaux. Effet convexe 3D via `RadialGradient` + `BoxShadow` (Light: Alabaster convexe; Dark: Abyssal lumineux). Animation press 150ms. `AppColors.*` exclusif. Spécification complète en Partie III.4. |
| 9 | `lib/shared/widgets/currency_badge.dart` | Badge montant MAD coloré vert/rouge/gris. |
| 10 | `lib/shared/widgets/status_badge.dart` | `StatusBadge({label, color, icon?})`. |
| 11 | `lib/shared/widgets/role_chip.dart` | `RoleChip({role})`, couleur sémantique. |

### ⚔️ GATE P8
- [ ] `BuildingTopologyWidget` affiche exactement 15 appartements en 3 rangées (1-5, 6-10, 11-15). Niveau 3 EN HAUT. **Si non → MUTATION I DÉFAILLANTE — ÉCHEC.**
- [ ] `BuildingTopologyWidget` utilise `RadialGradient` pour l'effet convexe — jamais de `Color(0xFF...)` hardcodé. **Si non → VIOLATION LOI IV — ÉCHEC.**
- [ ] `ContextualHeader` ne peut pas compiler sans `ClockWidget` ET `WeatherWidget`. **Si non → ÉCHEC.**
- [ ] `ClockWidget.dispose()` contient `_timer?.cancel()`. **Si non → FUITE MÉMOIRE — ÉCHEC.**
- [ ] `PinKeyboard` affiche **8** points par défaut. **Si 4 → VIOLATION LOI III — ÉCHEC.**
- [ ] Aucun import de `lib/modules/`. **Si oui → VIOLATION LOI I — ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 9 — Shared Widgets L4 : Cards, Listes, Widgets Spécialisés ════

*(Inchangée par rapport à V5.)*

**Fichiers `.dart` livrés : 10**
**Durée estimée :** 1 jour | **Prérequis :** PHASE 7 + PHASE 8

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/shared/widgets/cards/info_card.dart` | `InfoCard({title, value, icon?, color?})`. |
| 2 | `lib/shared/widgets/cards/stat_card.dart` | KPI card Playfair Display, variation `+X%`/`-X%`. |
| 3 | `lib/shared/widgets/cards/transaction_card.dart` | `TransactionCard({label, amountCentimes, date, status})`. |
| 4 | `lib/shared/widgets/cards/apartment_card.dart` | `ApartmentCard({number, residentName, balanceCentimes, onTap})`. Couleurs seuils `lightDebt*`. |
| 5 | `lib/shared/widgets/cards/incident_card.dart` | `IncidentCard({title, status, priority, daysOpen, onTap})`. |
| 6 | `lib/shared/widgets/cards/task_card.dart` | `TaskCard({title, recurrence, status, hasPhotoProof, onTap})`. |
| 7 | `lib/shared/widgets/lists/paginated_list.dart` | Cursor-pagination stricte (`cursor: String?` — jamais `page` ou `offset`). Pull-to-refresh. |
| 8 | `lib/shared/widgets/lists/transaction_list_tile.dart` | Tile `const` constructeur. |
| 9 | `lib/shared/widgets/qr_code_widget.dart` | `QrCodeWidget({data, size})`. SHA-256 du contenu encodé. |
| 10 | `lib/shared/widgets/treasury_gauge.dart` | `TreasuryGauge({caisseCentimes, banqueCentimes, targetCentimes})`. `fl_chart`. |

### ⚔️ GATE P9
- [ ] `PaginatedList` : zéro paramètre `page` ou `offset`. Uniquement `cursor: String?`. **Si non → ANTI-PATTERN — ÉCHEC.**
- [ ] `ApartmentCard` : futur impayé jamais rouge — `lightFutureUnpaid`. **Si rouge → ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 10 — Module Auth V6 ════

> 🔒 **RAPPEL ABSOLU PHASE 10 :**
> (1) Ce module livre **13 fichiers** (vs 10 en V5) — les 3 nouveaux écrans de la Mutation I.
> (2) `auth_repository_impl.dart` V6 appelle **deux RPC distincts** : `fn_validate_pin` (staff) et `fn_validate_pin_resident(apartmentId, pinHash)` (résident).
> (3) `AuthNotifier` V6 gère les états `AuthResidentBuildingDisplayed` et `AuthResidentApartmentSelected`.
> (4) `logout()` V6 appelle `_authContractImpl.emitLogout()` — remplace l'ancien `AppEventBus.emit(UserLoggedOutEvent())`.
> (5) Zéro import cross-module dans `lib/modules/auth/`.

**Fichiers `.dart` livrés : 13**
**Durée estimée :** 2.5 jours | **Prérequis :** PHASE 9

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/modules/auth/auth_module.dart` | `AuthModule implements IModuleRegistrar`. `onAppStart` : enregistre `authContractProvider` override dans le container. |
| 2 | `lib/modules/auth/domain/app_role.dart` | `AppRole` enum **5 valeurs exactes** : `syndicAlpha`, `adjointLecture`, `conceierge`, `menageRestreint`, `resident`. |
| 3 | `lib/modules/auth/domain/user_model.dart` | `@freezed UserModel` : `id`, `name`, `phone`, `role`, `apartmentId (String?)`, `lastSeenAt`. |
| 4 | `lib/modules/auth/domain/session_model.dart` | `@freezed SessionModel` V6 : `userId`, `role`, `token`, `apartmentId (String?)`, `createdAt`. **`apartmentId` non-null pour `AppRole.resident`.** |
| 5 | `lib/modules/auth/data/auth_repository.dart` | Interface : `validatePinStaff(AppRole, String pinHash)`, `validatePinResident(String apartmentId, String pinHash)`, `setPin(...)`, `getCurrentUser()`, `logout()`. |
| 6 | `lib/modules/auth/data/auth_repository_impl.dart` | Staff → `fn_validate_pin(role, pin_hash)`. Résident → `fn_validate_pin_resident(apartment_id, pin_hash)`. Salt récupéré via `fn_get_auth_salt(role?)` ou `fn_get_auth_salt_resident(apartment_id)`. PIN en clair jamais > 50ms en mémoire. |
| 7 | `lib/modules/auth/application/auth_provider.dart` | `@riverpod AuthNotifier` : FSM V6 complète (Partie III.6). `selectStaffRole(AppRole)`, `displayBuilding()`, `selectApartment(int, String)`, `submitPin(String)`, `logout()`. |
| 8 | `lib/modules/auth/application/auth_contract_impl.dart` | `AuthContractImpl implements IAuthContract` V6. Maintient `StreamController.broadcast()` pour `watchLogout()` et `watchSetupCompleted()`. `emitLogout()` et `emitSetupCompleted()` appelés par `AuthNotifier`. |
| 9 | `lib/modules/auth/application/pin_provider.dart` | `@riverpod PinNotifier` : comptage tentatives, `isBlocked`, `remainingAttempts`. Block après 5 tentatives. |
| 10 | `lib/modules/auth/presentation/splash_screen.dart` | Vérification session `SecureStorage` (< 500ms). Redirect dashboard si valide, `authBifurcation` sinon. |
| 11 | `lib/modules/auth/presentation/auth_bifurcation_screen.dart` | **NOUVEAU V6.** Deux grandes tuiles tactiles : STAFF et RÉSIDENT. `ContextualHeader` intégré. Fond Alabaster. Navigation via `context.go(RouteNames.staffRoleSelector)` ou `context.go(RouteNames.residentBuildingSelector)`. |
| 12 | `lib/modules/auth/presentation/staff_role_selector_screen.dart` | **NOUVEAU V6.** Grille 4 tuiles rôles Staff uniquement. Tap → `AuthNotifier.selectStaffRole(role)` → `context.go(RouteNames.pinEntry)`. |
| 13 | `lib/modules/auth/presentation/resident_building_selector_screen.dart` | **NOUVEAU V6.** Instancie `BuildingTopologyWidget(isInteractive: true, onApartmentTap: ...)`. Tap nœud → `AuthNotifier.selectApartment(number, id)` → `context.go(RouteNames.pinEntry, extra: {apartmentId: id})`. |
| *(inclus)* | `lib/modules/auth/presentation/pin_entry_screen.dart` | **MODIFIÉ V6.** Context-aware : affiche "Rôle: [label]" pour staff, "Appartement [N]" pour résident. Reçoit `role?` ou `apartmentId?` via route params. `PinKeyboard(pinLength: 8)`. |

> **Note quota :** `pin_entry_screen.dart` est inclus dans les 13 fichiers livrés. Il est modifié mais pas compté en plus.

### ⚔️ GATE P10
- [ ] `AppRole` a **exactement 5 valeurs**. Aucun `EMPLOYE_ADMIN`. **Si 6 → VIOLATION SPEC — ÉCHEC.**
- [ ] `resident_building_selector_screen.dart` existe et compile. **Si absent → MUTATION I NON LIVRÉE — ÉCHEC ABSOLU.**
- [ ] `auth_bifurcation_screen.dart` : tap RÉSIDENT navigue vers `RouteNames.residentBuildingSelector`, tap STAFF vers `RouteNames.staffRoleSelector`. **Si inversé → ÉCHEC.**
- [ ] `auth_repository_impl.dart` appelle `fn_validate_pin_resident` pour les résidents, `fn_validate_pin` pour le staff. Zéro chemin commun non-différencié. **Si non → MUTATION I DÉFAILLANTE — ÉCHEC.**
- [ ] `SessionModel.apartmentId` est non-null après auth résident. **Si null → PERTE CONTEXTE RÉSIDENT — ÉCHEC.**
- [ ] `AuthContractImpl` expose `watchLogout()` et `watchSetupCompleted()` via `StreamController.broadcast()`. **Si `StreamController` absent → LOI V VIOLÉE — ÉCHEC.**
- [ ] `logout()` appelle `_authContractImpl.emitLogout()` — **AUCUN** `AppEventBus.emit(...)`. **Si bus utilisé → VIOLATION LOI V — ÉCHEC ABSOLU.**
- [ ] PIN hash : jamais en clair dans les appels RPC. **Si non → FAILLE SÉCURITÉ CRITIQUE.**
- [ ] `lib/modules/auth/` : zéro import cross-module. **Si oui → VIOLATION LOI I — ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 11 — Module Wizard Séquence Genèse ════

> 🔒 **RAPPEL ABSOLU PHASE 11 :**
> (1) Le Wizard V6 suit **strictement les 5 étapes Genèse** de la Partie IV. Aucune autre structure de steps n'est acceptée.
> (2) L'email du syndic (Step 1) est **purement informatif** — aucun mécanisme d'auth email.
> (3) `SignedAmountField` (Phase 7) est utilisé dans Step 5 pour les soldes résidents négatifs.
> (4) `fn_complete_wizard_setup` est idempotent via clé d'idempotency.
> (5) `WizardNotifier.finalize()` → appelle `_authContractImpl.emitSetupCompleted()` (Riverpod pur, pas de bus).

**Fichiers `.dart` livrés : 10**
**Durée estimée :** 2 jours | **Prérequis :** PHASE 10

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/modules/wizard/domain/wizard_state_model.dart` | `@freezed WizardStateModel` V6. Champs Step 1 : `mandateStartDate`, `syndicFirstName`, `syndicLastName`, `syndicPhone`, `syndicEmail`. Step 2 : `salaireConcierge`, `salaireMenage`, `assurance`, `maintenanceAscenseur`, `servicesEauElectricite` (tous `int?` centimes). Step 3 : `provisionConsommables`, `provisionImprevus` (int?). Step 4 : `soldeInitialCaisse`, `soldeInitialBanque` (int?). Step 5 : `residentVectors (Map<String, int>?)` — clé `apartmentId`, valeur centimes signés. Persisté entre étapes. |
| 2 | `lib/modules/wizard/domain/mandate_config_model.dart` | `@freezed MandateConfigModel` : `setupFinalizedAt (DateTime?)` nullable, `setupFinalizedBy (String?)`. |
| 3 | `lib/modules/wizard/domain/residence_config_model.dart` | `@freezed ResidenceConfigModel` : `setupCompleted`, `cotisationMensuelle (int centimes)`, `totalApartments`. |
| 4 | `lib/modules/wizard/data/wizard_repository.dart` | Interface : `saveStep(WizardStep, Map<String, dynamic>)`, `finalize(String idempotencyKey)`. |
| 5 | `lib/modules/wizard/data/wizard_repository_impl.dart` | Appelle `fn_complete_wizard_setup` (idempotent), `fn_provision_resident_accesses`, `fn_backfill_cotisations`. Clé idempotency obligatoire. Payload inclut tous les champs Step 1-5. |
| 6 | `lib/modules/wizard/application/wizard_provider.dart` | `@riverpod WizardNotifier` : `WizardStateModel`, `nextStep()`, `prevStep()`, `saveStep(step, data)`, `finalize()`. `finalize()` → `authContractImpl.emitSetupCompleted()` via `ref.read(authContractProvider)`. **Zéro `AppEventBus.emit(WizardCompletedEvent())`.** |
| 7 | `lib/modules/wizard/presentation/wizard_shell_screen.dart` | Shell Wizard : `ContextualHeader`, barre progression **5/5** (Step 1→5), navigation Précédent/Suivant. |
| 8 | `lib/modules/wizard/presentation/step_screens.dart` | **6 classes dans 1 fichier** : `Step1MandateIdentity`, `Step2ChargesFixesMensuelles`, `Step3ProvisionsVariables`, `Step4EtatZeroAbsolu`, `Step5VecteursResidents`, `StepFinalRecapitulatif`. Chaque step valide ses champs avant `nextStep()`. Step 5 utilise `SignedAmountField` pour les 15 appartements. |
| 9 | `lib/modules/wizard/presentation/widgets/wizard_widgets.dart` | `WizardProgressBar` (5 étapes avec labels FR), `WizardSummaryRow` (ligne récapitulatif clé:valeur), `WizardApartmentSoldeRow` (ligne appartement avec `SignedAmountField`). |
| 10 | `lib/modules/wizard/wizard_module.dart` | `WizardModule implements IModuleRegistrar`. Route : `RouteNames.wizard`. QuickActions : aucune. `onAppStart` : rien. |

### ⚔️ GATE P11
- [ ] `SetupGuard` bloque l'accès pour tout rôle autre que `syndicAlpha`. **Si non → ÉCHEC.**
- [ ] `WizardStateModel` contient exactement les champs des 5 étapes Genèse. Vérifier `residentVectors (Map<String, int>?)`. **Si absent → STEP 5 NON MODÉLISÉ — ÉCHEC.**
- [ ] `step_screens.dart` contient 6 classes (`Step1`→`Step5` + `Recap`). `Step2ChargesFixesMensuelles` a exactement 5 champs. `Step5VecteursResidents` utilise `SignedAmountField`. **Si non → MUTATION II NON CONFORME — ÉCHEC.**
- [ ] `syndicEmail` dans `WizardStateModel` est stocké en DB sans déclencher aucun appel d'authentification email. Grep `signInWithOtp`, `signInWithPassword` dans `lib/modules/wizard/` → **0 résultat.** Si > 0 → **VIOLATION LOI III — ÉCHEC ABSOLU.**
- [ ] `WizardNotifier.finalize()` appelle `ref.read(authContractProvider).watchSetupCompleted()` source — et **non** `AppEventBus.emit(...)`. **Si bus utilisé → VIOLATION LOI V — ÉCHEC.**
- [ ] `MandateConfigModel.setupFinalizedAt` est nullable. **Si non → CRASH INITIAL GARANTI — ÉCHEC.**
- [ ] `lib/modules/wizard/` : zéro import cross-module. **Si oui → VIOLATION LOI I — ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 12 — Module Dashboard ════

> 🔒 **RAPPEL ABSOLU PHASE 12 :**
> (1) `ContextualHeader` en **premier enfant** de chaque dashboard — non-négociable.
> (2) Dashboard importe uniquement `lib/core/contracts/` — jamais un module.
> (3) `BurnRate = (m1 + m2 + m3) / 3` — Dart-side, division par 3 hardcodée.
> (4) `ResidentDashboardScreen` V6 : le résident est identifié par `authContractProvider.currentApartmentId` (jamais par rôle seul).

**Fichiers `.dart` livrés : 10**
**Durée estimée :** 2 jours | **Prérequis :** PHASE 11

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/modules/dashboard/application/dashboard_provider.dart` | `@riverpod DashboardNotifier` : agrège via contrats. Invalidation 5min. `ref.listen(authLogoutStreamProvider, ...)` → reset state au logout. |
| 2 | `lib/modules/dashboard/presentation/syndic_dashboard_screen.dart` | `ContextualHeader` ▸ `TreasurySummaryCard` ▸ `KpiRow` ▸ `QuickActionsGrid` ▸ `AnnoncePreview`. |
| 3 | `lib/modules/dashboard/presentation/adjoint_dashboard_screen.dart` | `ContextualHeader` ▸ `TreasurySummaryCard` (lecture) ▸ `KpiRow` ▸ `BuildingTopologyWidget(isInteractive: false, apartmentColors: ...)`. |
| 4 | `lib/modules/dashboard/presentation/concierge_dashboard_screen.dart` | `ContextualHeader` ▸ `UpcomingTasksWidget` ▸ `IncidentBadgeRow` ▸ `QuickActionsGrid`. |
| 5 | `lib/modules/dashboard/presentation/menage_dashboard_screen.dart` | `ContextualHeader` ▸ `TaskInstanceListWidget` ▸ `CompletionProgressBar`. |
| 6 | `lib/modules/dashboard/presentation/resident_dashboard_screen.dart` | `ContextualHeader` ▸ `ApartmentBalanceCard` (données via `IFinanceContract.getApartmentBalanceCentimes(currentApartmentId!)`) ▸ `CotisationTimelinePreview` ▸ `ActiveAnnoncesList` ▸ bouton "Déclarer un incident". |
| 7 | `lib/modules/dashboard/presentation/widgets/treasury_summary_card.dart` | Hero card Playfair Display, dégradé `AppGradients.heroGradient*`. |
| 8 | `lib/modules/dashboard/presentation/widgets/kpi_row.dart` | BurnRate (`sum3months/3`), Runway (`total/burnRate`), TauxRecouvrement. |
| 9 | `lib/modules/dashboard/presentation/widgets/quick_actions_grid.dart` | Grille 2×3. Navigation URI. |
| 10 | `lib/modules/dashboard/dashboard_module.dart` | `DashboardModule`. `onAppStart` : `ref.listen(authLogoutStreamProvider, ...)` → invalide dashboard au logout. |

### ⚔️ GATE P12
- [ ] **Chaque** dashboard a `ContextualHeader` en premier enfant. **Si manquant → VIOLATION — ÉCHEC.**
- [ ] `ResidentDashboardScreen` utilise `ref.read(authContractProvider).currentApartmentId` — jamais une heuristique basée sur le rôle seul. **Si non → MUTATION I FONCTIONNELLEMENT BRISÉE — ÉCHEC.**
- [ ] `KpiRow.burnRate` = `(m1 + m2 + m3) / 3` Dart-side. **Si non → ÉCHEC.**
- [ ] `DashboardModule` : zéro import de `lib/modules/[autre]/`. **Si oui → VIOLATION LOI I — ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 13 — Module Finance : Domaine & Data ════

*(Inchangée fonctionnellement par rapport à V5. Rappels critiques.)*

> 🔒 **RAPPEL ABSOLU PHASE 13 :**
> (1) Tous montants = `int` centimes. Zéro `double`.
> (2) `.single()` banni — uniquement `.maybeSingle()`.
> (3) Chaque mutation : `Either<Failure, T>` + clé idempotency.

**Fichiers `.dart` livrés : 10**
**Durée estimée :** 1 jour | **Prérequis :** PHASE 12

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/modules/finance/domain/cotisation_model.dart` | `@freezed CotisationModel`. `isOverdue` : vrai si passé ET `enAttente`/`partielle`. |
| 2 | `lib/modules/finance/domain/payment_model.dart` | `@freezed PaymentModel` : `methode`, `recuSerialId?`, `montantCentimes (int)`. |
| 3 | `lib/modules/finance/domain/expense_model.dart` | `@freezed ExpenseModel` : `photoUrls`, `bonSerialId?`, `montantCentimes (int)`. |
| 4 | `lib/modules/finance/domain/treasury_model.dart` | `@freezed TreasuryModel` : `caisseCentimes`, `banqueCentimes`. Getter `totalCentimes`. |
| 5 | `lib/modules/finance/domain/legacy_debt_model.dart` | `@freezed LegacyDebtModel` : `montantCentimes (int)`, `status`. |
| 6 | `lib/modules/finance/domain/exceptional_call_model.dart` | `@freezed ExceptionalCallModel` : `montantParApartmentCentimes (int)`. |
| 7 | `lib/modules/finance/domain/income_models.dart` | `@freezed GeneralIncomeModel` + `OtherIncomeModel`. |
| 8 | `lib/modules/finance/domain/payment_exceptionnel_model.dart` | `@freezed PaymentExceptionnelModel` : `notes (String?)`, `recuSerialId (String?)` — colonnes NON-NULLABLES dans le modèle Dart. |
| 9 | `lib/modules/finance/data/finance_repository.dart` | Interface complète (inchangée V5). |
| 10 | `lib/modules/finance/data/finance_repository_impl.dart` | Implémentation `SupabaseService`. Clé idempotency sur toutes mutations. `.maybeSingle()` exclusif. |

### ⚔️ GATE P13
- [ ] Zéro `double` pour montants. **Si non → VIOLATION — ÉCHEC.**
- [ ] Zéro `.single()`. **Si non → PGRST116 PRODUCTION — ÉCHEC.**
- [ ] `CotisationModel.isOverdue` retourne `false` pour périodes futures. **Si non → COLORATION TIMELINE CASSÉE — ÉCHEC.**
- [ ] `lib/modules/finance/` : zéro import cross-module. **Si oui → VIOLATION LOI I — ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 14 — Module Finance : Application ════

> 🔒 **RAPPEL ABSOLU PHASE 14 :**
> (1) **Toute mutation suit le protocole Optimistic UI V6 (Partie I.5).** C'est la Loi VI en action.
> (2) `FinanceContractImpl` V6 : maintient `StreamController<PaymentRecordedPayload>.broadcast()`. `emitPayment(payload)` est appelé par `PaymentNotifier` après succès RPC.
> (3) `BurnRate = sum(3 derniers mois dépenses) / 3` — Dart-side, division par 3 fixe.

**Fichiers `.dart` livrés : 10**
**Durée estimée :** 2 jours | **Prérequis :** PHASE 13

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/modules/finance/application/cotisation_provider.dart` | `@riverpod CotisationNotifier`. Optimiste sur paiement. |
| 2 | `lib/modules/finance/application/payment_provider.dart` | `@riverpod PaymentNotifier`. `fifoPayment()` et `paySpecificPeriod()` : **protocole Optimistic UI V6 complet** (Partie I.5). Sur succès → `_financeContractImpl.emitPayment(PaymentRecordedPayload(...))`. **Zéro `AppEventBus.emit(PaymentRecordedEvent(...))`.** |
| 3 | `lib/modules/finance/application/expense_provider.dart` | `@riverpod ExpenseNotifier`. `recordExpense()` optimiste. Upload photo avant RPC. |
| 4 | `lib/modules/finance/application/treasury_provider.dart` | `@riverpod TreasuryNotifier`. Invalidation 2min. |
| 5 | `lib/modules/finance/application/exceptional_call_provider.dart` | `@riverpod ExceptionalCallNotifier`. |
| 6 | `lib/modules/finance/application/finance_kpi_computer.dart` | Service pur : `computeBurnRate(List<ExpenseModel>) → int` (divise par 3 hardcodé), `computeRunway(int, int) → double`, `computeCollectionRate(int, int) → double`. |
| 7 | `lib/modules/finance/application/income_provider.dart` | `@riverpod IncomeNotifier`. |
| 8 | `lib/modules/finance/application/finance_contract_impl.dart` | `FinanceContractImpl implements IFinanceContract` V6. Maintient `_paymentCtrl = StreamController<PaymentRecordedPayload>.broadcast()`. Override `watchPayments()` → `_paymentCtrl.stream`. `emitPayment(payload)` callable par `PaymentNotifier`. |
| 9 | `lib/modules/finance/application/building_matrix_provider.dart` | `@riverpod BuildingMatrixNotifier`. Cache 60s. |
| 10 | `lib/modules/finance/application/audit_provider.dart` | `@riverpod AuditNotifier`. Rôle `syndicAlpha` uniquement. |

### ⚔️ GATE P14
- [ ] `PaymentNotifier.fifoPayment()` : état optimiste mis à jour **AVANT** l'appel RPC. Si RPC échoue, rollback. **Si non → VIOLATION LOI VI — ÉCHEC.**
- [ ] `PaymentNotifier.fifoPayment()` : si `connectivityProvider == offline`, enqueue dans `OfflineQueueService`, **aucun rollback**. **Si non → OFFLINE DOCTRINE BRISÉE — ÉCHEC.**
- [ ] `FinanceContractImpl.watchPayments()` retourne un `Stream` fonctionnel. `emitPayment()` déclenche les listeners. **Si non → PONT RÉACTIF CASSÉ — ÉCHEC.**
- [ ] **Zéro `AppEventBus.emit(PaymentRecordedEvent(...))` dans ce module.** Grep → 0 résultat. **Si trouvé → VIOLATION LOI V — ÉCHEC ABSOLU.**
- [ ] `FinanceKpiComputer.computeBurnRate` divise par **3** (constante, jamais variable). **Si non → BURN RATE FAUX — ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 15 — Module Finance : Présentation ════

*(Inchangée par rapport à V5. Rappels critiques de coloration timeline.)*

**Fichiers `.dart` livrés : 10**
**Durée estimée :** 2 jours | **Prérequis :** PHASE 14

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/modules/finance/presentation/screens/building_matrix_screen.dart` | Grille 3×5, couleur-coding dette. Utilise `BuildingTopologyWidget(isInteractive: false, apartmentColors: ...)`. `ContextualHeader`. |
| 2 | `lib/modules/finance/presentation/screens/apartment_detail_screen.dart` | `CotisationTimeline`, historique paginé, bouton paiement FIFO. |
| 3 | `lib/modules/finance/presentation/screens/record_payment_screen.dart` | Saisie FIFO/période spécifique. UI optimiste (badge "En cours" pendant RPC). |
| 4 | `lib/modules/finance/presentation/screens/record_expense_screen.dart` | Dépense, photos, prestataire. |
| 5 | `lib/modules/finance/presentation/screens/treasury_screen.dart` | `TreasuryGauge` + `RunwayGauge` + graphe `fl_chart`. `ContextualHeader`. |
| 6 | `lib/modules/finance/presentation/screens/grand_livre_screen.dart` | Grand livre, filtres, export PDF via `IDocumentContract`. |
| 7 | `lib/modules/finance/presentation/screens/exceptional_call_screen.dart` | Appels de fonds. |
| 8 | `lib/modules/finance/presentation/widgets/cotisation_timeline.dart` | **Coloration STRICTE :** gris (`lightFutureUnpaid`) futur impayé, rouge (`lightError`) retard passé, vert payé, bleu crédit. |
| 9 | `lib/modules/finance/presentation/widgets/finance_widgets.dart` | `PaymentMethodSelector`, `BalanceIndicator`, `RunwayGauge`, `CollectionRateChart`, `ExceptionalCallProgress`. |
| 10 | `lib/modules/finance/finance_module.dart` | `FinanceModule`. Routes complètes. |

### ⚔️ GATE P15
- [ ] `CotisationTimeline` : mois futur impayé = **gris**. Jamais rouge. **Si rouge → INVARIANT ABSOLU VIOLÉ — ÉCHEC.**
- [ ] `BuildingMatrixScreen` utilise `BuildingTopologyWidget` (réutilise le widget partagé V6). **Si non → DUPLICATION CODE — ÉCHEC.**
- [ ] `ContextualHeader` présent sur `BuildingMatrixScreen`, `TreasuryScreen`, `GrandLivreScreen`. **Si manquant → ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 16 — Module Incidents ════

> 🔒 **RAPPEL ABSOLU PHASE 16 :**
> (1) FSM Incidents : `nouveau → assigné → en_cours → résolu → clôturé`. Toute transition invalide = `ValidationFailure`.
> (2) Escalade → `_incidentContractImpl.emitEscalation(IncidentEscalationPayload(...))`. **ZÉRO `AppEventBus.emit(IncidentEscalatedToTaskEvent(...))`.** 
> (3) Changement statut → `_incidentContractImpl.emitStatusChange(...)`.
> (4) LOI VI : `createIncident()` suit le protocole Optimistic UI.

**Fichiers `.dart` livrés : 10**
**Durée estimée :** 2 jours | **Prérequis :** PHASE 15

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/modules/incidents/domain/incident_model.dart` | `@freezed IncidentModel` : `status (IncidentStatus)`, `priorite (IncidentPriority)`, `imageUrls`. |
| 2 | `lib/modules/incidents/domain/incident_template_model.dart` | `@freezed IncidentTemplateModel`. |
| 3 | `lib/modules/incidents/domain/incident_fsm.dart` | `IncidentFsm.canTransition(from, to) → bool`. `assertTransition()`. |
| 4 | `lib/modules/incidents/data/incident_repository.dart` | Interface complète. |
| 5 | `lib/modules/incidents/data/incident_repository_impl.dart` | `fn_transition_incident` pour toutes mutations statut. Clé idempotency. |
| 6 | `lib/modules/incidents/application/incident_provider.dart` | `@riverpod IncidentNotifier`. `escalateToTask()` → `_contractImpl.emitEscalation(payload)`. `transition()` → `_contractImpl.emitStatusChange(payload)`. LOI VI sur `createIncident()`. |
| 7 | `lib/modules/incidents/application/incident_contract_impl.dart` | `IncidentContractImpl implements IIncidentContract` V6. Deux `StreamController.broadcast()` : `_escalationCtrl`, `_statusCtrl`. `emitEscalation()` et `emitStatusChange()` publics (mais internes au module). |
| 8 | `lib/modules/incidents/presentation/incident_list_screen.dart` | `ContextualHeader`. `PaginatedList`. |
| 9 | `lib/modules/incidents/presentation/incident_detail_screen.dart` | Stepper statut, photos, assignation, "Escalader en Tâche". |
| 10 | `lib/modules/incidents/incidents_module.dart` | `IncidentsModule`. `onAppStart` : override `incidentContractProvider` dans le container. |

### ⚔️ GATE P16
- [ ] `escalateToTask()` : **zéro** `AppEventBus.emit(...)`. Grep `AppEventBus` dans `lib/modules/incidents/` → 0. **Si trouvé → VIOLATION LOI V — ÉCHEC ABSOLU.**
- [ ] `IncidentContractImpl` possède deux `StreamController.broadcast()` fonctionnels. **Si non → PONTS RÉACTIFS CASSÉS — ÉCHEC.**
- [ ] `IncidentFsm.canTransition(resolu, nouveau)` retourne `false`. **Si true → FSM INVALIDE — ÉCHEC.**
- [ ] `fn_transition_incident` seul chemin de mutation statut. Grep `.update({'status':` → 0. **Si > 0 → CONTOURNEMENT FSM — ÉCHEC.**
- [ ] Résident peut déclarer un incident en ≤ 3 taps. **Si > 3 → VIOLATION UX — ÉCHEC.**
- [ ] `lib/modules/incidents/` : zéro import cross-module. **Si oui → VIOLATION LOI I — ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 17 — Module Tasks ════

> 🔒 **RAPPEL ABSOLU PHASE 17 :**
> (1) Ce module **s'abonne** à `incidentEscalationStreamProvider` via `ref.listen`. Il ne connaît pas le module Incidents.
> (2) **Zéro** subscription à `AppEventBus.stream.whereType<IncidentEscalatedToTaskEvent>()` — cette API n'existe plus.
> (3) `TaskContractImpl` V6 : maintient `_overdueCtrl = StreamController<TaskOverduePayload>.broadcast()`.

**Fichiers `.dart` livrés : 10**
**Durée estimée :** 2 jours | **Prérequis :** PHASE 16

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/modules/tasks/domain/task_master_model.dart` | `@freezed TaskMasterModel` : `recurrence`, `assignedUserId?`, `requiredPhotoProof`. |
| 2 | `lib/modules/tasks/domain/task_instance_model.dart` | `@freezed TaskInstanceModel` : `dueDate`, `status`, `photoUrl?`, `completedAt?`. |
| 3 | `lib/modules/tasks/domain/tache_model.dart` | `@freezed TacheModel` legacy : `parentId?`, `status`, `courrier flag`. |
| 4 | `lib/modules/tasks/data/task_repository.dart` | Interface : `getTaskInstances`, `completeInstance`, `createMaster`. |
| 5 | `lib/modules/tasks/data/tache_repository.dart` | Interface legacy : `getTaches`, `createCourrier`, `rescheduleTache`. |
| 6 | `lib/modules/tasks/data/task_repository_impl.dart` | `fn_complete_task_instance`. Idempotency. |
| 7 | `lib/modules/tasks/application/task_provider.dart` | `@riverpod TaskNotifier`. Dans `build()` : `ref.listen(incidentEscalationStreamProvider, (_, next) { next.whenData((p) => _createTaskFromEscalation(p)); })`. **Remplace l'ancienne subscription bus.** `_createTaskFromEscalation()` suit le protocole LOI VI. Sur overdue : `_taskContractImpl.emitOverdue(TaskOverduePayload(...))`. |
| 8 | `lib/modules/tasks/application/task_contract_impl.dart` | `TaskContractImpl implements ITaskContract` V6. `_overdueCtrl = StreamController<TaskOverduePayload>.broadcast()`. |
| 9 | `lib/modules/tasks/presentation/task_screens.dart` | `TaskInstanceListScreen`, `CreateTaskMasterScreen`, `TacheListScreen`, `TacheDetailScreen`. |
| 10 | `lib/modules/tasks/tasks_module.dart` | `TasksModule`. `onAppStart` : override `taskContractProvider`. |

### ⚔️ GATE P17
- [ ] `TaskNotifier.build()` contient `ref.listen(incidentEscalationStreamProvider, ...)`. **Si absent → COMMUNICATION INCIDENTS→TASKS BRISÉE — ÉCHEC.**
- [ ] **Zéro** `AppEventBus` dans `lib/modules/tasks/`. Grep → 0. **Si trouvé → VIOLATION LOI V — ÉCHEC ABSOLU.**
- [ ] `lib/modules/tasks/` : zéro import de `lib/modules/incidents/`. **Si oui → VIOLATION LOI I — ÉCHEC ABSOLU.**
- [ ] `TaskContractImpl.watchOverdue()` fonctionne. **Si non → PONT TÂCHES→NOTIFS CASSÉ — ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 18 — Module HR ════

*(Inchangée fonctionnellement par rapport à V5. LOI VI s'applique à `paySalary()`.)*

> 🔒 `SalaryPaymentModel.compte` et `.type` sont **non-nullable**. `paySalary()` suit le protocole Optimistic UI V6.

**Fichiers `.dart` livrés : 10**
**Durée estimée :** 2 jours | **Prérequis :** PHASE 17

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/modules/hr/domain/salary_payment_model.dart` | `SalaryPaymentModel` : `compte (String)` NOT NULL, `type (String)` NOT NULL. |
| 2 | `lib/modules/hr/domain/salary_advance_model.dart` | FSM avances : `pending → approved → deducted`. |
| 3 | `lib/modules/hr/domain/staff_model.dart` | `StaffModel` : `salaireBaseCentimes (int)`. |
| 4 | `lib/modules/hr/data/hr_repository.dart` | Interface complète. |
| 5 | `lib/modules/hr/data/hr_repository_impl.dart` | `fn_record_staff_salary_payment`, `fn_request_salary_advance`, etc. Colonnes `compte`, `type` correctes. |
| 6 | `lib/modules/hr/application/hr_provider.dart` | `@riverpod HrNotifier`. `paySalary()` : protocole LOI VI. Émet via `_hrContractImpl` si implémenté. |
| 7 | `lib/modules/hr/application/hr_contract_impl.dart` | `HrContractImpl implements IHrContract`. |
| 8 | `lib/modules/hr/presentation/hr_screens.dart` | `StaffListScreen`, `PaySalaryScreen`, `SalaryHistoryScreen`. `ContextualHeader` sur racines. |
| 9 | `lib/modules/hr/presentation/widgets/hr_widgets.dart` | `SalarySlipCard`, `AdvanceApprovalTile`. |
| 10 | `lib/modules/hr/hr_module.dart` | `HrModule`. Rôles : `{syndicAlpha, adjointLecture}`. |

### ⚔️ GATE P18
- [ ] `SalaryPaymentModel.compte` et `.type` sont non-nullables. **Si nullable → VIOLATION DDL — ÉCHEC.**
- [ ] PDF fiche de paie via `PdfGenerationService.compute(Map<String, dynamic>)` — primitives uniquement. **Si non → CRASH ISOLATE — ÉCHEC.**
- [ ] `paySalary()` : état optimiste avant RPC. **Si non → VIOLATION LOI VI — ÉCHEC.**
- [ ] `lib/modules/hr/` : zéro import cross-module. **Si oui → VIOLATION LOI I — ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 19 — Module Prestataires ════

*(Inchangée par rapport à V5.)*

**Fichiers `.dart` livrés : 10**
**Durée estimée :** 1 jour | **Prérequis :** PHASE 18

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/modules/prestataires/domain/prestataire_model.dart` | `averageRating → double` calculé Dart-side. |
| 2 | `lib/modules/prestataires/domain/prestataire_rating_model.dart` | `score (int 1-5)`, `comment?`. |
| 3-10 | *[fichiers V5 identiques]* | Data, application, présentation, module. |

### ⚔️ GATE P19
- [ ] `averageRating` calculé Dart-side. **Si round-trip → ÉCHEC.**
- [ ] `RatingStars` : `AppColors.*` exclusif. **Si `Colors.yellow` → VIOLATION LOI IV — ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 20 — Module Annonces + Module Notifications ════

> 🔒 **RAPPEL ABSOLU PHASE 20 :**
> (1) `NotificationsModule.onAppStart` : `ref.listen(paymentRecordedStreamProvider, ...)`, `ref.listen(incidentStatusStreamProvider, ...)`, `ref.listen(taskOverdueStreamProvider, ...)`. **ZÉRO subscription `AppEventBus`.** 
> (2) Annonces expirées invisibles aux résidents — filtre `expires_at > now()` Supabase.

**Fichiers `.dart` livrés : 10**
**Durée estimée :** 2 jours | **Prérequis :** PHASE 19

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/modules/annonces/domain/annonce_model.dart` | `typeAnnonce`, `expiresAt`, `priorite`, `isActive → bool`. |
| 2 | `lib/modules/annonces/data/annonce_repository_impl.dart` | Filtre `expires_at > now()` pour résidents. |
| 3 | `lib/modules/annonces/application/annonce_provider.dart` | `@riverpod AnnonceNotifier`. |
| 4 | `lib/modules/annonces/application/annonce_contract_impl.dart` | `AnnonceContractImpl`. |
| 5 | `lib/modules/annonces/presentation/annonce_screens.dart` | `AnnonceListScreen`, `AnnonceDetailScreen`, `CreateAnnonceScreen`. |
| 6 | `lib/modules/annonces/annonces_module.dart` | `AnnoncesModule`. |
| 7 | `lib/modules/notifications/domain/notification_model.dart` | `@freezed NotificationModel` : `isRead`, `readAt?`, `fcmSent`, `payload`. |
| 8 | `lib/modules/notifications/application/notification_provider.dart` | `@riverpod NotificationNotifier`. Dans `build()` : **trois `ref.listen`** sur `paymentRecordedStreamProvider`, `incidentStatusStreamProvider`, `taskOverdueStreamProvider`. Sur événement → `fn_create_notification_log(...)`. **Zéro `AppEventBus.stream.whereType<...>()`.** |
| 9 | `lib/modules/notifications/application/notification_contract_impl.dart` | `NotificationContractImpl`. |
| 10 | `lib/modules/notifications/notifications_module.dart` | `NotificationsModule`. `onAppStart` : override `notificationContractProvider`. |

### ⚔️ GATE P20
- [ ] `NotificationNotifier.build()` : 3 `ref.listen` sur les ponts réactifs. **Si abonnement bus → VIOLATION LOI V — ÉCHEC ABSOLU.**
- [ ] Grep `AppEventBus` dans `lib/modules/notifications/` et `lib/modules/annonces/` → **0 résultat.** **Si > 0 → ÉCHEC.**
- [ ] Annonce expirée invisible pour résident. Test avec `expiresAt` dans le passé. **Si visible → ÉCHEC.**
- [ ] `lib/modules/notifications/` : zéro import de `lib/modules/finance/`, `incidents/`, `tasks/`. **Si oui → VIOLATION LOI I — ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 21 — Module Documents + Module Profile ════

*(Inchangée fonctionnellement par rapport à V5.)*

**Fichiers `.dart` livrés : 10**
**Durée estimée :** 2 jours | **Prérequis :** PHASE 20

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1-5 | Documents | `DocumentModel`, `document_provider.dart`, `document_contract_impl.dart`, `document_screens.dart`, `documents_module.dart`. PDFs via `PdfGenerationService.compute(Map<String, dynamic>)`. |
| 6-10 | Profile | `ProfileModel`, `profile_provider.dart`, `profile_contract_impl.dart`, `profile_screens.dart` (incl. `ChangePinScreen` 8ch), `profile_module.dart`. |

### ⚔️ GATE P21
- [ ] `DocumentContractImpl` : `PdfGenerationService.compute(Map<String, dynamic>)` — zéro instance custom dans `SendPort`. **Si non → CRASH ISOLATE — ÉCHEC.**
- [ ] `ChangePinScreen` : ancien PIN 8ch validé avant saisie nouveau. Nouveau PIN = 8ch. **Si non → FAILLE SÉCURITÉ — ÉCHEC.**
- [ ] `AuditLogScreen` : cursor-pagination. Zéro offset. **Si offset → ANTI-PATTERN — ÉCHEC.**
- [ ] `flutter analyze` → 0 erreur.

---

## ════ PHASE 22 — Shell : Module Registry, Intégration Finale ════

> 🔒 **RAPPEL ABSOLU PHASE 22 :**
> (1) `ModuleRegistry` override les providers de contrats dans le `ProviderContainer` : `financeContractProvider`, `incidentContractProvider`, `taskContractProvider`, `authContractProvider`.
> (2) **Test d'amputation :** `lib/modules/finance/` supprimé → app compile et tourne.
> (3) `AppModuleLoader.onAppStart()` initialise les modules dans l'ordre déterministe.

**Fichiers `.dart` livrés : 3 + validation finale**
**Durée estimée :** 1 jour | **Prérequis :** TOUTES phases P0–P21

| # | Fichier | Responsabilité |
|---|---------|----------------|
| 1 | `lib/shell/module_registry.dart` | `ModuleRegistry` : liste ordonnée de `IModuleRegistrar`. `routesForRole(AppRole)`, `quickActionsForRole(AppRole)`, `isModulePresent(String)`. **Override des providers de contrats** dans le `ProviderContainer` au démarrage. |
| 2 | `lib/shell/app_module_loader.dart` | `AppModuleLoader` : appelle `module.onAppStart(container)` dans l'ordre déterministe : auth → wizard → dashboard → finance → incidents → tasks → hr → prestataires → annonces → notifications → documents → profile. |
| 3 | `lib/shell/dynamic_nav_builder.dart` | `DynamicNavBuilder` : `BottomNavBar` depuis `ModuleRegistry.quickActionsForRole(role)`. |

### ⚔️ GATE P22 — VALIDATION FINALE ABSOLUE V6
**CETTE GATE EST LA PORTE DE PRODUCTION. AUCUN RACCOURCI N'EST TOLÉRÉ.**

**Éradication AppEventBus (LOI V) :**
- [ ] Grep `AppEventBus` dans tout `lib/` → **0 résultat.** `Si > 0 → VIOLATION LOI V — RELEASE BLOQUÉE.**
- [ ] Grep `AppEvent` dans tout `lib/` → **0 résultat.** **Si > 0 → VIOLATION LOI V — RELEASE BLOQUÉE.**
- [ ] Grep `app_event_bus` ou `app_events` dans `pubspec.yaml`, imports, routes → **0 résultat.**
- [ ] Répertoire `lib/core/events/` : **INEXISTANT.** **Si présent → VIOLATION LOI V — RELEASE BLOQUÉE.**

**Ponts Réactifs V6 (LOI V) :**
- [ ] `lib/core/providers/reactive_bridges.dart` **existe**. **Si absent → LOI V NON IMPLÉMENTÉE — RELEASE BLOQUÉE.**
- [ ] 6 `StreamProvider` dans `reactive_bridges.dart` : `incidentEscalationStreamProvider`, `paymentRecordedStreamProvider`, `incidentStatusStreamProvider`, `taskOverdueStreamProvider`, `authLogoutStreamProvider`, `wizardCompletedStreamProvider`. **Si manquant → PONT INCOMPLET — RELEASE BLOQUÉE.**

**Authentification Topologique (LOI III — Mutation I) :**
- [ ] `resident_building_selector_screen.dart` **existe et compile.** **Si absent → MUTATION I NON LIVRÉE — RELEASE BLOQUÉE.**
- [ ] `auth_bifurcation_screen.dart` **existe.** **Si absent → MUTATION I NON LIVRÉE — RELEASE BLOQUÉE.**
- [ ] `BuildingTopologyWidget` affiche 15 nœuds en 3 rangées (1-5, 6-10, 11-15). Vérification manuelle. **Si non → MUTATION I DÉFAILLANTE — RELEASE BLOQUÉE.**
- [ ] `fn_validate_pin_resident` est appelé pour les résidents (grep dans `auth_repository_impl.dart`). **Si absent → RÉSOLUTION ENTITÉ BRISÉE — RELEASE BLOQUÉE.**
- [ ] Zéro flux email/password. Grep `signInWithPassword`, `signInWithOtp`, `EmailAuthProvider` → **0 résultat.** **Si > 0 → VIOLATION LOI III — RELEASE BLOQUÉE.**
- [ ] `AppConstants.pinLength == 8`. **Si 4 → VIOLATION LOI III — RELEASE BLOQUÉE.**

**Séquence Genèse (LOI — Mutation II) :**
- [ ] `WizardStateModel` contient `mandateStartDate`, `syndicFirstName`, `syndicLastName`, `syndicPhone`, `syndicEmail`, 5 champs charges fixes, 2 provisions, `soldeInitialCaisse`, `soldeInitialBanque`, `residentVectors`. **Si manquant → MUTATION II INCOMPLÈTE — RELEASE BLOQUÉE.**
- [ ] `step_screens.dart` : 5 steps input + 1 recap. `Step5VecteursResidents` utilise `SignedAmountField`. **Si non → MUTATION II INCOMPLÈTE — RELEASE BLOQUÉE.**
- [ ] Grep `signInWithOtp`, `signInWithPassword` dans `lib/modules/wizard/` → **0 résultat.** **Si > 0 → VIOLATION LOI III (email auth) — RELEASE BLOQUÉE.**

**Offline-First (LOI VI — Mutation IV) :**
- [ ] `PaymentNotifier.fifoPayment()` : état optimiste avant RPC. Grep `previousState = state` dans `payment_provider.dart` → **présent.** **Si absent → VIOLATION LOI VI — RELEASE BLOQUÉE.**
- [ ] `OfflineQueueService.enqueue()` appelé si `connectivityProvider == offline` dans au moins `fifoPayment`, `recordExpense`, `createIncident`. **Si non → DOCTRINE OFFLINE PARTIELLE — RELEASE BLOQUÉE.**

**Isolation Modulaire (LOI I) :**
- [ ] Scanner `lib/modules/[A]/` pour import de `lib/modules/[B]/` (A≠B) → **0 résultat.**
- [ ] **Test d'amputation :** Renommer `lib/modules/finance/` → `_finance_REMOVED/`, `flutter build apk` → compile sans erreur. **Si crash → LOI I VIOLÉE — RELEASE BLOQUÉE.**
- [ ] Grep `lib/modules/incidents/` dans `lib/modules/tasks/` → **0 résultat.**

**Thème (LOI IV) :**
- [ ] Scanner `Color(0xFF` hors `app_colors.dart` → **0 résultat.**
- [ ] `AppColors.lightBackground = Color(0xFFF0E8DC)`. **Si `0xFFFFFFFF` → RELEASE BLOQUÉE.**
- [ ] `BuildingTopologyWidget` : uniquement `AppColors.*`, zéro hex hardcodé.

**Module AG (LOI II) :**
- [ ] Grep `ag_session`, `ag_presence`, `ag_vote`, `quorum`, `assemblee_generale` → **0 résultat.**

**Invariants Financiers :**
- [ ] Zéro `double` monétaire. Grep `double.*mad`, `double.*amount` → **0 résultat.**
- [ ] Zéro `.single()` dans repositories. **Si > 0 → PGRST116 PRODUCTION.**
- [ ] `CotisationTimeline` : futur impayé = gris. Vérification manuelle.

**Sécurité & Qualité :**
- [ ] Zéro `setState` (Riverpod exclusif).
- [ ] Clé idempotency sur chaque mutation RPC.
- [ ] `_timer?.cancel()` dans chaque `dispose()` avec `Timer`.
- [ ] `mounted` vérifié avant `BuildContext` async.
- [ ] PDF via `PdfGenerationService.compute(Map<String, dynamic>)` — primitives uniquement.

**Build Final :**
- [ ] `flutter analyze` → **0 erreur, 0 warning. Si non → RELEASE BLOQUÉE.**
- [ ] `flutter test` → **100% pass.** Si non → RELEASE BLOQUÉE.
- [ ] `flutter build apk --release` → **succès, zéro warning R8.** Si non → RELEASE BLOQUÉE.
- [ ] `flutter build ios --release` → **succès, zéro warning Xcode.** Si non → RELEASE BLOQUÉE.

---

# ═══════════════════════════════════════════════════════════
# PARTIE VI — RÉFÉRENCES CROISÉES & CONVENTIONS CODE V6
# ═══════════════════════════════════════════════════════════

## VI.1 — Nommage

| Élément | Convention | Exemple |
|---------|------------|---------|
| Fichier | snake_case | `payment_provider.dart` |
| Classe | PascalCase | `PaymentNotifier` |
| Provider Riverpod | camelCase + Provider | `paymentListProvider` |
| Notifier Riverpod | PascalCase + Notifier | `RecordPaymentNotifier` |
| Contrat interface | `I` + PascalCase | `IFinanceContract` |
| Implémentation contrat | PascalCase + Impl | `FinanceContractImpl` |
| Payload de stream | PascalCase + Payload | `PaymentRecordedPayload` |
| Pont réactif provider | camelCase + StreamProvider | `paymentRecordedStreamProvider` |
| Module registrar | PascalCase + Module | `FinanceModule` |
| Constante | SCREAMING_SNAKE | `PIN_MAX_ATTEMPTS` |
| Paramètre RPC Supabase | préfixe `p_` | `p_idempotency_key` |
| Route GoRouter | kebab-case URI | `/finance/payment` |
| ~~Événement bus~~ | ~~PascalCase + Event~~ | **ABOLI EN V6** |

## VI.2 — Patrons de Code Obligatoires V6

```dart
// ✅ CORRECT V6 — Montant en centimes (int)
const int montantCentimes = 25000; // = 250.00 MAD
final String display = AmountFormatter.format(montantCentimes);

// ❌ INTERDIT — Double pour les montants
// final double montantMad = 250.0; // → REJET IMMÉDIAT

// ✅ CORRECT V6 — Optimistic UI (LOI VI)
Future<void> fifoPayment({required String apartmentId, required int montantCentimes}) async {
  final previousState = state;
  state = state.copyWith(balance: state.balance - montantCentimes, isOptimistic: true);

  final key = IdempotencyService.generate(userId: _userId, action: 'FIFO_${apartmentId}');
  final isOnline = ref.read(connectivityProvider) == ConnectivityStatus.online;

  if (!isOnline) {
    await ref.read(offlineQueueServiceProvider).enqueue(OfflineMutation(
      rpcName: 'fn_apply_fifo_payment',
      params: {'p_idempotency_key': key, 'p_apartment_id': apartmentId,
               'p_montant_centimes': montantCentimes},
      idempotencyKey: key,
    ));
    return; // État optimiste maintenu — SyncService confirmera
  }

  final result = await ref.read(supabaseServiceProvider).rpcEither<PaymentResult>(
    'fn_apply_fifo_payment',
    params: {'p_idempotency_key': key, 'p_apartment_id': apartmentId,
             'p_montant_centimes': montantCentimes},
  );

  result.fold(
    (failure) => state = previousState.copyWith(isOptimistic: false), // rollback
    (data) {
      state = state.copyWith(isOptimistic: false);
      _financeContractImpl.emitPayment(PaymentRecordedPayload(
        apartmentId: apartmentId, amountCentimes: montantCentimes, periode: data.periode,
      ));
    },
  );
}

// ✅ CORRECT V6 — Communication inter-module (ref.listen, pas AppEventBus)
@riverpod
class TaskNotifier extends _$TaskNotifier {
  @override
  TaskState build() {
    ref.listen(incidentEscalationStreamProvider, (_, next) {
      next.whenData((payload) => _createTaskFromEscalation(payload));
    });
    return const TaskState.initial();
  }
}

// ❌ ABOLI EN V6 — AppEventBus
// ref.read(appEventBusProvider).emit(IncidentEscalatedToTaskEvent(...));     // INTERDIT
// _eventBus.stream.whereType<IncidentEscalatedToTaskEvent>().listen(...);    // INTERDIT

// ✅ CORRECT V6 — Emission cross-module via ContractImpl
// Dans IncidentNotifier (après escalation réussie) :
_incidentContractImpl.emitEscalation(IncidentEscalationPayload(
  incidentId: incident.id, title: incident.title,
  description: incident.description, assignedUserId: incident.assignedUserId,
));

// ✅ CORRECT V6 — .maybeSingle() (jamais .single())
final user = await supabase
    .from('users').select().eq('id', userId).maybeSingle();

// ✅ CORRECT — Timer avec dispose
class _ClockWidgetState extends State<ClockWidget> {
  Timer? _timer;
  @override
  void initState() {
    super.initState();
    _timer = Timer.periodic(const Duration(seconds: 1), (_) => setState(() {}));
  }
  @override
  void dispose() {
    _timer?.cancel(); // OBLIGATOIRE
    super.dispose();
  }
}

// ✅ CORRECT — Context async safety
Future<void> _doAsyncAction() async {
  final result = await someOperation();
  if (!mounted) return; // OBLIGATOIRE avant BuildContext
  context.go(RouteNames.success);
}

// ✅ CORRECT V6 — BuildingTopologyWidget usage dans ResidentBuildingSelectorScreen
BuildingTopologyWidget(
  isInteractive: true,
  onApartmentTap: (int number, String id) {
    ref.read(authNotifierProvider.notifier).selectApartment(number, id);
    context.go(RouteNames.pinEntry, extra: {'apartmentId': id, 'apartmentNumber': number});
  },
)

// ✅ CORRECT V6 — SignedAmountField pour dettes résidents (Wizard Step 5)
SignedAmountField(
  label: 'Appartement $aptNumber',
  initialCentimes: currentBalance, // peut être négatif
  onChanged: (int centimes) { // centimes peut être négatif = dette
    ref.read(wizardNotifierProvider.notifier)
       .updateResidentVector(apartmentId, centimes);
  },
)
```

## VI.3 — Variables d'Environnement

```bash
# .env (ne jamais commiter — dans .gitignore)
SUPABASE_URL=https://fypbkzlfyjjchaislaia.supabase.co
SUPABASE_ANON_KEY=[anon-key]
FCM_SERVER_KEY=[fcm-server-key]
OPENWEATHER_API_KEY=[openweathermap-api-key]
ENVIRONMENT=development
```

**Build :** `flutter run --dart-define-from-file=.env`

## VI.4 — Invariants Métier Absolus V6

| Invariant | Règle | Contrôle |
|-----------|-------|----------|
| PIN length | `AppConstants.pinLength == 8` | Gate chaque phase auth |
| PIN tentatives | `AppConstants.pinMaxAttempts == 5` | Gate P10 |
| Montants | `int` centimes exclusif, zéro `double` | Gate chaque phase finance |
| Futur impayé | Gris (`lightFutureUnpaid`), JAMAIS rouge | Gate P9, P15 |
| BurnRate | `sum_depenses_3_mois / 3` (Dart-side, div par 3 fixe) | Gate P12, P14 |
| Queries | `.maybeSingle()` exclusif, zéro `.single()` | Gate P13, chaque module |
| PDF isolate | `Map<String, dynamic>` primitives uniquement via `SendPort` | Gate P5, P21 |
| Modules | Zéro import cross-module | Gate chaque phase module |
| AG | Zéro trace dans la codebase | Gate P22 |
| Idempotency | Clé obligatoire sur toute mutation RPC | Gate chaque module data |
| AppEventBus | **ABOLI** — zéro instance dans la codebase | Gate P2, P22 |
| Optimistic UI | Mise à jour état local AVANT RPC sur toute mutation | Gate P14, P16, P18 |
| Auth email | Interdit comme mécanisme d'authentification | Gate P10, P11, P22 |
| Bifurcation auth | Staff ≠ Résident — deux chemins distincts | Gate P10 |
| Building selector | Nœuds 1-5 Level1, 6-10 Level2, 11-15 Level3 | Gate P8, P10 |
| Soldes résidents T=0 | Peut être négatif, nul, ou positif (int centimes signé) | Gate P11 |

---

# ═══════════════════════════════════════════════════════════
# PARTIE VII — CHECKLIST GLOBALE DE VALIDATION V6
# ═══════════════════════════════════════════════════════════

```
ÉRADICATION APPEVENTBUS (LOI V)
□ Grep AppEventBus dans lib/ → 0 résultat
□ Grep AppEvent dans lib/ → 0 résultat
□ Répertoire lib/core/events/ inexistant
□ lib/core/providers/reactive_bridges.dart existe avec 6 StreamProviders

PONTS RÉACTIFS V6
□ incidentEscalationStreamProvider → Stream<IncidentEscalationPayload> fonctionnel
□ paymentRecordedStreamProvider → Stream<PaymentRecordedPayload> fonctionnel
□ incidentStatusStreamProvider → Stream<IncidentStatusChangePayload> fonctionnel
□ taskOverdueStreamProvider → Stream<TaskOverduePayload> fonctionnel
□ authLogoutStreamProvider + wizardCompletedStreamProvider fonctionnels

AUTHENTIFICATION TOPOLOGIQUE V6 (LOI III — MUTATION I)
□ auth_bifurcation_screen.dart existe et s'affiche en post-splash
□ staff_role_selector_screen.dart : 4 tuiles staff uniquement
□ resident_building_selector_screen.dart : BuildingTopologyWidget 3×5 rendu
□ BuildingTopologyWidget Level1=Apts1-5, Level2=Apts6-10, Level3=Apts11-15
□ BuildingTopologyWidget : effet convexe 3D via RadialGradient (pas de hex hardcodé)
□ Tap nœud appartement → AuthNotifier.selectApartment(number, id) avant PIN
□ fn_validate_pin_resident appelé pour résidents, fn_validate_pin pour staff
□ SessionModel.apartmentId non-null après auth résident
□ Zéro flux email/password : grep signInWithOtp, signInWithPassword → 0

SÉQUENCE GENÈSE V6 (MUTATION II)
□ Step 1 : mandateStartDate, syndicFirstName, syndicLastName, syndicPhone, syndicEmail
□ Step 2 : 5 charges fixes mensuelles (salaires, assurance, ascenseur, eau/élec)
□ Step 3 : provisionConsommables, provisionImprevus
□ Step 4 : soldeInitialCaisse, soldeInitialBanque (int centimes, ≥ 0)
□ Step 5 : 15 SignedAmountField (int centimes signés — négatif = dette)
□ syndicEmail : informatif uniquement — aucun mécanisme auth email déclenché

OFFLINE-FIRST DOCTRINAL (LOI VI — MUTATION IV)
□ fifoPayment() : previousState sauvegardé, état optimiste AVANT RPC
□ Si offline : enqueue OfflineQueueService, aucun rollback
□ Si RPC server failure : rollback vers previousState
□ SyncService rejoue la queue au retour de connectivité
□ recordExpense(), createIncident() : même protocole

ISOLEMENT MODULAIRE (LOI I)
□ Zéro import lib/modules/[A]/ → lib/modules/[B]/ (A≠B)
□ Test d'amputation finance : app compile avec lib/modules/finance/ absent
□ lib/modules/tasks/ : zéro import lib/modules/incidents/
□ lib/modules/notifications/ : zéro import modules finance/incidents/tasks

THÈME (LOI IV)
□ Zéro Color(0xFF...) hors app_colors.dart
□ lightBackground = Color(0xFFF0E8DC) (Alabaster)
□ lightFutureUnpaid = Color(0xFF9E9488) (gris chaud, jamais rouge)
□ BuildingTopologyWidget : AppColors.* exclusif

AUTHENTIFICATION — INVARIANTS COMMUNS
□ AppRole exactement 5 valeurs
□ pinLength = 8
□ PIN hashé SHA-256 avant envoi réseau
□ Zéro email/password auth

MODULE AG (LOI II)
□ Grep ag_session, ag_presence, ag_vote, quorum → 0 résultat

FINANCE
□ Zéro double monétaire
□ Zéro .single()
□ BurnRate = sum3mois / 3 (Dart-side)
□ CotisationTimeline : gris futur, rouge retard passé, vert payé

SÉCURITÉ & QUALITÉ
□ Zéro setState (Riverpod exclusif)
□ Clé idempotency sur chaque mutation RPC
□ _timer?.cancel() dans chaque dispose() avec Timer
□ mounted vérifié avant BuildContext async
□ PDF via PdfGenerationService.compute(Map<String, dynamic>)

BUILD
□ flutter analyze → 0 erreur, 0 warning
□ flutter test → 100% pass
□ flutter build apk --release → succès
□ flutter build ios --release → succès
```

---

*Document généré le 09/05/2026 par Claude Sonnet 4.6 — Architecte Principal*
*Exécutant syntaxique désigné : Jules (AI Coding Agent)*
*Résidence L'Amandier B — Bouskoura, Maroc*
*Projet Supabase : fypbkzlfyjjchaislaia*
*Ce document annihile et invalide toutes les versions précédentes (V1.0 à V5.0)*

```
T = 227 fichiers .dart
N = 23 phases acycliques (P0 → P22)
Quota par phase : ≤ 10 fichiers (tolérance ±3 sur phases structurelles V6)
Delta V5→V6 : −2 (events/ éradiqués) +1 (reactive_bridges) +1 (building_topology_widget) +3 (auth screens) = +3
Mutations V6 : Résolution Entité Topologique | Séquence Genèse | Purisme Réactif Riverpod | Enforcement Offline-First
```
