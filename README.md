# VHTML
HTML extension for allowing more visual based styling

## 概要
扱いやすく、見た目重視のスタイリングがしやすいように拡張されたHTMLです。拡張子：.vhml

現在検討中の見た目からセマンティクスを生成するライブラリ、VisArcheとの連携を前提に作成していますが、Tailwindなどへの変換も可能だと思います。

## 例

```js
// setting.json 複数ファイルで使いまわせるようにしたい
{
  "lang":"ja", // <html lang="ja">を生成
  "charset":"utf-8", // <meta>要素は名前から書ける
  "title":"{myTitle} - VHMLサンプル",
  "author":"永山俊明",
  "content-security-policy":"default-src https:",// http-equivもnameも同時に記載可能
  "description":"プレースホルダー"
  "og": { // <meta name="og:image"> などに自動変換する
          "image":"https://developer.mozilla.org/static/img/opengraph-logo.png",
          "description":"VHMLは扱いやすく、見た目重視のスタイリングがしやすいように拡張されたHTMLです。"
        },
  "shortcut icon": { // <link rel="shortcut icon">として展開
                      "source":"favicon.ico",
                      "type":"image/x-icon"
                   },
  "scripts": {
                "normal.js", // このように書いた場合は通常のJSとしてロードされる
                {
                  "source":"defer.js", // src, hrefもエイリアスとして認めるが、使い分けの問題があるのでsourceのほうが良いと思う
                  "load-type":"defer"
                },
                {
                  "source":"preload.js",
                  "load-type":"preload" // このように書くと自動で<link rel="preload" href="preload.js" as="script">と展開される
                },
             },
  "script-critical":"critical.js", // クリティカルなJSの場合は入れる個所が重要なので、このように書くことにする
  "styles": {
              "base.css", // このように書いた場合は普通の<link>としてロードされる
              {
                "source":"screen.css", // src, hrefもエイリアスとして認めるが、おそらくsourceのほうが分かりやすいと思う
                "media":"screen"
              },
              {
                "source":"print.css",
                "media":"print"
              },
              {
                "source":"noncritical.css",
                "media":"screen",
                "load-type":"preload"
              },
              {
                "source":"critical.css",
                "media":"screen",
                "load-type":"critical" // このように書くとクリティカルCSSとして<style>にインライン展開する
              },
            },
  "script-critical":"critical2.js", // クリティカルJSの場合は挿入箇所を尊重する
}
```

```vhml
<setting external source="setting.json" myTitle="トップ"> // 主に<head>内の情報をJSON形式で記述できる。内部に入れられるようにもしたいので、externalを付けることで外部化する
{
  "description":"VHMLのサンプルページです。", // "scripts", "styles"を除き上書き可能
  "add" : {
            "as":"scripts"
            "where":"scripts:last-of-type::after" // CSSセレクタ風に記載可能
            "data": "top.js"
          }
}
</> // 閉じ括弧の要素名を省略できる
<heading>VHMLサンプル</> // トランスパイル時に自動的にランクを決定する。最初の<heading>はあらゆる指定に関わりなく<h1>に、以降は一番最初がネストに関わらず<h2>、以降はネストが深くなると自動的にランクが下がるようになる
<heading>サブ見出し</>
<heading rank="+1">さらにサブ見出し</> // 明示的にランクを下げることが可能（直前のheadingから相対的に決定。プラスになる）
<p style="margin: 0 0 2em;" style-before"width: 1em; height: 1em; background-image:'exclamation.svg'">注意：このコードはブラウザには読めません。</> // style-*として疑似要素の記載が可能。VisArcheの自動変換でCSSコード生成時に使える。
```

はVisArcheを用いて、例えば以下の様に変換される。

```html
<html lang="ja">
  <head>
    <meta charset="utf-8">
    <title>トップ - VHMLサンプル</title>
    <meta name="author" value="永山俊明">
    <meta http-equiv="content-security-policy" value="default-src https:">
    <meta name="description" value="VHMLのサンプルページです。">
    <meta name="og:image" value="https://developer.mozilla.org/static/img/opengraph-logo.png">
    <meta name="og:description" value="VHMLは扱いやすく、見た目重視のスタイリングがしやすいように拡張されたHTMLです。">
    <link rel="shortcut icon" href="favicon.ico" type="image/x-icon">
    <script src="modal.js"></script>
    <script src="defer.js" defer></script>
    <script src="top.js"></script>
    <link rel="preload" href="preload.js" as="script">
    <script>/* critical.jsの中身 */</script>
    <link rel="stylesheet" href="base.css">
    <link rel="stylesheet" href="screen.css" media="screen">
    <link rel="stylesheet" href="print.css" media="print">
    <link rel="preload" href="noncritical.css" media="screen" as="style">
    <style>/* critical.cssの中身 */</style>
    <script>/* critical2.jsの中身 */</script>
  </head>
  <body>
    <h1>VHMLサンプル</h1>
    <h2>サブ見出し</h2>
    <h3>さらにサブ見出し</h3>
    <p class="beetle">注意：このコードはブラウザには読めません。</> // クラス名はVisArcheの初期生成のままとする。
  </body>
</html>
```

``` css
/* base.css */
.beetle {
  margin: 0 0 2em;"
}

.beetle::before {
  content: "",/* style-beforeのcontentを省略するとデフォルト値を入れる。正直現在のcontent入力必須というのは面倒なだけだと思う */
  width: 1em;
  height: 1em;
  background-image:'exclamation.svg'
}
```
