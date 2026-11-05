# FLUTTER DEVELOPMENT TUTORIAL
## A Beginner-Friendly Guide to Building Mobile Applications

---

# 1. What is Flutter?

Flutter is an open-source UI framework developed by Google.

It allows developers to build applications using a single codebase.

You can use Flutter to create:

- Android applications
- iOS applications
- Web applications
- Desktop applications (Windows, macOS, Linux)

### What programming language does Flutter use?

Flutter uses **Dart**.

Dart is a programming language with syntax similar to Java, C#, and JavaScript.

### How Flutter Works

    Developer writes Dart code
              |
              v
         Flutter Framework
              |
              v
         Flutter Engine
              |
              v
        Running Application

---

# 2. Installing Flutter

Before creating an application, you need to install the required tools.

### Required Tools

1. Flutter SDK - Provides the Flutter development framework.
2. Dart SDK - Programming language used by Flutter (included with Flutter).
3. Android Studio - Provides Android development tools and emulator.
4. Visual Studio Code - Code editor for writing Flutter applications.

### Check Flutter Installation

Open your terminal or command prompt.

Type:

```bash
flutter doctor
```

This command checks whether your development environment is properly configured.

### Create Your First Flutter Project

```bash
flutter create my_first_app
```

Navigate to your project:

```bash
cd my_first_app
```

Run your application:

```bash
flutter run
```

---

# 3. Understanding Flutter Project Structure

When you create a Flutter project, you will see several folders and files.

```text
my_first_app/
|
|-- android/
|-- ios/
|-- lib/
|   |
|   |-- main.dart
|
|-- test/
|-- pubspec.yaml
```

### Important Files and Folders

| File / Folder | Purpose |
|---|---|
| android/ | Android-specific files |
| ios/ | iOS-specific files |
| lib/ | Main application source code |
| main.dart | Starting point of the application |
| test/ | Application testing |
| pubspec.yaml | Dependencies, assets, and project configuration |

**Important:** Most of your Flutter development will happen inside the `lib/` folder.

---

# 4. Understanding Dart Basics

Before learning Flutter widgets, understand some basic Dart concepts.

## 4.1 Variables

Variables store information.

```dart
void main() {
  String name = "Wakai";
  int age = 21;
  double height = 1.75;
  bool isStudent = true;

  print(name);
  print(age);
  print(height);
  print(isStudent);
}
```

### Common Data Types

| Data Type | Description | Example |
|---|---|---|
| String | Text | "Hello" |
| int | Whole numbers | 100 |
| double | Decimal numbers | 10.5 |
| bool | True or false | true |

## 4.2 Functions

Functions contain reusable instructions.

```dart
void sayHello() {
  print("Hello, Flutter!");
}

void main() {
  sayHello();
}
```

A function can also return a value.

```dart
int add(int a, int b) {
  return a + b;
}

void main() {
  print(add(10, 5));
}
```

Output:

```text
15
```

## 4.3 Conditions

Conditions allow your application to make decisions.

```dart
void main() {
  int age = 20;

  if (age >= 18) {
    print("You are an adult.");
  } else {
    print("You are a minor.");
  }
}
```

## 4.4 Loops

Loops repeat instructions.

```dart
void main() {
  for (int i = 1; i <= 5; i++) {
    print(i);
  }
}
```

Output:

```text
1
2
3
4
5
```

---

# 5. Understanding Flutter Widgets

Widgets are the building blocks of Flutter applications.

Almost everything you see on the screen is a widget.

Examples:

- Text
- Buttons
- Images
- Containers
- Rows
- Columns
- Text fields

### Common Flutter Widgets

| Widget | Purpose |
|---|---|
| MaterialApp | Main application configuration |
| Scaffold | Basic screen layout |
| AppBar | Top application bar |
| Text | Displays text |
| Container | Creates a customizable box |
| Row | Arranges widgets horizontally |
| Column | Arranges widgets vertically |
| Image | Displays images |
| Icon | Displays icons |
| ElevatedButton | Creates a button |
| TextField | Accepts user input |

---

# 6. Your First Flutter Application

Open `lib/main.dart`.

Replace its contents with:

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(
          title: const Text("My First App"),
        ),

        body: const Center(
          child: Text(
            "Hello, Flutter!",
            style: TextStyle(
              fontSize: 25,
              fontWeight: FontWeight.bold,
            ),
          ),
        ),
      ),
    );
  }
}
```

### Understanding the Code

**1. Import Flutter**

```dart
import 'package:flutter/material.dart';
```

Imports Flutter's Material Design widgets.

**2. Main Function**

```dart
void main() {
  runApp(const MyApp());
}
```

The main function starts your application.

**3. MaterialApp**

```dart
MaterialApp(
  home: Scaffold(),
)
```

MaterialApp provides the main application structure and configuration.

**4. Scaffold**

```dart
Scaffold(
  appBar: AppBar(),
  body: Center(),
)
```

Scaffold provides a basic screen layout.

**5. Text**

```dart
Text("Hello, Flutter!")
```

Displays text on the screen.

---

# 7. Understanding Layout Widgets

Flutter uses layout widgets to organize the appearance of an application.

## 7.1 Container

Container is used to create a box with customizable properties.

```dart
Container(
  width: 200,
  height: 100,
  color: Colors.blue,
  child: const Center(
    child: Text(
      "Hello!",
      style: TextStyle(
        color: Colors.white,
      ),
    ),
  ),
)
```

Common Container properties:

- width
- height
- color
- padding
- margin
- alignment
- decoration

## 7.2 Column

Column arranges widgets vertically.

```dart
Column(
  children: const [
    Text("First"),
    Text("Second"),
    Text("Third"),
  ],
)
```

Layout:

```text
First
  |
Second
  |
Third
```

## 7.3 Row

Row arranges widgets horizontally.

```dart
Row(
  children: const [
    Text("Left"),
    Text("Center"),
    Text("Right"),
  ],
)
```

Layout:

```text
Left | Center | Right
```

## 7.4 Padding

Padding adds space inside a widget's surrounding area.

```dart
Padding(
  padding: const EdgeInsets.all(16),
  child: const Text("Hello!"),
)
```

## 7.5 Margin

Margin adds space outside a widget.

```dart
Container(
  margin: const EdgeInsets.all(20),
  child: const Text("Hello!"),
)
```

---

# 8. Buttons and User Interaction

Buttons allow users to interact with your application.

### ElevatedButton

```dart
ElevatedButton(
  onPressed: () {
    print("Button clicked!");
  },
  child: const Text("Click Me"),
)
```

### IconButton

```dart
IconButton(
  onPressed: () {
    print("Favorite clicked!");
  },
  icon: const Icon(Icons.favorite),
)
```

### Understanding onPressed

`onPressed` is a callback function.

It tells Flutter what to do when the user clicks a button.

---

# 9. Understanding State Management

State refers to information that can change while your application is running.

Example:

- Counter value
- Login status
- Selected item
- Shopping cart quantity
- Form information

Flutter has two common widget types:

| Widget | Description |
|---|---|
| StatelessWidget | UI does not manage changing internal state |
| StatefulWidget | UI can update when its state changes |

## 9.1 StatelessWidget

Use StatelessWidget when the widget does not need to manage changing internal state.

```dart
class WelcomeMessage extends StatelessWidget {
  const WelcomeMessage({super.key});

  @override
  Widget build(BuildContext context) {
    return const Text("Welcome!");
  }
}
```

## 9.2 StatefulWidget

Use StatefulWidget when the interface needs to change.

### Example: Counter Application

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(
    home: CounterPage(),
  ));
}

class CounterPage extends StatefulWidget {
  const CounterPage({super.key});

  @override
  State<CounterPage> createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {

  int counter = 0;

  void increaseCounter() {
    setState(() {
      counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("Counter App"),
      ),

      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              "Counter: $counter",
              style: const TextStyle(fontSize: 25),
            ),

            ElevatedButton(
              onPressed: increaseCounter,
              child: const Text("Increase"),
            ),
          ],
        ),
      ),
    );
  }
}
```

### What is setState()?

`setState()` tells Flutter that the state has changed and the affected UI needs to be rebuilt.

Example:

```dart
setState(() {
  counter++;
});
```

Without setState, changing the counter variable alone does not automatically update the screen.

---

# 10. TextField and User Input

TextField allows users to enter information.

Example:

```dart
TextField(
  decoration: const InputDecoration(
    labelText: "Enter your name",
    border: OutlineInputBorder(),
  ),
)
```

### Getting User Input

Use TextEditingController to access the text entered by the user.

```dart
final TextEditingController nameController =
    TextEditingController();
```

Complete example:

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(
    home: InputPage(),
  ));
}

class InputPage extends StatefulWidget {
  const InputPage({super.key});

  @override
  State<InputPage> createState() => _InputPageState();
}

class _InputPageState extends State<InputPage> {

  final TextEditingController nameController =
      TextEditingController();

  String name = "";

  @override
  void dispose() {
    nameController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("User Input"),
      ),

      body: Padding(
        padding: const EdgeInsets.all(20),

        child: Column(
          children: [
            TextField(
              controller: nameController,
              decoration: const InputDecoration(
                labelText: "Your Name",
                border: OutlineInputBorder(),
              ),
            ),

            ElevatedButton(
              onPressed: () {
                setState(() {
                  name = nameController.text;
                });
              },
              child: const Text("Submit"),
            ),

            Text("Hello, $name"),
          ],
        ),
      ),
    );
  }
}
```

---

# 11. Navigation Between Screens

Navigation allows users to move between different pages.

For example:

```text
Home Page
    |
    v
Profile Page
    |
    v
Settings Page
```

### Create Another Screen

```dart
class ProfilePage extends StatelessWidget {
  const ProfilePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("Profile"),
      ),

      body: const Center(
        child: Text("Welcome to Profile!"),
      ),
    );
  }
}
```

### Navigate to Another Screen

```dart
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => const ProfilePage(),
  ),
);
```

### Return to Previous Screen

```dart
Navigator.pop(context);
```

---

# 12. Working with Images

Flutter supports displaying images from local assets and the internet.

## 12.1 Local Images

First, create an assets folder:

```text
my_first_app/
|
|-- assets/
|   |-- images/
|       |-- logo.png
|
|-- lib/
```

Register the assets in `pubspec.yaml`:

```yaml
flutter:
  assets:
    - assets/images/
```

Display the image:

```dart
Image.asset(
  'assets/images/logo.png',
  width: 150,
  height: 150,
)
```

## 12.2 Internet Images

```dart
Image.network(
  'https://example.com/image.png',
  width: 200,
  height: 200,
)
```

---

# 13. Lists and ListView

ListView displays multiple items in a scrollable layout.

Example:

```dart
ListView(
  children: const [
    ListTile(
      title: Text("Apple"),
    ),

    ListTile(
      title: Text("Banana"),
    ),

    ListTile(
      title: Text("Orange"),
    ),
  ],
)
```

### ListView.builder

Use ListView.builder when displaying a dynamic or larger collection of items.

```dart
final List<String> fruits = [
  "Apple",
  "Banana",
  "Orange",
  "Mango",
];

ListView.builder(
  itemCount: fruits.length,

  itemBuilder: (context, index) {
    return ListTile(
      title: Text(fruits[index]),
    );
  },
)
```

---

# 14. Connecting Flutter to a Backend API

Most real-world applications need a backend.

For example:

```text
Flutter Application
        |
        | HTTP Request
        v
    Backend API
        |
        v
      Database
```

A backend can be developed using:

- Node.js + Express
- ASP.NET Core Web API
- Java Spring Boot
- Python FastAPI

### Install HTTP Package

Run:

```bash
flutter pub add http
```

### Example GET Request

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

Future<void> getUsers() async {

  final response = await http.get(
    Uri.parse('https://jsonplaceholder.typicode.com/users'),
  );

  if (response.statusCode == 200) {

    final data = jsonDecode(response.body);

    print(data);

  } else {

    print("Failed to load users");

  }
}
```

### Important HTTP Methods

| Method | Purpose |
|---|---|
| GET | Retrieve information |
| POST | Create new information |
| PUT | Update existing information |
| PATCH | Partially update information |
| DELETE | Remove information |

---

# 15. Database and Local Storage

Applications often need to save information.

There are two common approaches:

### Local Storage

Stores data directly on the device.

Examples:

- SharedPreferences
- SQLite
- Hive

### Cloud Database

Stores information on an external server.

Examples:

- Firebase Firestore
- Supabase
- MySQL through a backend API
- MongoDB through a backend API

### Basic Architecture

```text
Flutter Mobile App
        |
        v
    Backend API
        |
        v
      Database
```

Remember: Flutter should generally communicate with remote databases through a secure backend API rather than connecting directly to a database using exposed credentials.

---

# 16. Recommended Flutter Folder Structure

As your project grows, organize your code into separate folders.

Example:

```text
lib/
|
|-- main.dart
|
|-- models/
|   |-- user.dart
|
|-- screens/
|   |-- home_screen.dart
|   |-- login_screen.dart
|   |-- profile_screen.dart
|
|-- widgets/
|   |-- custom_button.dart
|   |-- custom_textfield.dart
|
|-- services/
|   |-- api_service.dart
|
|-- providers/
|   |-- user_provider.dart
|
|-- utils/
    |-- constants.dart
```

### Folder Responsibilities

- models/ - Represents application data.
- screens/ - Contains application pages.
- widgets/ - Contains reusable UI components.
- services/ - Handles API requests and external services.
- providers/ - Handles shared application state when using Provider.
- utils/ - Contains helper functions and constants.

---

# 17. Common Flutter Commands

| Command | Description |
|---|---|
| flutter doctor | Checks Flutter installation |
| flutter create app_name | Creates a new project |
| flutter run | Runs the application |
| flutter devices | Lists connected devices |
| flutter pub get | Downloads dependencies |
| flutter pub add package_name | Adds a package |
| flutter clean | Clears generated build files |
| flutter build apk | Builds an Android APK |
| flutter build appbundle | Builds an Android App Bundle |

---

# 18. Beginner Practice Projects

The best way to learn Flutter is by building small applications.

### Level 1 - Basic UI

1. Hello World App
2. Personal Profile App
3. Simple Calculator
4. Digital Counter
5. Login Screen Design

### Level 2 - User Interaction

1. To-Do List
2. Notes Application
3. Student Information Form
4. Simple Expense Tracker
5. Shopping Cart UI

### Level 3 - Database and Backend

1. Login and Registration System
2. Student Management System
3. Inventory Management System
4. CRUD Application
5. Online Ordering Application

---

# 19. Recommended Learning Path

Follow this order to avoid learning too many concepts at once.

```text
1. Dart Fundamentals
        |
        v
2. Flutter Installation
        |
        v
3. Flutter Project Structure
        |
        v
4. Widgets and Layouts
        |
        v
5. Buttons and User Input
        |
        v
6. StatelessWidget and StatefulWidget
        |
        v
7. Navigation
        |
        v
8. Lists and Images
        |
        v
9. State Management
        |
        v
10. API Integration
        |
        v
11. Database Integration
        |
        v
12. Build and Deploy
```

---

# 20. Important Things to Remember

- Flutter is a framework.
- Dart is the programming language.
- Widgets are the building blocks of Flutter.
- MaterialApp configures the application.
- Scaffold provides the basic screen structure.
- Row arranges widgets horizontally.
- Column arranges widgets vertically.
- StatelessWidget is for UI without internally managed changing state.
- StatefulWidget manages changing state.
- setState() triggers a UI rebuild.
- Navigator handles screen navigation.
- TextField accepts user input.
- ListView displays scrollable collections.
- HTTP requests allow Flutter to communicate with backend APIs.

## Final Advice

Do not memorize every Flutter widget.

Instead:

1. Understand what each widget does.
2. Practice writing small code examples.
3. Build simple applications.
4. Learn how widgets communicate.
5. Practice connecting your application to an API.
6. Gradually build a complete mobile application.

**The goal is not just to learn Flutter syntax. The goal is to understand how to build a working application from beginning to end.**