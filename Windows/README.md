# Windows

## Windows環境で複数のSQLファイルを1ファイルにまとめる

```sh
copy /b *.sql merged.sql
```

## 環境変数を最短で開く方法

1. Win + R を押す(ファイル名を指定して実行)
2. 「SystemPropertiesAdvanced」を入力してEnter
3. 「システムのプロパティ」が開かれる
4. 下にある「環境変数(N)...」をクリック

## よく開くフォルダを環境変数に登録して簡単に開けるようにする

1. 「環境変数」を開く
2. ユーザー環境変数に新規追加
   - 変数名：playground
   - 変数値：C:\Users\ユーザー名\Playground
3. 「ファイル名を指定して実行」で「%playground%」を入力してEnter
4. 指定の変数値のフォルダがエクスプローラで開かれる

ただし playground 単体ではなく %playground% が必要
