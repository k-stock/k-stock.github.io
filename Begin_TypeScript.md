# for beginning

```bash
npm init -y
npm install --save-dev typescript eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
npx tsc --init
npx eslint --init
npm install --save-dev gts
npx gts init
code .
```

.vscode/settings.json:
```json
{
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
        "source.fixAll": true,
        "source.fixAll.eslint": true
    },
    "eslint.validate": [
        "typescript"
    ]
}
```

tsconfig.json:
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "allowSyntheticDefaultImports": true,
    "noImplicitAny": true
  }
}
```

# 型定義がない問題


✅ 1. DefinitelyTyped の @types/ パッケージを探す
📦 原則：npm にある @types/ プレフィックスの型パッケージを使う

npm install --save-dev @types/lodash

    lodash, express, moment など多くの有名ライブラリはこれで解決します。

    公式サイト: https://definitelytyped.org/

    💡 npm install <ライブラリ名> のあとに npm install --save-dev @types/<ライブラリ名> を試すのが定石です。

✅ 2. 型が付属しているライブラリを優先的に使う

最近は多くのライブラリが 型を同梱（types フィールドを package.json に含む）しています。
✔ 見分け方：

"types": "./dist/index.d.ts"

が package.json に含まれているライブラリは型定義が付属しています。
✅ 3. 自作型ファイルを global.d.ts にまとめる

もし型が存在しない・非公式型もない場合は、自分で global.d.ts に簡易的な型を書くこともできます。
例：src/global.d.ts

declare module 'some-legacy-lib' {
  const whatever: any;
  export = whatever;
}

📍 TIPS:

    こうすることで最低限エラーは消える。

    本格的に使う場合は、実際の構造を元に型を少しずつ強くしていくのがおすすめ。

✅ 4. --noImplicitAny を一時的に緩める（非推奨）

最終手段として、tsconfig.json の noImplicitAny を false にすれば「型がないもの」も使えます。

{
  "compilerOptions": {
    "noImplicitAny": false
  }
}

ただし、これは 型安全性を大きく損なうため、なるべく使わないようにしましょう。
✅ 5. よく使う型がないライブラリを型付きの代替に置き換える

例：
ライブラリ	代替（型付き）
uuid（旧v3）	uuid v8+（型付き）
axios-mock-adapter	msw（TypeScriptサポート強力）

「型定義がない」こと自体がそのライブラリのメンテ状況を示すこともあるので、型がある代替ライブラリを探すのも賢い選択です。
✅ 6. 補助ツールで自動型生成（上級者向け）
🔧 ツール例：

    dts-gen: 簡易的な .d.ts ファイルを自動生成

    ts-migrate: JS → TS 変換時に型注釈を自動で推測

💡 結論：現実的な対策まとめ
目的	対策
よく使うOSSなら	@types を探す
可能なら	型付きライブラリを選ぶ
自作でもOKなら	global.d.ts で型を書く
とりあえず使いたい	any 型や noImplicitAny を一時的に緩める（要注意）

# 型定義の自動作成

✅ 方法1：dts-gen で型定義ファイルを自動生成（基本）
📦 インストール

npm install -g dts-gen

またはローカルに：

npm install --save-dev dts-gen

🛠 使い方

dts-gen -m ライブラリ名

例：

dts-gen -m some-legacy-lib

これで以下のような some-legacy-lib.d.ts が生成されます。

declare module "some-legacy-lib" {
  export function example(): any;
}

📁 配置場所

types/ など任意の場所に配置し、tsconfig.json に以下を追加：

{
  "compilerOptions": {
    "typeRoots": ["./node_modules/@types", "./types"]
  }
}

✅ 方法2：ts-migrate でJS → TS自動変換（Airbnb製）

Airbnb が開発した移行支援ツールで、JavaScript コードを TypeScript に変換しつつ、型情報を補完してくれます。
📦 インストール

npx ts-migrate-full <your-js-project>

特徴：

    JavaScript ファイルを .ts に変換

    推論できる範囲で型を自動付与（すべてではない）

    any や TODO: 型も含まれるので、部分的に補完しながら使える

対象がライブラリ本体でなくプロジェクト内コードなら便利です。
✅ 方法3：VSCode / TypeScript Language Server に型を推論させる（手動補完）

    TypeScript は .js ファイルの JSDoc コメントを元に型を推論できます。

    ライブラリのソースコードが読めるなら、VSCode 上で hover するだけでも補完される場合があります。

✅ 方法4：JSDoc コメントから .d.ts を生成（ツール：tsc）

もし拾ってきた JavaScript ライブラリが JSDoc コメント付きなら、TypeScript のコンパイラ（tsc）で .d.ts を生成できます。
📦 tsconfig.json に追加：

{
  "compilerOptions": {
    "allowJs": true,
    "declaration": true,
    "emitDeclarationOnly": true,
    "outDir": "./types"
  },
  "include": ["lib/**/*.js"]
}

npx tsc

これで lib/ 以下の .js ファイルから .d.ts が生成されます。
✅ 使い分けガイド
ライブラリの状態	推奨ツール・方法
npmパッケージだが型がない	dts-gen
ソースコードがJSで手元にある	ts-migrate or tsc + JSDoc
最低限anyで使いたい	手動で global.d.ts
しっかり型定義したい	JSDoc → tsc or 自作 .d.ts
💡 例：型がないライブラリ some-legacy-lib を使いたい

    npm install some-legacy-lib

    npx dts-gen -m some-legacy-lib

    生成された some-legacy-lib.d.ts を types/ に配置

    tsconfig.json に typeRoots でそのフォルダを追加

🎁 補足：最小限の型で回避する手書き方法

// global.d.ts
declare module 'some-legacy-lib' {
  const whatever: any;
  export = whatever;
}

