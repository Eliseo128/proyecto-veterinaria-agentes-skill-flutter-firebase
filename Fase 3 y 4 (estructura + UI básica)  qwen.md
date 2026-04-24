Aquí tienes el código completo y estructurado para la **Fase 3 (Estructura del Proyecto)** y **Fase 4 (UI, Navegación y Arquitectura)**. Está listo para copiar, pegar y ejecutar. Incluye navegación moderna con `go_router`, gestión de estado con `provider`, y una UI funcional con datos de prueba para validar el flujo visual antes de conectar Firestore (Fase 5).

---

## 📦 PASO 1: `pubspec.yaml`
Reemplaza el contenido actual con:
```yaml
name: proyectoveterinaria
description: App de gestión veterinaria - CRUD Mascotas
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: '>=3.1.0 <4.0.0'
  flutter: '>=3.19.0'

dependencies:
  flutter:
    sdk: flutter
  go_router: ^14.0.0
  provider: ^6.1.2
  intl: ^0.19.0
  uuid: ^4.3.3
  firebase_core: ^2.27.0
  cloud_firestore: ^4.15.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.1

flutter:
  uses-material-design: true
```
Ejecuta en terminal: `flutter pub get`

---

## 📁 PASO 2: Estructura de Carpetas
Crea manualmente o con tu IDE:
```
lib/
├── core/
│   ├── constants/app_constants.dart
│   ├── theme/app_theme.dart
│   └── router/app_router.dart
├── data/models/mascota_model.dart
├── presentation/
│   ├── providers/mascota_provider.dart
│   ├── screens/
│   │   ├── home_screen.dart
│   │   ├── mascota_list_screen.dart
│   │   ├── mascota_form_screen.dart
│   │   └── mascota_detail_screen.dart
│   └── widgets/
│       ├── custom_button.dart
│       └── empty_state.dart
└── main.dart
```

---

## 🎨 PASO 3: Archivos Core

### `lib/core/constants/app_constants.dart`
```dart
class AppConstants {
  static const String appName = 'Veterinaria App';
  static const String routeHome = '/';
  static const String routeList = '/mascotas';
  static const String routeForm = '/mascota/form';
  static const String routeDetail = '/mascota/:id';
}
```

### `lib/core/theme/app_theme.dart`
```dart
import 'package:flutter/material.dart';

class AppTheme {
  static ThemeData get lightTheme => ThemeData(
        primaryColor: const Color(0xFF4CAF50),
        scaffoldBackgroundColor: Colors.grey[100],
        appBarTheme: AppBarTheme(
          backgroundColor: const Color(0xFF4CAF50),
          foregroundColor: Colors.white,
          elevation: 0,
        ),
        elevatedButtonTheme: ElevatedButtonThemeData(
          style: ElevatedButton.styleFrom(
            backgroundColor: const Color(0xFF4CAF50),
            foregroundColor: Colors.white,
            shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
          ),
        ),
        inputDecorationTheme: InputDecorationTheme(
          filled: true,
          fillColor: Colors.white,
          border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)),
          contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 14),
        ),
      );
}
```

### `lib/core/router/app_router.dart`
```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import '../../presentation/screens/home_screen.dart';
import '../../presentation/screens/mascota_list_screen.dart';
import '../../presentation/screens/mascota_form_screen.dart';
import '../../presentation/screens/mascota_detail_screen.dart';
import '../constants/app_constants.dart';

final appRouter = GoRouter(
  initialLocation: AppConstants.routeHome,
  routes: [
    GoRoute(
      path: AppConstants.routeHome,
      builder: (_, __) => const HomeScreen(),
    ),
    GoRoute(
      path: AppConstants.routeList,
      builder: (_, __) => const MascotaListScreen(),
    ),
    GoRoute(
      path: AppConstants.routeForm,
      builder: (_, state) => MascotaFormScreen(mascotaId: state.extra as String?),
    ),
    GoRoute(
      path: AppConstants.routeDetail,
      builder: (_, state) => MascotaDetailScreen(id: state.pathParameters['id']!),
    ),
  ],
);
```

---

## 🧱 PASO 4: Modelo & Provider (Scaffold)

### `lib/data/models/mascota_model.dart`
```dart
class MascotaModel {
  final String id;
  final String nombre;
  final String especie;
  final String raza;
  final int edad;
  final String propietario;
  final DateTime fechaRegistro;

  MascotaModel({
    required this.id,
    required this.nombre,
    required this.especie,
    required this.raza,
    required this.edad,
    required this.propietario,
    DateTime? fechaRegistro,
  }) : fechaRegistro = fechaRegistro ?? DateTime.now();

  factory MascotaModel.empty() => MascotaModel(
        id: '', nombre: '', especie: '', raza: '', edad: 0, propietario: '');

  MascotaModel copyWith({
    String? id, String? nombre, String? especie,
    String? raza, int? edad, String? propietario,
  }) => MascotaModel(
        id: id ?? this.id,
        nombre: nombre ?? this.nombre,
        especie: especie ?? this.especie,
        raza: raza ?? this.raza,
        edad: edad ?? this.edad,
        propietario: propietario ?? this.propietario,
      );
}
```

### `lib/presentation/providers/mascota_provider.dart`
```dart
import 'package:flutter/foundation.dart';
import '../../data/models/mascota_model.dart';

class MascotaProvider extends ChangeNotifier {
  final List<MascotaModel> _mascotas = [];
  bool _isLoading = false;

  List<MascotaModel> get mascotas => List.unmodifiable(_mascotas);
  bool get isLoading => _isLoading;

  // 🔧 Fase 5: Aquí se conectará Firestore. Por ahora, datos mock para UI.
  Future<void> init() async {
    _isLoading = true;
    notifyListeners();
    await Future.delayed(const Duration(milliseconds: 800));
    _mascotas.addAll([
      MascotaModel(id: '1', nombre: 'Max', especie: 'Perro', raza: 'Labrador', edad: 3, propietario: 'Carlos'),
      MascotaModel(id: '2', nombre: 'Luna', especie: 'Gato', raza: 'Siamés', edad: 2, propietario: 'Ana'),
    ]);
    _isLoading = false;
    notifyListeners();
  }

  void add(MascotaModel mascota) {
    _mascotas.add(mascota);
    notifyListeners();
  }

  void update(MascotaModel mascota) {
    final index = _mascotas.indexWhere((m) => m.id == mascota.id);
    if (index != -1) _mascotas[index] = mascota;
    notifyListeners();
  }

  void delete(String id) {
    _mascotas.removeWhere((m) => m.id == id);
    notifyListeners();
  }

  MascotaModel? getById(String id) => _mascotas.firstWhere((m) => m.id == id, orElse: () => MascotaModel.empty());
}
```

---

## 🖼️ PASO 5: Widgets & Pantallas

### `lib/presentation/widgets/custom_button.dart`
```dart
import 'package:flutter/material.dart';

class CustomButton extends StatelessWidget {
  final String text;
  final VoidCallback onPressed;
  final Color? color;
  final IconData? icon;

  const CustomButton({super.key, required this.text, required this.onPressed, this.color, this.icon});

  @override
  Widget build(BuildContext context) {
    return ElevatedButton.icon(
      onPressed: onPressed,
      icon: icon != null ? Icon(icon, size: 20) : const SizedBox.shrink(),
      label: Text(text),
      style: ElevatedButton.styleFrom(
        backgroundColor: color ?? Theme.of(context).primaryColor,
        padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 14),
      ),
    );
  }
}
```

### `lib/presentation/widgets/empty_state.dart`
```dart
import 'package:flutter/material.dart';

class EmptyState extends StatelessWidget {
  final String message;
  const EmptyState({super.key, required this.message});

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.pets_outlined, size: 80, color: Colors.grey[400]),
          const SizedBox(height: 16),
          Text(message, style: TextStyle(color: Colors.grey[600], fontSize: 16)),
        ],
      ),
    );
  }
}
```

### `lib/presentation/screens/home_screen.dart`
```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import '../../core/constants/app_constants.dart';
import '../widgets/custom_button.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Veterinaria')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.pets, size: 100, color: Color(0xFF4CAF50)),
            const SizedBox(height: 24),
            const Text('Gestión de Mascotas', style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold)),
            const SizedBox(height: 40),
            CustomButton(
              text: 'Ver Mascotas',
              icon: Icons.list,
              onPressed: () => context.push(AppConstants.routeList),
            ),
          ],
        ),
      ),
    );
  }
}
```

### `lib/presentation/screens/mascota_list_screen.dart`
```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import 'package:provider/provider.dart';
import '../../core/constants/app_constants.dart';
import '../providers/mascota_provider.dart';
import '../widgets/empty_state.dart';

class MascotaListScreen extends StatefulWidget {
  const MascotaListScreen({super.key});

  @override
  State<MascotaListScreen> createState() => _MascotaListScreenState();
}

class _MascotaListScreenState extends State<MascotaListScreen> {
  @override
  void initState() {
    super.initState();
    context.read<MascotaProvider>().init();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Mascotas Registradas')),
      body: Consumer<MascotaProvider>(
        builder: (_, provider, __) {
          if (provider.isLoading) return const Center(child: CircularProgressIndicator());
          if (provider.mascotas.isEmpty) return const EmptyState(message: 'No hay mascotas registradas');

          return ListView.builder(
            padding: const EdgeInsets.all(16),
            itemCount: provider.mascotas.length,
            itemBuilder: (_, i) {
              final m = provider.mascotas[i];
              return Card(
                margin: const EdgeInsets.only(bottom: 12),
                child: ListTile(
                  leading: CircleAvatar(child: Text(m.nombre[0].toUpperCase())),
                  title: Text(m.nombre, style: const TextStyle(fontWeight: FontWeight.bold)),
                  subtitle: Text('${m.especie} • ${m.raza} • ${m.edad} años'),
                  trailing: IconButton(icon: const Icon(Icons.delete, color: Colors.red), onPressed: () => provider.delete(m.id)),
                  onTap: () => context.push('${AppConstants.routeDetail}/${m.id}'),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => context.push(AppConstants.routeForm, extra: null),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### `lib/presentation/screens/mascota_form_screen.dart`
```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import 'package:provider/provider.dart';
import 'package:uuid/uuid.dart';
import '../../data/models/mascota_model.dart';
import '../providers/mascota_provider.dart';
import '../widgets/custom_button.dart';

class MascotaFormScreen extends StatefulWidget {
  final String? mascotaId;
  const MascotaFormScreen({super.key, this.mascotaId});

  @override
  State<MascotaFormScreen> createState() => _MascotaFormScreenState();
}

class _MascotaFormScreenState extends State<MascotaFormScreen> {
  final _formKey = GlobalKey<FormState>();
  final _nombreCtrl = TextEditingController();
  final _especieCtrl = TextEditingController();
  final _razaCtrl = TextEditingController();
  final _edadCtrl = TextEditingController();
  final _propietarioCtrl = TextEditingController();
  bool _isEdit = false;

  @override
  void initState() {
    super.initState();
    if (widget.mascotaId != null) {
      _isEdit = true;
      final mascota = context.read<MascotaProvider>().getById(widget.mascotaId!);
      if (mascota != null) {
        _nombreCtrl.text = mascota.nombre;
        _especieCtrl.text = mascota.especie;
        _razaCtrl.text = mascota.raza;
        _edadCtrl.text = mascota.edad.toString();
        _propietarioCtrl.text = mascota.propietario;
      }
    }
  }

  @override
  void dispose() {
    _nombreCtrl.dispose(); _especieCtrl.dispose(); _razaCtrl.dispose();
    _edadCtrl.dispose(); _propietarioCtrl.dispose();
    super.dispose();
  }

  void _guardar() {
    if (_formKey.currentState!.validate()) {
      final mascota = MascotaModel(
        id: _isEdit ? widget.mascotaId! : const Uuid().v4(),
        nombre: _nombreCtrl.text.trim(),
        especie: _especieCtrl.text.trim(),
        raza: _razaCtrl.text.trim(),
        edad: int.parse(_edadCtrl.text.trim()),
        propietario: _propietarioCtrl.text.trim(),
      );
      _isEdit ? context.read<MascotaProvider>().update(mascota) : context.read<MascotaProvider>().add(mascota);
      context.pop();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(_isEdit ? 'Editar Mascota' : 'Nueva Mascota')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: ListView(
            children: [
              TextFormField(controller: _nombreCtrl, decoration: const InputDecoration(labelText: 'Nombre'), validator: (v) => v!.isEmpty ? 'Obligatorio' : null),
              const SizedBox(height: 12),
              TextFormField(controller: _especieCtrl, decoration: const InputDecoration(labelText: 'Especie (Perro/Gato)')),
              const SizedBox(height: 12),
              TextFormField(controller: _razaCtrl, decoration: const InputDecoration(labelText: 'Raza')),
              const SizedBox(height: 12),
              TextFormField(controller: _edadCtrl, decoration: const InputDecoration(labelText: 'Edad'), keyboardType: TextInputType.number),
              const SizedBox(height: 12),
              TextFormField(controller: _propietarioCtrl, decoration: const InputDecoration(labelText: 'Propietario')),
              const SizedBox(height: 24),
              CustomButton(text: _isEdit ? 'Actualizar' : 'Registrar', icon: Icons.save, onPressed: _guardar),
            ],
          ),
        ),
      ),
    );
  }
}
```

### `lib/presentation/screens/mascota_detail_screen.dart`
```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import 'package:provider/provider.dart';
import '../../core/constants/app_constants.dart';
import '../providers/mascota_provider.dart';

class MascotaDetailScreen extends StatelessWidget {
  final String id;
  const MascotaDetailScreen({super.key, required this.id});

  @override
  Widget build(BuildContext context) {
    final mascota = context.read<MascotaProvider>().getById(id);
    if (mascota == null || mascota.id.isEmpty) {
      return Scaffold(appBar: AppBar(title: const Text('Detalle')), body: const Center(child: Text('Mascota no encontrada')));
    }

    return Scaffold(
      appBar: AppBar(title: Text(mascota.nombre)),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            _detailRow('Especie', mascota.especie),
            _detailRow('Raza', mascota.raza),
            _detailRow('Edad', '${mascota.edad} años'),
            _detailRow('Propietario', mascota.propietario),
            const SizedBox(height: 32),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                OutlinedButton.icon(
                  onPressed: () => context.push(AppConstants.routeForm, extra: mascota.id),
                  icon: const Icon(Icons.edit), label: const Text('Editar'),
                ),
                ElevatedButton.icon(
                  onPressed: () {
                    context.read<MascotaProvider>().delete(mascota.id);
                    context.pop();
                  },
                  icon: const Icon(Icons.delete), label: const Text('Eliminar'),
                  style: ElevatedButton.styleFrom(backgroundColor: Colors.red),
                ),
              ],
            )
          ],
        ),
      ),
    );
  }

  Widget _detailRow(String label, String value) => Padding(
        padding: const EdgeInsets.symmetric(vertical: 8),
        child: Row(children: [Text('$label: ', style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 16)), Text(value, style: const TextStyle(fontSize: 16))]),
      );
}
```

---

## 🚀 PASO 6: `main.dart`
```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:provider/provider.dart';
import 'core/router/app_router.dart';
import 'core/theme/app_theme.dart';
import 'presentation/providers/mascota_provider.dart';
import 'firebase_options.dart'; // Generado por flutterfire configure

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // ⚠️ Si aún no configuraste Firebase, comenta estas 3 líneas temporalmente
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );

  runApp(
    ChangeNotifierProvider(
      create: (_) => MascotaProvider(),
      child: const MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Veterinaria App',
      debugShowCheckedModeBanner: false,
      theme: AppTheme.lightTheme,
      routerConfig: appRouter,
    );
  }
}
```

---

## ✅ VERIFICACIÓN RÁPIDA
1. Ejecuta: `flutter clean && flutter pub get`
2. Si ya hiciste `flutterfire configure`, descomenta las líneas de Firebase en `main.dart`. Si no, mantenlas comentadas temporalmente.
3. Ejecuta: `flutter run`
4. Deberías ver:
   - Pantalla de inicio → Botón "Ver Mascotas"
   - Lista con 2 mascotas de prueba
   - Botón `+` abre formulario → Guarda → Aparece en lista
   - Tap en mascota → Detalle → Editar/Eliminar funcionan
   - Navegación fluida con `GoRouter`

---

## 🔜 Siguiente Paso: **Fase 5 (CRUD Real con Firestore)**
Cuando confirmes `✅ Fases 3 y 4 completadas`, generaré:
- Conexión real a Firestore (`cloud_firestore`)
- Repositorio `MascotaRepository` con `Stream` en tiempo real
- Manejo de errores, loading states, y sincronización offline
- Configuración de índices y reglas de seguridad
- Pruebas de escritura/lectura desde consola Firebase

¿Listo para avanzar a la Fase 5 o necesitas ajustes en la UI/estructura actual?
