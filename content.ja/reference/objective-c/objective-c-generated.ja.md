+++
title = "Objective-C 生成コードガイド"
weight = 710
linkTitle = "生成コードガイド"
description = "任意のプロトコル定義に対してプロトコルバッファコンパイラが生成する Objective-C コードを正確に説明します。"
type = "docs"
+++

proto2 と proto3 生成コードの違いが強調されています。このドキュメントを読む前に、[proto2 言語ガイド](/programming-guides/proto2) および/または [proto3 言語ガイド](/programming-guides/proto3) を読むべきです。

## コンパイラの呼び出し {#invocation}

プロトコルバッファコンパイラは、`--objc_out=` コマンドラインフラグを使用して呼び出されると、Objective-C 出力を生成します。`--objc_out=` オプションのパラメータは、コンパイラが Objective-C 出力を書き込むディレクトリです。コンパイラは、各 `.proto` ファイルの入力に対してヘッダーファイルと実装ファイルを作成します。出力ファイルの名前は、`.proto` ファイルの名前を取り、以下の変更を行うことで計算されます:

-   ファイル名は、`.proto` ファイルのベース名をキャメルケースに変換して決定されます。例えば、`foo_bar.proto` は `FooBar` になります。
-   拡張子（`.proto`）は、ヘッダーファイルまたは実装ファイルに対してそれぞれ `pbobjc.h` または `pbobjc.m` に置き換えられます。
-   proto パス（`--proto_path=` または `-I` コマンドラインフラグで指定）は、出力パス（`--objc_out=` フラグで指定）に置き換えられます。

例えば、次のようにコンパイラを呼び出すと:

```shell
protoc --proto_path=src --objc_out=build/gen src/foo.proto src/bar/baz.proto
```

コンパイラはファイル `src/foo.proto` と `src/bar/baz.proto` を読み込み、4 つの出力ファイルを生成します: `build/gen/Foo.pbobjc.h`, `build/gen/Foo.pbobjc.m`, `build/gen/bar/Baz.pbobjc.h`, `build/gen/bar/Baz.pbobjc.m`。コンパイラは必要に応じてディレクトリ `build/gen/bar` を自動的に作成しますが、`build` または `build/gen` を作成しません。これらはすでに存在している必要があります。

## パッケージ {#package}

プロトコルバッファコンパイラによって生成される Objective-C コードは、`.proto` ファイルで定義されたパッケージ名には全く影響を受けません。Objective-C には言語による名前空間がないため、Objective-C クラス名は接頭辞を使用して区別されます。これについては、次のセクションで詳細を確認できます。

## クラス接頭辞 {#prefix}

以下の[file option](/programming-guides/proto3#options)が与えられた場合：

```proto
option objc_class_prefix = "CGOOP";
```

指定された文字列 - この場合は `CGOOP` - は、この `.proto` ファイルのために生成されたすべての Objective-C クラスの前に付けられます。Apple が推奨するように、3文字以上の接頭辞を使用してください。2文字の接頭辞はすべて Apple によって予約されていることに注意してください。

## キャメルケース変換 {#case}

慣用的な Objective-C では、すべての識別子にキャメルケースが使用されます。

メッセージの名前は変換されません。なぜなら、proto ファイルの標準では既にメッセージの名前がキャメルケースであることが想定されているからです。ユーザーが意図的に規則を逸脱していると仮定し、実装はその意図に従うものとします。

フィールド名や `oneof`、`enum` 宣言、拡張アクセサから生成されるメソッドは、キャメルケースで名前が付けられます。一般的に、proto 名をキャメルケースの Objective-C 名に変換するには：

-   最初の文字を大文字に変換します（ただし、フィールドは常に小文字で始まります）。
-   名前内のアンダースコアごとに、アンダースコアを削除し、次の文字を大文字にします。

例えば、フィールド `foo_bar_baz` は `fooBarBaz` になります。フィールド `FOO_bar` は `fooBar` になります。

## メッセージ {#message}

単純なメッセージ宣言が与えられた場合：

```proto
message Foo {}
```

プロトコルバッファコンパイラは `Foo` というクラスを生成します。[`objc_class_prefix` ファイルオプション](#prefix)を指定した場合、このオプションの値が生成されたクラス名の前に付加されます。

C/C++ または Objective-C のキーワードと一致する名前を持つ外部メッセージの場合：

```proto
message static {}
```

生成されるインターフェースは、次のように `_Class` で接尾辞が付けられます：

```objc
@interface static_Class {}
```

[キャメルケース変換ルール](#case)に従うと、名前 `static` は変換されません。`FieldNumber` または `OneOfCase` というキャメルケースの名前を持つ内部メッセージの場合、生成されるインターフェースは、生成された名前が `FieldNumber` 列挙型や `OneOfCase` 列挙型と競合しないように、キャメルケースの名前に `_Class` が付けられます。

メッセージ内に別のメッセージを宣言することもできます。

```proto
message Foo {
  message Bar {}
}
```

これにより、次のように生成されます。

```objc
@interface Foo_Bar : GPBMessage
@end
```

生成されたネストされたメッセージ名は、生成された含まれるメッセージ名（`Foo`）にアンダースコア（`_`）が追加されたものと、ネストされたメッセージ名（`Bar`）です。

> **注意:** 衝突を最小限に抑えるよう努めていますが、アンダースコアとキャメルケースの変換によるメッセージ名の衝突が発生する可能性がある場合があります。
> 例:

```objc
@interface GPBMessage : NSObject
@end
```

```objc
// ディープコピーを行います。
- (id)copy;
// ディープ等価比較を行います。
- (BOOL)isEqual:(id)value;
```

```objc
@property(nonatomic, copy, nullable) GPBUnknownFieldSet *unknownFields;
```

```proto
message Foo {
  message Bar {
    int32 int32_value = 1;
  }
  enum Qux {...}
  int32 int32_value = 1;
  string string_value = 2;
  Bar message_value = 3;
  Qux enum_value = 4;
  bytes bytes_value = 5;
}
```

```objc
typedef GPB_ENUM(Foo_Bar_FieldNumber) {
  // 生成されたフィールド番号名は、アンダースコアで区切られた囲むメッセージ名に続いて "FieldNumber"、その後にフィールド名のキャメルケースが続きます。
  Foo_Bar_FieldNumber_Int32Value = 1,
};

@interface Foo_Bar : GPBMessage
@property(nonatomic, readwrite) int32_t int32Value;
@end

typedef GPB_ENUM(Foo_FieldNumber) {
  Foo_FieldNumber_Int32Value = 1,
  Foo_FieldNumber_StringValue = 2,
  Foo_FieldNumber_MessageValue = 3,
  Foo_FieldNumber_EnumValue = 4,
  Foo_FieldNumber_BytesValue = 5,
};

typedef GPB_ENUM(Foo_Qux) {
  Foo_Qux_GPBUnrecognizedEnumeratorValue = kGPBUnrecognizedEnumeratorValue,
  ...
};

@interface Foo : GPBMessage
// フィールド名はキャメルケースです。
@property(nonatomic, readwrite) int32_t int32Value;
@property(nonatomic, readwrite, copy, null_resettable) NSString *stringValue;
@property(nonatomic, readwrite) BOOL hasMessageValue;
@property(nonatomic, readwrite, strong, null_resettable) Foo_Bar *messageValue;
@property(nonatomic, readwrite) Foo_Qux enumValue;
@property(nonatomic, readwrite, copy, null_resettable) NSData *bytesValue;
@end

```proto
message Foo {
  int32 foo_array = 1;      // 「Array」で終わります
  int32 bar_OneOfCase = 2;  // 「oneofcase」で終わります
  int32 id = 3;             // C/C++/Objective-C キーワードです
}

```objc
typedef GPB_ENUM(Foo_FieldNumber) {
  // 非繰り返しフィールド名が「Array」で終わる場合、「_p」が付加され、名前が繰り返しタイプと区別されます。
  Foo_FieldNumber_FooArray_p = 1,
  // フィールド名が「OneOfCase」で終わる場合、「_p」が付加され、OneOfCaseプロパティと名前が区別されます。
  Foo_FieldNumber_BarOneOfCase_p = 2,
  // フィールド名がC/C++/ObjectiveCのキーワードである場合、「_p」が付加され、コンパイルが可能になります。
  Foo_FieldNumber_Id_p = 3,
};

@interface Foo : GPBMessage
@property(nonatomic, readwrite) int32_t fooArray_p;
@property(nonatomic, readwrite) int32_t barOneOfCase_p;
@property(nonatomic, readwrite) int32_t id_p;
@end
```

```proto
message Foo {
  message Bar {
     int32 b;
  }
  Bar a;
}
```

```objc
Foo *foo = [[Foo alloc] init];
foo.a.b = 2;
```

```proto
message Foo {
  message Bar {
    int32 int32_value = 1;
  }
  enum Qux {...}
  optional int32 int32_value = 1;
  optional string string_value = 2;
  optional Bar message_value = 3;
  optional Qux enum_value = 4;
  optional bytes bytes_value = 5;
}
```

```objc
# Enum Foo_Qux

typedef GPB_ENUM(Foo_Qux) {
  Foo_Qux_Flupple = 0,
};

GPBEnumDescriptor *Foo_Qux_EnumDescriptor(void);

BOOL Foo_Qux_IsValidValue(int32_t value);

# Message Foo

typedef GPB_ENUM(Foo_FieldNumber) {
  Foo_FieldNumber_Int32Value = 2,
  Foo_FieldNumber_MessageValue = 3,
  Foo_FieldNumber_EnumValue = 4,
  Foo_FieldNumber_BytesValue = 5,
  Foo_FieldNumber_StringValue = 6,
};

@interface Foo : GPBMessage

@property(nonatomic, readwrite) BOOL hasInt32Value;
@property(nonatomic, readwrite) int32_t int32Value;

@property(nonatomic, readwrite) BOOL hasStringValue;
@property(nonatomic, readwrite, copy, null_resettable) NSString *stringValue;

@property(nonatomic, readwrite) BOOL hasMessageValue;
@property(nonatomic, readwrite, strong, null_resettable) Foo_Bar *messageValue;

@property(nonatomic, readwrite) BOOL hasEnumValue;
@property(nonatomic, readwrite) Foo_Qux enumValue;

@property(nonatomic, readwrite) BOOL hasBytesValue;
@property(nonatomic, readwrite, copy, null_resettable) NSData *bytesValue;

@end

# Message Foo_Bar

typedef GPB_ENUM(Foo_Bar_FieldNumber) {
  Foo_Bar_FieldNumber_Int32Value = 1,
};

@interface Foo_Bar : GPBMessage

@property(nonatomic, readwrite) BOOL hasInt32Value;
@property(nonatomic, readwrite) int32_t int32Value;

@end
```

```proto
message Foo {
  optional int32 foo_array = 1;      // Ends with Array
  optional int32 bar_OneOfCase = 2;  // Ends with oneofcase
  optional int32 id = 3;             // Is a C/C++/Objective-C keyword
}
```

```objc
typedef GPB_ENUM(Foo_FieldNumber) {
  // 非繰り返しフィールド名が「Array」で終わる場合、「_p」が付加されます
  // 名前が繰り返しタイプと区別されるようにします。
  Foo_FieldNumber_FooArray_p = 1,
  // フィールド名が「OneOfCase」で終わる場合、「_p」が付加されます
  // OneOfCaseプロパティと名前が区別されるようにします。
  Foo_FieldNumber_BarOneOfCase_p = 2,
  // フィールド名がC/C++/ObjectiveCのキーワードの場合、「_p」が付加されます
  // コンパイルできるようにします。
  Foo_FieldNumber_Id_p = 3,
};

@interface Foo : GPBMessage
@property(nonatomic, readwrite) int32_t fooArray_p;
@property(nonatomic, readwrite) int32_t barOneOfCase_p;
@property(nonatomic, readwrite) int32_t id_p;
@end
```

```proto
message Foo {
  message Bar {
     int32 b;
  }
  Bar a;
}
```

```objc
Foo *foo = [[Foo alloc] init];
foo.a.b = 2;
```

```proto
message Foo {
  message Bar {}
  enum Qux {}
  repeated int32 int32_value = 1;
  repeated string string_value = 2;
  repeated Bar message_value = 3;
  repeated Qux enum_value = 4;
}
```

```objc
typedef GPB_ENUM(Foo_FieldNumber) {
  Foo_FieldNumber_Int32ValueArray = 1,
  Foo_FieldNumber_StringValueArray = 2,
  Foo_FieldNumber_MessageValueArray = 3,
  Foo_FieldNumber_EnumValueArray = 4,
};

@interface Foo : GPBMessage
// 繰り返しタイプのフィールド名はキャメルケースの名前に
// 「Array」が付加されます。
@property(nonatomic, readwrite, strong, null_resettable)
 GPBInt32Array *int32ValueArray;
@property(nonatomic, readonly) NSUInteger int32ValueArray_Count;

@property(nonatomic, readwrite, strong, null_resettable)
 NSMutableArray *stringValueArray;
@property(nonatomic, readonly) NSUInteger stringValueArray_Count;

@property(nonatomic, readwrite, strong, null_resettable)
 NSMutableArray *messageValueArray;
@property(nonatomic, readonly) NSUInteger messageValueArray_Count;

@property(nonatomic, readwrite, strong, null_resettable)
 GPBEnumArray *enumValueArray;
@property(nonatomic, readonly) NSUInteger enumValueArray_Count;
@end
```

```objc
if (myFoo.messageValueArray_Count) {
  // 配列に何かが入っています...
}
```

```objc
@interface GPBArray : NSObject
@property (nonatomic, readonly) NSUInteger count;
+ (instancetype)array;
+ (instancetype)arrayWithValue:()value;
+ (instancetype)arrayWithValueArray:(GPBArray *)array;
+ (instancetype)arrayWithCapacity:(NSUInteger)count;

// 配列を初期化し、値をコピーします。
- (instancetype)initWithValueArray:(GPBArray *)array;
- (instancetype)initWithValues:(const  [])values
                         count:(NSUInteger)count NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithCapacity:(NSUInteger)count;

- ()valueAtIndex:(NSUInteger)index;

- (void)enumerateValuesWithBlock:
     (void (^)( value, NSUInteger idx, BOOL *stop))block;
- (void)enumerateValuesWithOptions:(NSEnumerationOptions)opts
    usingBlock:(void (^)( value, NSUInteger idx, BOOL *stop))block;

- (void)addValue:()value;
- (void)addValues:(const  [])values count:(NSUInteger)count;
- (void)addValuesFromArray:(GPBArray *)array;

- (void)removeValueAtIndex:(NSUInteger)count;
- (void)removeAll;

- (void)exchangeValueAtIndex:(NSUInteger)idx1
            withValueAtIndex:(NSUInteger)idx2;
- (void)insertValue:()value atIndex:(NSUInteger)count;
- (void)replaceValueAtIndex:(NSUInteger)index withValue:()value;


@end
```

```objc
@interface GPBEnumArray : NSObject
@property (nonatomic, readonly) NSUInteger count;
@property (nonatomic, readonly) GPBEnumValidationFunc validationFunc;

+ (instancetype)array;
+ (instancetype)arrayWithValidationFunction:(nullable GPBEnumValidationFunc)func;
+ (instancetype)arrayWithValidationFunction:(nullable GPBEnumValidationFunc)func
                                   rawValue:value;
+ (instancetype)arrayWithValueArray:(GPBEnumArray *)array;
+ (instancetype)arrayWithValidationFunction:(nullable GPBEnumValidationFunc)func
                                   capacity:(NSUInteger)count;

- (instancetype)initWithValidationFunction:
  (nullable GPBEnumValidationFunc)func;

// 配列を初期化し、値をコピーします。
- (instancetype)initWithValueArray:(GPBEnumArray *)array;
- (instancetype)initWithValidationFunction:(nullable GPBEnumValidationFunc)func
    values:(const int32_t [])values
    count:(NSUInteger)count NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithValidationFunction:(nullable GPBEnumValidationFunc)func
                                  capacity:(NSUInteger)count;

// これらは、validationFuncで定義された有効な列挙子でない場合、
// indexの値がkGPBUnrecognizedEnumeratorValueを返します。実際の
// 値が必要な場合は、メソッドの「raw」バージョンを使用してください。
- (int32_t)valueAtIndex:(NSUInteger)index;
- (void)enumerateValuesWithBlock:
    (void (^)(int32_t value, NSUInteger idx, BOOL *stop))block;
- (void)enumerateValuesWithOptions:(NSEnumerationOptions)opts
    usingBlock:(void (^)(int32_t value, NSUInteger idx, BOOL *stop))block;

// これらのメソッドは、バイナリがコンパイルされた時点で知られていない値に
// アクセスするために、validationFuncをバイパスします。
- (int32_t)rawValueAtIndex:(NSUInteger)index;

- (void)enumerateRawValuesWithBlock:
    (void (^)(int32_t value, NSUInteger idx, BOOL *stop))block;
- (void)enumerateRawValuesWithOptions:(NSEnumerationOptions)opts
    usingBlock:(void (^)(int32_t value, NSUInteger idx, BOOL *stop))block;

// valueがvalidationFuncで定義された有効な列挙子でない場合、
// これらのメソッドはデバッグ時にアサートし、リリース時にログを出力し、
// デフォルト値に値を割り当てます。非列挙子値を割り当てるには、
// 以下のrawValueメソッドを使用してください。
- (void)addValue:(int32_t)value;
- (void)addValues:(const int32_t [])values count:(NSUInteger)count;
- (void)insertValue:(int32_t)value atIndex:(NSUInteger)count;
- (void)replaceValueAtIndex:(NSUInteger)index withValue:(int32_t)value;

// これらのメソッドは、バイナリがコンパイルされた時点で知られていない値に
// アクセスするために、validationFuncをバイパスします。
- (void)addRawValue:(int32_t)rawValue;
- (void)addRawValuesFromEnumArray:(GPBEnumArray *)array;
- (void)addRawValues:(const int32_t [])values count:(NSUInteger)count;
- (void)replaceValueAtIndex:(NSUInteger)index withRawValue:(int32_t)rawValue;
- (void)insertRawValue:(int32_t)value atIndex:(NSUInteger)count;

// これらのメソッドには検証が適用されません。
- (void)removeValueAtIndex:(NSUInteger)count;
- (void)removeAll;
- (void)exchangeValueAtIndex:(NSUInteger)idx1
            withValueAtIndex:(NSUInteger)idx2;

@end
```

```proto
message Order {
  oneof OrderID {
    string name = 1;
    int32 address = 2;
  };
  int32 quantity = 3;
};
```

```objc
typedef GPB_ENUM(Order_OrderID_OneOfCase) {
  Order_OrderID_OneOfCase_GPBUnsetOneOfCase = 0,
  Order_OrderID_OneOfCase_Name = 1,
  Order_OrderID_OneOfCase_Address = 2,
};

typedef GPB_ENUM(Order_FieldNumber) {
  Order_FieldNumber_Name = 1,
  Order_FieldNumber_Address = 2,
  Order_FieldNumber_Quantity = 3,
};

@interface Order : GPBMessage
@property (nonatomic, readwrite) Order_OrderID_OneOfCase orderIDOneOfCase;
@property (nonatomic, readwrite, copy, null_resettable) NSString *name;
@property (nonatomic, readwrite) int32_t address;
@property (nonatomic, readwrite) int32_t quantity;
@end

void Order_ClearOrderIDOneOfCase(Order *message);
```

```proto
message Bar {...}
message Foo {
  map<int32, string> a_map = 1;
  map<string, Bar> b_map = 2;
};
```

```objc
typedef GPB_ENUM(Foo_FieldNumber) {
  Foo_FieldNumber_AMap = 1,
  Foo_FieldNumber_BMap = 2,
};

@interface Foo : GPBMessage
// Map names are the camel case version of the field name.
@property (nonatomic, readwrite, strong, null_resettable) GPBInt32ObjectDictionary *aMap;
@property(nonatomic, readonly) NSUInteger aMap_Count;
@property (nonatomic, readwrite, strong, null_resettable) NSMutableDictionary *bMap;
@property(nonatomic, readonly) NSUInteger bMap_Count;
@end
```

```objc
GBP<KEY><VALUE>Dictionary
```

```objc
if (myFoo.myMap_Count) {
  // There is something in the map...
}
```

```objc
@interface GPB<KEY>Dictionary : NSObject
@property (nonatomic, readonly) NSUInteger count;

+ (instancetype)dictionary;
+ (instancetype)dictionaryWithValue:(const )value
                             forKey:(const <KEY>)key;
+ (instancetype)dictionaryWithValues:(const  [])values
                             forKeys:(const <KEY> [])keys
                               count:(NSUInteger)count;
+ (instancetype)dictionaryWithDictionary:(GPB<KEY>Dictionary *)dictionary;
+ (instancetype)dictionaryWithCapacity:(NSUInteger)numItems;
```

```objc
- (instancetype)initWithValues:(const  [])values
                       forKeys:(const <KEY> [])keys
                         count:(NSUInteger)count NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithDictionary:(GPB<KEY>Dictionary *)dictionary;
- (instancetype)initWithCapacity:(NSUInteger)numItems;

- (BOOL)valueForKey:(<KEY>)key value:(VALUE *)value;

- (void)enumerateKeysAndValuesUsingBlock:
    (void (^)(<KEY> key,  value, BOOL *stop))block;

- (void)removeValueForKey:(<KEY>)aKey;
- (void)removeAll;
- (void)setValue:()value forKey:(<KEY>)key;
- (void)addEntriesFromDictionary:(GPB<KEY>Dictionary *)otherDictionary;
@end

The `GBP<KEY>ObjectDictionary` interface is:

```objc
@interface GPB<KEY>ObjectDictionary : NSObject
@property (nonatomic, readonly) NSUInteger count;

+ (instancetype)dictionary;
+ (instancetype)dictionaryWithObject:(id)object
                             forKey:(const <KEY>)key;
+ (instancetype)
  dictionaryWithObjects:(const id GPB_UNSAFE_UNRETAINED [])objects
                forKeys:(const <KEY> [])keys
                  count:(NSUInteger)count;
+ (instancetype)dictionaryWithDictionary:(GPB<KEY>ObjectDictionary *)dictionary;
+ (instancetype)dictionaryWithCapacity:(NSUInteger)numItems;

- (instancetype)initWithObjects:(const id GPB_UNSAFE_UNRETAINED [])objects
                        forKeys:(const <KEY> [])keys
                          count:(NSUInteger)count NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithDictionary:(GPB<KEY>ObjectDictionary *)dictionary;
- (instancetype)initWithCapacity:(NSUInteger)numItems;

- (id)objectForKey:(uint32_t)key;

- (void)enumerateKeysAndObjectsUsingBlock:
    (void (^)(<KEY> key, id object, BOOL *stop))block;

- (void)removeObjectForKey:(<KEY>)aKey;
- (void)removeAll;
- (void)setObject:(id)object forKey:(<KEY>)key;
- (void)addEntriesFromDictionary:(GPB<KEY>ObjectDictionary *)otherDictionary;
@end

`GBP<KEY>EnumDictionary` has a slightly different interface to handle the
validation function and to access raw values.

```objc
@interface GPB<KEY>EnumDictionary : NSObject

@property(nonatomic, readonly) NSUInteger count;
@property(nonatomic, readonly) GPBEnumValidationFunc validationFunc;
```  

```objective-c
+ (instancetype)dictionary;
+ (instancetype)dictionaryWithValidationFunction:(nullable GPBEnumValidationFunc)func;
+ (instancetype)dictionaryWithValidationFunction:(nullable GPBEnumValidationFunc)func
                                        rawValue:(int32_t)rawValue
                                          forKey:(<KEY>_t)key;
+ (instancetype)dictionaryWithValidationFunction:(nullable GPBEnumValidationFunc)func
                                       rawValues:(const int32_t [])values
                                         forKeys:(const <KEY>_t [])keys
                                           count:(NSUInteger)count;
+ (instancetype)dictionaryWithDictionary:(GPB<KEY>EnumDictionary *)dictionary;
+ (instancetype)dictionaryWithValidationFunction:(nullable GPBEnumValidationFunc)func
                                        capacity:(NSUInteger)numItems;

- (instancetype)initWithValidationFunction:(nullable GPBEnumValidationFunc)func;
- (instancetype)initWithValidationFunction:(nullable GPBEnumValidationFunc)func
                                 rawValues:(const int32_t [])values
                                   forKeys:(const <KEY>_t [])keys
                                     count:(NSUInteger)count NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithDictionary:(GPB<KEY>EnumDictionary *)dictionary;
- (instancetype)initWithValidationFunction:(nullable GPBEnumValidationFunc)func
                                  capacity:(NSUInteger)numItems;

// These will return kGPBUnrecognizedEnumeratorValue if the value for the key
// is not a valid enumerator as defined by validationFunc. If the actual value is
// desired, use "raw" version of the method.

- (BOOL)valueForKey:(<KEY>_t)key value:(nullable int32_t *)value;

- (void)enumerateKeysAndValuesUsingBlock:
    (void (^)(<KEY>_t key, int32_t value, BOOL *stop))block;

// These methods bypass the validationFunc to provide access to values that were not
// known at the time the binary was compiled.

- (BOOL)valueForKey:(<KEY>_t)key rawValue:(nullable int32_t *)rawValue;
```

```objc
- (void)enumerateKeysAndRawValuesUsingBlock:
    (void (^)(<KEY>_t key, int32_t rawValue, BOOL *stop))block;

- (void)addRawEntriesFromDictionary:(GPB<KEY>EnumDictionary *)otherDictionary;

// もしvalueがvalidationFuncで定義された有効なenumeratorでない場合、これらのメソッドはデバッグ時にはアサートし、リリース時にはログを出力してデフォルト値に値を割り当てます。enumeratorでない値を割り当てるには、以下のrawValueメソッドを使用してください。

- (void)setValue:(int32_t)value forKey:(<KEY>_t)key;

// このメソッドはvalidationFuncをバイパスして、バイナリがコンパイルされた時点で知られていなかった値の設定を提供します。
- (void)setRawValue:(int32_t)rawValue forKey:(<KEY>_t)key;

// これらのメソッドにはバリデーションが適用されません。

- (void)removeValueForKey:(<KEY>_t)aKey;
- (void)removeAll;

@end

## Enumerations {#enum}

Given an enum definition like:

```proto
enum Foo {
  VALUE_A = 0;
  VALUE_B = 1;
  VALUE_C = 5;
}

the generated code will be:

```objc
// 生成されたenum値の名前は、列挙名の後にアンダースコアが続き、その後にキャメルケースに変換されたenumerator名が続きます。
// GPB_ENUMは、Objective-C Protocol Bufferヘッダーで定義されたマクロで、すべてのenum値がint32であることを強制し、Swift Enumerationのサポートを支援します。
typedef GPB_ENUM(Foo) {
  Foo_GPBUnrecognizedEnumeratorValue = kGPBUnrecognizedEnumeratorValue, //proto3 only
  Foo_ValueA = 0,
  Foo_ValueB = 1;
  Foo_ValueC = 5;
};

// このenum型が定義する値に関する情報を返します。
GPBEnumDescriptor *Foo_EnumDescriptor();

Each enumeration has a validation function declared for it:

```objc
// 与えられた数値がFooの定義された値（0、1、5）のいずれかと一致する場合、YESを返します。
BOOL Foo_IsValidValue(int32_t value);

and an enumeration descriptor accessor function declared for it:

```objc
// GPBEnumDescriptorはランタイムで定義され、enum名、enum値、enum値バリデーション関数など、enum定義に関する情報を含んでいます。
typedef GPBEnumDescriptor *(*GPBEnumDescriptorAccessorFunc)();

The enum descriptor accessor functions are C functions, as opposed to methods on
the enumeration class, because they are rarely used by client software. This
will cut down on the amount of Objective-C runtime information generated, and
potentially allow the linker to deadstrip them.

In the case of outer enums that have names matching any C/C++ or Objective-C
keywords, such as:

```proto
enum method {}

the generated interfaces are suffixed with `_Enum`, as follows:

```objc
// 生成された列挙名は、キーワードの後に_Enumが付いたものです。
typedef GPB_ENUM(Method_Enum) {}
```

```proto
message Foo {
  enum Bar {
    VALUE_A = 0;
    VALUE_B = 1;
    VALUE_C = 5;
  }
  Bar aBar = 1;
  Bar aDifferentBar = 2;
  repeated Bar aRepeatedBar = 3;
}
```{/*ee56da7929e2b91e*/}

```objc
typedef GPB_ENUM(Foo_Bar) {
  Foo_Bar_GPBUnrecognizedEnumeratorValue = kGPBUnrecognizedEnumeratorValue, //proto3 only
  Foo_Bar_ValueA = 0;
  Foo_Bar_ValueB = 1;
  Foo_Bar_ValueC = 5;
};

GPBEnumDescriptor *Foo_Bar_EnumDescriptor();

BOOL Foo_Bar_IsValidValue(int32_t value);

@interface Foo : GPBMessage
@property (nonatomic, readwrite) Foo_Bar aBar;
@property (nonatomic, readwrite) Foo_Bar aDifferentBar;
@property (nonatomic, readwrite, strong, null_resettable)
 GPBEnumArray *aRepeatedBarArray;
@end

// proto3 only Every message that has an enum field will have an accessor function to get
// the value of that enum as an integer. This allows clients to deal with
// raw values if they need to.
int32_t Foo_ABar_RawValue(Foo *message);
void SetFoo_ABar_RawValue(Foo *message, int32_t value);
int32_t Foo_ADifferentBar_RawValue(Foo *message);
void SetFoo_ADifferentBar_RawValue(Foo *message, int32_t value);
```{/*a4ad48ab04a2485a*/}

```proto
// Proto
enum Foo {
  VALUE_A = 0;
}
```{/*1456f155ce2f9376*/}

```objc
// Objective-C
typedef GPB_ENUM(Foo) {
  Foo_GPBUnrecognizedEnumeratorValue = kGPBUnrecognizedEnumeratorValue,
  Foo_ValueA = 0,
};
```{/*4c1c3379489c717f*/}

```swift
// Swift
let aValue = Foo.ValueA
let anotherValue: Foo = .GPBUnrecognizedEnumeratorValue
```{/*c591d6a721a1cd9c*/}

```objc
@interface GPBTimeStamp (GPBWellKnownTypes)
@property (nonatomic, readwrite, strong) NSDate *date;
@property (nonatomic, readwrite) NSTimeInterval timeIntervalSince1970;
- (instancetype)initWithDate:(NSDate *)date;
- (instancetype)initWithTimeIntervalSince1970:
    (NSTimeInterval)timeIntervalSince1970;
@end
```{/*59b5eac6e84d4535*/}

```objc
@interface GPBDuration (GPBWellKnownTypes)
@property (nonatomic, readwrite) NSTimeInterval timeIntervalSince1970;
- (instancetype)initWithTimeIntervalSince1970:
    (NSTimeInterval)timeIntervalSince1970;
@end
```{/*8e96d4dcc2f6b774*/}

```proto
message Foo {
  extensions 100 to 199;
}
```{/*3cd09c235effbb52*/}

```objc
# ファイル Test2Root

@interface Test2Root : GPBRootObject

// ベースクラスは提供します:
//   + (GPBExtensionRegistry *)extensionRegistry;
// これは、このファイルとその依存ファイルで定義されたすべての拡張機能を含む GPBExtensionRegistry です。

@end

@interface Test2Root (DynamicMethods)
+ (GPBExtensionDescriptor *)foo;
+ (GPBExtensionDescriptor *)repeatedFoo;
@end

# メッセージ Foo

@interface Foo : GPBMessage

@end

# メッセージ Bar

@interface Bar : GPBMessage

@end

@interface Bar (DynamicMethods)

+ (GPBExtensionDescriptor *)bar;
+ (GPBExtensionDescriptor *)repeatedBar;
@end
```
