---
title: "改めてSvelteに入門してみる"
emoji: "🕺🏿"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['Svelte']
published: false
---

::: message alert
2024年5月時点でlatestなバージョン(svelte@4.2.17)の情報を掲載しています。
:::


Svelteについては少しチュートリアルを触ったぐらいなのでちょっとずつdeep diveして行きたいと思います。

公式ドキュメントとGoogle Gemini Advancedさんの力を借り執筆しています、間違っているところがあれば教えてください！🫠


# Svelteってなに？
ReactやVueと同じようなフロントエンドフレームワークです。

ReactやVueでは仮想DOMを用いてリアクティブなUIを実現していますが、Svelteでは**仮想DOMの管理をしなくてよい、仮想DOMとリアルDOMの差分を検出する処理が必要ない**ように設計がなされています。

なのでバンドルサイズやメモリ使用率などが最軽量クラスになっています。



# Svelteのはじめ方

Svelteにdiveしましょう！
今回はせっかくなので(?)[Bun](https://bun.sh/docs/installation#installing)を使ってはじめていきます！


## プロジェクトの初期化

まずはプロジェクトを初期化していきましょう！

::: message
公式ドキュメントの通りに`bun create svelte@latest PROJECT_NAME`のようなコマンド実行するとSvelteKitが初期化されるので注意が必要です。
:::

Svelte単体で開発するためにはViteを使うしかないようです・・・！
なのでViteプロジェクトを作成します。

```zsh
$ bun create vite@latest
```

このコマンドを実行したらプロジェクト名、TypeScriptを使うかどうか、どのフレームワークを使うのかを聞かれるので下記のように選択しましよう。

下記のようになるはずです！

```zsh
✔ Project name: … quickstart-svelte
✔ Select a framework: › Svelte
✔ Select a variant: › TypeScript

Scaffolding project in /Users/USER_NAME/code/quickstart-svelte...

Done. Now run:

  cd quickstart-svelte
  bun install
  bun run dev
```


## 実際にファイルを覗いてみる

まずどのような構成になっているのか確認するため`App.svelte`を覗いてみましょう。

::: details App.svelte

```svelte
<script lang="ts">
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from '/vite.svg'
  import Counter from './lib/Counter.svelte'
</script>

<main>
  <div>
    <a href="https://vitejs.dev" target="_blank" rel="noreferrer">
      <img src={viteLogo} class="logo" alt="Vite Logo" />
    </a>
    <a href="https://svelte.dev" target="_blank" rel="noreferrer">
      <img src={svelteLogo} class="logo svelte" alt="Svelte Logo" />
    </a>
  </div>
  <h1>Vite + Svelte</h1>

  <div class="card">
    <Counter />
  </div>

  <p>
    Check out <a href="https://github.com/sveltejs/kit#readme" target="_blank" rel="noreferrer">SvelteKit</a>, the official Svelte app framework powered by Vite!
  </p>

  <p class="read-the-docs">
    Click on the Vite and Svelte logos to learn more
  </p>
</main>

<style>
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
    transition: filter 300ms;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
  .logo.svelte:hover {
    filter: drop-shadow(0 0 2em #ff3e00aa);
  }
  .read-the-docs {
    color: #888;
  }
</style>
```
:::


一見すると非常にVueの書き方に似ていると思いますが、VueではHTMLを拡張したtemplate内に記述しますがSvelteではtemplateのような特別なものはなくHTMLを記述していきます。
加えて、制御フローなどはVueにある`v-if`のような形ではなく`{#if 条件式}...{/if}`のように記述します。

HTMLブロックについては次回以降書きます。


大雑把にいうと以下のようになります。
これを踏まえてもう少しdeep diveしてみます。

```svelte
<script>
// ロジックを記述
</script>

<!-- HTMLを記述 -->

<style>
/* styleを記述 */
</style>
```

では続いては各ブロックについてdeep diveしていきます。
<br/>

# script　ブロック内でできること

変数、関数の定義を行います。
リアクティブな変数（`let`での定義、`$:`ではじまるリアクティブ宣言）をトップレベルで宣言するとコンポーネントがその変数をウォッチしてくれて、よしなに画面の更新などを行ってくれます。

大雑把にいうとこんな感じで、次に`script`ブロックではどのようなことができるのかと一定のルールなどについて知っていきましょう。



## TypeScriptを使う

`script`ブロックではJavaScript、もしくはTypeScriptでロジックを記述します。


TypeScriptで実装する場合

`script`タグのプロパティに`lan='ts'`を追加する必要があります。
```svelte
<script lang='ts'>
// ロジックを記述
</script>
```

## コンポーネントのpropを作成する

Svelteでは`export`を用いた変数宣言はそのコンポーネントのpropとして認識されます。

ReactやVueからSvelteへ入門する人がいたら手癖で`const`と書いてしまいがちですが、`const`で宣言した変数を`export`するとreadonlyの変数になってしまい、propと認識されないので注意が必要です。


`export`なしの`const`のみで変数を宣言するとリアクティブではない変数になります。


```svelte
<script>
  // 初期値を指定しないと undefined になる
  // 開発環境では警告が出るので明示的に undefined を指定してあげるほうが良いかも
  export let foo = undefined;

  // barの初期値は　０　になる
  export let bar = 0;

  // ただの const はリアクティブではないただの変数になる
  const fizz = 'fizz';

  // これらはreadonly
　export const buzz = 'buzz';

  export function callName(name: string){
    return `Hello, ${name}!`
  }

  // これはpropとなる    
  export let format = (n) => n.toFixed(2);
</script>
```


## 再レンダリングのトリガーを記述

まず結論からいうと、Svelteにおいて**再レンダリングのトリガーは変数への代入、または更新が行われた時**です。

例えば、`count += 1`と`object.x = 'foo'`は同じように再レンダリングのトリガーになります。

Svelteでは変数などの代入がトリガーとなっているため、配列を操作する系のメソッドで配列に変更を加えたタイミングでは再レンダリングは走りません。

```svelte
<script>
  let arr = [0, 1];　// letで定義された変数は監視対象、変更があれば再レンダリング

  function handleClick() {
    arr.push(2); // この時点ではトリガーにはならない
    arr = arr; // 代入が行われているので再レンダリング　される
  }
</script>
```

## 副作用を起こす

変数への代入がなされたときに副次的効果を起こしたいときには`$:`を文頭につけてマークする」必要があります。

簡単にいうとReactでいうところの`useEffect`などとほぼほぼ同じようなものだと思っておけば良いのかな？と思っています。

例を見てみましょう。

```svelte
<script lang="ts">
  let x = 0;
  let y = 0;
	
  function yPlusAValue(value: number) {
    return value + y;
  }
	
  $: total = yPlusAValue(x);
</script>

Total: {total}
<button on:click={() => x++}> Increment X </button>
<button on:click={() => y++}> Increment Y </button>
```


上記のコードでは`x`と`y`、`total`がリアクティブな変数として宣言されており、`yPlusAValue()`関数が定義されています。

ここで注目したいのは`$: total = yPlusAValue(x)`という式です。

`$:`で宣言された式（リアクティブブロック）というのは依存している変数(dependencies)に変化があると実行されると説明しました。

引数に`x`が代入されているので`x`に変化が起こると`total` が更新されるということです。

注意が必要なのは`y`に変化が起きても`total`の値には更新されません。

つまり、リアクティブブロックで呼び出される関数内にある変数は依存変数として扱われないということで、依存変数として認識してもらうためには`x`のように直接リアクティブブロックに記述しなければいけません。


## ライフサイクルメソッドを使った処理

`Svelte` にももちろんライフサイクルメソッドは存在します。

その中でもこの記事では`onMount()`, `createEventDispatcher()`について少しだけdiveします。


### onMount()

- コンポーネントがDOMにマウントされた直後に実行されます
- DOM操作、外部APIとの通信、イベントリスナーの登録など、コンポーネントの初期化処理に利用されます
- 関数が返された場合はアンマウント時に呼び出されます
- 必ずコンポーネント内で定義する必要はありません、外部から呼び出すことも可能です
- ※ サーバーサイドコンポーネントでは動作しません


コンポーネントが作成されるタイミングでAPIからデータを取得するような処理を例として書いてみます。

```svelte
<script>
	import { onMount } from 'svelte';

	let data = [];

	onMount(async () => {
		const res = await fetch(`/sample/api/data`);
		data = await res.json();
	});
</script>
```


### createEventDispatcher()

- 子コンポーネント →　親コンポーネントへカスタムイベントを送信するための関数を作成します
- 関数の第一引数にはイベント名を指定します
- 第二引数には、任意のデータをオブジェクトとして渡すことができます
- イベント名は、コンポーネント内で一意である必要があります
- dispatch 関数に3番目のパラメータを渡すことで、イベントがキャンセルできるようになります

例のコードを見てみましょう。


```svelte
// 子コンポーネント
<script>
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  let count = 0;

  function increment() {
    count++;
    dispatch('countChanged', { count });
  }
</script>

<button on:click={increment}>Increment</button>
```

```svelte
// 親コンポーネント
<script>
  import ChildComponent from './ChildComponent.svelte';

  let childCount = 0;
</script>

<ChildComponent on:countChanged={(event) => childCount = event.detail.count} />
<p>Child count: {childCount}</p>
```


::: message
`createEventDispatcher()`は、子コンポーネントから親コンポーネントへの一方向の通信にのみ使用できます。
兄弟コンポーネント間の通信や、親コンポーネントから子コンポーネントへの通信には他のライフサイクルメソッドを使用する必要があります。
:::


<br/>

---


今回は`script`ブロックについて多く書きすぎたので次回以降マークアップブロック、`style`ブロックについて書こうと思います。

ありがとうございました。