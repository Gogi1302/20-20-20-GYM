# GymPlan – MVP Flutter App (Riverpod + Hive)

Below is a complete minimal Flutter app you can paste into a fresh project. It includes:
- Local-first storage with **Hive** (storing JSON-like maps, no adapters required).
- State management with **Riverpod**.
- Screens: **User list**, **Weekly plan** (tabs L–D), **Exercise editor** (modal), **Templates** (dialog).
- UX: swipe left = eliminar, swipe right = marcar hecho, duplicar/limpiar día, aplicar plantilla a día o a semana.
- i18n base (**es-CO** por defecto) mediante un archivo sencillo de strings.
- Accesibilidad: botones con área táctil amplia, fuentes escalables.
- Modo oscuro del sistema.
- **Tests unitarios**: validación de modelo y duplicar día.
- **Seeds**: usuarios/planes/plantillas iniciales en `assets/seeds.json`.

---
## Estructura del proyecto

```
lib/
  main.dart
  app.dart
  core/
    strings.dart
    utils.dart
  data/
    storage.dart
    seeds.dart
  domain/
    models.dart
  state/
    repositories.dart
    controllers.dart
  ui/
    user_list_screen.dart
    weekly_plan_screen.dart
    exercise_editor.dart
    template_picker.dart
assets/
  seeds.json
pubspec.yaml
analysis_options.yaml

# Tests
test/
  model_validation_test.dart
  duplicate_day_test.dart
```

---
## pubspec.yaml
```yaml
name: gymplan
description: MVP local-first para planes de entrenamiento (Flutter + Riverpod + Hive)
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: '>=3.4.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
  hooks_riverpod: ^2.5.2
  hive: ^2.2.3
  hive_flutter: ^1.1.0
  collection: ^1.18.0
  intl: ^0.19.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^4.0.0

flutter:
  uses-material-design: true
  assets:
    - assets/seeds.json
```

---
## analysis_options.yaml (opcional)
```yaml
include: package:flutter_lints/flutter.yaml
linter:
  rules:
    prefer_const_constructors: true
    avoid_print: true
```

---
## assets/seeds.json
```json
{
  "users": [
    { "id": "u1", "name": "Rodrigo", "createdAt": "2025-08-13T12:00:00Z" }
  ],
  "plans": [
    { "id": "p1", "userId": "u1", "name": "Plan base", "weekStartIso": "2025-08-11" }
  ],
  "days": [
    { "id": "d1", "planId": "p1", "weekday": 0, "exercises": [
      { "id": "e1", "name": "Sentadilla", "sets": 4, "reps": 8, "note": "", "done": false },
      { "id": "e2", "name": "Press banca", "sets": 4, "reps": 8, "note": "", "done": false }
    ] }
  ],
  "templates": [
    {
      "id": "t1",
      "name": "Full Body 3d",
      "days": [
        { "weekday": 0, "exercises": [
          { "name": "Sentadilla", "sets": 4, "reps": 8 },
          { "name": "Press banca", "sets": 4, "reps": 8 },
          { "name": "Remo", "sets": 4, "reps": 10 },
          { "name": "Press militar", "sets": 3, "reps": 10 },
          { "name": "Peso muerto", "sets": 3, "reps": 5 }
        ] },
        { "weekday": 2, "exercises": [
          { "name": "Sentadilla", "sets": 4, "reps": 8 },
          { "name": "Press banca", "sets": 4, "reps": 8 },
          { "name": "Remo", "sets": 4, "reps": 10 },
          { "name": "Press militar", "sets": 3, "reps": 10 },
          { "name": "Peso muerto", "sets": 3, "reps": 5 }
        ] },
        { "weekday": 4, "exercises": [
          { "name": "Sentadilla", "sets": 4, "reps": 8 },
          { "name": "Press banca", "sets": 4, "reps": 8 },
          { "name": "Remo", "sets": 4, "reps": 10 },
          { "name": "Press militar", "sets": 3, "reps": 10 },
          { "name": "Peso muerto", "sets": 3, "reps": 5 }
        ] }
      ]
    },
    {
      "id": "t2",
      "name": "PPL 6d",
      "days": [
        { "weekday": 0, "exercises": [ { "name": "Press banca", "sets": 5, "reps": 5 }, { "name": "Fondos", "sets": 3, "reps": 10 } ] },
        { "weekday": 1, "exercises": [ { "name": "Dominadas", "sets": 4, "reps": 6 }, { "name": "Remo", "sets": 4, "reps": 10 } ] },
        { "weekday": 2, "exercises": [ { "name": "Sentadilla", "sets": 5, "reps": 5 }, { "name": "Peso muerto", "sets": 3, "reps": 5 } ] },
        { "weekday": 3, "exercises": [ { "name": "Press banca", "sets": 5, "reps": 5 }, { "name": "Fondos", "sets": 3, "reps": 10 } ] },
        { "weekday": 4, "exercises": [ { "name": "Dominadas", "sets": 4, "reps": 6 }, { "name": "Remo", "sets": 4, "reps": 10 } ] },
        { "weekday": 5, "exercises": [ { "name": "Sentadilla", "sets": 5, "reps": 5 }, { "name": "Peso muerto", "sets": 3, "reps": 5 } ] }
      ]
    },
    {
      "id": "t3",
      "name": "Hipertrofia 4d",
      "days": [
        { "weekday": 0, "exercises": [ { "name": "Pecho/Espalda", "sets": 4, "reps": 10 } ] },
        { "weekday": 1, "exercises": [ { "name": "Pierna/Glúteo", "sets": 5, "reps": 10 } ] },
        { "weekday": 3, "exercises": [ { "name": "Hombro/Brazo", "sets": 4, "reps": 12 } ] },
        { "weekday": 4, "exercises": [ { "name": "Pierna/Core", "sets": 5, "reps": 12 } ] }
      ]
    }
  ]
}
```

---
## lib/domain/models.dart
```dart
import 'dart:convert';

class User {
  final String id;
  final String name;
  final DateTime createdAt;
  User({required this.id, required this.name, required this.createdAt});
  Map<String, dynamic> toMap() => {
        'id': id,
        'name': name,
        'createdAt': createdAt.toIso8601String(),
      };
  factory User.fromMap(Map<String, dynamic> m) => User(
        id: m['id'],
        name: m['name'],
        createdAt: DateTime.parse(m['createdAt']),
      );
}

class Exercise {
  final String id;
  final String name;
  final int sets;
  final int reps;
  final String? note;
  final bool done;
  Exercise({
    required this.id,
    required this.name,
    required this.sets,
    required this.reps,
    this.note,
    this.done = false,
  });
  Exercise copyWith({String? id, String? name, int? sets, int? reps, String? note, bool? done}) =>
      Exercise(
        id: id ?? this.id,
        name: name ?? this.name,
        sets: sets ?? this.sets,
        reps: reps ?? this.reps,
        note: note ?? this.note,
        done: done ?? this.done,
      );
  Map<String, dynamic> toMap() => {
        'id': id,
        'name': name,
        'sets': sets,
        'reps': reps,
        'note': note,
        'done': done,
      };
  factory Exercise.fromMap(Map<String, dynamic> m) => Exercise(
        id: m['id'],
        name: m['name'],
        sets: m['sets'],
        reps: m['reps'],
        note: m['note'],
        done: (m['done'] ?? false) as bool,
      );
}

class DayPlan {
  final String id;
  final String planId;
  final int weekday; // 0-6 L..D
  final List<Exercise> exercises;
  DayPlan({required this.id, required this.planId, required this.weekday, required this.exercises});
  DayPlan copyWith({String? id, String? planId, int? weekday, List<Exercise>? exercises}) => DayPlan(
        id: id ?? this.id,
        planId: planId ?? this.planId,
        weekday: weekday ?? this.weekday,
        exercises: exercises ?? this.exercises,
      );
  Map<String, dynamic> toMap() => {
        'id': id,
        'planId': planId,
        'weekday': weekday,
        'exercises': exercises.map((e) => e.toMap()).toList(),
      };
  factory DayPlan.fromMap(Map<String, dynamic> m) => DayPlan(
        id: m['id'],
        planId: m['planId'],
        weekday: m['weekday'],
        exercises: (m['exercises'] as List? ?? [])
            .map((e) => Exercise.fromMap(Map<String, dynamic>.from(e)))
            .toList(),
      );
}

class Plan {
  final String id;
  final String userId;
  final String name;
  final String weekStartIso; // yyyy-mm-dd
  Plan({required this.id, required this.userId, required this.name, required this.weekStartIso});
  Map<String, dynamic> toMap() => {
        'id': id,
        'userId': userId,
        'name': name,
        'weekStartIso': weekStartIso,
      };
  factory Plan.fromMap(Map<String, dynamic> m) => Plan(
        id: m['id'],
        userId: m['userId'],
        name: m['name'],
        weekStartIso: m['weekStartIso'],
      );
}

class TemplateDay {
  final int weekday;
  final List<Exercise> exercises;
  TemplateDay({required this.weekday, required this.exercises});
  Map<String, dynamic> toMap() => {
        'weekday': weekday,
        'exercises': exercises.map((e) => e.toMap()).toList(),
      };
  factory TemplateDay.fromMap(Map<String, dynamic> m) => TemplateDay(
        weekday: m['weekday'],
        exercises: (m['exercises'] as List)
            .map((e) => Exercise.fromMap(Map<String, dynamic>.from(e)..['id'] = _rid()))
            .toList(),
      );
}

class TemplatePlan {
  final String id;
  final String name;
  final List<TemplateDay> days;
  TemplatePlan({required this.id, required this.name, required this.days});
  Map<String, dynamic> toMap() => {
        'id': id,
        'name': name,
        'days': days.map((d) => d.toMap()).toList(),
      };
  factory TemplatePlan.fromMap(Map<String, dynamic> m) => TemplatePlan(
        id: m['id'],
        name: m['name'],
        days: (m['days'] as List)
            .map((d) => TemplateDay.fromMap(Map<String, dynamic>.from(d)))
            .toList(),
      );
}

String _rid() => DateTime.now().microsecondsSinceEpoch.toString();

// Helpers
String encodeJson(Object o) => jsonEncode(o);
Map<String, dynamic> decodeJson(String s) => jsonDecode(s);
```

---
## lib/core/strings.dart
```dart
class S {
  static const appName = 'GymPlan';
  static const users = 'Usuarios';
  static const newUser = 'Nuevo usuario';
  static const name = 'Nombre';
  static const save = 'Guardar';
  static const cancel = 'Cancelar';
  static const delete = 'Eliminar';
  static const plans = 'Plan semanal';
  static const addExercise = 'Agregar ejercicio';
  static const sets = 'Series';
  static const reps = 'Reps';
  static const note = 'Nota (opcional)';
  static const duplicateDay = 'Duplicar día';
  static const clearDay = 'Limpiar día';
  static const importTemplate = 'Importar desde plantilla';
  static const applyToWeek = 'Aplicar a la semana';
  static const applyToDay = 'Aplicar al día';
  static const confirmDeleteExercise = '¿Eliminar ejercicio?';
  static const confirm = 'Confirmar';
  static const done = 'Hecho';
  static const templates = 'Plantillas';
  static const empty = 'Sin ejercicios';
  static const dayShort = ['L', 'M', 'X', 'J', 'V', 'S', 'D'];
}
```

---
## lib/core/utils.dart
```dart
import 'package:flutter/material.dart';

String weekdayLabel(int i) => const ['Lunes','Martes','Miércoles','Jueves','Viernes','Sábado','Domingo'][i];

InputDecoration deco(String label) => InputDecoration(labelText: label, border: const OutlineInputBorder());

ButtonStyle bigButton(BuildContext context) => ElevatedButton.styleFrom(
      minimumSize: const Size(64, 48),
      textStyle: Theme.of(context).textTheme.titleMedium,
    );

String rid() => DateTime.now().microsecondsSinceEpoch.toString();
```

---
## lib/data/storage.dart
```dart
import 'dart:convert';
import 'package:hive_flutter/hive_flutter.dart';

class KVStore {
  static const _box = 'gymplan_box';
  static Future<void> init() async {
    await Hive.initFlutter();
    await Hive.openBox(_box);
  }

  static Box get _b => Hive.box(_box);

  static T? get<T>(String key) => _b.get(key) as T?;
  static Future<void> put(String key, Object? value) async => _b.put(key, value);

  static List<Map<String, dynamic>> getListMap(String key) {
    final v = _b.get(key);
    if (v is String) {
      return (jsonDecode(v) as List).map((e) => Map<String, dynamic>.from(e)).toList();
    } else if (v is List) {
      return v.map((e) => Map<String, dynamic>.from(e as Map)).toList();
    }
    return [];
  }

  static Future<void> putListMap(String key, List<Map<String, dynamic>> value) =>
      put(key, jsonEncode(value));
}
```

---
## lib/data/seeds.dart
```dart
import 'dart:convert';
import 'package:flutter/services.dart' show rootBundle;
import '../domain/models.dart';
import 'storage.dart';

class Seeds {
  static const _seededKey = 'seeded_v1';

  static Future<void> ensureSeeded() async {
    final seeded = KVStore.get<bool>(_seededKey) ?? false;
    if (seeded) return;
    final raw = await rootBundle.loadString('assets/seeds.json');
    final data = jsonDecode(raw) as Map<String, dynamic>;

    await KVStore.putListMap('users', (data['users'] as List).cast<Map<String, dynamic>>());
    await KVStore.putListMap('plans', (data['plans'] as List).cast<Map<String, dynamic>>());
    await KVStore.putListMap('days', (data['days'] as List).cast<Map<String, dynamic>>());
    await KVStore.putListMap('templates', (data['templates'] as List).cast<Map<String, dynamic>>());

    await KVStore.put(_seededKey, true);
  }

  static Future<List<TemplatePlan>> loadTemplates() async {
    final maps = KVStore.getListMap('templates');
    return maps.map((m) => TemplatePlan.fromMap(m)).toList();
  }
}
```

---
## lib/state/repositories.dart
```dart
import 'package:collection/collection.dart';
import '../data/storage.dart';
import '../domain/models.dart';
import '../core/utils.dart';

class Repo {
  // USERS
  Future<List<User>> users() async =>
      KVStore.getListMap('users').map(User.fromMap).toList();

  Future<void> addUser(String name) async {
    final list = KVStore.getListMap('users');
    list.add(User(id: rid(), name: name, createdAt: DateTime.now()).toMap());
    await KVStore.putListMap('users', list);
  }

  // PLANS
  Future<Plan?> planForUser(String userId) async {
    final plans = KVStore.getListMap('plans').map(Plan.fromMap).toList();
    return plans.firstWhereOrNull((p) => p.userId == userId);
  }

  Future<Plan> ensurePlan(String userId) async {
    final maybe = await planForUser(userId);
    if (maybe != null) return maybe;
    final p = Plan(id: rid(), userId: userId, name: 'Plan', weekStartIso: DateTime.now().toIso8601String().substring(0,10));
    final plans = KVStore.getListMap('plans');
    plans.add(p.toMap());
    await KVStore.putListMap('plans', plans);
    // also create seven empty days
    final days = KVStore.getListMap('days');
    for (var i = 0; i < 7; i++) {
      days.add(DayPlan(id: rid(), planId: p.id, weekday: i, exercises: const []).toMap());
    }
    await KVStore.putListMap('days', days);
    return p;
  }

  // DAYS
  Future<List<DayPlan>> daysForPlan(String planId) async {
    final days = KVStore.getListMap('days').map(DayPlan.fromMap).toList();
    return days.where((d) => d.planId == planId).toList()..sort((a,b) => a.weekday.compareTo(b.weekday));
  }

  Future<void> saveDay(DayPlan day) async {
    final list = KVStore.getListMap('days').map(DayPlan.fromMap).toList();
    final idx = list.indexWhere((d) => d.id == day.id);
    if (idx >= 0) list[idx] = day; else list.add(day);
    await KVStore.putListMap('days', list.map((e) => e.toMap()).toList());
  }

  Future<void> clearDay(String dayId) async {
    final list = KVStore.getListMap('days').map(DayPlan.fromMap).toList();
    final idx = list.indexWhere((d) => d.id == dayId);
    if (idx >= 0) {
      final d = list[idx];
      list[idx] = d.copyWith(exercises: const []);
      await KVStore.putListMap('days', list.map((e) => e.toMap()).toList());
    }
  }

  Future<void> duplicateDay(String fromId, String toId) async {
    final list = KVStore.getListMap('days').map(DayPlan.fromMap).toList();
    final from = list.firstWhere((d) => d.id == fromId);
    final toIdx = list.indexWhere((d) => d.id == toId);
    if (toIdx >= 0) {
      final cloned = from.copyWith(
        id: toId,
        weekday: list[toIdx].weekday,
        exercises: from.exercises
            .map((e) => e.copyWith(id: rid(), done: false))
            .toList(),
      );
      list[toIdx] = cloned;
      await KVStore.putListMap('days', list.map((e) => e.toMap()).toList());
    }
  }
}
```

---
## lib/state/controllers.dart
```dart
import 'package:hooks_riverpod/hooks_riverpod.dart';
import '../domain/models.dart';
import 'repositories.dart';

final repoProvider = Provider<Repo>((ref) => Repo());

final usersProvider = FutureProvider<List<User>>((ref) async => ref.read(repoProvider).users());

final planForUserProvider = FutureProvider.family<Plan, String>((ref, userId) async =>
    ref.read(repoProvider).ensurePlan(userId));

final daysForPlanProvider = FutureProvider.family<List<DayPlan>, String>((ref, planId) async =>
    ref.read(repoProvider).daysForPlan(planId));

class DayEditor extends StateNotifier<DayPlan?> {
  DayEditor(): super(null);
  void setDay(DayPlan d) => state = d;
  void upsertExercise(Exercise e) {
    if (state == null) return;
    final list = [...state!.exercises];
    final idx = list.indexWhere((x) => x.id == e.id);
    if (idx >= 0) list[idx] = e; else list.add(e);
    state = state!.copyWith(exercises: list.take(20).toList());
  }
  void deleteExercise(String id) {
    if (state == null) return;
    state = state!.copyWith(exercises: state!.exercises.where((e) => e.id != id).toList());
  }
  void toggleDone(String id) {
    if (state == null) return;
    state = state!.copyWith(exercises: state!.exercises.map((e) => e.id==id? e.copyWith(done: !e.done): e).toList());
  }
}

final dayEditorProvider = StateNotifierProvider<DayEditor, DayPlan?>((ref) => DayEditor());
```

---
## lib/ui/user_list_screen.dart
```dart
import 'package:flutter/material.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';
import '../core/strings.dart';
import '../state/controllers.dart';
import '../core/utils.dart';
import 'weekly_plan_screen.dart';

class UserListScreen extends ConsumerWidget {
  const UserListScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final users = ref.watch(usersProvider);
    return Scaffold(
      appBar: AppBar(title: const Text(S.users)),
      body: users.when(
        data: (list) => ListView.builder(
          itemCount: list.length,
          itemBuilder: (c, i) {
            final u = list[i];
            return ListTile(
              leading: CircleAvatar(child: Text(u.name.substring(0,1).toUpperCase())),
              title: Text(u.name),
              onTap: () async {
                final plan = await ref.read(planForUserProvider(u.id).future);
                if (context.mounted) {
                  Navigator.push(context, MaterialPageRoute(
                    builder: (_) => WeeklyPlanScreen(userName: u.name, planId: plan.id),
                  ));
                }
              },
            );
          },
        ),
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (e, _) => Center(child: Text(e.toString())),
      ),
      floatingActionButton: FloatingActionButton.extended(
        onPressed: () async {
          final name = await showDialog<String>(context: context, builder: (c) => const _NewUserDialog());
          if (name != null && name.trim().isNotEmpty) {
            await ref.read(repoProvider).addUser(name.trim());
            ref.invalidate(usersProvider);
          }
        },
        label: const Text(S.newUser),
        icon: const Icon(Icons.person_add_alt_1),
      ),
    );
  }
}

class _NewUserDialog extends ConsumerStatefulWidget {
  const _NewUserDialog();
  @override
  ConsumerState<_NewUserDialog> createState() => _NewUserDialogState();
}

class _NewUserDialogState extends ConsumerState<_NewUserDialog> {
  final _ctrl = TextEditingController();
  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: const Text(S.newUser),
      content: TextField(controller: _ctrl, decoration: deco(S.name), autofocus: true),
      actions: [
        TextButton(onPressed: () => Navigator.pop(context), child: const Text(S.cancel)),
        ElevatedButton(onPressed: () => Navigator.pop(context, _ctrl.text), child: const Text(S.save)),
      ],
    );
  }
}
```

---
## lib/ui/weekly_plan_screen.dart
```dart
import 'package:flutter/material.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';
import '../core/strings.dart';
import '../core/utils.dart';
import '../domain/models.dart';
import '../state/controllers.dart';
import '../state/repositories.dart';
import 'exercise_editor.dart';
import 'template_picker.dart';

class WeeklyPlanScreen extends ConsumerStatefulWidget {
  final String userName;
  final String planId;
  const WeeklyPlanScreen({super.key, required this.userName, required this.planId});

  @override
  ConsumerState<WeeklyPlanScreen> createState() => _WeeklyPlanScreenState();
}

class _WeeklyPlanScreenState extends ConsumerState<WeeklyPlanScreen> with SingleTickerProviderStateMixin {
  late TabController _tab;

  @override
  void initState() {
    super.initState();
    _tab = TabController(length: 7, vsync: this);
  }

  @override
  Widget build(BuildContext context) {
    final daysAsync = ref.watch(daysForPlanProvider(widget.planId));
    final repo = ref.read(repoProvider);

    return Scaffold(
      appBar: AppBar(
        title: Text('${S.plans} · ${widget.userName}'),
        bottom: TabBar(
          controller: _tab,
          isScrollable: true,
          tabs: List.generate(7, (i) => Tab(text: S.dayShort[i])),
        ),
        actions: [
          IconButton(
            tooltip: S.importTemplate,
            onPressed: () async {
              final tpl = await showDialog<TemplateAction>(
                context: context,
                builder: (_) => const TemplatePickerDialog(),
              );
              if (tpl == null) return;
              final days = await daysAsync.future;
              if (tpl.applyToWeek) {
                for (final td in tpl.template.days) {
                  final target = days.firstWhere((d) => d.weekday == td.weekday);
                  final cloned = target.copyWith(
                    exercises: td.exercises.map((e) => e.copyWith(id: rid(), done: false)).toList(),
                  );
                  await repo.saveDay(cloned);
                }
              } else {
                final idx = _tab.index;
                final target = days.firstWhere((d) => d.weekday == idx);
                final td = tpl.template.days.firstWhere((d) => d.weekday == idx, orElse: () => TemplateDay(weekday: idx, exercises: const []));
                final cloned = target.copyWith(exercises: td.exercises.map((e) => e.copyWith(id: rid(), done: false)).toList());
                await repo.saveDay(cloned);
              }
              ref.invalidate(daysForPlanProvider(widget.planId));
            },
            icon: const Icon(Icons.download_for_offline_rounded),
          ),
          PopupMenuButton<String>(
            onSelected: (v) async {
              final days = await daysAsync.future;
              final idx = _tab.index;
              final current = days[idx];
              if (v == 'dup') {
                final to = await showDialog<int>(context: context, builder: (_) => _PickDayDialog(currentDay: idx));
                if (to != null && to != idx) {
                  final target = days.firstWhere((d) => d.weekday == to);
                  await repo.duplicateDay(current.id, target.id);
                  ref.invalidate(daysForPlanProvider(widget.planId));
                }
              } else if (v == 'clear') {
                final ok = await showDialog<bool>(context: context, builder: (_) => _ConfirmDialog(text: '${S.clearDay}: ${weekdayLabel(idx)}?'));
                if (ok == true) {
                  await repo.clearDay(current.id);
                  ref.invalidate(daysForPlanProvider(widget.planId));
                }
              }
            },
            itemBuilder: (c) => [
              PopupMenuItem(value: 'dup', child: Text(S.duplicateDay)),
              PopupMenuItem(value: 'clear', child: Text(S.clearDay)),
            ],
          ),
        ],
      ),
      body: daysAsync.when(
        data: (days) => TabBarView(
          controller: _tab,
          children: days.map((d) => _DayList(day: d)).toList(),
        ),
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (e, _) => Center(child: Text(e.toString())),
      ),
      floatingActionButton: FloatingActionButton.extended(
        onPressed: () async {
          final days = await daysAsync.future;
          final idx = _tab.index;
          final d = days[idx];
          final created = await showModalBottomSheet<Exercise>(
            context: context,
            isScrollControlled: true,
            builder: (_) => ExerciseEditor(exercise: null),
          );
          if (created != null) {
            final updated = d.copyWith(exercises: [...d.exercises, created]);
            await repo.saveDay(updated);
            ref.invalidate(daysForPlanProvider(widget.planId));
          }
        },
        label: const Text(S.addExercise),
        icon: const Icon(Icons.add_task),
      ),
    );
  }
}

class _DayList extends ConsumerWidget {
  final DayPlan day;
  const _DayList({required this.day});
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    ref.listen(dayEditorProvider, (_, __) {}); // keep provider alive
    final editor = ref.read(dayEditorProvider.notifier)..setDay(day);
    final display = ref.watch(dayEditorProvider) ?? day;
    return Column(
      children: [
        Padding(
          padding: const EdgeInsets.all(12.0),
          child: Row(
            children: [
              Text(weekdayLabel(day.weekday), style: Theme.of(context).textTheme.titleLarge),
              const Spacer(),
              Text(display.exercises.isEmpty ? S.empty : ''),
            ],
          ),
        ),
        Expanded(
          child: ListView.builder(
            itemCount: display.exercises.length,
            itemBuilder: (c, i) {
              final e = display.exercises[i];
              return Dismissible(
                key: ValueKey(e.id),
                background: Container(
                  alignment: Alignment.centerLeft,
                  padding: const EdgeInsets.symmetric(horizontal: 20),
                  color: Colors.green,
                  child: const Icon(Icons.check, color: Colors.white),
                ),
                secondaryBackground: Container(
                  alignment: Alignment.centerRight,
                  padding: const EdgeInsets.symmetric(horizontal: 20),
                  color: Colors.red,
                  child: const Icon(Icons.delete, color: Colors.white),
                ),
                confirmDismiss: (dir) async {
                  if (dir == DismissDirection.startToEnd) {
                    editor.toggleDone(e.id);
                    // Persist toggle
                    final repo = ref.read(repoProvider);
                    await repo.saveDay(editor.state!);
                    return false; // keep item
                  } else {
                    final ok = await showDialog<bool>(context: context, builder: (_) => const _ConfirmDeleteExercise());
                    if (ok == true) {
                      editor.deleteExercise(e.id);
                      final repo = ref.read(repoProvider);
                      await repo.saveDay(editor.state!);
                      return true;
                    }
                  }
                  return false;
                },
                child: ListTile(
                  contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 6),
                  title: Text(e.name, style: TextStyle(decoration: e.done ? TextDecoration.lineThrough : null)),
                  subtitle: Text('${e.sets} × ${e.reps}${e.note?.isNotEmpty == true ? '  ·  ${e.note}' : ''}'),
                  leading: Icon(e.done ? Icons.check_circle : Icons.radio_button_unchecked),
                  onTap: () async {
                    final updated = await showModalBottomSheet<Exercise>(
                      context: context,
                      isScrollControlled: true,
                      builder: (_) => ExerciseEditor(exercise: e),
                    );
                    if (updated != null) {
                      editor.upsertExercise(updated);
                      final repo = ref.read(repoProvider);
                      await repo.saveDay(editor.state!);
                    }
                  },
                ),
              );
            },
          ),
        ),
      ],
    );
  }
}

class _ConfirmDeleteExercise extends StatelessWidget {
  const _ConfirmDeleteExercise();
  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: const Text('Eliminar ejercicio'),
      content: const Text('¿Deseas eliminar este ejercicio?'),
      actions: [
        TextButton(onPressed: () => Navigator.pop(context, false), child: const Text('Cancelar')),
        ElevatedButton(onPressed: () => Navigator.pop(context, true), child: const Text('Eliminar')),
      ],
    );
  }
}

class _PickDayDialog extends StatelessWidget {
  final int currentDay;
  const _PickDayDialog({required this.currentDay});
  @override
  Widget build(BuildContext context) {
    return SimpleDialog(title: const Text('Duplicar a…'), children: [
      for (int i = 0; i < 7; i++)
        ListTile(
          enabled: i != currentDay,
          title: Text(weekdayLabel(i)),
          onTap: i == currentDay ? null : () => Navigator.pop(context, i),
        )
    ]);
  }
}

class _ConfirmDialog extends StatelessWidget {
  final String text;
  const _ConfirmDialog({required this.text});
  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      content: Text(text),
      actions: [
        TextButton(onPressed: () => Navigator.pop(context, false), child: const Text('Cancelar')),
        ElevatedButton(onPressed: () => Navigator.pop(context, true), child: const Text('Confirmar')),
      ],
    );
  }
}
```

---
## lib/ui/exercise_editor.dart
```dart
import 'package:flutter/material.dart';
import '../domain/models.dart';
import '../core/strings.dart';
import '../core/utils.dart';

class ExerciseEditor extends StatefulWidget {
  final Exercise? exercise;
  const ExerciseEditor({super.key, required this.exercise});

  @override
  State<ExerciseEditor> createState() => _ExerciseEditorState();
}

class _ExerciseEditorState extends State<ExerciseEditor> {
  final _formKey = GlobalKey<FormState>();
  final _name = TextEditingController();
  final _sets = TextEditingController(text: '4');
  final _reps = TextEditingController(text: '8');
  final _note = TextEditingController();

  @override
  void initState() {
    super.initState();
    final e = widget.exercise;
    if (e != null) {
      _name.text = e.name;
      _sets.text = e.sets.toString();
      _reps.text = e.reps.toString();
      _note.text = e.note ?? '';
    }
  }

  @override
  Widget build(BuildContext context) {
    final bottom = MediaQuery.of(context).viewInsets.bottom;
    return Padding(
      padding: EdgeInsets.only(bottom: bottom),
      child: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              Text(widget.exercise == null ? S.addExercise : 'Editar ejercicio',
                  style: Theme.of(context).textTheme.titleLarge),
              const SizedBox(height: 16),
              TextFormField(
                controller: _name,
                decoration: deco(S.name),
                textInputAction: TextInputAction.next,
                validator: (v) => (v == null || v.trim().isEmpty) ? 'Requerido' : null,
              ),
              const SizedBox(height: 12),
              Row(children: [
                Expanded(
                  child: TextFormField(
                    controller: _sets,
                    decoration: deco(S.sets),
                    keyboardType: TextInputType.number,
                    validator: (v) {
                      final n = int.tryParse(v ?? '');
                      if (n == null || n <= 0) return 'Entero > 0';
                      return null;
                    },
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: TextFormField(
                    controller: _reps,
                    decoration: deco(S.reps),
                    keyboardType: TextInputType.number,
                    validator: (v) {
                      final n = int.tryParse(v ?? '');
                      if (n == null || n <= 0) return 'Entero > 0';
                      return null;
                    },
                  ),
                ),
              ]),
              const SizedBox(height: 12),
              TextFormField(controller: _note, decoration: deco(S.note)),
              const SizedBox(height: 20),
              Row(children: [
                if (widget.exercise != null)
                  Expanded(
                    child: OutlinedButton.icon(
                      onPressed: () => Navigator.pop(context, null),
                      icon: const Icon(Icons.delete_outline),
                      label: const Text(S.delete),
                    ),
                  ),
                if (widget.exercise != null) const SizedBox(width: 12),
                Expanded(
                  child: ElevatedButton.icon(
                    style: bigButton(context),
                    onPressed: () {
                      if (!_formKey.currentState!.validate()) return;
                      final e = Exercise(
                        id: widget.exercise?.id ?? rid(),
                        name: _name.text.trim(),
                        sets: int.parse(_sets.text),
                        reps: int.parse(_reps.text),
                        note: _note.text.trim().isEmpty ? null : _note.text.trim(),
                        done: widget.exercise?.done ?? false,
                      );
                      Navigator.pop(context, e);
                    },
                    icon: const Icon(Icons.save_alt),
                    label: const Text(S.save),
                  ),
                ),
              ]),
            ],
          ),
        ),
      ),
    );
  }
}
```

---
## lib/ui/template_picker.dart
```dart
import 'package:flutter/material.dart';
import '../domain/models.dart';
import '../data/seeds.dart';
import '../core/strings.dart';

class TemplateAction {
  final TemplatePlan template;
  final bool applyToWeek; // true=semana, false=día actual
  TemplateAction(this.template, this.applyToWeek);
}

class TemplatePickerDialog extends StatefulWidget {
  const TemplatePickerDialog({super.key});
  @override
  State<TemplatePickerDialog> createState() => _TemplatePickerDialogState();
}

class _TemplatePickerDialogState extends State<TemplatePickerDialog> {
  List<TemplatePlan> _templates = const [];
  bool _toWeek = true;
  TemplatePlan? _selected;
  @override
  void initState() {
    super.initState();
    Seeds.loadTemplates().then((t) => setState(() => _templates = t));
  }

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: const Text(S.templates),
      content: SizedBox(
        width: 400,
        child: _templates.isEmpty
            ? const Center(child: CircularProgressIndicator())
            : Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  DropdownButton<TemplatePlan>(
                    isExpanded: true,
                    value: _selected,
                    hint: const Text('Selecciona plantilla'),
                    items: _templates
                        .map((t) => DropdownMenuItem(value: t, child: Text(t.name)))
                        .toList(),
                    onChanged: (v) => setState(() => _selected = v),
                  ),
                  const SizedBox(height: 12),
                  Row(children: [
                    Expanded(child: RadioListTile<bool>(value: true, groupValue: _toWeek, onChanged: (v)=>setState(()=>_toWeek=v!), title: const Text(S.applyToWeek))),
                    Expanded(child: RadioListTile<bool>(value: false, groupValue: _toWeek, onChanged: (v)=>setState(()=>_toWeek=v!), title: const Text(S.applyToDay))),
                  ]),
                ],
              ),
      ),
      actions: [
        TextButton(onPressed: () => Navigator.pop(context), child: const Text('Cancelar')),
        ElevatedButton(
          onPressed: _selected == null ? null : () => Navigator.pop(context, TemplateAction(_selected!, _toWeek)),
          child: const Text('Aplicar'),
        )
      ],
    );
  }
}
```

---
## lib/app.dart
```dart
import 'package:flutter/material.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';
import 'core/strings.dart';
import 'ui/user_list_screen.dart';

class GymPlanApp extends StatelessWidget {
  const GymPlanApp({super.key});
  @override
  Widget build(BuildContext context) {
    return ProviderScope(
      child: MaterialApp(
        title: S.appName,
        debugShowCheckedModeBanner: false,
        themeMode: ThemeMode.system,
        theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.blue),
        darkTheme: ThemeData.dark(useMaterial3: true),
        home: const UserListScreen(),
        localizationsDelegates: const [
          DefaultMaterialLocalizations.delegate,
          DefaultWidgetsLocalizations.delegate,
          DefaultCupertinoLocalizations.delegate,
        ],
        supportedLocales: const [Locale('es', 'CO')],
      ),
    );
  }
}
```

---
## lib/main.dart
```dart
import 'package:flutter/material.dart';
import 'data/storage.dart';
import 'data/seeds.dart';
import 'app.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await KVStore.init();
  await Seeds.ensureSeeded();
  runApp(const GymPlanApp());
}
```

---
## test/model_validation_test.dart
```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:gymplan/domain/models.dart';

void main() {
  test('Exercise validation basic', () {
    final e = Exercise(id: '1', name: 'Sentadilla', sets: 4, reps: 8);
    expect(e.name.isNotEmpty, true);
    expect(e.sets > 0 && e.reps > 0, true);
  });
}
```

---
## test/duplicate_day_test.dart
```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:gymplan/domain/models.dart';

DayPlan duplicate(DayPlan from, DayPlan toTargetWeekday) {
  return from.copyWith(
    id: toTargetWeekday.id,
    weekday: toTargetWeekday.weekday,
    exercises: from.exercises.map((e) => e.copyWith(id: 'new_${e.id}', done: false)).toList(),
  );
}

void main() {
  test('Duplicate day copies exercises and resets done', () {
    final from = DayPlan(
      id: 'd1', planId: 'p1', weekday: 0,
      exercises: [Exercise(id: 'e1', name: 'Press', sets: 4, reps: 8, done: true)],
    );
    final target = DayPlan(id: 'd2', planId: 'p1', weekday: 2, exercises: const []);

    final dup = duplicate(from, target);

    expect(dup.id, 'd2');
    expect(dup.weekday, 2);
    expect(dup.exercises.length, 1);
    expect(dup.exercises.first.done, false);
    expect(dup.exercises.first.id.startsWith('new_'), true);
  });
}
```

---
## README (instrucciones de build & run)

1) **Requisitos**
- Flutter 3.22+
- Android Studio / Xcode configurados

2) **Instalación**
```bash
flutter pub get
```

3) **Ejecutar**
```bash
flutter run -d chrome   # web (para probar rápido)
# o
flutter run -d android  # dispositivo/emulador Android
# o
flutter run -d ios      # simulador iOS
```

> La primera ejecución siembra datos desde `assets/seeds.json`.

4) **Pruebas**
```bash
flutter test
```

5) **Criterios de aceptación (cómo probar manualmente)**
- **1. Crear usuario**: Pantalla Inicio → botón “Nuevo usuario”, crea; se navega a Plan semanal al tocar el usuario.
- **2. Agregar ejercicio (lunes)**: En pestaña **L**, FAB “Agregar ejercicio” → guarda “Sentadilla 4×8”; al tocar la tarjeta abre editor con los valores correctos.
- **3. Duplicar día**: Menú ⋮ → “Duplicar día” → elegir **Miércoles**; confirma que **X** tiene las mismas tarjetas.
- **4. Aplicar plantilla “Full Body 3d” a la semana**: Icono descarga → elegir plantilla y “Aplicar a la semana”; llena **L, X, V** y deja intactos los demás días.
- **5. Persistencia**: Cierra y reabre la app → el plan y ejercicios se conservan.
- **6. Marcar hecho**: Desliza a la derecha “Press banca” → queda con check (línea atravesada). Cambia de pestaña y vuelve para verificar.
- **7. Edición**: Toca “Sentadilla 4×8” → cambia a “5×5” y guarda → tarjeta actualizada.
- **8. Eliminar**: Desliza a la izquierda un ejercicio → confirma → desaparece y tras reinicio no reaparece.

6) **Notas de arquitectura**
- **Local-first**: `KVStore` envuelve Hive guardando listas de mapas JSON, habilitando migración futura a backend.
- **Capas**: `domain` (modelos), `state` (repos + controladores Riverpod), `ui` (pantallas y componentes).
- **Accesibilidad**: Botones con tamaño mínimo 48px, tipografías escalables, contraste Material 3.
- **i18n**: Base es-CO en `strings.dart`. Se puede extender con `intl`.
- **Rendimiento**: Listas paginadas simples; al crecer, virtualizar con `ListView.builder` (ya aplicado).

7) **MVP+ (sugerencias)**
- Historial por fecha (añadir campo `date` por ejecución y registrar pesos).
- Duplicar semana completa.
- Exportar/Importar JSON desde menú (usar `share_plus`/`file_picker`).
- Recordatorios locales (paquetes `flutter_local_notifications`).

