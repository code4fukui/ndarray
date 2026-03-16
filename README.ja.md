# ndarray

Javascriptの多次元配列ライブラリ。

## 機能
- 1次元配列から多次元配列のビューを作成可能
- 定数時間での切り取り、転置、リサイズに対応
- Node.jsやブラウザでの利用が可能
- 様々なデータ型に対応（typed array、汎用データストア）

## 必要環境
Node.js または ブラウザ

## 使い方
インストール:
```
npm install ndarray
```

サンプルコード:
```javascript
import ndarray from "ndarray";

var mat = ndarray(new Float64Array([1, 0, 0, 1]), [2,2])
// 2x2の配列が作成されます
```

## ライセンス
このプロジェクトは [MIT License](LICENSE) のもとで公開されています。
