---
title: Cross-Platform Development
date: 2026-01-31
tags:
  - react-native
  - flutter
  - cross-platform
  - interview
---

# Cross-Platform Development

## React Native

### Core Concepts

```jsx
import React, { useState } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';

const Counter = () => {
  const [count, setCount] = useState(0);
  
  return (
    <View style={styles.container}>
      <Text style={styles.text}>Count: {count}</Text>
      <Button title="Increment" onPress={() => setCount(count + 1)} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  text: { fontSize: 24, marginBottom: 20 }
});
```

### Navigation

```jsx
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';

const Stack = createStackNavigator();

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Details" component={DetailsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

// Navigate
navigation.navigate('Details', { itemId: 42 });
```

### State Management

```jsx
// Redux Toolkit
import { createSlice, configureStore } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: state => { state.value += 1 },
    decrement: state => { state.value -= 1 }
  }
});

const store = configureStore({ reducer: { counter: counterSlice.reducer } });
```

### Bridge Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   JavaScript    │────▶│     Bridge      │────▶│     Native      │
│    (React)      │◀────│   (JSON msgs)   │◀────│  (iOS/Android)  │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

## Flutter

### Core Concepts

```dart
import 'package:flutter/material.dart';

class Counter extends StatefulWidget {
  @override
  _CounterState createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _count = 0;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Count: $_count', style: TextStyle(fontSize: 24)),
            ElevatedButton(
              onPressed: () => setState(() => _count++),
              child: Text('Increment'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Widget Types

| StatelessWidget | StatefulWidget |
|-----------------|----------------|
| Immutable | Mutable state |
| No `setState()` | Uses `setState()` |
| Rebuilt by parent | Can rebuild itself |
| Example: Text, Icon | Example: Checkbox, TextField |

### Navigation

```dart
// Push
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => DetailScreen(id: 42)),
);

// Named routes
Navigator.pushNamed(context, '/details', arguments: {'id': 42});

// Pop
Navigator.pop(context);
```

### State Management (Provider)

```dart
// Provider
class Counter with ChangeNotifier {
  int _count = 0;
  int get count => _count;
  
  void increment() {
    _count++;
    notifyListeners();
  }
}

// Usage
ChangeNotifierProvider(
  create: (_) => Counter(),
  child: MyApp(),
)

// Consume
Consumer<Counter>(
  builder: (context, counter, child) => Text('${counter.count}'),
)
```

### Flutter Architecture

```
┌─────────────────────────────────────────┐
│              Dart Framework             │
│  (Widgets, Material, Cupertino, etc.)   │
├─────────────────────────────────────────┤
│              Flutter Engine             │
│    (Skia, Dart Runtime, Platform)       │
├─────────────────────────────────────────┤
│           Platform Specific             │
│         (iOS / Android / Web)           │
└─────────────────────────────────────────┘
```

## Comparison

| Aspect | React Native | Flutter |
|--------|--------------|---------|
| Language | JavaScript/TypeScript | Dart |
| UI | Native components | Custom rendering (Skia) |
| Performance | Good (bridge overhead) | Excellent (compiled) |
| Hot Reload | Yes | Yes |
| Learning Curve | Easy (if know React) | Moderate |
| Community | Large | Growing fast |

## When to Use Cross-Platform

> [!tip] Use Cross-Platform When
> - Rapid prototyping needed
> - Budget constraints
> - Simple to medium complexity apps
> - Consistent UI across platforms desired

> [!warning] Use Native When
> - Heavy platform-specific features
> - Maximum performance required
> - Complex animations/graphics
> - Deep OS integration needed

## Questions & Answers

> [!question]- Q1: What is the React Native Bridge?
> **Answer:** 
> The bridge is the communication layer between JavaScript and native code. It serializes data to JSON and passes messages asynchronously between the two realms.
> 
> Limitation: Can become a bottleneck for heavy data transfer or frequent updates.

> [!question]- Q2: How does Flutter achieve high performance?
> **Answer:** 
> Flutter compiles to native ARM code and uses Skia graphics engine to render UI directly, bypassing platform UI components.
> 
> Key factors:
> - No bridge overhead
> - Direct rendering to canvas
> - Ahead-of-time (AOT) compilation

> [!question]- Q3: What is the difference between StatelessWidget and StatefulWidget?
> **Answer:** 
> - **StatelessWidget**: Immutable, no internal state, rebuilt only when parent rebuilds
> - **StatefulWidget**: Has mutable state, can trigger rebuilds via `setState()`
> 
> Use StatelessWidget when UI doesn't change based on user interaction.

> [!question]- Q4: How do you handle navigation in React Native?
> **Answer:** 
> Use React Navigation library:
> - Stack Navigator for screen stacks
> - Tab Navigator for bottom tabs
> - Drawer Navigator for side menus
> 
> Pass params via `navigation.navigate('Screen', { params })`.

> [!question]- Q5: What is Provider in Flutter?
> **Answer:** 
> Provider is a state management solution that uses InheritedWidget under the hood.
> 
> Types:
> - `Provider` - Basic value provider
> - `ChangeNotifierProvider` - For objects that notify listeners
> - `Consumer` - Rebuilds when provider changes
> - `context.watch()` / `context.read()` - Access provider values

> [!question]- Q6: When would you choose native over cross-platform?
> **Answer:** 
> Choose native when:
> - App requires heavy platform-specific features
> - Maximum performance is critical
> - Complex animations or graphics
> - Deep hardware integration (AR, sensors)
> - Long-term maintainability is priority

> [!question]- Q7: What is Hot Reload and how does it work?
> **Answer:** 
> Hot Reload injects updated code into running app without losing state.
> 
> - React Native: Uses Metro bundler to push changes
> - Flutter: Injects updated Dart code into Dart VM
> 
> Speeds up development by avoiding full rebuilds.

> [!question]- Q8: How do you handle platform-specific code in React Native?
> **Answer:** 
> Options:
> 1. `Platform.OS` check: `Platform.OS === 'ios'`
> 2. Platform-specific files: `Component.ios.js`, `Component.android.js`
> 3. `Platform.select()` for style differences

> [!question]- Q9: What is the widget tree in Flutter?
> **Answer:** 
> Flutter UI is built as a tree of widgets. Each widget describes part of the UI.
> 
> Three trees:
> - Widget tree (configuration)
> - Element tree (lifecycle)
> - RenderObject tree (layout/painting)
> 
> Flutter efficiently updates only changed parts.

> [!question]- Q10: How do you optimize React Native performance?
> **Answer:** 
> Techniques:
> - Use `FlatList` instead of `ScrollView` for lists
> - Memoize components with `React.memo`
> - Avoid inline functions in render
> - Use `useCallback` and `useMemo`
> - Minimize bridge traffic
> - Use Hermes engine
