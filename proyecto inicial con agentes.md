Para desarrollar la habilidad global `.agents` y el proyecto **proyectoveterinaria**, seguiremos una estructura modular. Este enfoque permite que tus agentes de diseño, código y scraping trabajen de forma sincronizada.

---

## 1. Estructura de la Habilidad Global `.agents`

Primero, configuramos el entorno de trabajo para tus agentes inteligentes.

### Archivo: `SKILL.md`
Este archivo sirve como el "manual de instrucciones" para tus agentes.
```markdown
# Skill: Agente Global Veterinario (.agents)
**Descripción:** Automatización para el desarrollo de apps veterinarias.
**Capacidades:** - Generación de UI (Diseño)
- Lógica de Firebase (Código)
- Extracción de datos médicos (Scraping)

## Estructura de Carpetas
- /scripts: Automatismos de despliegue y limpieza.
- /ejemplos: Templates de pantallas CRUD.
- /resources: Assets, iconos y temas de color.
```

---

## 2. Prerrequisitos y Configuración del Entorno

Antes de codificar, debemos asegurar que el entorno en **VS Code** o **Antigravity** sea estable.

### Verificación de Herramientas
Ejecuta estos comandos en tu terminal:
1. **Flutter:** `flutter doctor` (Verifica que Flutter esté instalado).
2. **Firebase CLI:** Si no lo tienes, instala con `npm install -g firebase-tools`.
3. **Login:** `firebase login` para vincular tu cuenta de Google.

### Configuración de Firebase
1. Ve a [Firebase Console](https://console.firebase.google.com/).
2. Crea el proyecto: `veterinaria-app`.
3. Habilita **Cloud Firestore** en modo de prueba.
4. Registra tu app Android (usa el nombre del paquete `com.veterinaria.app`).
5. Descarga el archivo `google-services.json` y colócalo en `android/app/`.

---

## 3. Estructura del Proyecto: `proyectoveterinaria`

### Configuración de `pubspec.yaml`
Añade las dependencias necesarias para conectar con Firestore:
```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.0.0
  cloud_firestore: ^4.0.0
  cupertino_icons: ^1.0.2
```

---

## 4. Implementación del CRUD de Mascotas

### Modelo de Datos: `lib/models/mascota_model.dart`
```dart
class Mascota {
  String id;
  String nombre;
  String especie;
  int edad;

  Mascota({required this.id, required this.nombre, required this.especie, required this.edad});

  Map<String, dynamic> toMap() {
    return {'nombre': nombre, 'especie': especie, 'edad': edad};
  }
}
```

### Servicio Firestore: `lib/services/firebase_service.dart`
```dart
import 'package:cloud_firestore/cloud_firestore.dart';

FirebaseFirestore db = FirebaseFirestore.instance;

Future<List> getMascotas() async {
  List mascotas = [];
  QuerySnapshot querySnapshot = await db.collection('mascotas').get();
  for (var doc in querySnapshot.docs) {
    mascotas.add(doc.data());
  }
  return mascotas;
}

Future<void> addMascota(String nombre, String especie, int edad) async {
  await db.collection('mascotas').add({'nombre': nombre, 'especie': especie, 'edad': edad});
}
```

### Interfaz de Usuario (UI)

#### Pantalla de Inicio: `lib/pages/home_page.dart`
Esta pantalla muestra la lista y permite navegar al formulario.
```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';

class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Veterinaria .agents')),
      body: FutureBuilder(
        future: getMascotas(),
        builder: (context, snapshot) {
          if (snapshot.hasData) {
            return ListView.builder(
              itemCount: snapshot.data?.length,
              itemBuilder: (context, index) {
                return ListTile(
                  title: Text(snapshot.data?[index]['nombre']),
                  subtitle: Text(snapshot.data?[index]['especie']),
                  trailing: Icon(Icons.pets),
                );
              },
            );
          } else {
            return Center(child: CircularProgressIndicator());
          }
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => Navigator.pushNamed(context, '/add'),
        child: Icon(Icons.add),
      ),
    );
  }
}
```

---

## 5. Secuencia Lógica de Ejecución

1.  **Inicialización:** Ejecuta `flutter create proyectoveterinaria`.
2.  **Vinculación:** Usa `flutterfire configure` para enlazar automáticamente el proyecto con Firebase (esto genera el archivo `firebase_options.dart`).
3.  **Main:** En `lib/main.dart`, asegúrate de inicializar Firebase:
    ```dart
    void main() async {
      WidgetsFlutterBinding.ensureInitialized();
      await Firebase.initializeApp();
      runApp(MaterialApp(
        initialRoute: '/',
        routes: {
          '/': (context) => HomePage(),
          // Agregar rutas de editar y borrar aquí
        },
      ));
    }
    ```

## 6. Siguientes Pasos para tus Sub-Skills
* **Skill de Diseño:** Personalizar `resources/theme.dart` con colores pasteles para la clínica.
* **Skill de Scraping:** Crear un script en `/scripts` que extraiga nombres de razas de perros desde una web externa para pre-llenar los formularios.
* **Skill de Código:** Implementar las funciones `update` y `delete` en el service de Firestore.

¿Deseas que profundice en el código de la pantalla para "Actualizar" o prefieres configurar el script de scraping primero?
