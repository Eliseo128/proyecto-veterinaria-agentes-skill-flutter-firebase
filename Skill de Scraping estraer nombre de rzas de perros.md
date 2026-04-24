Para activar el **Agente de Scraping**, utilizaremos el paquete `http` y `html` (basado en Beautiful Soup de Python) para extraer datos de una fuente confiable. En este caso, simularemos la extracción de una lista de razas para que el usuario no tenga que escribirlas manualmente en el CRUD.

---

## 1. Configuración del Agente de Scraping

Primero, añade las dependencias necesarias en tu `pubspec.yaml`:

```yaml
dependencies:
  http: ^1.1.0
  html: ^0.15.4
```

### Archivo: `scripts/breed_scraper.dart`
Este script actúa de forma independiente o como un servicio dentro de la app para alimentar la base de datos de Firestore con razas oficiales.

```dart
import 'package:http/http.dart' as http;
import 'package:html/parser.dart' show parse;

class BreedScraper {
  // URL de ejemplo (Wikipedia o sitio de razas)
  final String url = "https://en.wikipedia.org/wiki/List_of_dog_breeds";

  Future<List<String>> fetchBreeds() async {
    List<String> breeds = [];

    try {
      final response = await http.get(Uri.parse(url));

      if (response.statusCode == 200) {
        var document = parse(response.body);
        
        // Buscamos los elementos en la tabla o lista de la web
        // Este selector varía según la estructura del sitio
        var elements = document.querySelectorAll('.mw-parser-output ul li a');

        for (var element in elements) {
          String name = element.text.trim();
          // Filtro simple para evitar textos irrelevantes
          if (name.length > 3 && !name.contains('^')) {
            breeds.add(name);
          }
        }
        print("Scraping exitoso: ${breeds.length} razas encontradas.");
      }
    } catch (e) {
      print("Error en el Agente de Scraping: $e");
    }

    return breeds.take(20).toList(); // Retornamos las primeras 20 para el ejemplo
  }
}
```

---

## 2. Integración con el Formulario (Skill de Código)

Ahora, el **Agente de Código** toma la información del scraper para mejorar el formulario de registro de mascotas en la carpeta `lib/pages/add_mascota_page.dart`.

### Archivo: `lib/pages/add_mascota_page.dart`

```dart
import 'package:flutter/material.dart';
import '../../scripts/breed_scraper.dart'; // Import del script
import '../resources/palette.dart';

class AddMascotaPage extends StatefulWidget {
  @override
  _AddMascotaPageState createState() => _AddMascotaPageState();
}

class _AddMascotaPageState extends State<AddMascotaPage> {
  final _scraper = BreedScraper();
  List<String> _razas = ["Cargando razas..."];
  String? _razaSeleccionada;

  @override
  void initState() {
    super.initState();
    _cargarRazas();
  }

  void _cargarRazas() async {
    var lista = await _scraper.fetchBreeds();
    setState(() {
      _razas = lista;
      _razaSeleccionada = _razas.isNotEmpty ? _razas[0] : null;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Nueva Mascota")),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          children: [
            TextField(
              decoration: InputDecoration(labelText: "Nombre de la mascota"),
            ),
            const SizedBox(height: 20),
            DropdownButtonFormField<String>(
              value: _razaSeleccionada,
              decoration: InputDecoration(labelText: "Raza (Auto-completada)"),
              items: _razas.map((raza) {
                return DropdownMenuItem(value: raza, child: Text(raza));
              }).toList(),
              onChanged: (val) => setState(() => _razaSeleccionada = val),
            ),
            const SizedBox(height: 30),
            ElevatedButton(
              style: ElevatedButton.styleFrom(
                backgroundColor: Palette.mintPastel,
                minimumSize: Size(double.infinity, 50),
              ),
              onPressed: () {
                // Lógica para guardar en Firebase
              },
              child: Text("Guardar en Expediente"),
            )
          ],
        ),
      ),
    );
  }
}
```

---

## 3. Verificación de Funcionalidad

Para asegurar que todo esté conectado correctamente bajo la arquitectura `.agents`:

1.  **Terminal:** Ejecuta `flutter pub get` para instalar las librerías de scraping.
2.  **Firebase:** Asegúrate de que la colección `mascotas` en Firestore tenga permisos de escritura.
3.  **Debug:** Si el scraper falla por CORS (en web), recuerda que este script está diseñado primordialmente para **Android/iOS** o para ser ejecutado como un script de servidor en la carpeta `/scripts`.

### Resumen de la secuencia logica:
1.  **Diseño** creó la paleta pastel.
2.  **Scraping** obtuvo los datos del mundo real.
3.  **Código** unificó ambos en un formulario funcional.

¿Deseas que implementemos ahora la funcionalidad de **Cámara** (para fotos de las mascotas) o prefieres finalizar el **CRUD** con la lógica de "Borrar" y "Editar"?
