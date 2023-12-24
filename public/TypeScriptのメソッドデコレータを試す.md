---
title: TypeScriptのメソッドデコレータを試す
tags:
  - TypeScript
private: false
updated_at: '2022-09-26T10:44:44+09:00'
id: 62b1c15e9b09794aa22d
organization_url_name: null
slide: false
ignorePublish: false
---
筆者がTypeScriptを勉強する中でデコレータについて学んだのですが、どうにもデコレータの記法や動作が記憶に定着しないので、備忘録も兼ねて記事化します。
（勉強中につき、内容に誤りがある可能性があります。）

TypeScriptのデコレータには以下の５種類があります。
- クラスデコレータ
- メソッドデコレータ
- プロパティデコレータ
- アクセサデコレータ
- パラメータデコレータ

が、本記事では、
- メソッドデコレータ

について動作を確認します。

デコレータに関する公式のドキュメントは以下になります。

https://www.typescriptlang.org/docs/handbook/decorators.html

# サンプルプロジェクトの作成

デコレータの動作を確認するためのサンプルプロジェクトを作成します。
とりあえずTypeScriptが動くまでのところは、筆者が以前投稿した記事に沿って作成します。

https://qiita.com/plumchang/items/a800be904936e86d6f44

Webページとコンソールそれぞれに`Hello`と表示されるだけの状態になります。
![typescript.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/495712a1-dca1-55d6-318f-dcec0bea1771.png)


# メソッドデコレータを試す

まず、デコレータを使うには、`tsconfig.json`を編集する必要があります。`experimentalDecorators`のコメントアウトを外します。

```diff_json:./tsconfig.json
{
  "compilerOptions": {
    /* Visit https://aka.ms/tsconfig to read more about this file */

    /* Projects */
    // "incremental": true,                              /* Save .tsbuildinfo files to allow for incremental compilation of projects. */
    // "composite": true,                                /* Enable constraints that allow a TypeScript project to be used with project references. */
    // "tsBuildInfoFile": "./.tsbuildinfo",              /* Specify the path to .tsbuildinfo incremental compilation file. */
    // "disableSourceOfProjectReferenceRedirect": true,  /* Disable preferring source files instead of declaration files when referencing composite projects. */
    // "disableSolutionSearching": true,                 /* Opt a project out of multi-project reference checking when editing. */
    // "disableReferencedProjectLoad": true,             /* Reduce the number of projects loaded automatically by TypeScript. */

    /* Language and Environment */
    "target": "es2016",                                  /* Set the JavaScript language version for emitted JavaScript and include compatible library declarations. */
    // "lib": [],                                        /* Specify a set of bundled library declaration files that describe the target runtime environment. */
    // "jsx": "preserve",                                /* Specify what JSX code is generated. */
-   // "experimentalDecorators": true,                   /* Enable experimental support for TC39 stage 2 draft decorators. */
+   "experimentalDecorators": true,                   /* Enable experimental support for TC39 stage 2 draft decorators. */

    // "emitDecoratorMetadata": true,                    /* Emit design-type metadata for decorated declarations in source files. */

    // <中略>

  },
  "exclude": ["node_modules"]
}

```

これで、TypeScriptでデコレータを使えるようになります。

次に、`./index.ts`を編集します。

```ts:./index.ts
function methodDecoratorFactorySample(str: string) {
  console.log(`メソッドデコレータファクトリが呼ばれました: ${str}`);

  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log(`メソッドデコレータが呼ばれました: ${str}`);

    descriptor.value = () => {
      console.log(`対象のメソッドが呼ばれました: ${str}`);
    };
  };
}

class SampleClass {
  str: string;
  constructor(str: string) {
    console.log(`コンストラクタが呼び出されました: ${str}`);
    this.str = str;
  }

  @methodDecoratorFactorySample("おはよう")
  printStr() {
    console.log(this.str);
  }
}

var sampleClass = new SampleClass("こんにちは");
sampleClass.printStr();
```

この状態で実行すると、以下のようになります。

```bash
# TypeScriptコンパイラをWatchモードで起動
npx tsc -w
# lite-serverを起動
npm start
```
![スクリーンショット 2022-09-22 19.46.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/121cb822-b79a-1f17-a183-0e177fe5177f.png)
デコレータファクトリ  
→デコレータ関数  
→コンストラクタ  
→対象メソッド

の順で実行されているように見えます。

## 書き方・使い方

メソッドのデコレータファクトリとデコレータですが、以下のように定義します。

```ts
// デコレータファクトリ（引数は何でも良い）
function methodDecoratorFactorySample(str: string) {
  // デコレータ関数を返す
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    // 何かする
  };
}
```

これを使う際は、以下のようにメソッドの上に`@デコレータファクトリ名(引数)`とします。
```ts
class SampleClass {
  // <中略>

  @methodDecoratorFactorySample("おはよう")
  printStr() {
    console.log(this.str);
  }
}
```

## 呼ばれるタイミング

デコレータファクトリと、デコレータファクトリが返すデコレータ関数は、<font color="red">対象のメソッドが定義されたタイミング</font>で呼ばれます。  
なので、↑の例でも、コンストラクタやメソッドの実行よりも前に実行されました。

## メソッドデコレータ関数の引数

メソッドデコレータ関数は、以下の引数を受け取ります。
- target: any,
- propertyKey: string,
- descriptor: PropertyDescriptor

これらを１つ１つ見てみました。

### target

とりあえず、以下のような感じで`target`をコンソール出力してみます。

```ts
function methodDecoratorFactorySample(str: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log(target);
  };
}
```

#### インスタンスメンバのメソッドの場合
コンソール出力の結果は以下です。
![スクリーンショット 2022-09-23 11.25.01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/0c1b53fe-dea7-72fc-b4c4-4fd515d646ce.png)
targetには、<font color="red">クラスのプロトタイプ</font>が入っています。

なので、例えば以下のようにメソッドデコレータ関数の中で、クラスのインスタンス化もできます（これが何に使えるかはさておき…）。
```diff_typescript
function methodDecoratorFactorySample(str: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log(target);
+   var tmp = new target.constructor("こんばんは");
+   tmp.printStr();
  };
}
```

![スクリーンショット 2022-09-23 11.47.47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/90421501-9089-9872-b61c-814fd34d5a3e.png)

#### staticメソッドの場合

まず、`SampleClass`にstaticメソッドを定義していなかったので、以下のように定義します。
分かりやすさのため、インスタンスメンバのメソッド（`printStr`）に付けていたデコレータは、一旦消しておきます。

```diff_typescript
class SampleClass {
  // <中略>

+ @methodDecoratorFactorySample("")
+ static printHello() {
+   console.log("Hello from static method");
+ }

- @methodDecoratorFactorySample("おはよう")
  printStr() {
    console.log(this.str);
  }
}
```
↑で記述したクラスのインスタンス化の部分も一旦消しておきます。
```diff_typescript
function methodDecoratorFactorySample(str: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log(target);
-   var tmp = new target.constructor("こんばんは");
-   tmp.printStr();
  };
}
```
この時のコンソール出力が以下です。
![スクリーンショット 2022-09-23 19.20.31.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/092c8f3e-58eb-d428-8f76-e9e35dc41b8d.png)
targetには、<font color="red">クラス（コンストラクタ関数）</font>が入っています。

なので、同様にクラスをインスタンス化する場合は、以下のようになります。


```diff_typescript
function methodDecoratorFactorySample(str: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log(target);
+   var tmp = new target("こんばんは");
+   tmp.printStr();
  };
}
```

### propertyKey

`propertyKey`をコンソールに出力します。

```diff_typescript
function methodDecoratorFactorySample(str: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log(propertyKey);
  };
}
```

#### インスタンスメンバのメソッドの場合
propertyKeyには、<font color="red">メソッド名</font>が入っています。
![スクリーンショット 2022-09-24 14.34.58.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/b3683d14-60f2-e2f5-5d73-ee152797bdd9.png)

#### staticメソッドの場合
こちらでも同様に、propertyKeyには、<font color="red">メソッド名</font>が入っています。
![スクリーンショット 2022-09-24 14.36.05.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/9f7e9494-5ee1-a646-6778-da7038f7e8db.png)

### descriptor
`descriptor`をコンソールに出力します。

```diff_typescript
function methodDecoratorFactorySample(str: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log(descriptor);
  };
}
```
#### インスタンスメンバのメソッドの場合
descriptorには、<font color="red">PropertyDescriptorのオブジェクト</font>が入っています。
![スクリーンショット 2022-09-24 14.39.44.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/19319fb3-8695-22f2-5cbf-72eb2b683bcd.png)
#### staticメソッドの場合
descriptorには、<font color="red">PropertyDescriptorのオブジェクト</font>が入っています。オブジェクトの内容は、インスタンスメンバのメソッドの場合と同じです。
![スクリーンショット 2022-09-24 14.42.22.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/797971bf-5aef-8fe9-d72a-43f352e0c9e2.png)

#### PropertyDescriptorオブジェクトの内容
メソッドデコレータにおける、PropertyDescriptorオブジェクトには以下の属性が含まれています。

- configurable
- enumerable
- value
- writable
- (get)
- (set)

以下のWebページを参考に、1つ1つ動作を確認してみます。

https://ja.javascript.info/property-descriptors

##### configurable

メソッドの場合、デフォルトでは`true`が設定されていました。

> configurable: false はプロパティフラグの変更や削除を禁止しますが、値を変更することは可能です

`プロパティの削除`ができなくなることを確認してみます。（「プロパティフラグの変更」については、確認方法が分かりませんでした…）

- まずは、`configurable: true`のパターン。

```diff_typescript
// デコレータファクトリ
function methodDecoratorFactorySample(str: string) {
  // デコレータ関数を返す
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    // descriptorの確認
    console.log(descriptor);
  };
}

class SampleClass {
  str: string;
  constructor(str: string) {
    console.log(`コンストラクタが呼び出されました: ${str}`);
    this.str = str;
  }

  @methodDecoratorFactorySample("instance method")
  printStr() {
    console.log(this.str);
  }
}

var sampleClass = new SampleClass("こんにちは");

+ // configurableの確認
+ console.log(
+   Object.getOwnPropertyDescriptor(
+     Object.getPrototypeOf(sampleClass),
+     "printStr"
+   )
+ );
+ // プロトタイプから、printStrメソッドを削除する
+ delete Object.getPrototypeOf(sampleClass).printStr;
+ sampleClass.printStr();
```
printStrメソッドの削除には成功し、メソッドが実行できなくなっていることが分かります。
![スクリーンショット 2022-09-24 15.44.50.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/a843aa7f-5fb7-587a-6b94-138625ee8cd2.png)

- 次に、`configurable: false`のパターン。

```diff_typescript
// デコレータファクトリ
function methodDecoratorFactorySample(str: string) {
  // デコレータ関数を返す
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    // descriptorの確認
    console.log(descriptor);
+   descriptor.configurable = false;
  };
}

class SampleClass {
  str: string;
  constructor(str: string) {
    console.log(`コンストラクタが呼び出されました: ${str}`);
    this.str = str;
  }

  @methodDecoratorFactorySample("instance method")
  printStr() {
    console.log(this.str);
  }
}

var sampleClass = new SampleClass("こんにちは");

// configurableの確認
console.log(
  Object.getOwnPropertyDescriptor(
    Object.getPrototypeOf(sampleClass),
    "printStr"
  )
);
// プロトタイプから、printStrメソッドを削除する
delete Object.getPrototypeOf(sampleClass).printStr;
sampleClass.printStr();
```
エラーメッセージの内容が、「printStrメソッドが削除できなかった」旨のもの変わっています。`configurable: false`の場合は、メソッドを削除できないことが確認できました。
![スクリーンショット 2022-09-24 15.47.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/f22754fd-9c3a-ab22-9669-b0d0088e22f7.png)

##### enumerable

メソッドの場合、デフォルトでは`false`が設定されていました。

オブジェクトの要素として列挙可能であるかどうかを設定できる項目になります。
以下のように、SampleClassのインスタンスを`for..in`に渡して、要素を1つ1つ出力することで動作を確認してみます。

```diff_typescript
var sampleClass = new SampleClass("こんにちは");

// PropertyDescriptorの出力（デコレータで変更された後の値確認）
console.log(
  Object.getOwnPropertyDescriptor(
    Object.getPrototypeOf(sampleClass),
    "printStr"
  )
);

// enumerableの確認
+for (var key in sampleClass) {
+  console.log(`要素の列挙：${key}`);
+}

```

- `enumerable: false`のパターン。

`printStr`メソッドは列挙されていません。
![スクリーンショット 2022-09-25 18.38.14.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/546014c1-d60d-f2dd-2a6d-553f85d4538a.png)

- `enumerable: true`のパターン。

デコレータ関数内で、`enumerable`を`true`に設定します。

```diff_typescript
// デコレータファクトリ
function methodDecoratorFactorySample(str: string) {
  // デコレータ関数を返す
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    // -- descriptorの確認 --
    console.log(descriptor);
+   // - enumerableの確認
+   descriptor.enumerable = true;
  };
}
```
`printStr`メソッドが列挙されました。
![スクリーンショット 2022-09-25 18.38.36.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/ebce8c4b-7fb2-127a-b8f8-f11f387731fa.png)

##### value

`value`には、対象のメソッドが入っています。
デコレータ内で、メソッド実行とメソッドの書き換えを試してみます。

```diff_typescript
// デコレータファクトリ
function methodDecoratorFactorySample(str: string) {
  // デコレータ関数を返す
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    // -- descriptorの確認 --
    console.log(descriptor);
+   // - valueの確認
+   descriptor.value();
+   // -- メソッドの書き換え --
+   descriptor.value = () => {
+     console.log(`対象のメソッドが呼ばれました: ${str}`);
+   };
  };
}

// サンプルクラス
class SampleClass {
  str: string;
  constructor(str: string) {
    console.log(`コンストラクタが呼び出されました: ${str}`);
    this.str = str;
  }

  //@methodDecoratorFactorySample("static method")
  static printHello() {
    console.log("Hello from static method");
  }

  @methodDecoratorFactorySample("instance method")
  printStr() {
    console.log(this.str);
  }
}

var sampleClass = new SampleClass("こんにちは");

// PropertyDescriptorの出力（デコレータで変更された後の値確認）
console.log(
  Object.getOwnPropertyDescriptor(
    Object.getPrototypeOf(sampleClass),
    "printStr"
  )
);

+// valueの確認
+sampleClass.printStr();
```
実行結果が以下になります。
メソッドの実行については、デコレータ関数の実行時点で`this.str`が定義されていないため、`undefined`となっています。
また、メソッドの書き換えができていることも確認できました。
![スクリーンショット 2022-09-25 18.57.52.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/d0223e09-7f49-14aa-611f-2e1f02abfc0a.png)

##### writable
メソッドの場合、デフォルトでは`true`が設定されていました。
書き込み（再代入）を許可するかどうかを設定することができます。

実際に試してみます。

- `writable: true`のパターン

`printStr`メソッドに、メソッドの再代入を行ってみます。
```diff_typescript
var sampleClass = new SampleClass("こんにちは");

// PropertyDescriptorの出力（デコレータで変更された後の値確認）
console.log(
  Object.getOwnPropertyDescriptor(
    Object.getPrototypeOf(sampleClass),
    "printStr"
  )
);

+// writableの確認（メソッドの内容変更）
+sampleClass.printStr = () => {
+  console.log("changed");
+};
+sampleClass.printStr();
```
問題なく、メソッドの再代入が行われています。
![スクリーンショット 2022-09-25 22.13.08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/66a4e26a-9e90-92a6-8be6-fabc0dae31c5.png)

- `writable:false`のパターン

デコレータ関数内で、`writable`に`false`を設定します。

```diff_typescript
// デコレータファクトリ
function methodDecoratorFactorySample(str: string) {
  // デコレータ関数を返す
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    // -- descriptorの確認 --
    console.log(descriptor);
+   // - writableの確認
+   descriptor.writable = false;
  };
}

// サンプルクラス
class SampleClass {
  str: string;
  constructor(str: string) {
    console.log(`コンストラクタが呼び出されました: ${str}`);
    this.str = str;
  }

  //@methodDecoratorFactorySample("static method")
  static printHello() {
    console.log("Hello from static method");
  }

  @methodDecoratorFactorySample("instance method")
  printStr() {
    console.log(this.str);
  }
}

var sampleClass = new SampleClass("こんにちは");

// PropertyDescriptorの出力（デコレータで変更された後の値確認）
console.log(
  Object.getOwnPropertyDescriptor(
    Object.getPrototypeOf(sampleClass),
    "printStr"
  )
);

// writableの確認（メソッドの内容変更）
sampleClass.printStr = () => {
  console.log("changed");
};
sampleClass.printStr();
```
この場合は、メソッドの再代入を行うことはできませんでした。`printStr`メソッドは`read only property`となりました。
![スクリーンショット 2022-09-25 22.11.09.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/11eebeb5-711c-b56d-399d-b7d3542880b8.png)

##### get

対象メソッドのgetterです。PropertyDescriptorオブジェクトの中で、`value`および`writable`属性とは同居できません。

以下、実際に試してみます。
これまでは引数のdescriptorを直接変更していましたが、`get`は`value`・`writable`属性と同居できないので、新しくPropertyDescriptorオブジェクトを生成して、メソッドデコレータ関数のreturn値としています（メソッドデコレータ関数は、PropertyDescriptorオブジェクトをreturnすることで対象メソッドのPropertyDescriptorを上書きできます）。

```diff_typescript
// デコレータファクトリ
function methodDecoratorFactorySample(str: string) {
  // デコレータ関数を返す
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    // -- descriptorの確認 --
    console.log(descriptor);
+   // - getの確認
+   var retDesc = {
+     configurable: true,
+     enumerable: false,
+     get() {
+       console.log("getter called");
+       return descriptor.value;
+     },
+   };
+   return retDesc;
  };
}

// サンプルクラス
class SampleClass {
  str: string;
  constructor(str: string) {
    console.log(`コンストラクタが呼び出されました: ${str}`);
    this.str = str;
  }

  //@methodDecoratorFactorySample("static method")
  static printHello() {
    console.log("Hello from static method");
  }

  @methodDecoratorFactorySample("instance method")
  printStr() {
    console.log(this.str);
  }
}

var sampleClass = new SampleClass("こんにちは");

// PropertyDescriptorの出力（デコレータで変更された後の値確認）
console.log(
  Object.getOwnPropertyDescriptor(
    Object.getPrototypeOf(sampleClass),
    "printStr"
  )
);

+// getの確認
+sampleClass.printStr();
```
結果です。
`printStr`メソッドの実行時に、設定したgetterが呼ばれたことが分かります。
![スクリーンショット 2022-09-26 0.06.19.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/d30f0475-0621-6039-1b63-d6873bbc3bef.png)

##### set

対象メソッドのsetterです。getと同じく、PropertyDescriptorオブジェクトの中で、`value`および`writable`属性とは同居できません。

以下、実際に試してみます。

```diff_typescript
// デコレータファクトリ
function methodDecoratorFactorySample(str: string) {
  // デコレータ関数を返す
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    // -- descriptorの確認 --
    console.log(descriptor);
+   // - get・setの確認
+   var orgMethod = descriptor.value;
    var retDesc = {
      configurable: true,
      enumerable: false,
      get() {
        console.log("getter called");
+       return orgMethod;
      },
+     set(value: any) {
+       console.log("setter called");
+       orgMethod = value;
      },
    };
    return retDesc;
  };
}

// サンプルクラス
class SampleClass {
  str: string;
  constructor(str: string) {
    console.log(`コンストラクタが呼び出されました: ${str}`);
    this.str = str;
  }

  //@methodDecoratorFactorySample("static method")
  static printHello() {
    console.log("Hello from static method");
  }

@methodDecoratorFactorySample("instance method")
  printStr() {
    console.log(this.str);
  }
}

var sampleClass = new SampleClass("こんにちは");

// PropertyDescriptorの出力（デコレータで変更された後の値確認）
console.log(
  Object.getOwnPropertyDescriptor(
    Object.getPrototypeOf(sampleClass),
    "printStr"
  )
);

+// setの確認
+sampleClass.printStr = () => {
+  console.log("changed by setter");
+};
+sampleClass.printStr();
```
結果です。
setterが呼び出され、`printStr`メソッドが書き変わったことが分かります。
![スクリーンショット 2022-09-26 0.37.49.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/898e4b7f-3d59-d39e-013b-d6093c86fd5f.png)

# まとめ
メソッドデコレータ関数の引数をまとめます。

<table>
  <thead>
    <tr>
      <th>引数名</th> <th>インスタンスメンバのメソッドの場合</th> <th>staticメソッドの場合</th>
    </tr>
  </thead>
  <tr>
    <th> target </th> <td>クラスのプロトタイプ</td> <td>クラス（コンストラクタ関数）</td>
  </tr>
  <tr>
    <th> propertyKey </th> <td>メソッド名</td> <td>メソッド名</td>
  </tr>
  <tr>
    <th> descriptor </th> <td>PropertyDescriptorのオブジェクト</td> <td>PropertyDescriptorのオブジェクト</td>
  </tr>
</table>

メソッドデコレータ関数は、PropertyDescripterオブジェクトをreturnすることで、対象メソッドのPropertyDescriptorを上書きすることができました。

PropertyDescriptorオブジェクトの属性をまとめます。

<table>
  <thead>
    <tr>
      <th>属性名</th> <th>説明</th>
    </tr>
  </thead>
  <tr>
    <th> configurable </th> <td>プロパティフラグの変更やプロパティの削除を許可するか否か</td>
  </tr>
  <tr>
    <th> enumerable </th> <td>プロパティを列挙可能にするかどうか</td>
  </tr>
  <tr>
    <th> value </th> <td>対象のメソッドそのもの</td>
  </tr>
  <tr>
    <th> writable </th> <td>プロパティの値書き込み（再代入）を許可するかどうか</td>
  </tr>
  <tr>
    <th> get </th> <td>対象メソッドのgetter（value, writableとは同居できない）</td>
  </tr>
  <tr>
    <th> set </th> <td>対象メソッドのsetter（value, writableとは同居できない）</td>
  </tr>
</table>

本記事のサンプルコードはGitHubに上げています。

https://github.com/plumchang/typescript_decorator_sample

以上。
