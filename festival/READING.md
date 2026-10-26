# 読解ノート：自分のアプリのコード3箇所

> **氏名：** 26CM0120 竹岡　修一
> **AI利用レベル：** 1（質問のみ。説明は自分の言葉で書く）
> **最終更新：** 2026-10-26

生成AIに書いてもらったコードでも、自分で書いたコードでもかまいません。自分のアプリの中から3箇所を選んで、次の3つの問いに答えてください。分からないところは、生成AIに「この行は何をしていますか」「この行を消すとどうなりますか」と聞いてかまいません。ただし、書くのは自分の言葉です。

> 💡 担任の先生の指示で、項目が増えることがあります。

---

## 箇所1：（例：プレイヤーが剣のあるマスに入ったとき、剣を取得して所持状態を更新する処理）

```swift
struct SwordEvent: TileEvent {
    func onEnter(state: inout GameState, from previousPosition: GridPos) {
        if state.hasSword {
            state.message = "剣を装備している。"
            return
        }
        state.hasSword = true
        state.message = "剣を手に入れた！攻撃力が上がった！"
    }
}
```

**何をしているか：**
プレイヤーが剣のマスに入ると　onEnter　というメソッドが呼び出され　剣をすでに持っているか？の確認を　if state.hasSword　で行っている　True なら 剣を装備している。と表示　falseなら 剣を手に入れた！を表示する。

**なぜこう書くのか：**
異なるイベントを共通の仕組みで管理しやすくするため、ゲーム全体の状態である GameState を更新する必要があるから、プレイヤーが剣を所持しているのか分かりやすくするため

**もしこう書かなかったら：**

inout を書かなかったら
GameState が構造体で、hasSword が変更可能なプロパティである場合、引数は通常変更できないためコンパイルエラーになる。
GameStateを更新できなかったら、イベント管理ができてないと見なされる？

---

## 箇所2：（鍵を取得する前の状態を保存し、初回取得かどうかによってチュートリアルとメッセージを変更している処理）

```swift
 func onEnter(state: inout GameState, from previousPosition: GridPos) {
        let alreadyHadKey = state.hasKey
        state.hasKey = true
        if !alreadyHadKey && (state.tutorialStep == .findKey || state.tutorialStep == .zoomMap) {
            state.tutorialStep = .reachGoal
            state.message = "鍵を手に入れた！ゴールを探そう！"
        } else {
            state.message = alreadyHadKey ? "空の宝箱がある" : "鍵を手に入れた"
        }
    }
}
```

**何をしているか：**
onEnterで鍵を手に入れた時の処理をまとめ、inout GameStateで変更可能にしている、その後let alreadyHadKey = state.hasKeyというコードで鍵を取得する前の状態を保存している
if文で鍵を取得した時の処理を作成し、&& はAND条件、|| はOR条件のどちらかでチュートリアルの画面かそれ以外のマップかを判断しています。取得した後はstate.tutorialStep = .reachGoalで
ゴールを探すチュートリアルを促しています。state.messageは鍵を持っていたかによって状況メッセージを変更しています。


**なぜこう書くのか：**
初回取得かどうかを判定するため、鍵を取得したことを記録するため、2種類のチュートリアル状態に対応し状況によって案内を変えている

**もしこう書かなかったら：**

&& を || に変えたら、鍵をすでに持っていても、チュートリアルの状態によっては条件が成立してしまい、意図しないタイミングでチュートリアルが進む可能性がある。
---

## 箇所3：（ゴールに入れるかどうかの判定と、ゴールに入った後の処理を別々の関数に分けている処理）

```swift
struct GoalEvent: TileEvent {
    func blockReason(state: GameState) -> String? {
        // 鍵が必要な面で、鍵を持っていないときだけ入れない
        if state.requiresKey && !state.hasKey {
            return "ゴールの扉には鍵が必要です。まず鍵を探しましょう。"
        }
        return nil
    }

    func onEnter(state: inout GameState, from previousPosition: GridPos) {
        state.isCleared = true      // クリア状態にする(仕様書§3・§11)
        state.message = "ゴール!"
    }
```

**何をしているか：**
struct GoalEvent でゴールイベントを作成し、func blockReason(...)でゴールへの進入を妨げる理由を設定している、その後　if state.requiresKey && !state.hasKey　で
鍵が必要で、鍵を持っていないか判定している。return nilで返すことによって理由がないことを返すか、返さないかの処理が行われている。

**なぜこう書くのか：**
クリア条件を確認するため、入れない理由の有無を表すため、プレイヤーに必要な条件を伝えるため、ゴール到達を記録しプレイヤーに知らせるため。: TileEventに繋げると画面を遷移できる。

**もしこう書かなかったら：**

! を書かなかったら　鍵が必要なステージで、鍵を持っているプレイヤーをブロックしてしまう。
---

## 自分ではまだ説明できない箇所（正直に）
（アプリの中で「動いているけれど何をしているか分からない」ところを書く。ここに書いたものは、後期の文法の授業で分かるようになります。）
 色々なファイルを作成し結びつけてしまったため、どこの関数や変数で処理が連動しているかが、まだ読み取れていない。
 また、IOSのバージョンによって使える構文の違いや、IPadへの画面構成の分岐処理などが分からない。
