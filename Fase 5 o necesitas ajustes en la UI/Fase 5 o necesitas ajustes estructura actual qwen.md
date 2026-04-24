¡Perfecto! Aquí tienes la **Fase 5 completa**: integración real con **Firebase Firestore**, arquitectura de repositorio, gestión de estados en tiempo real y manejo de errores. El código reemplaza los datos mock y conecta directamente a la nube.

---

## 📦 PASO 1: Actualizar `pubspec.yaml`
Asegúrate de tener estas versiones (ya incluidas en la fase anterior):
```yaml
dependencies:
  firebase_core: ^2.27.0
  cloud_firestore: ^4.15.0
  provider: ^6.1.2
  go_router: ^14.0.0
  uuid: ^4.3.3
```
Ejecuta: `flutter pub get`

---

## 🗃️ PASO 2: Actualizar Modelo (`lib/data/models/mascota_model.dart`)
Agregamos `toMap()` para serialización y mejoramos la seguridad de tipos:
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

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'nombre': nombre,
      'especie': especie,
      'raza': raza,
      'edad': edad,
      'propietario': propietario,
      'fechaRegistro': fechaRegistro,
    };
  }

  factory MascotaModel.fromFirestore(String id, Map<String, dynamic> data) {
    return MascotaModel(
      id: id,
      nombre: data['nombre'] as String? ?? '',
      especie: data['especie'] as String? ?? 'Sin especificar',
      raza: data['raza'] as String? ?? 'Mestizo',
      edad: data['edad'] as int? ?? 0,
      propietario: data['propietario'] as String? ?? 'Sin asignar',
      fechaRegistro: (data['fechaRegistro'] as dynamic?)?.toDate() ?? DateTime.now(),
    );
  }

  MascotaModel copyWith({
    String? id, String? nombre, String? especie,
    String? raza, int? edad, String? propietario, DateTime? fechaRegistro,
  }) => MascotaModel(
        id: id ?? this.id,
        nombre: nombre ?? this.nombre,
        especie: especie ?? this.especie,
        raza: raza ?? this.raza,
        edad: edad ?? this.edad,
        propietario: propietario ?? this.propietario,
        fechaRegistro: fechaRegistro ?? this.fechaRegistro,
      );
}
```

---

## ☁️ PASO 3: Repositorio Firestore (`lib/data/repositories/mascota_repository.dart`)
```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/mascota_model.dart';

class MascotaRepository {
  final FirebaseFirestore _db = FirebaseFirestore.instance;
  final String _collection = 'mascotas';

  Stream<List<MascotaModel>> streamMascotas() {
    return _db
        .collection(_collection)
        .orderBy('fechaRegistro', descending: true)
        .snapshots()
        .map((snapshot) =>
            snapshot.docs.map((doc) => MascotaModel.fromFirestore(doc.id, doc.data())).toList());
  }

  Future<String> crearMascota(MascotaModel mascota) async {
    final docRef = _db.collection(_collection).doc();
    final data = mascota.copyWith(id: docRef.id).toMap();
    // Firestore maneja DateTime nativamente, pero usamos serverTimestamp para auditoría
    data['fechaRegistro'] = FieldValue.serverTimestamp();
    await docRef.set(data);
    return docRef.id;
  }

  Future<void> actualizarMascota(MascotaModel mascota) async {
    final data = mascota.toMap();
    data.remove('id'); // El ID no se actualiza
    data.remove('fechaRegistro'); // No modificamos fecha de registro
    await _db.collection(_collection).doc(mascota.id).update(data);
  }

  Future<void> eliminarMascota(String id) async {
    await _db.collection(_collection).doc(id).delete();
  }

  Future<MascotaModel?> obtenerPorId(String id) async {
    final doc = await _db.collection(_collection).doc(id).get();
    return doc.exists ? MascotaModel.fromFirestore(doc.id, doc.data()!) : null;
  }
}
```

---

## 🔄 PASO 4: Provider Actualizado (`lib/presentation/providers/mascota_provider.dart`)
Gestión de suscripción en tiempo real + manejo de errores:
```dart
import 'dart:async';
import 'package:flutter/foundation.dart';
import '../../data/models/mascota_model.dart';
import '../../data/repositories/mascota_repository.dart';

class MascotaProvider extends ChangeNotifier {
  final MascotaRepository _repository = MascotaRepository();
  StreamSubscription<List<MascotaModel>>? _subscription;
  List<MascotaModel> _mascotas = [];
  bool _isLoading = true;
  String? _errorMessage;

  List<MascotaModel> get mascotas => _mascotas;
  bool get isLoading => _isLoading;
  String? get errorMessage => _errorMessage;

  void startListening() {
    _isLoading = true;
    _errorMessage = null;
    notifyListeners();

    _subscription?.cancel();
    _subscription = _repository.streamMascotas().listen(
      (mascotas) {
        _mascotas = mascotas;
        _isLoading = false;
        notifyListeners();
      },
      onError: (error) {
        _errorMessage = 'Error de conexión: ${error.toString()}';
        _isLoading = false;
        notifyListeners();
      },
    );
  }

  @override
  void dispose() {
    _subscription?.cancel();
    super.dispose();
  }

  Future<void> crearMascota(MascotaModel mascota) async {
    try {
      await _repository.crearMascota(mascota);
    } catch (e) {
      _errorMessage = 'Error al registrar: $e';
      notifyListeners();
    }
  }

  Future<void> actualizarMascota(MascotaModel mascota) async {
    try {
      await _repository.actualizarMascota(mascota);
    } catch (e) {
      _errorMessage = 'Error al actualizar: $e';
      notifyListeners();
    }
  }

  Future<void> eliminarMascota(String id) async {
    try {
      await _repository.eliminarMascota(id);
    } catch (e) {
      _errorMessage = 'Error al eliminar: $e';
      notifyListeners();
    }
  }

  Future<MascotaModel?> cargarMascota(String id) async {
    return await _repository.obtenerPorId(id);
  }

  void clearError() {
    _errorMessage = null;
    notifyListeners();
  }
}
```

---

## 📱 PASO 5: Pantallas Actualizadas

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
    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (mounted) context.read<MascotaProvider>().startListening();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Mascotas Registradas')),
      body: Consumer<MascotaProvider>(
        builder: (_, provider, __) {
          if (provider.errorMessage != null) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Icon(Icons.error_outline, color: Colors.red, size: 48),
                  const SizedBox(height: 12),
                  Text(provider.errorMessage!, textAlign: TextAlign.center, style: const TextStyle(color: Colors.red)),
                  const SizedBox(height: 16),
                  ElevatedButton(onPressed: provider.startListening, child: const Text('Reintentar')),
                ],
              ),
            );
          }

          if (provider.isLoading) return const Center(child: CircularProgressIndicator());
          if (provider.mascotas.isEmpty) return const EmptyState(message: 'No hay mascotas registradas');

          return ListView.builder(
            padding: const EdgeInsets.all(16),
            itemCount: provider.mascotas.length,
            itemBuilder: (_, i) {
              final m = provider.mascotas[i];
              return Dismissible(
                key: Key(m.id),
                background: Container(color: Colors.red, alignment: Alignment.centerRight, padding: const EdgeInsets.only(right: 20), child: const Icon(Icons.delete, color: Colors.white)),
                onDismissed: (_) => provider.eliminarMascota(m.id),
                child: Card(
                  margin: const EdgeInsets.only(bottom: 12),
                  child: ListTile(
                    leading: CircleAvatar(backgroundColor: Theme.of(context).primaryColor.withOpacity(0.2), child: Text(m.nombre.isNotEmpty ? m.nombre[0].toUpperCase() : '?')),
                    title: Text(m.nombre, style: const TextStyle(fontWeight: FontWeight.bold)),
                    subtitle: Text('${m.especie} • ${m.raza} • ${m.edad} años'),
                    onTap: () => context.push('${AppConstants.routeDetail}/${m.id}'),
                  ),
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
  bool _submitting = false;

  @override
  void initState() {
    super.initState();
    _loadData();
  }

  Future<void> _loadData() async {
    if (widget.mascotaId != null) {
      _isEdit = true;
      final mascota = await context.read<MascotaProvider>().cargarMascota(widget.mascotaId!);
      if (mascota != null && mounted) {
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

  Future<void> _guardar() async {
    if (!_formKey.currentState!.validate()) return;

    setState(() => _submitting = true);
    final provider = context.read<MascotaProvider>();

    final mascota = MascotaModel(
      id: _isEdit ? widget.mascotaId! : const Uuid().v4(),
      nombre: _nombreCtrl.text.trim(),
      especie: _especieCtrl.text.trim(),
      raza: _razaCtrl.text.trim(),
      edad: int.tryParse(_edadCtrl.text.trim()) ?? 0,
      propietario: _propietarioCtrl.text.trim(),
    );

    if (_isEdit) {
      await provider.actualizarMascota(mascota);
    } else {
      await provider.crearMascota(mascota);
    }

    if (mounted) {
      setState(() => _submitting = false);
      if (provider.errorMessage == null) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text(_isEdit ? 'Mascota actualizada' : 'Mascota registrada'), backgroundColor: Colors.green),
        );
        context.pop();
      } else {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text(provider.errorMessage!), backgroundColor: Colors.red),
        );
      }
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
              TextFormField(controller: _especieCtrl, decoration: const InputDecoration(labelText: 'Especie')),
              const SizedBox(height: 12),
              TextFormField(controller: _razaCtrl, decoration: const InputDecoration(labelText: 'Raza')),
              const SizedBox(height: 12),
              TextFormField(controller: _edadCtrl, decoration: const InputDecoration(labelText: 'Edad'), keyboardType: TextInputType.number),
              const SizedBox(height: 12),
              TextFormField(controller: _propietarioCtrl, decoration: const InputDecoration(labelText: 'Propietario')),
              const SizedBox(height: 24),
              _submitting
                  ? const Center(child: CircularProgressIndicator())
                  : CustomButton(
                      text: _isEdit ? 'Actualizar' : 'Registrar',
                      icon: Icons.save,
                      onPressed: _guardar,
                    ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## ⚙️ PASO 6: Configuración Firebase & Pruebas

1. **Ejecuta configuración:**
   ```bash
   flutterfire configure --project=proyectoveterinaria
   ```
   Selecciona `android` e `ios` según tu plataforma.

2. **Reglas de Firestore (`firestore.rules`):**
   ```text
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /mascotas/{mascotaId} {
         allow read, write: if request.auth != null || true; // Modo prueba
       }
     }
   }
   ```
   Despliega: `firebase deploy --only firestore`

3. **Verificación en consola:**
   - Abre Firebase Console → Firestore → Colección `mascotas`
   - Registra una mascota desde la app → debe aparecer en tiempo real
   - Edita/elimina desde la app → los cambios se reflejan instantáneamente
   - Desconecta internet → verifica que la app no crashea (Firestore cachea localmente)

---

## ✅ CHECKLIST FASE 5
- [x] Modelo con `toMap()` y `fromFirestore()`
- [x] Repositorio con Stream, CRUD y manejo de Timestamps
- [x] Provider con suscripción controlada y manejo de errores
- [x] Pantallas con estados de carga, error y éxito
- [x] Integración real con Firestore en tiempo real

---

¿Confirmas `✅ Fase 5 completada` para proceder a la **Fase 6** (Optimización, Reglas de Seguridad en producción, validaciones avanzadas, y generación de scripts de automatización `.agents/scripts`)?
