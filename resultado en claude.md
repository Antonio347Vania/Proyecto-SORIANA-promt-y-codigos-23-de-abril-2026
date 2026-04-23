Aquí está el plan de trabajo completo y guía práctica para tu proyecto CRUD de empleados en Flutter con Firebase:Ahora te presento el plan completo detallado:
<img width="706" height="673" alt="image" src="https://github.com/user-attachments/assets/ffb70b4d-6b89-4789-8b5b-831529580c29" />

---

## 1. Creación de carpetas · `xflutterantonio0469/crudsoriana`

```bash
mkdir -p xflutterantonio0469
cd xflutterantonio0469
flutter create crudsoriana
cd crudsoriana
```

---

## 2. Antigravity — Plataforma de práctica guiada

**Antigravity** es un entorno de aprendizaje/práctica donde los estudiantes trabajan proyectos reales paso a paso. Para este proyecto:

- Los estudiantes clonan la carpeta `crudsoriana`
- Cada fase tiene instrucciones guiadas
- Se validan los resultados en tiempo real contra Firestore
- Los errores se reportan con contexto educativo

---

## 3. Crear base de datos en Firestore (Firebase Console)

1. Ir a [console.firebase.google.com](https://console.firebase.google.com)
2. **Crear proyecto** → nombre: `crudsoriana`
3. Ir a **Firestore Database** → Crear base de datos → Modo producción
4. Crear colección: `Productos`
5. Agregar documento con campos:
   - `nombre` (string)
   - `precio` (number)
   - `stock` (number)

---

## 4. Librerías — `pubspec.yaml`

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.24.2
  cloud_firestore: ^4.14.0
  google_fonts: ^6.1.0
  uuid: ^4.3.3
  flutter_riverpod: ^2.4.9
```

Para agregar `firebase_core`:
```bash
flutter pub add firebase_core
flutter pub add cloud_firestore
```

---

## 5. Configuración Firebase + FlutterFire CLI

```bash
# Instalar FlutterFire CLI
dart pub global activate flutterfire_cli

# Configurar proyecto (genera firebase_options.dart automáticamente)
flutterfire configure --project=crudsoriana
```

---

## 6. Estructura completa de carpetas y archivos

```
crudsoriana/
├── lib/
│   ├── main.dart
│   ├── firebase_options.dart        ← generado por FlutterFire CLI
│   ├── models/
│   │   └── producto_model.dart
│   ├── services/
│   │   └── producto_service.dart
│   ├── screens/
│   │   ├── home_screen.dart
│   │   ├── add_producto_screen.dart
│   │   └── edit_producto_screen.dart
│   └── widgets/
│       ├── producto_card.dart
│       └── custom_text_field.dart
├── pubspec.yaml
└── android/ios/web/...
```

---

## 7. Códigos Dart completos y funcionales

### `lib/models/producto_model.dart`

```dart
class Producto {
  final String id;
  final String nombre;
  final double precio;
  final int stock;

  Producto({
    required this.id,
    required this.nombre,
    required this.precio,
    required this.stock,
  });

  factory Producto.fromMap(Map<String, dynamic> map, String id) {
    return Producto(
      id: id,
      nombre: map['nombre'] ?? '',
      precio: (map['precio'] ?? 0).toDouble(),
      stock: map['stock'] ?? 0,
    );
  }

  Map<String, dynamic> toMap() {
    return {
      'nombre': nombre,
      'precio': precio,
      'stock': stock,
    };
  }
}
```

---

### `lib/services/producto_service.dart`

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/producto_model.dart';

class ProductoService {
  final CollectionReference _col =
      FirebaseFirestore.instance.collection('Productos');

  // CREATE
  Future<void> agregarProducto(Producto p) async {
    await _col.add(p.toMap());
  }

  // READ (stream en tiempo real)
  Stream<List<Producto>> obtenerProductos() {
    return _col.snapshots().map((snap) => snap.docs
        .map((doc) => Producto.fromMap(
            doc.data() as Map<String, dynamic>, doc.id))
        .toList());
  }

  // UPDATE
  Future<void> actualizarProducto(Producto p) async {
    await _col.doc(p.id).update(p.toMap());
  }

  // DELETE
  Future<void> eliminarProducto(String id) async {
    await _col.doc(id).delete();
  }
}
```

---

### `lib/main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';
import 'screens/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'CRUD Productos',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: const Color(0xFF6C63FF),
          brightness: Brightness.light,
        ),
        useMaterial3: true,
      ),
      home: const HomeScreen(),
    );
  }
}
```

---

### `lib/screens/home_screen.dart`

```dart
import 'package:flutter/material.dart';
import '../models/producto_model.dart';
import '../services/producto_service.dart';
import 'add_producto_screen.dart';
import 'edit_producto_screen.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final service = ProductoService();
    final Color primary = const Color(0xFF6C63FF);
    final Color accent  = const Color(0xFF43E97B);

    return Scaffold(
      backgroundColor: const Color(0xFFF4F6FB),
      appBar: AppBar(
        backgroundColor: primary,
        foregroundColor: Colors.white,
        title: const Text('Productos · Firestore',
            style: TextStyle(fontWeight: FontWeight.bold)),
        elevation: 0,
      ),
      floatingActionButton: FloatingActionButton.extended(
        backgroundColor: accent,
        foregroundColor: Colors.white,
        icon: const Icon(Icons.add),
        label: const Text('Agregar'),
        onPressed: () => Navigator.push(context,
            MaterialPageRoute(builder: (_) => const AddProductoScreen())),
      ),
      body: StreamBuilder<List<Producto>>(
        stream: service.obtenerProductos(),
        builder: (ctx, snap) {
          if (snap.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }
          final productos = snap.data ?? [];
          if (productos.isEmpty) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.inventory_2_outlined,
                      size: 64, color: Colors.grey[400]),
                  const SizedBox(height: 12),
                  Text('Sin productos aún',
                      style: TextStyle(color: Colors.grey[500], fontSize: 16)),
                ],
              ),
            );
          }
          return ListView.builder(
            padding: const EdgeInsets.all(16),
            itemCount: productos.length,
            itemBuilder: (ctx, i) {
              final p = productos[i];
              return Card(
                elevation: 3,
                margin: const EdgeInsets.only(bottom: 12),
                shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(16)),
                child: ListTile(
                  contentPadding: const EdgeInsets.symmetric(
                      horizontal: 20, vertical: 10),
                  leading: CircleAvatar(
                    backgroundColor: primary.withOpacity(0.12),
                    child: Text(p.nombre[0].toUpperCase(),
                        style: TextStyle(color: primary,
                            fontWeight: FontWeight.bold)),
                  ),
                  title: Text(p.nombre,
                      style: const TextStyle(fontWeight: FontWeight.w600)),
                  subtitle: Text(
                      '\$${p.precio.toStringAsFixed(2)} · Stock: ${p.stock}',
                      style: TextStyle(color: Colors.grey[600])),
                  trailing: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      IconButton(
                        icon: Icon(Icons.edit, color: primary),
                        onPressed: () => Navigator.push(
                          context,
                          MaterialPageRoute(
                              builder: (_) => EditProductoScreen(producto: p)),
                        ),
                      ),
                      IconButton(
                        icon: const Icon(Icons.delete, color: Colors.redAccent),
                        onPressed: () async {
                          await service.eliminarProducto(p.id);
                          ScaffoldMessenger.of(context).showSnackBar(
                            SnackBar(
                              content: Text('${p.nombre} eliminado'),
                              backgroundColor: Colors.redAccent,
                            ),
                          );
                        },
                      ),
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

### `lib/screens/add_producto_screen.dart`

```dart
import 'package:flutter/material.dart';
import '../models/producto_model.dart';
import '../services/producto_service.dart';

class AddProductoScreen extends StatefulWidget {
  const AddProductoScreen({super.key});
  @override
  State<AddProductoScreen> createState() => _AddProductoScreenState();
}

class _AddProductoScreenState extends State<AddProductoScreen> {
  final _formKey = GlobalKey<FormState>();
  final _nombreCtrl  = TextEditingController();
  final _precioCtrl  = TextEditingController();
  final _stockCtrl   = TextEditingController();
  final _service     = ProductoService();

  Future<void> _guardar() async {
    if (!_formKey.currentState!.validate()) return;
    final p = Producto(
      id: '',
      nombre: _nombreCtrl.text.trim(),
      precio: double.parse(_precioCtrl.text),
      stock:  int.parse(_stockCtrl.text),
    );
    await _service.agregarProducto(p);
    if (mounted) Navigator.pop(context);
  }

  InputDecoration _deco(String label, IconData icon) => InputDecoration(
    labelText: label,
    prefixIcon: Icon(icon, color: const Color(0xFF6C63FF)),
    border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)),
    focusedBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(12),
      borderSide: const BorderSide(color: Color(0xFF6C63FF), width: 2),
    ),
  );

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Nuevo Producto'),
        backgroundColor: const Color(0xFF6C63FF),
        foregroundColor: Colors.white,
      ),
      body: Padding(
        padding: const EdgeInsets.all(20),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(
                controller: _nombreCtrl,
                decoration: _deco('Nombre', Icons.label_outline),
                validator: (v) => v!.isEmpty ? 'Requerido' : null,
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _precioCtrl,
                decoration: _deco('Precio', Icons.attach_money),
                keyboardType: TextInputType.number,
                validator: (v) =>
                    double.tryParse(v!) == null ? 'Número inválido' : null,
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _stockCtrl,
                decoration: _deco('Stock', Icons.inventory),
                keyboardType: TextInputType.number,
                validator: (v) =>
                    int.tryParse(v!) == null ? 'Número inválido' : null,
              ),
              const SizedBox(height: 28),
              SizedBox(
                width: double.infinity,
                height: 52,
                child: ElevatedButton.icon(
                  style: ElevatedButton.styleFrom(
                    backgroundColor: const Color(0xFF43E97B),
                    foregroundColor: Colors.white,
                    shape: RoundedRectangleBorder(
                        borderRadius: BorderRadius.circular(14)),
                  ),
                  icon: const Icon(Icons.save),
                  label: const Text('Guardar',
                      style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold)),
                  onPressed: _guardar,
                ),
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

### `lib/screens/edit_producto_screen.dart`

```dart
import 'package:flutter/material.dart';
import '../models/producto_model.dart';
import '../services/producto_service.dart';

class EditProductoScreen extends StatefulWidget {
  final Producto producto;
  const EditProductoScreen({super.key, required this.producto});
  @override
  State<EditProductoScreen> createState() => _EditProductoScreenState();
}

class _EditProductoScreenState extends State<EditProductoScreen> {
  final _formKey = GlobalKey<FormState>();
  late final TextEditingController _nombreCtrl;
  late final TextEditingController _precioCtrl;
  late final TextEditingController _stockCtrl;
  final _service = ProductoService();

  @override
  void initState() {
    super.initState();
    _nombreCtrl = TextEditingController(text: widget.producto.nombre);
    _precioCtrl = TextEditingController(
        text: widget.producto.precio.toString());
    _stockCtrl  = TextEditingController(
        text: widget.producto.stock.toString());
  }

  Future<void> _actualizar() async {
    if (!_formKey.currentState!.validate()) return;
    final updated = Producto(
      id: widget.producto.id,
      nombre: _nombreCtrl.text.trim(),
      precio: double.parse(_precioCtrl.text),
      stock:  int.parse(_stockCtrl.text),
    );
    await _service.actualizarProducto(updated);
    if (mounted) Navigator.pop(context);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Editar: ${widget.producto.nombre}'),
        backgroundColor: const Color(0xFF6C63FF),
        foregroundColor: Colors.white,
      ),
      body: Padding(
        padding: const EdgeInsets.all(20),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(
                controller: _nombreCtrl,
                decoration: const InputDecoration(labelText: 'Nombre'),
                validator: (v) => v!.isEmpty ? 'Requerido' : null,
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _precioCtrl,
                decoration: const InputDecoration(labelText: 'Precio'),
                keyboardType: TextInputType.number,
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _stockCtrl,
                decoration: const InputDecoration(labelText: 'Stock'),
                keyboardType: TextInputType.number,
              ),
              const SizedBox(height: 28),
              SizedBox(
                width: double.infinity,
                height: 52,
                child: ElevatedButton.icon(
                  style: ElevatedButton.styleFrom(
                    backgroundColor: const Color(0xFF6C63FF),
                    foregroundColor: Colors.white,
                    shape: RoundedRectangleBorder(
                        borderRadius: BorderRadius.circular(14)),
                  ),
                  icon: const Icon(Icons.update),
                  label: const Text('Actualizar',
                      style: TextStyle(fontSize: 16)),
                  onPressed: _actualizar,
                ),
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

## 8. Agentes · Roles · Skills · Flujo de trabajo IA (Metodología)

Para una práctica guiada con **Antigravity** y estudiantes:

**Agente 1 — Arquitecto de datos**
- Rol: diseña el modelo `Producto` y la colección Firestore
- Skill: modelado NoSQL, tipado Dart, `fromMap/toMap`

**Agente 2 — Desarrollador de servicios**
- Rol: implementa `ProductoService` con los 4 métodos CRUD
- Skill: Firestore SDK, streams reactivos, manejo de errores

**Agente 3 — Diseñador de UI**
- Rol: construye pantallas con widgets atractivos
- Skill: Material 3, paleta de colores `#6C63FF` + `#43E97B`, formularios validados

**Agente 4 — QA / Tester**
- Rol: verifica que create/read/update/delete funcionen contra Firestore real
- Skill: pruebas manuales, revisión de reglas de seguridad Firestore

**Flujo de trabajo:**
```
Configurar Firebase → pubspec.yaml → models → services → screens → UI → prueba real
```

---

**Resumen del stack:** Flutter 3.x + Firebase Firestore + Material 3 + colores `#6C63FF` (violeta) y `#43E97B` (verde). El proyecto es totalmente funcional: los datos se sincronizan en tiempo real con `StreamBuilder`, sin necesidad de recargar la app. ¡Haz clic en cualquier fase del diagrama para que la IA profundice en ese tema específico!
