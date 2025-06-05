# Riverpod


## Introducción a Riverpod

Riverpod es un sistema de gestión de estado moderno y flexible para Flutter que se enfoca en:

- Enfoque Declarativo.
- Flexibilidad y Escalabilidad.
- Inyección de Dependencias Integrada.
- Extensibilidad.

<!-- abstrae el concepto de estado reactivo, permitiendo dependencias declarativas y reactivas entre estados -->

<!-- Riverpod intenta mantener la inmutabilidad, que garantiza que el estado no puede ser modificado directamente, lo que promueve un flujo de datos más predecible y seguro.

Otro valor clave es el desacoplamiento: Riverpod no depende del contexto de Flutter, lo que permite que el estado y la lógica de negocio se gestionen de forma independiente de la interfaz de usuario. Este diseño favorece también la testabilidad, ya que los providers se pueden simular o sustituir fácilmente en entornos de pruebas sin requerir el árbol de widgets.

haremos un resumen sobre alguno de los puntos a tener en cuenta y realizaremos un ejemplo con aplicabilidad con la aplicación contador que construye por defecto el SDK de Flutter. -->

## Conceptos clave

### Programación Declarativa

### Estado reactivo

### Providers en Riverpod

En Riverpod, los providers representan fuentes de estado o lógica de negocio, respetando el principio de responsabilidad única (SRP):

- Un **Provider** representa un valor inmutable.
- Un **StateProvider** representa un estado simple mutable.
- Un **StateNotifierProvider** representa un estado complejo/controlado.
- Un **FutureProvider** representa lógica asíncrona.
- Un **ChangeNotifierProvider** representa integración con clases existentes con ChangeNotifier.

## Widgets en Riverpod

**ProviderScope**	Necesario para inicializar Riverpod en la aplicación.

**ConsumerWidget**	Escucha providers y reconstruye en cada cambio.

**Consumer**	Permite consumir un provider dentro de cualquier widget sin convertir todo el widget.

## Ejemplificando la aplicación de riverpod en Flutter

Vamos a migrar el contador clásico de Flutter para usar Riverpod con principios de Clean Architecture.

Uno de los conceptos clave en Riverpod es el ProviderScope. Este componente actúa como un "ámbito" o un espacio donde puedes definir y gestionar tus proveedores de estado. Los proveedores son contenedores de datos que almacenan estados compartidos en tu aplicación. Para comenzar a utilizar Riverpod, debes envolver tu aplicación con un ProviderScope.

```dart
import 'package:flutter/material.dart';
import 'package:riverpod/riverpod.dart';

void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}
```

Paso 1: Definir el Provider

`features/counter/application/providers.dart`

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

/// Provider que representa el estado actual del contador.
final counterProvider = StateProvider<int>((ref) => 0);
````

Este provider:

Representa el estado del contador (SRP).

Es simple y adecuado para un valor entero.

Puede evolucionar a un StateNotifierProvider si la lógica crece (abierto a extensión, cerrado a modificación - OCP).

✅ Paso 2: Crear la pantalla

`features/counter/presentation/counter_page.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../application/providers.dart';

class CounterPage extends ConsumerWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final counter = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Contador')),
      body: Center(
        child: Text(
          '$counter',
          style: const TextStyle(fontSize: 40),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // Aumentar el contador
          ref.read(counterProvider.notifier).state++;
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}
````

✅ Paso 3: Integrar Riverpod en la app

`main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'features/counter/presentation/counter_page.dart';

void main() {
  runApp(const ProviderScope(child: MyApp()));
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Contador con Riverpod',
      home: const CounterPage(),
    );
  }
}
```