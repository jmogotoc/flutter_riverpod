# Riverpod

## Introducción a Riverpod

Riverpod es un sistema de gestión de estado moderno y flexible para Flutter que se enfoca en:

- Programación declarativo y reactiva.
- Flexibilidad y escalabilidad.
- Inyección de dependencias integrada.
- Extensibilidad.

Mediante programación declarativa y reactiva, Riverpod gestiona gran parte de la lógica de su aplicación e intenta resolver el problema de la gestión de estado en una aplicación, ofreciendo una forma única de escribir lógica de negocio, inspirada en los widgets de Flutter. 

Si bien para dominar Riverpod es importante tener claros varios conceptos fundamentales de Dart, en este artículo haremos un repaso de los aspectos clave a tener en cuenta y construiremos un ejemplo aplicando Riverpod en la clásica aplicación de contador que incluye por defecto el SDK de Flutter.

## Conceptos clave

### Programación Declarativa

La programación declarativa se enfoca en describir qué se quiere lograr, en lugar de cómo hacerlo paso a paso. En Flutter con Riverpod, esto significa declarar cómo debe lucir la interfaz según el estado de la app, y dejar que el sistema la actualice automáticamente. Este enfoque facilita un código más limpio, legible y fácil de mantener.

### Estado reactivo

El estado reactivo se refiere a la capacidad de un sistema para responder automáticamente a cambios en los datos. Aunque Riverpod no implementa estrictamente el patrón State, abstrae este concepto al permitir dependencias declarativas y reactivas entre estados. Esto significa que, cuando un estado cambia, los widgets o valores que dependen de él se actualizan automáticamente, manteniendo la interfaz sincronizada con la lógica de la aplicación de forma clara y eficiente.

### Providers en Riverpod

En Riverpod, los providers representan fuentes de estado o lógica de negocio, respetando el principio de responsabilidad única (SRP):

- Un **Provider** representa un valor inmutable.
- Un **StateProvider** representa un estado simple mutable.
- Un **StateNotifierProvider** representa un estado complejo/controlado.
- Un **FutureProvider** representa lógica asíncrona.
- Un **ChangeNotifierProvider** representa integración con clases existentes con ChangeNotifier.

## Widgets en Riverpod

**ProviderScope**:	Necesario para inicializar Riverpod en la aplicación.

**ConsumerWidget**: Escucha providers y reconstruye en cada cambio.

**ConsumerStatefulWidget**: Es un StatefulWidget que escucha providers y reconstruye en cada cambio.

**Consumer**: Permite consumir un provider dentro de cualquier widget sin convertir todo el widget.

## Ejemplificando la aplicación de riverpod en Flutter

Vamos a migrar el contador clásico de Flutter para usar Riverpod con principios de Clean Architecture.

1. La aplicación tendrá tres botones:
    - Incrementar el contador: actualizará el valor del contador en +1 inmediatamente.
    - Disminuir el contador: actualizará el valor del contador en -1 inmediatamente.
    - Resetear el contador: actualizará el valor del contador en 0 inmediatamente.

2. Para este ejemplo haremos uso del paquete de [`flutter_riverpod`](https://pub.dev/packages/flutter_riverpod) ya que contiene el núcleo de Riverpod, sumado a todas las clases y widgets que se utilizan para enlazar la lógica con la interfaz.

    ```yaml
    dependencies:
      flutter_riverpod: ^2.6.1
    ```

3. Tendremos la siguiente estructura en nuestro proyecto, imaginémonos que la aplicación estará construída bajo Clean Architecture y una organización layer-first, quedaría de la siguiente manera. Pueden hacer falta muchos más archivos, pero para ser un ejemplo base se manejará de esta manera, es solo para tener una aproximación general.

```bash
lib/
├── main.dart
├── presentation/
│   └── counter/
│       ├── pages/
│       │   └── counter_page.dart
│       ├── provider/
│       │   └── counter_provider.dart
```

La interacción general del provider de riverpod en la aplicación de contador la podemos observar en el siguiente diagrama, para entender su relación general.

![Diagram](readme_assets/diagram.png)

4. Uno de los conceptos clave en Riverpod es el `ProviderScope`. Este componente actúa como un "ámbito" o un espacio donde puedes definir y gestionar tus proveedores de estado. Los proveedores son contenedores de datos que almacenan estados compartidos en tu aplicación. Entonces vamos a definir nuestro `ProviderScope` en `main.dart`

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

5. Ya con eso podemos definir nuestro provider creando el archivo `counter_provider.dart`, donde definimos el estado del contador utilizando un `StateProvider`. En este caso, declaramos un `counterProvider` que inicializa el estado con un valor de 0. A través de este provider, podremos leer y modificar el valor del contador desde cualquier parte de la aplicación. 

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

final counterProvider = StateProvider<int>((_) => 0);
```

6. Para la construcción de nuestra pantalla usaremos `ConsumerStatefulWidget` que contiene la abstracción de un `StatefulWidget`que nos provee riverpod para facilitar la lectura de providers y adicional usaremos `ConsumerState` que sería la abstracción de riverpod para un `State`.

```dart
class CounterPage extends ConsumerStatefulWidget {
  const CounterPage({
    super.key,
  });

  @override
  ConsumerState<CounterPage> createState() => _CounterPageState();
}

class _CounterPageState extends ConsumerState<CounterPage> {
  @override
  Widget build(BuildContext context) {
    // ...
  }
}
```

7. Ya en nuestro `counter_page.dart` podemos hacer uso de nuestro provider registrado en el     `ProviderScope` mediante `ref` y la función `watch` para que nuestro widget esté escuchando los cambios en nuestro provider.

```dart
final int counter = ref.watch(counterProvider);
```

Dentro de nuestra vista será tan simple como utilizar nuestra variable `counter` la cual por medio de la función `watch` se encuentra escuchando los cambios de estado.

```dart
Column(
  mainAxisAlignment: MainAxisAlignment.center,
  children: <Widget>[
    const Text(
      'You have pushed the button this many times:',
    ),
    Text(
      '$counter',
      style: Theme.of(context).textTheme.headlineMedium,
    ),
  ],
),
```

Para los cambios de estado en nuestro `counterProvider` utilizamos la referencia `ref` y la función `read` para tener acceso al estado y realizar la modificación.

```dart
void _onIncrement() {
  ref.read(counterProvider.notifier).state++;
}
```

Y haciendo uso de la función `_onIncrement` podemos dispararla desde nuestro `CounterPage` por medio de un `FloatingActionButton`.

```dart
Scaffold(
  // ...
  floatingActionButton: : FloatingActionButton(
    onPressed: _onIncrement,
    child: const Icon(Icons.add),
  ),
)
```

Nuestro resultado final será así:

<img src="readme_assets/demo.gif" alt="drawing" width="200"/>