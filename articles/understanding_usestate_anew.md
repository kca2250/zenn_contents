---
title: "あらためて理解するuseState"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react"]
published: false
---

普段何気なく使っているuseStateを改めて公式ドキュメントを読みながら整理していきたいと思います。


# useState の基本的な使い方

useState といえばこんな感じで書きますよね

```useState.tsx
function example() {
const [count, setCount] = useState<number>(0);

    const countUp = () => {
        setCount(count + 1);
    }

    return (
        <>
            <p>{count}</p>
            <button onClick={countUp}>count up</button>
        </>
    )
}
```

コンポーネントのトップレベルで state を宣言し、なんらかの関数で state の更新処理を記述します。
上記のコードでいうと初回レンダーでは`useState<number>(0)`の`0`がレンダリングされ、`countUp()`が実行される毎に state が更新されていきます。

これが超基本的な useState の使い方です。

:::message
useState は hook なのでコンポーネント、custom hook 内でしか宣言できないので注意が必要です。
:::



## コンポーネントの props を初期値に設定する

必ずしも useState の初期値はコンポーネント内で宣言しなければならないというわけではありません。

下記のようにコンポーネントの props で初期値をとることもできます。

```propsIsDefaultValue.tsx
type Props = {
    defaultValue: number
}

export function PropsIsInitialState({defaultValue}: Props){
    const [number, setNumber] = useState<number>(defaultValue);
    ....
}
```

## 関数を初期値に設定する

実は関数も初期値に設定することもできます。

```getRandomNumber.ts
// 0から99までのランダムな数値を返す
function getRandomNumber(): number {
    return Math.floor(Math.random() * 100);
}
```

```functionIsDefaultState.tsx

export function FunctionIsDefaultState() {
    const [number, setNumber] = useState<number>(getRandomNumber);
    ...
}
```

:::message
関数を初期値に設定する際の記述で少し挙動が変わるので注意が必要です。（Lazy initial state と呼ぶ）
`useState<number>(getRandomNumber)`と書くと初回レンダー時のみ実行され、`useState<number>(getRandomNumber())`と書くとレンダー毎に値が計算されてしまいます。
つまり、初期値に関数を渡す際に初期値を再生成されるの防ぐためには前者の`()`なしを書くようにしましょう。
:::

## 直前の state に応じて更新する

下記のコードを見てみましょう、set 関数を並べて記述するとどのような挙動になるのか考えてみましょう。

`count = 5`になる？それとも`count = 3`になる？

```updateAccordingLastState.ts
// count = 2
function updateAccordingLastState() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
}
```

正解は`count = 3`です。

であれば、同じように set 関数を並べて`count`を`5`に更新するにはどうしたらよいでしょうか？

```anewUpdateAccordingLastState.ts
function anewUpdateAccordingLastState(){
    setCount(prevCount => prevCount + 1);
    setCount(prevCount => prevCount + 1);
    setCount(prevCount => prevCount + 1);
}
```

上記のように set 関数を用いると`count = 5`になります。

これを更新用関数(updater function)と呼びます。

このような形で 1 つのイベントで複数回同じ state に変化を加えたい場合に用います。


## オブジェクトのstateの更新方法

例えば以下の様なオブジェクトがあるとします

```object.ts
const object = {
    age: 23,
    name: 'stan lee',
    isAdult: true
}
```

上記のオブジェクトが初期値に設定されたstateを更新する時どうするでしょうか？

下記のようにスプレッド構文でobjectを展開しつつ更新したいプロパティのみを変更することでオブジェクトの一部だけを更新することができます。

```changeObjectState.tsx
setObject({
    ...object,
    name: e.target.value,
})
```


## keyを用いてstateをリセットする

key属性はリストを表示させるコンポーネントでよく使われるかと思います。

実はstateをリセットするためにも使うことが可能です。

下記のようにトップレベルに`version`stateを持たせて、対象のコンポーネントのkey属性にわたすことでリセットすることが可能になります。


```resetStateUsingKey.tsx
export default function App() {
  const [version, setVersion] = useState(0);

  function handleReset() {
    setVersion(version + 1);
  }

  return (
    <>
      <button onClick={handleReset}>Reset</button>
      <Form key={version} />
    </>
  );
}

function Form() {
  const [name, setName] = useState('Taylor');

  return (
    <>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <p>Hello, {name}.</p>
    </>
  );
}
```



# 番外編：レンダーとコミットについて考え方

少しstateとは話しがそれますが、番外編として書いておきます

基本的なところから話すと React におけるレンダーのトリガーは state が変更が通知されたときです。

次に、このレンダーとはどういうものかを理解して useState をもうちょっと深ぼってみます。

まず結論から言うとレンダーとは React がコンポーネント（関数）を呼び出すこと、コミットとは React が DOM にコンポーネントを反映させることです。
この流れを state が変更されたときにおこなっているというわけです。

ではざっくり簡単に React がどのようなステップをもって描画に変化を加えているかを書きます。

- レンダーのトリガー　（state の更新通知）
- 更新済みのコンポーネントをレンダー（state を変更する）
- コンポーネントを DOM にコミット（ブラウザに表示）

この 3 段階の段階を踏んでようやくブラウザに表示されることになります。
次回以降の state の更新も同じ段階を踏みます。


**イメージ図**
トリガー、レンダー、コミットの図

:::message
更新された（される）コンポーネントがどの位置に配置されているかでレンダーされる範囲が変化するのでパフォーマンスの観点からどのような設計にするか考える必要がありそうです。
:::


## Fiber Reconcilerという概念について

ここではあまり深く書きませんが、ReactにはFiber Reconcilerという仕組みがあります。

これは簡単にいうと**ReactがUIを効率的に更新するために使われる仕組みのこと**です。

すべてのコンポーネントをFiberという単位に分割し、それぞれ効率的にレンダーからコミットまでを行ってくれます。

一旦説明はここまでにしておきます（今後まとめる予定です）。


<br />

<br />

以上です、最後まで読んでいただきありがとうございます。

間違っている内容などあればコメントなどで指摘していただければと思います、よろしくお願いします。

----

参考記事たち
https://zenn.dev/porokyu32/articles/960d9d6e45533b

https://ja.react.dev/reference/react/useState#resetting-state-with-a-key

