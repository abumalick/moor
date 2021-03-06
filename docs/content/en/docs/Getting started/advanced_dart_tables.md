---
title: "Dart tables"
description: Further information on Dart tables
weight: 150
---

{{% pageinfo %}}
__Prefer sql?__: If you prefer, you can also declare tables via `CREATE TABLE` statements.
Moor's sql analyzer will generate matching Dart code. [Details]({{< ref "starting_with_sql.md" >}}).
{{% /pageinfo %}}

As shown in the [getting started guide]({{<relref "_index.md">}}), sql tables can be written in Dart:
```dart
class Todos extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 6, max: 32)();
  TextColumn get content => text().named('body')();
  IntColumn get category => integer().nullable()();
}
```

In this article, we'll cover some advanced features of this syntax.

## Names

By default, moor uses the `snake_case` name of the Dart getter in the database. For instance, the
table
```dart
class EnabledCategories extends Table {
    IntColumn get parentCategory => integer()();
    // ..
}
```

Would be generated as `CREATE TABLE enabled_categories (parent_category INTEGER NOT NULL)`.

To override the table name, simply override the `tableName` getter. An explicit name for
columns can be provided with the `named` method:
```dart
class EnabledCategories extends Table {
    String get tableName => 'categories';

    IntColumn get parentCategory => integer().named('parent')();
}
```

The updated class would be generated as `CREATE TABLE categories (parent INTEGER NOT NULL)`.

To update the name of a column when serializing data to json, annotate the getter with 
[`@JsonKey`](https://pub.dev/documentation/moor/latest/moor_web/JsonKey-class.html).

## Nullability

By default, columns may not contain null values. When you forgot to set a value in an insert,
an exception will be thrown. When using sql, moor also warns about that at compile time.

If you do want to make a column nullable, just use `nullable()`:
```dart
class Items {
    IntColumn get category => integer().nullable();
    // ...
}
```

## Default values

You can set a default value for a column. When not explicitly set, the default value will
be used when inserting a new row. To set a constant default value, use `withDefault`:

```dart
class Preferences extends Table {
  TextColumn get name => integer().autoIncrement()();
  BoolColumn get enabled => boolean().withDefault(const Constant(false))();
}
```

When you later use `into(preferences).insert(PreferencesCompanion.forInsert(name: 'foo'));`, the new
row will have its `enabled` column set to false (and not to null, as it normally would).

Of course, constants can only be used for static values. But what if you want to generate a dynamic
default value for each column? For that, you can use `clientDefault`. It takes a function returning
the desired default value. The function will be called for each insert. For instance, here's an
example generating a random Uuid using the `uuid` package:
```dart
final _uuid = Uuid();

class Users extends Table {
    TextColumn get id => text().clientDefault(() => _uuid.v4())();
    // ...
}
```

Don't know when to use which? Prefer to use `withDefault` when the default value is constant, or something
simple like `currentDate`. For more complicated values, like a randomly generated id, you need to use
`clientDefault`. Internally, `withDefault` writes the default value into the `CREATE TABLE` statement. This
can be more efficient, but doesn't suppport dynamic values.