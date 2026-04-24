Para completar el ciclo del **CRUD** (Create, Read, Update, Delete), activamos el **Agente de Código**. En esta fase, conectaremos las acciones de la interfaz de usuario con los métodos específicos de Firebase Firestore, manteniendo la estética definida por el Agente de Diseño.

Aquí tienes la implementación lógica y estructurada para tu proyecto veterinario.

---

### 1. Actualización del Servicio: `lib/services/firebase_service.dart`

Añadiremos las funciones `updateMascota` y `deleteMascota`. Observa cómo usamos el `id` del documento de Firestore para asegurar que afectamos al registro correcto.

```dart
import 'package:cloud_firestore/cloud_firestore.dart';

FirebaseFirestore db = FirebaseFirestore.instance;
final CollectionReference collection = db.collection('mascotas');

// --- READ (Obtener con IDs) ---
Future<List<Map<String, dynamic>>> getMascotas() async {
  List<Map<String, dynamic>> mascotas = [];
  QuerySnapshot querySnapshot = await collection.get();
  
  for (var doc in querySnapshot.docs) {
    final Map<String, dynamic> data = doc.data() as Map<String, dynamic>;
    mascotas.add({
      "id": doc.id, // Guardamos el ID de Firebase para editar/borrar
      "nombre": data['nombre'],
      "especie": data['especie'],
      "edad": data['edad'],
    });
  }
  return mascotas;
}

// --- UPDATE (Actualizar) ---
Future<void> updateMascota(String id, String nuevoNombre, String nuevaEspecie, int nuevaEdad) async {
  await collection.doc(id).update({
    'nombre': nuevoNombre,
    'especie': nuevaEspecie,
    'edad': nuevaEdad,
  });
}

// --- DELETE (Borrar) ---
Future<void> deleteMascota(String id) async {
  await collection.doc(id).delete();
}
```

---

### 2. Integración en la UI: `lib/pages/home_page.dart`

Para que el usuario pueda borrar, implementaremos el widget `Dismissible` (deslizar para borrar) o un botón de eliminar con un diálogo de confirmación, usando los colores de `Palette.coralSoft` para acciones de peligro.



```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';
import '../resources/palette.dart';

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Expediente Mascotas')),
      body: FutureBuilder(
        future: getMascotas(),
        builder: (context, snapshot) {
          if (snapshot.hasData) {
            return ListView.builder(
              itemCount: snapshot.data?.length,
              itemBuilder: (context, index) {
                var mascota = snapshot.data![index];
                return Dismissible(
                  key: Key(mascota['id']),
                  direction: DismissDirection.endToStart,
                  background: Container(
                    color: Palette.coralSoft,
                    alignment: Alignment.centerRight,
                    padding: const EdgeInsets.symmetric(horizontal: 20),
                    child: const Icon(Icons.delete, color: Colors.red),
                  ),
                  onDismissed: (direction) async {
                    await deleteMascota(mascota['id']);
                    ScaffoldMessenger.of(context).showSnackBar(
                      SnackBar(content: Text("${mascota['nombre']} eliminado")),
                    );
                  },
                  child: Card(
                    margin: const EdgeInsets.all(8),
                    child: ListTile(
                      title: Text(mascota['nombre']),
                      subtitle: Text("Raza: ${mascota['especie']}"),
                      leading: const CircleAvatar(
                        backgroundColor: Palette.bluePastel,
                        child: Icon(Icons.pets, color: Colors.white),
                      ),
                      onTap: () {
                        // Aquí llamaríamos a la lógica de UPDATE (Abrir formulario con datos)
                        _mostrarDialogoEdicion(context, mascota);
                      },
                    ),
                  ),
                );
              },
            );
          }
          return const Center(child: CircularProgressIndicator());
        },
      ),
    );
  }

  // Función interna para el UPDATE mediante un Dialog
  void _mostrarDialogoEdicion(BuildContext context, Map mascota) {
    TextEditingController controller = TextEditingController(text: mascota['nombre']);
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text("Editar ${mascota['nombre']}"),
        content: TextField(controller: controller),
        actions: [
          TextButton(onPressed: () => Navigator.pop(context), child: const Text("Cancelar")),
          ElevatedButton(
            onPressed: () async {
              await updateMascota(mascota['id'], controller.text, mascota['especie'], mascota['edad']);
              Navigator.pop(context);
              setState(() {}); // Refrescar UI
            },
            child: const Text("Actualizar"),
          )
        ],
      ),
    );
  }
}
```

---

### 3. Verificación Final de la Secuencia Logica

Con estos cambios, tu agente `.agents` ha completado:
1.  **Prerrequisitos:** Conectividad con Firebase Console.
2.  **Diseño:** Tematización pastel en `resources/`.
3.  **Scraping:** Obtención de razas para el `Dropdown` de creación.
4.  **Código:** Lógica completa de persistencia en Firestore (CRUD).

### Pruebas de funcionamiento:
* **Crear:** Usa el botón `+` y verás las razas extraídas por el scraper.
* **Leer:** La lista se carga automáticamente desde Firestore al iniciar.
* **Actualizar:** Toca cualquier tarjeta para cambiar el nombre.
* **Borrar:** Desliza una tarjeta hacia la izquierda para eliminar el registro.

El proyecto **proyectoveterinaria** ya es totalmente funcional. ¿Te gustaría generar ahora el archivo `scripts/deploy.sh` para automatizar la subida de reglas de seguridad a Firebase o prefieres añadir una pantalla de estadísticas de mascotas?
