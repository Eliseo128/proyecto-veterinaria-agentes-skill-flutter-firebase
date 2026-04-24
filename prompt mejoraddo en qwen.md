Aquí tienes una versión **optimizada, modular y lista para usar** con un agente de IA o como guía de desarrollo. Sigue una secuencia lógica (Setup → Firebase → Arquitectura → UI → Lógica → Verificación) y está estructurada para que la IA genere el código paso a paso sin saturar el contexto.

---

# 🐾 PROMPT MAESTRO: Agente de Desarrollo Veterinaria (Flutter + Firestore CRUD)

## 🎯 Objetivo
Crear una aplicación móvil funcional para una veterinaria usando **Flutter**, centrada inicialmente en un **CRUD completo de mascotas** (Crear, Leer, Actualizar, Eliminar) con **Firebase Firestore** como backend. El desarrollo seguirá una secuencia lógica, modular y escalable, utilizando la estructura de agente `.agents`.

---

## 📁 FASE 0: Estructura del Agente Global (`.agents`)
1. Crear carpeta `.agents/` en la raíz del proyecto.
2. Dentro, generar:
   - `SKILL.md`: Definir 3 habilidades:
     - 🎨 `diseño`: Patrones UI, temas, accesibilidad.
     - 💻 `código`: Arquitectura, mejores prácticas Flutter/Dart, gestión de estado.
     - 🕷️ `scraping/automatización`: Scripts de entorno, generación de boilerplate, validación de configuraciones.
   - `/scripts`: Archivos `.sh` o `.bat` para automatizar `pub get`, `flutterfire configure`, limpieza de caché, etc.
   - `/ejemplos`: Snippets de referencia (ej. modelo Firestore, formulario validado, stream builder).
   - `/resources`: Reglas de Firestore, estructura de colección, documentación técnica breve.

---

## 🔧 FASE 1: Prerrequisitos y Verificación del Entorno
- Ejecutar y validar:
  ```bash
  flutter --version
  dart --version
  firebase --version
  ```
- Si falta alguna herramienta, guiar paso a paso con enlaces oficiales y comandos de instalación.
- Autenticación y conectividad Firebase:
  ```bash
  firebase login
  firebase projects:list
  ```
- IDE principal: **VS Code** (extensiones: Flutter, Dart, Firebase). Si se usa otro IDE (ej. Antigravity), indicar cómo importar el proyecto y activar hot-reload.
- Ejecutar `flutter doctor` y resolver warnings antes de continuar.

---

## ☁️ FASE 2: Proyecto Firebase y Firestore
1. Crear proyecto en Firebase Console: `proyectoveterinaria`.
2. Registrar app Android/Flutter.
3. Instalar CLI de FlutterFire:
   ```bash
   dart pub global activate flutterfire_cli
   flutterfire configure
   ```
4. Habilitar **Firestore Database** (modo prueba inicial).
5. Configurar reglas básicas (`firestore.rules` en `/resources`):
   ```text
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /{document=**} {
         allow read, write: if request.time < timestamp.date(2027, 1, 1);
       }
     }
   }
   ```
6. Definir colección `mascotas` con campos: `id`, `nombre`, `especie`, `raza`, `edad`, `propietario`, `fechaRegistro`, `imagenUrl` (opcional).

---

## 🏗️ FASE 3: Estructura del Proyecto Flutter
1. Crear proyecto:
   ```bash
   flutter create proyectoveterinaria
   cd proyectoveterinaria
   ```
2. Estructura recomendada (layer-first):
   ```
   lib/
   ├── core/           (config, theme, utils, constants)
   ├── data/           (models, repositories, datasources)
   ├── domain/         (entities, usecases)
   ├── presentation/   (screens, widgets, controllers)
   ├── main.dart
   └── firebase_options.dart (generado automáticamente)
   ```
3. Configurar `pubspec.yaml`:
   ```yaml
   dependencies:
     flutter: { sdk: flutter }
     firebase_core: ^latest
     cloud_firestore: ^latest
     provider: ^latest  # o riverpod
     uuid: ^latest
     intl: ^latest
   ```

---

## 🖼️ FASE 4: UI, Navegación y Arquitectura
- Implementar `MaterialApp` con **GoRouter** o `Navigator 2.0`.
- Pantallas requeridas:
  1. `HomeScreen`: Dashboard con acceso al CRUD.
  2. `MascotaListScreen`: Lista con `StreamBuilder`, indicador de carga, estado vacío, swipe-to-delete.
  3. `MascotaFormScreen`: Crear/Editar con `Form`, validación `TextFormField`, selección de especie/raza.
  4. `MascotaDetailScreen`: Vista de lectura con opción a editar.
- Tema unificado, widgets reutilizables (`CustomButton`, `LoadingIndicator`, `ErrorBanner`).
- Navegación tipada y manejo de rutas protegidas si aplica.

---

## ⚙️ FASE 5: Lógica CRUD + Integración Firestore
- **Modelo** `lib/data/models/mascota_model.dart`:
  - `fromFirestore()`, `toMap()`, `copyWith()`.
- **Repositorio** `lib/data/repositories/mascota_repository.dart`:
  - `create()`, `streamAll()`, `update()`, `delete()`.
  - Manejo de excepciones `FirebaseException`.
- **Estado/Controlador** (Provider/Riverpod):
  - `MascotaController` con `StateNotifier` o `ChangeNotifier`.
  - Estados: `loading`, `data`, `error`.
- Validaciones: campos obligatorios, límites de edad, confirmación de eliminación.

---

## 🧪 FASE 6: Pruebas y Verificación Final
1. Limpieza y configuración:
   ```bash
   flutter clean && flutter pub get
   flutterfire configure
   ```
2. Ejecutar:
   ```bash
   flutter run
   ```
3. Verificar:
   - Conexión Firebase (`firebase_options.dart` cargado).
   - Operaciones CRUD completas sin errores.
   - UI fluida, manejo de errores, estados de carga.
   - Logs útiles en consola para depuración.
4. Incluir comandos de debugging y cómo revisar Firestore Console en tiempo real.

---

## 📤 FORMATO DE SALIDA ESPERADO (PARA LA IA)
- Generar archivos `.dart` **con rutas exactas**.
- Código completo, comentado, listo para copiar/pegar.
- Instrucciones claras por fase. No avances a la siguiente hasta recibir `✅ Fase X completada`.
- Si requiere interacción del usuario (ej. descargar `google-services.json` o aceptar permisos), indicarlo explícitamente.
- Mantener compatibilidad con **Dart 3.x** y **Flutter 3.19+**.

---

## ✅ INSTRUCCIÓN FINAL PARA EL AGENTE
> Ejecuta este flujo paso a paso. Solicita confirmación al final de cada fase antes de continuar. Genera el código estructurado, respetando la arquitectura definida, y asegura que el proyecto sea funcional desde la primera ejecución. Usa `.agents/SKILL.md` como referencia interna para mantener coherencia de diseño, código y automatización.

---

### 💡 Cómo usarlo:
1. Copia y pega este prompt completo en tu agente IA.
2. La IA devolverá la **Fase 0 y 1** primero. Responde `✅ Fase 1 completada` cuando termines los pasos locales.
3. Avanza fase por fase. Esto evita cortes de contexto y garantiza código probado y modular.

¿Quieres que te genere directamente el código de la **Fase 3 y 4** (estructura + UI básica) para comenzar con la implementación?
