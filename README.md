# observable_ish

Write elegant reactive cross-platform client side application using observable states and event emitters. 

Provides:

1. [Reactive Values][RxValue]
2. [Reactive Lists][RxList]
3. [Reactive Sets][RxSet]
4. [Reactive Maps][RxMap]
5. [Event emitter][Emitter]

## Philosophy

Observable-ish provides a light-weight non-intrusive reactive framework to build cross-platform UI. It uses Dart's 
asynchronous `Stream`s to emit and listen to changes.

Various observable types like [`RxValue`][RxValue], [`RxList`][RxList], [`RxSet`][RxSet] and [`RxMap`][RxMap] can be 
used to update UI automatically on changes. Events can be passed up the widget tree using event [`Emitter`][Emitter].

## Reactive values

[`RxValue`][RxValue] can be used to encapsulate a simple observable value. 

### Getting and setting value

[`RxValue`][RxValue] exposes field [`value`][RxValue_value] to get set current value. 


```dart
main() {
  final rxInts = RxValue<int>(initial: 5);
  int got = rxInts.value; // Gets current value
  rxInts.value = 10;      // Sets current value
}
```

When a value that is different from the existing value is set, the change is notified through various ways explained in the
section.

### Listening to changes
It provides few flexible ways to listen to changes:

1. [onChange: Record of changes][RxValue_onChange]
2. [values: Stream of new values][RxValue_values]
3. [listen: Callback function with new value][RxValue_listen]

```dart
main() {
  final rxInts = RxValue<int>(initial: 5);
  print(rxInts.value);  // => 5
  rxInts.values.listen((int v) => print(v));  // => 5, 20, 25
  rxInts.value = 20;
  rxInts.value = 25;
}
```

### Binding to a value

Binding an `RxValue` to a `Stream` (using method [`bindStream`][RxValue_bindStream]) or another `RxValue` 
(using method [`bind`][RxValue_bind]) changes its value when the source `Stream` emits or `RxValue` changes. This is very 
useful in scenarios where one would like to change a model's value or widget's property when control changes. For example, 
change a text field's value when a checkbox is toggled.

```dart
  textBox.value.bindStream(checkBox.checked.map((bool v) => v?'Female': 'Male'));
```

### Full examples

```dart
main() {
  final rxInts = RxValue<int>(initial: 5);
  print(rxInts.value);  // => 5
  rxInts.value = 10;
  rxInts.value = 15;
  rxInts.values.listen((int v) => print(v));  // => 15, 20, 25
  rxInts.value = 20;
  rxInts.value = 25;
}
```

## Composite reactive objects

Observable-ish is designed to be non-intrusive. The philosophy is to separate the model and its reactive cousin into 
different classes.

```dart
class RxUser {
  final name = RxValue<String>();
  final age = RxValue<int>();
}

class User {
  final rx = RxUser();

  User({String name, int age}) {
    this.name = name;
    this.age = age;
  }

  String get name => rx.name.value;
  set name(String value) => rx.name.value = value;

  int get age => rx.age.value;
  set age(int value) => rx.age.value = value;
}

main() {
  final user = User(name: 'Messi', age: 30);
  user.age = 31;
  print(user.age);  // => 31
  print('---------');
  user.age = 32;
  user.rx.age.listen((int v) => print(v));  // => 20, 25
  user.age = 33;
  user.age = 34;
  user.age = 35;
}
```

## Event emitter

`Emitter`s provide a simple interface to emit and listen to events. It is designed to inter-operate with `RxValue` to provide
maximum productivity.

### Listening to a event

1. [on: Execute callback on event][Emitter_on]
2. [listen: Similar to Stream][Emitter_listen]
3. [asStream: Obtain event as Stream][Emitter_asStream]

### Piping events

[`pipeTo`][Emitter_pipeTo] pipes events to another `Emitter`.

[`pipeToValue`][Emitter_pipeToValue] pipes events to the given `RxValue`. This could be very helpful in binding events 
to observable values.

### Emitting events

`emit`, `emitOne`, `emitAll`, `emitStream` and `emitRxValue` provides various ways to emit events using the `Emitter`

## Reactive Lists

`RxList` notifies changes (addition, removal, clear, setting) of its elements.

### Updating RxList

`RxList` implements Dart's `List`. 

Besides `List`'s methods, `RxList` provides convenient methods like [`addIf`][RxList_addIf] and [`addAllIf`][RxList_addAllIf] 
to add elements based on a condition. This is very useful in writing UI in Dart DSL (as in Flutter and Nuts).

```dart
main() {
  final rxInts = RxList<int>();
  rxInts.onChange.listen((c) => print(c.element)); // => 5
  rxInts.addIf(5 < 10, 5);
  rxInts.addIf(5 > 9, 9);
}
```

Use `assign` and `assignAll` methods to replace existing contents of the list with new content.

### Listening for changes

[`onChange`][RxList_onChange] exposes a `Stream` of record of change of the `List`.

## Reactive Sets

`RxSet` notifies changes (addition and removal) of its elements.

### Updating RxSet

`RxSet` implements Dart's `Set`. 

Besides `Set`'s methods, `RxSet` provides convenient methods like [`addIf`][RxSet_addIf] and [`addAllIf`][RxSet_addAllIf] 
to add elements based on a condition. This is very useful in writing UI in Dart DSL (as in Flutter and Nuts).

```dart
main() {
  final rxInts = RxSet<int>();
  rxInts.onChange.listen((c) => print(c.element)); // => 5
  rxInts.addIf(5 < 10, 5);
  rxInts.addIf(5 > 9, 9);
}
```

### Listening for changes

[`onChange`][RxSet_onChange] exposes a `Stream` of record of change of the `Set`.

### Binding

`bindBool` and `bindBoolValue` allows removing or adding the given element based on the `Stream` of
`bool`s or `RxValue` of `bool`s.

`bindOneByIndexStream` and `bindOneByIndex` allows removing all but the one element from a given `Iterable`
of elements based on index `Stream` or `RxValue`.

## Reactive Maps

`RxMap` notifies changes (addition, removal, clear, setting) of its elements.

### Updating RxMap

`RxMap` implements Dart's `Map`. 

Besides `Map`'s methods, `RxMap` provides convenient methods like `addIf` and `addAllIf` to add elements based on a 
condition. This is very useful in writing UI in Dart DSL (as in Flutter and Nuts).

### Listening for changes

`onChange` exposes a `Stream` of record of change of the `Map`.

[RxValue]: https://pub.dartlang.org/documentation/observable_ish/latest/value_value/RxValue-class.html
[RxList]: https://pub.dartlang.org/documentation/observable_ish/latest/list_list/RxList-class.html
[RxSet]: https://pub.dartlang.org/documentation/observable_ish/latest/set_set/RxSet-class.html
[RxMap]: https://pub.dartlang.org/documentation/observable_ish/latest/map_map/RxMap-class.html
[Emitter]: https://pub.dartlang.org/documentation/observable_ish/latest/event_event/Emitter-class.html
[RxValue_value]: https://pub.dartlang.org/documentation/observable_ish/latest/value_value/RxValue/value.html
[RxValue_onChange]: https://pub.dartlang.org/documentation/observable_ish/latest/value_value/RxValue/onChange.html
[RxValue_values]: https://pub.dartlang.org/documentation/observable_ish/latest/value_value/RxValue/values.html
[RxValue_listen]: https://pub.dartlang.org/documentation/observable_ish/latest/value_value/RxValue/listen.html
[RxValue_bindStream]: https://pub.dartlang.org/documentation/observable_ish/latest/value_value/RxValue/bindStream.html
[RxValue_bind]: https://pub.dartlang.org/documentation/observable_ish/latest/value_value/RxValue/bind.html
[RxList_addIf]: https://pub.dartlang.org/documentation/observable_ish/latest/list_list/RxList/addIf.html
[RxList_addAllIf]: https://pub.dartlang.org/documentation/observable_ish/latest/list_list/RxList/addAllIf.html
[RxList_onChange]: https://pub.dartlang.org/documentation/observable_ish/latest/list_list/RxList/onChange.html
[RxSet_addIf]: https://pub.dartlang.org/documentation/observable_ish/latest/set_set/RxSet/addIf.html
[RxSet_addAllIf]: https://pub.dartlang.org/documentation/observable_ish/latest/set_set/RxSet/addAllIf.html
[RxSet_onChange]: https://pub.dartlang.org/documentation/observable_ish/latest/set_set/RxSet/onChange.html
[Emitter_on]: https://pub.dartlang.org/documentation/observable_ish/latest/event_event/Emitter/on.html
[Emitter_listen]: https://pub.dartlang.org/documentation/observable_ish/latest/event_event/Emitter/listen.html
[Emitter_asStream]: https://pub.dartlang.org/documentation/observable_ish/latest/event_event/Emitter/asStream.html
[Emitter_pipeTo]: https://pub.dartlang.org/documentation/observable_ish/latest/event_event/Emitter/pipeTo.html
[Emitter_pipeToValue]: https://pub.dartlang.org/documentation/observable_ish/latest/event_event/Emitter/pipeToValue.html