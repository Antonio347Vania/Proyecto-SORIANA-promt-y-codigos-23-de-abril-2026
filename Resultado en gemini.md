¡Hola! Como desarrollador de software, me encanta la estructura que planteas. Vamos a construir este ecosistema utilizando **Flutter**, **Firebase** y el framework de agentes **Antigravity**.

Este no es solo un CRUD convencional; es un sistema orientado a agentes donde cada operación está delegada a una entidad lógica.

---

## 1. Metodología de Trabajo: Estructura de Agentes
Para que los estudiantes entiendan **Antigravity**, dividiremos la lógica en roles específicos:

* **Agente de Persistencia (Firebase Agent):** Encargado de la comunicación directa con Firestore.
* **Agente de Validación:** Revisa que los datos (precio, stock) sean correctos antes de enviarlos.
* **Flujo de Trabajo (Workflow):** Coordina la UI con los agentes.

---

## 2. Configuración Inicial y Directorios

Primero, prepárate en tu terminal:

```bash
# Crear estructura de carpetas
mkdir xflutterantonio0469
cd xflutterantonio0469
flutter create crudsoriana
cd crudsoriana
```

### Estructura de Archivos Recomendada
```text
crudsoriana/
├── lib/
│   ├── agents/
│   │   ├── firebase_agent.dart (Skills: CRUD)
│   │   └── validator_agent.dart (Skills: Check Data)
│   ├── models/
│   │   └── producto_model.dart
│   ├── screens/
│   │   └── home_screen.dart (UI Atractiva)
│   └── main.dart
└── pubspec.yaml
```

---

## 3. Dependencias (pubspec.yaml)
Para integrar Firebase y Antigravity, edita tu archivo `pubspec.yaml` bajo la sección `dependencies`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.24.2
  cloud_firestore: ^4.14.0
  antigravity: ^latest_version # Framework de agentes
  google_fonts: ^6.1.0        # Para UI atractiva
```
*Luego ejecuta `flutter pub get` en la terminal.*

---

## 4. Configuración en Consola Firebase
1.  Ve a [Firebase Console](https://console.firebase.google.com/).
2.  Crea un proyecto llamado `crudsoriana`.
3.  Habilita **Cloud Firestore** en modo de prueba.
4.  Crea una colección llamada **"productos"**.
5.  Agrega un documento de ejemplo con campos: `nombre` (string), `precio` (number), `stock` (number).

---

## 5. Implementación del Código (Funcional)

### A. El Modelo de Datos (`models/producto_model.dart`)
```dart
class Producto {
  String id;
  String nombre;
  double precio;
  int stock;

  Producto({required this.id, required this.nombre, required this.precio, required this.stock});

  Map<String, dynamic> toMap() => {
    "nombre": nombre,
    "precio": precio,
    "stock": stock,
  };

  factory Producto.fromFirestore(Map<String, dynamic> data, String id) {
    return Producto(
      id: id,
      nombre: data['nombre'] ?? '',
      precio: (data['precio'] ?? 0).toDouble(),
      stock: data['stock'] ?? 0,
    );
  }
}
```

### B. El Agente CRUD con Antigravity (`agents/firebase_agent.dart`)
Aquí definimos el **Rol** y las **Skills** (habilidades) del agente.

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/producto_model.dart';

class FirebaseAgent {
  final FirebaseFirestore _db = FirebaseFirestore.instance;

  // Skill: Crear
  Future<void> createProduct(Producto p) async {
    await _db.collection('productos').add(p.toMap());
  }

  // Skill: Leer (Stream para tiempo real)
  Stream<List<Producto>> getProducts() {
    return _db.collection('productos').snapshots().map((snapshot) =>
        snapshot.docs.map((doc) => Producto.fromFirestore(doc.data(), doc.id)).toList());
  }

  // Skill: Actualizar
  Future<void> updateProduct(Producto p) async {
    await _db.collection('productos').doc(p.id).update(p.toMap());
  }

  // Skill: Borrar
  Future<void> deleteProduct(String id) async {
    await _db.collection('productos').doc(id).delete();
  }
}
```

### C. Interfaz de Usuario (`screens/home_screen.dart`)
Diseño con colores atractivos (Indigo y Amber).

```dart
import 'package:flutter/material.dart';
import '../agents/firebase_agent.dart';
import '../models/producto_model.dart';

class HomeScreen extends StatelessWidget {
  final FirebaseAgent _agent = FirebaseAgent();

  void _showForm(BuildContext context, [Producto? producto]) {
    final nombreCtrl = TextEditingController(text: producto?.nombre ?? '');
    final precioCtrl = TextEditingController(text: producto?.precio.toString() ?? '');
    final stockCtrl = TextEditingController(text: producto?.stock.toString() ?? '');

    showModalBottomSheet(
      context: context,
      isScrollControlled: true,
      backgroundColor: Colors.grey[100],
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.vertical(top: Radius.circular(25))),
      builder: (_) => Padding(
        padding: EdgeInsets.only(bottom: MediaQuery.of(context).viewInsets.bottom, left: 20, right: 20, top: 20),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(producto == null ? "Nuevo Producto" : "Editar Producto", style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold, color: Colors.indigo)),
            TextField(controller: nombreCtrl, decoration: InputDecoration(labelText: "Nombre")),
            TextField(controller: precioCtrl, decoration: InputDecoration(labelText: "Precio"), keyboardType: TextInputType.number),
            TextField(controller: stockCtrl, decoration: InputDecoration(labelText: "Stock"), keyboardType: TextInputType.number),
            SizedBox(height: 20),
            ElevatedButton(
              style: ElevatedButton.styleFrom(backgroundColor: Colors.indigo, foregroundColor: Colors.white),
              onPressed: () {
                final p = Producto(
                  id: producto?.id ?? '',
                  nombre: nombreCtrl.text,
                  precio: double.parse(precioCtrl.text),
                  stock: int.parse(stockCtrl.text),
                );
                producto == null ? _agent.createProduct(p) : _agent.updateProduct(p);
                Navigator.pop(context);
              },
              child: Text("Guardar"),
            ),
            SizedBox(height: 10),
          ],
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("CRUD Soriana - Antigravity"), backgroundColor: Colors.indigo, foregroundColor: Colors.white),
      floatingActionButton: FloatingActionButton(
        backgroundColor: Colors.amber,
        child: Icon(Icons.add, color: Colors.black),
        onPressed: () => _showForm(context),
      ),
      body: StreamBuilder<List<Producto>>(
        stream: _agent.getProducts(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          final productos = snapshot.data!;
          return ListView.builder(
            itemCount: productos.length,
            itemBuilder: (context, index) {
              final p = productos[index];
              return Card(
                margin: EdgeInsets.symmetric(horizontal: 15, vertical: 8),
                elevation: 4,
                child: ListTile(
                  leading: CircleAvatar(backgroundColor: Colors.indigo, child: Text(p.nombre[0], style: TextStyle(color: Colors.white))),
                  title: Text(p.nombre, style: TextStyle(fontWeight: FontWeight.bold)),
                  subtitle: Text("Precio: \$${p.precio} | Stock: ${p.stock}"),
                  trailing: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      IconButton(icon: Icon(Icons.edit, color: Colors.blue), onPressed: () => _showForm(context, p)),
                      IconButton(icon: Icon(Icons.delete, color: Colors.red), onPressed: () => _agent.deleteProduct(p.id)),
                    ],
                  ),
                ),
              );
            },
          );
        },
      ),
    );
  }
}
```

---

## 6. Punto de Entrada (`main.dart`)
```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'screens/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Inicialización vital de Firebase
  runApp(MaterialApp(
    debugShowCheckedModeBanner: false,
    theme: ThemeData(primarySwatch: Colors.indigo, useMaterial3: true),
    home: HomeScreen(),
  ));
}
```

### Resumen para Estudiantes:
1.  **Agente:** El `FirebaseAgent` es el cerebro que sabe hablar con la nube.
2.  **Skill:** `createProduct` o `deleteProduct` son las habilidades de ese agente.
3.  **Flujo:** La UI captura el evento -> El agente procesa -> Firestore actualiza -> La UI reacciona mediante el `StreamBuilder`.

¿Te gustaría que añadamos una regla de validación especial con un segundo agente para el stock mínimo?
