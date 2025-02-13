# Обзор Flutter

## Содержание

1. [Основная идея Flutter](#основная-идея-flutter)
2. [Минимальное Flutter-приложение](#минимальное-flutter-приложение)
3. [StatelessWidget и StatefulWidget](#statelesswidget-и-statefulwidget)
4. [Взаимодействие с данными и setState()](#взаимодействие-с-данными-и-setstate)
5. [Provider, Riverpod, BLoC и др.](#provider-riverpod-bloc-и-др)
6. [Навигация и переход между экранами](#навигация-и-переход-между-экранами)
7. [Полезные команды](#полезные-команды)
8. [Советы по поддержке](#советы-по-поддержке)

---

## Основная идея Flutter

- Кроссплатформенный фреймворк на языке **Dart**.
- Интерфейс рендерится движком Flutter напрямую (не через нативные элементы).
- Всё представлено в виде **виджетов**, аналог компонентов в React.
- Поддержка **iOS**, **Android**, **Web**, а также **Desktop** (Windows, macOS, Linux).

---

## Минимальное Flutter-приложение

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: Text('Пример Flutter'),
        ),
        body: Center(
          child: Text('Всё – это виджет!'),
        ),
      ),
    );
  }
}
```

1. `main()` — точка входа.
2. `runApp()` — запуск приложения.
3. `MaterialApp` — обёртка для темы, роутинга.
4. `Scaffold` — каркас экрана с AppBar и телом.

---

## StatelessWidget и StatefulWidget

### StatelessWidget

Не содержит внутреннего состояния, весь интерфейс зависит только от входных параметров.

```dart
import 'package:flutter/material.dart';

class HelloWidget extends StatelessWidget {
  final String name;

  const HelloWidget({Key? key, required this.name}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Text('Привет, $name!', style: TextStyle(fontSize: 20));
  }
}
```

### StatefulWidget

Обладает собственным состоянием, обновляется при вызове `setState()`.

```dart
import 'package:flutter/material.dart';

class CounterWidget extends StatefulWidget {
  const CounterWidget({Key? key}) : super(key: key);

  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _count = 0;

  void _increment() {
    setState(() {
      _count++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        Text('Счётчик: $_count', style: TextStyle(fontSize: 24)),
        ElevatedButton(
          onPressed: _increment,
          child: Text('Увеличить'),
        ),
      ],
    );
  }
}
```

---

## Взаимодействие с данными и setState()

Для простых случаев можно управлять состоянием напрямую через `setState()`:

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'State Management',
      home: Scaffold(
        appBar: AppBar(
          title: Text('setState() пример'),
        ),
        body: CounterWidget(),
      ),
    );
  }
}

class CounterWidget extends StatefulWidget {
  @override
  _CounterWidgetState createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _count = 0;

  void _incrementCounter() {
    setState(() {
      _count++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text('$_count', style: TextStyle(fontSize: 40)),
          ElevatedButton(
            onPressed: _incrementCounter,
            child: Text('Увеличить'),
          ),
        ],
      ),
    );
  }
}
```

---

## Provider, Riverpod, BLoC и др.

Взаимодействие с данными и state management

### Provider

Используем `ChangeNotifier` и `ChangeNotifierProvider`:

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class CounterModel extends ChangeNotifier {
  int _count = 0;
  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();
  }
}

void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => CounterModel(),
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('Provider пример')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Consumer<CounterModel>(
                builder: (_, model, __) {
                  return Text('${model.count}', style: TextStyle(fontSize: 36));
                },
              ),
              ElevatedButton(
                onPressed: () {
                  Provider.of<CounterModel>(context, listen: false).increment();
                },
                child: Text('Увеличить'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### Riverpod

Используем глобальные провайдеры и `ConsumerWidget`:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

final counterProvider = StateProvider<int>((ref) => 0);

void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('Riverpod пример')),
        body: Center(child: CounterWidget()),
      ),
    );
  }
}

class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider.state).state;
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        Text('$count', style: TextStyle(fontSize: 36)),
        ElevatedButton(
          onPressed: () => ref.read(counterProvider.state).state++,
          child: Text('Увеличить'),
        ),
      ],
    );
  }
}
```

### BLoC

Разделяем события (Event), состояния (State) и бизнес-логику (Bloc):

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

// События
abstract class CounterEvent {}
class Increment extends CounterEvent {}

// Состояния
class CounterState {
  final int value;
  CounterState(this.value);
}

// Логика
class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(CounterState(0)) {
    on<Increment>((event, emit) {
      emit(CounterState(state.value + 1));
    });
  }
}

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  final CounterBloc _counterBloc = CounterBloc();

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: BlocProvider(
        create: (_) => _counterBloc,
        child: Scaffold(
          appBar: AppBar(title: Text('BLoC пример')),
          body: BlocBuilder<CounterBloc, CounterState>(
            builder: (context, state) {
              return Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Text('${state.value}', style: TextStyle(fontSize: 36)),
                    ElevatedButton(
                      onPressed: () => context.read<CounterBloc>().add(Increment()),
                      child: Text('Увеличить'),
                    ),
                  ],
                ),
              );
            },
          ),
        ),
      ),
    );
  }
}
```

---

## Навигация и переход между экранами

### 1. push/pop через `MaterialPageRoute`

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: FirstScreen(),
    );
  }
}

class FirstScreen extends StatelessWidget {
  const FirstScreen({Key? key}) : super(key: key);

  void _goToSecondScreen(BuildContext context) {
    Navigator.push(
      context,
      MaterialPageRoute(builder: (context) => const SecondScreen()),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Первый экран')),
      body: Center(
        child: ElevatedButton(
          onPressed: () => _goToSecondScreen(context),
          child: const Text('Перейти на второй экран'),
        ),
      ),
    );
  }
}

class SecondScreen extends StatelessWidget {
  const SecondScreen({Key? key}) : super(key: key);

  void _back(BuildContext context) {
    Navigator.pop(context);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Второй экран')),
      body: Center(
        child: ElevatedButton(
          onPressed: () => _back(context),
          child: const Text('Назад'),
        ),
      ),
    );
  }
}
```

### 2. Именованные маршруты (`pushNamed`/`pop`)

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(
    title: 'Навигация с именованными маршрутами',
    initialRoute: '/',
    routes: {
      '/': (context) => const FirstScreen(),
      '/second': (context) => const SecondScreen(),
    },
  ));
}

class FirstScreen extends StatelessWidget {
  const FirstScreen({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Первый экран')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            Navigator.pushNamed(context, '/second');
          },
          child: const Text('Перейти на второй экран'),
        ),
      ),
    );
  }
}

class SecondScreen extends StatelessWidget {
  const SecondScreen({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Второй экран')),
      body: Center(
        child: ElevatedButton(
          onPressed: () => Navigator.pop(context),
          child: const Text('Назад'),
        ),
      ),
    );
  }
}
```

### 3. Navigator 2.0 / Router

Более гибкая модель, используем декларативный подход (особенно для Web-приложений):

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatefulWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  String _path = '/';

  void _setPath(String newPath) {
    setState(() {
      _path = newPath;
    });
  }

  List<Page> _buildPages() {
    List<Page> pages = [
      MaterialPage(
        child: FirstScreen(onNavigate: () => _setPath('/second')),
      ),
    ];
    if (_path == '/second') {
      pages.add(MaterialPage(child: SecondScreen(onBack: () => _setPath('/'))));
    }
    return pages;
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Navigator(
        pages: _buildPages(),
        onPopPage: (route, result) {
          if (!route.didPop(result)) return false;
          _setPath('/');
          return true;
        },
      ),
    );
  }
}

class FirstScreen extends StatelessWidget {
  final VoidCallback onNavigate;
  const FirstScreen({Key? key, required this.onNavigate}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Первый экран')),
      body: Center(
        child: ElevatedButton(
          onPressed: onNavigate,
          child: const Text('Перейти на второй экран'),
        ),
      ),
    );
  }
}

class SecondScreen extends StatelessWidget {
  final VoidCallback onBack;
  const SecondScreen({Key? key, required this.onBack}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Второй экран')),
      body: Center(
        child: ElevatedButton(
          onPressed: onBack,
          child: const Text('Назад'),
        ),
      ),
    );
  }
}
```

---

## Полезные команды

```bash
# Проверка окружения (SDK, эмуляторы, IDE, плагины)
flutter doctor

# Установка/обновление зависимостей
flutter pub get

# Запуск приложения (c hot reload)
flutter run

# Сборка apk (Android) в релизном режиме
flutter build apk

# Сборка iOS в релизном режиме (только на macOS)
flutter build ios

# Запуск тестов
flutter test

# Просмотр устаревших версий пакетов
flutter pub outdated

# Обновление зависимостей до последних версий
flutter pub upgrade
```

---

## Советы по поддержке

1. **Структура проекта**

   - Разделение кода в папке `lib/` (экраны, модели, сервисы).
   - Проверяйте `pubspec.yaml` для понимания используемых пакетов.

2. **Совместимость SDK и зависимостей**

   - `flutter doctor` и `pubspec.lock` помогут выяснить версии.

3. **State Management**

   - Определите, каким способом управляется состояние (Provider/BLoC и т.д.).

4. **Тесты**

   - Используйте `flutter test` и обратите внимание на тестовые файлы в `test/`.

5. **Обновление пакетов**

   - Не забывайте регулярно проверять `flutter pub outdated`.

6. **Анализ кода**

   - `dart analyze` или встроенные инструменты в IDE помогут в рефакторинге.
