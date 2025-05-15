# 🧱 Inventario Ferretería con Flutter + Firebase

Este proyecto es una guía paso a paso para crear una aplicación móvil con Flutter que se conecta con Firebase para manejar una **base de datos de piezas de una ferretería**.

Aprenderás cómo:

- Crear un proyecto Flutter vacío en VSCode (macOS)
- Conectarlo con Firebase
- Crear y leer datos desde Firestore
- Mostrar un inventario simple en pantalla

---

## 🛠️ Crear un nuevo proyecto Flutter vacío en VSCode (macOS)

1. Abre **Visual Studio Code**.
2. Presiona `Cmd + Shift + P` para abrir la **paleta de comandos**.
3. Escribe y selecciona: `Flutter: New Project (Empty)`.
4. Escribe el nombre del proyecto, por ejemplo: `inventario_ferreteria`.
5. Selecciona la carpeta donde se guardará el proyecto.
6. Espera unos segundos mientras se genera la estructura básica del proyecto.

La estructura será algo así:

```
inventario_ferreteria/
├── lib/
│   └── main.dart
├── pubspec.yaml
└── README.md
```

---

## 🔥 Configurar Firebase para el proyecto

Vamos a conectar nuestro proyecto Flutter con Firebase para manejar el inventario en la nube usando Firestore.

### 1. Crear un proyecto en Firebase

1. Entra a [Firebase Console](https://console.firebase.google.com/).
2. Haz clic en **Agregar proyecto**.
3. Nombra tu proyecto, por ejemplo: `inventario-ferreteria`.
4. Sigue los pasos hasta finalizar.

---

## 🔥 Conectar el proyecto con Firebase

La forma más sencilla de empezar es usando **FlutterFire CLI**.

### 🛑 Antes de continuar, asegúrate de tener lo siguiente listo:

✅ Firebase CLI instalada y haber iniciado sesión  
→ Ejecuta: `firebase login`

✅ Flutter SDK instalado correctamente  
→ Verifica con: `flutter doctor`

✅ Un proyecto Flutter creado  

> ⚠️ Si alguno de estos pasos no está completo, detente aquí y complétalo antes de continuar. No funcionará correctamente si falta algo.

---

### ⚙️ Instalar FlutterFire CLI

Desde cualquier directorio, ejecuta:

```bash
dart pub global activate flutterfire_cli
```

Luego, en la raíz de tu proyecto Flutter, ejecuta el siguiente comando:

```bash
flutterfire configure --project=inventario-ferreteria
```

Esto generará el archivo `firebase_options.dart` dentro de la carpeta `lib/`.

---

## 🚀 Inicializar Firebase en tu proyecto Flutter

Modifica el archivo `lib/main.dart` así:

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'firebase_options.dart';

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
      title: 'Inventario Ferretería',
      home: const InventarioScreen(),
    );
  }
}

class InventarioScreen extends StatelessWidget {
  const InventarioScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Inventario')),
      body: StreamBuilder<QuerySnapshot>(
        stream: FirebaseFirestore.instance.collection('piezas').snapshots(),
        builder: (context, snapshot) {
          if (snapshot.hasError) {
            return const Center(child: Text('Error al cargar los datos'));
          }
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }

          final docs = snapshot.data!.docs;

          if (docs.isEmpty) {
            return const Center(child: Text('No hay piezas registradas'));
          }

          return ListView.builder(
            itemCount: docs.length,
            itemBuilder: (context, index) {
              final pieza = docs[index].data() as Map<String, dynamic>;
              return ListTile(
                title: Text(pieza['nombre']),
                subtitle: Text('Precio: \$${pieza['precio']} | Stock: ${pieza['stock']}'),
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

## 📦 Agregar los plugins necesarios de Firebase

Edita tu archivo `pubspec.yaml` e incluye:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.25.4
  cloud_firestore: ^4.13.3
```

Luego ejecuta:

```bash
flutter pub get
```

---

## ✅ Probar conexión con Firestore

Para comprobar que todo está funcionando correctamente, puedes añadir un documento manualmente desde la consola web de Firebase:

### 🧪 Añadir datos manualmente desde Firebase Console

1. Abre [Firebase Console](https://console.firebase.google.com/) y entra a tu proyecto (`inventario-ferreteria`).
2. En el menú lateral, haz clic en **Firestore Database**.
3. Si es la primera vez que lo usas, haz clic en **Crear base de datos** y selecciona el modo de prueba.
4. Haz clic en **Iniciar colección**.
   - **ID de la colección**: `piezas`
   - Presiona "Siguiente"
5. Agrega un documento con los siguientes campos:
   - `nombre` → `Tornillo 1/4` (tipo: string)
   - `precio` → `1.50` (tipo: number)
   - `stock` → `100` (tipo: number)
6. Haz clic en **Guardar**.

Una vez hecho esto, tu colección estará lista para ser leída desde tu app Flutter.

---


---

## 🍏 Configurar iOS correctamente (solo si estás en Mac)

Para evitar errores al compilar en iOS (por ejemplo, si usas un simulador o dispositivo real), asegúrate de modificar el archivo `ios/Podfile`.

### 🛠️ Pasos:

1. Abre el archivo `ios/Podfile` en tu proyecto.
2. Busca esta línea (normalmente viene comentada):

```ruby
# platform :ios, '9.0'
```

3. Reemplázala por:

```ruby
platform :ios, '13.0'
```

4. Guarda el archivo y ejecuta:

```bash
cd ios
pod install
cd ..
```

> ⚠️ Si no haces esto, puedes tener errores al construir la app en Xcode o al correrla en simuladores recientes.



---

## 🔐 Configurar reglas de seguridad en Firestore (modo prueba)

Para que puedas leer y escribir documentos desde tu app durante el desarrollo, necesitas cambiar temporalmente las reglas de Firestore.

### 🛠️ Pasos:

1. Ve a [Firebase Console](https://console.firebase.google.com/) y entra a tu proyecto.
2. En el menú lateral, haz clic en **Firestore Database**.
3. Ve a la pestaña **Reglas**.
4. Reemplaza el contenido con lo siguiente:

```plaintext
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true;
    }
  }
}
```

5. Haz clic en **Publicar**.

> ⚠️ Esta configuración es solo para desarrollo. No la uses en producción porque permite acceso total sin autenticación.


## 🎉 ¡Listo!

Tu proyecto Flutter ya está conectado con Firebase y listo para empezar a construir el sistema de inventario de tu ferretería. Puedes expandirlo agregando funcionalidades como edición de productos, eliminación, búsqueda, y más.

---

## 👤 Autor

- [@andrettiprz](https://www.github.com/andrettiprz)
