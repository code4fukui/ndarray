ndarray
=======
JavaScript用のモジュール式多次元配列。

[![browser support](https://ci.testling.com/mikolalysenko/ndarray.png)
](https://ci.testling.com/mikolalysenko/ndarray)

[![build status](https://secure.travis-ci.org/scijs/ndarray.png)](http://travis-ci.org/scijs/ndarray)

[![stable](https://rawgithub.com/hughsk/stability-badges/master/dist/frozen.svg)](http://github.com/hughsk/stability-badges)

##### [scijsドキュメント](http://scijs.net/packages)で多数のndarray互換モジュールを閲覧する
##### MATLABやnumpyから移行される方へ: [MATLABユーザー向けのscijs/ndarray](https://github.com/scijs/scijs-ndarray-for-matlab-users)をご覧ください
##### [ndarrayモジュールの膨大なリスト](https://github.com/scijs/ndarray/wiki/ndarray-module-list#core-module)

Introduction
============
`ndarrays`は、1次元配列のより高次元なビューを提供します。例えば、長さ4の型付き配列をnd-arrayに変換する方法は以下の通りです：

```javascript
var mat = ndarray(new Float64Array([1, 0, 0, 1]), [2,2])

//Now:
//
// mat = 1 0
//       0 1
//
```

nd-arrayを作成すると、`.set`と`.get`を使用して要素にアクセスできます。例えば、ndarrayを使用した[コンウェイのライフゲーム](http://en.wikipedia.org/wiki/Conway's_Game_of_Life)の実装は以下のようになります：

```javascript
function stepLife(next_state, cur_state) {

  //Get array shape
  var nx = cur_state.shape[0], 
      ny = cur_state.shape[1]

  //Loop over all cells
  for(var i=1; i<nx-1; ++i) {
    for(var j=1; j<ny-1; ++j) {

      //Count neighbors
      var n = 0
      for(var dx=-1; dx<=1; ++dx) {
        for(var dy=-1; dy<=1; ++dy) {
          if(dx === 0 && dy === 0) {
            continue
          }
          n += cur_state.get(i+dx, j+dy)
        }
      }
      
      //Update state according to rule
      if(n === 3 || n === 3 + cur_state.get(i,j)) {
        next_state.set(i,j,1)
      } else {
        next_state.set(i,j,0)
      }
    }
  }
}
```

また、基となる要素をコピーすることなく、ndarrayのビューを取り出すこともできます。以下は、部分配列の一部を更新する方法を示す例です：

```javascript
var x = ndarray(new Float32Array(25), [5, 5])
var y = x.hi(4,4).lo(1,1)

for(var i=0; i<y.shape[0]; ++i) {
  for(var j=0; j<y.shape[1]; ++j) {
    y.set(i,j,1)
  }
}

//Now:
//    x = 0 0 0 0 0
//        0 1 1 1 0
//        0 1 1 1 0
//        0 1 1 1 0
//        0 0 0 0 0
```

ndarrayは、操作ごとに定数時間で転置、反転、せん断、スライスが可能です。画像、音声、ボリュームグラフィックス、行列、文字列などを表現するのに役立ちます。node.jsと[browserify](http://browserify.org/)の両方で動作します。

Install
=======
[npm](http://npmjs.org)を使用してライブラリをインストールします：

```sh
npm install ndarray
```

CommonJS/nodeモジュールの規約に従うツールを使用すれば、ブラウザでndarrayを使用することもできます。これを行う最も直接的な方法は、[browserify](https://github.com/substack/node-browserify)を使用することです。より高速なデバッグのためにライブリロードが必要な場合は、[beefy](https://github.com/chrisdickinson/beefy)をチェックしてください。

API
===
ndarrayをインストールしたら、プロジェクトで次のように使用できます：

```javascript
var ndarray = require("ndarray")
```

## コンストラクタ

### `ndarray(data[, shape, stride, offset])`
デフォルトの`module.exports`メソッドは、ndarrayのコンストラクタです。これは、基となるストレージ型をラップするn次元配列ビューを作成します。

* `data` は1次元配列のストレージです。`Array`のインスタンス、型付き配列、または`get()`、`set()`、`.length`を実装するオブジェクトのいずれかです。
* `shape` はビューの形状です（デフォルト: `data.length`）。
* `stride` は新しい配列のストライドです。（デフォルト: 行優先）
* `offset` はビューを開始するオフセットです（デフォルト: `0`）。

**戻り値** バッファのn次元配列ビュー

## メンバー

`ndarray`の中心的な概念は、ビューという考え方です。これらの仕組みは[SciPyの配列スライス](http://docs.scipy.org/doc/numpy/reference/arrays.indexing.html)に非常に似ています。ビューは、1次元ストレージ型へのアフィン射影です。これが何を意味するのかをよりよく理解するために、まずビューオブジェクトのプロパティを見てみましょう。これには正確に4つの変数が含まれています：

* `array.data` - 多次元配列の基となる1次元ストレージ
* `array.shape` - 型付き配列の形状
* `array.stride` - メモリ内の型付き配列のレイアウト
* `array.offset` - メモリ内の配列の開始オフセット

ストライドを個別に保持することで、同じデータ構造を使用して[行優先および列優先のストレージ](http://en.wikipedia.org/wiki/Row-major_order)の両方をサポートできます。

## 要素へのアクセス
配列の要素にアクセスするには、`set/get`メソッドを使用できます：

### `array.get(i,j,...)`
配列から要素`i,j,...`を取得します。擬似コードでは、これは次のように実装されます：

```javascript
function get(i,j,...) {
  return this.data[this.offset + this.stride[0] * i + this.stride[1] * j + ... ]
}
```

### `array.set(i,j,...,v)`
要素`i,j,...`を`v`に設定します。これも擬似コードでは次のように機能します：

```javascript
function set(i,j,...,v) {
  return this.data[this.offset + this.stride[0] * i + this.stride[1] * j + ... ] = v
}
```

### `array.index(i,j, ...)`
基となるndarray内のセルのインデックスを取得します。JSでは、

```javascript
function index(i,j, ...) {
  return this.offset + this.stride[0] * i + this.stride[1] * j + ...
}
```

## プロパティ
以下のプロパティはObject.definePropertyを使用して作成され、物理メモリを消費しません。これらはndarrayを含む計算で役立ちます。

### `array.dtype`
ndarrayの基となるデータ型を表す文字列を返します。汎用データストアを除き、これらの型は[`typedarray-pool`](https://github.com/mikolalysenko/typedarray-pool)と互換性があります。これは以下のルールに従ってマッピングされます：

Data type | String
--------: | :-----
`Int8Array` | "int8"
`Int16Array` | "int16"
`Int32Array` | "int32"
`Uint8Array` | "uint8"
`Uint16Array` | "uint16"
`Uint32Array` | "uint32"
`BigInt64Array` | "bigint64"
`BigUint64Array` | "biguint64"
`Float32Array` | "float32"
`Float64Array` | "float64"
`Array` | "array"
`Uint8ArrayClamped` | "uint8_clamped"
`Buffer` | "buffer"
Other | "generic"

汎用配列（Generic arrays）は、配列アクセサの代わりにget()/set()を使用して、基となる1次元ストアの要素にアクセスします。

### `array.size`
論理要素数での配列のサイズを返します。

### `array.order`
配列のストライドの順序を、長さの昇順でソートして返します。最初の要素は最も短いストライドの最初のインデックスであり、最後の要素は最も長いストライドのインデックスです。

### `array.dimension`
配列の次元数を返します。

## スライス
ビューが与えられた場合、ストライドをシフト、切り詰め、または置換することでインデックス付けを変更できます。これにより、配列の反転や行列の転置などの操作を**定数時間**で実行できます（厳密には`O(shape.length)`ですが、通常shape.lengthは4未満であるため、実質的に定数時間と言えます）。操作を簡単にするために、以下のインターフェースが公開されています：

### `array.lo(i,j,k,...)`
これは配列のシフトされたビューを作成します。画像の左上隅を`(i,j,k...)`の量だけ内側にドラッグするようなものだと考えてください。

### `array.hi(i,j,k,...)`
これは`array.lo()`の双対を行います。左上からシフトする代わりに、配列の右下から切り詰め、より小さな配列オブジェクトを返します。`hi`と`lo`を組み合わせて使用することで、配列の中央の範囲を選択できます。

**注意:** `hi`と`lo`は交換可能ではありません。一般的に：

```javascript
a.hi(3,3).lo(3,3)  !=  a.lo(3,3).hi(3,3)
```

### `array.step(i,j,k...)`
スケーリングによってストライド長を変更します。負のインデックスは軸を反転させます。例えば、1次元配列の反転ビューを作成する方法は以下の通りです：

```javascript
var reversed = a.step(-1)
```

必要に応じてステップサイズを1より大きく変更し、リストのエントリをスキップすることもできます。例えば、配列を偶数と奇数のコンポーネントに分割する方法は以下の通りです：

```javascript
var evens = a.step(2)
var odds = a.lo(1).step(2)
```

### `array.transpose(p0, p1, ...)`
最後に、高次元配列の場合、データを複製することなくインデックスを転置できます。これは、形状とストライドの値を置換し、その結果を同じデータの新しいビューに配置する効果があります。例えば、2次元配列では、次のように行列の転置を計算できます：

```javascript
M.transpose(1, 0)
```

または、3Dボリューム画像がある場合、より一般的な変換を使用して軸をシフトできます：

```javascript
volume.transpose(2, 0, 1)
```

### `array.pick(p0, p1, ...)`
特定の軸を固定することで、ndarrayから部分配列を取り出すこともできます。これは、値のリストを指定して取り出す方向を指定することで機能します。例えば、nxmx3の配列として保存された画像がある場合、次のようにチャンネルを取り出すことができます：

```javascript
var red   = image.pick(null, null, 0)
var green = image.pick(null, null, 1)
var blue  = image.pick(null, null, 2)
```

上記の例が示すように、pickの座標に負の値または非数値を渡すと、そのインデックスはスキップされます。

# 詳細情報

ndarrayに関するさらなる議論については、以下のトーク、チュートリアル、記事をご覧ください：

* [ndarrayのプレゼンテーション](http://mikolalysenko.github.io/ndarray-presentation/)
* [JavaScriptでの多次元配列の実装](http://0fps.wordpress.com/2013/05/22/implementing-multidimensional-arrays-in-javascript/)
* [キャッシュ非依存の配列操作](http://0fps.wordpress.com/2013/05/28/cache-oblivious-array-operations/)
* [いくつかの実験](https://github.com/mikolalysenko/ndarray-experiments)

ライセンス
=======
(c) 2013-2016 Mikola Lysenko. MIT License
