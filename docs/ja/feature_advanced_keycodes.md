# レイヤーの切り替えとトグル :id=switching-and-toggling-layers

<!---
  original document: 5d5ff80:docs/feature_advanced_keycodes.md
  git diff 5d5ff80 HEAD -- docs/feature_advanced_keycodes.md | cat
-->

これらの機能により、様々な方法でレイヤーをアクティブ化することができます。レイヤーは一般的に独立したレイアウトでは無いことに注意してください -- 複数のレイヤーを一度にアクティブ化することができ、レイヤーが `KC_TRNS` を使ってキーの押下を下のレイヤーに渡すことが一般的です。レイヤーの詳細については、[キーマップの概要](ja/keymap.md#keymap-and-layers)を見てください。MO()、LM()、TT() あるいは LT() を使って一時的なレイヤーの切り替えを使う場合、上のレイヤーのキーを透過にするようにしてください。さもないと意図したように動作しないかもしれません。

* `DF(layer)` - デフォルトレイヤーを切り替えます。デフォルトレイヤーは、他のレイヤーがその上に積み重なっている、常にアクティブな基本レイヤーです。デフォルトレイヤーの詳細については以下を見てください。これは QWERTY から Dvorak レイアウトに切り替えるために使うことができます。(これは一時的な切り替えであり、キーボードの電源が切れるまでしか持続しないことに注意してください。デフォルトレイヤーを永続的に変更するには、[process_record_user](ja/custom_quantum_functions.md#programming-the-behavior-of-any-keycode) 内で `set_single_persistent_default_layer` 関数を呼び出すなど、より深いカスタマイズが必要です。)
* `MO(layer)` - 一時的に*レイヤー*をアクティブにします。キーを放すとすぐに、レイヤーは非アクティブになります。
* `LM(layer, mod)` - (`MO` のように)一時的に*レイヤー*をアクティブにしますが、モディファイア *mod* がアクティブな状態です。layer 0-15 と、左モディファイアのみをサポートします: `MOD_LCTL`、`MOD_LSFT`、`MOD_LALT`、`MOD_LGUI` (`KC_` の代わりに `MOD_` 定数を使うことに注意してください)。これらのモディファイアは、例えば `LM(_RAISE, MOD_LCTL | MOD_LALT)` のように、ビット単位の OR を使って組み合わせることができます。
* `LT(layer, kc)` - ホールドされた時に*レイヤー*を一時的にアクティブにし、タップされた時に *kc* を送信します。layer 0-15 のみをサポートします。
* `OSL(layer)` - 次のキーが押されるまで、一時的に*レイヤー*をアクティブにします。詳細と追加機能については、[ワンショットキー](ja/one_shot_keys.md)を見てください。
* `TG(layer)` - *レイヤー*を切り替えます。非アクティブな場合はアクティブにし、逆も同様です。
* `TO(layer)` - *レイヤー*をアクティブにし、他の全てのレイヤー(デフォルトレイヤーを除く)を非アクティブにします。この関数は特別です。1つのレイヤーをアクティブなレイヤースタックに追加/削除する代わりに、現在のアクティブなレイヤーを完全に置き換え、唯一上位のレイヤーを下位のレイヤーで置き換えることができるからです。これはキーダウンで(キーが押されるとすぐに)アクティブになります。
* `TT(layer)` - レイヤーのタップ切り替え。キーを押したままにすると*レイヤー*がアクティブにされ、放すと非アクティブになります (`MO` 風)。繰り返しタップすると、レイヤーはオンあるいはオフを切り替えます (`TG` 風)。デフォルトでは5回のタップが必要ですが、`TAPPING_TOGGLE` を定義することで変更することができます -- 例えば、2回のタップだけで切り替えるには、`#define TAPPING_TOGGLE 2` を定義します。

## 注意事項

現在のところ、`LT()` と `MT()` は[基本的なキーコードセット](ja/keycodes_basic.md)に制限されています。つまり、`LCTL()`、`KC_TILD` あるいは `0xFF` より大きなキーコードを使うことができません。レイヤータップあるいはモッドタップのキーコードの一部として指定されたモディファイアは無視されます。タップしたキーコードにモディファイアを適用する必要がある場合は、[タップダンス](ja/feature_tap_dance.md#example-5-using-tap-dance-for-advanced-mod-tap-and-layer-tap-keys)を使うことができます。

さらに、モッドタップあるいはレイヤータップで少なくとも1つの右手用のモディファイアが指定された場合、指定された全てのモディファイアが右手用になるため、2つをうまく組み合わせて一致させることはできません。

# レイヤーの使用

レイヤーを切り替える時は注意してください。(キーボードを取り外さずに)そのレイヤーを非アクティブにすることができずレイヤーから移動できなくなる可能性があります。最も一般的な問題を避けるためのガイドラインを作成しました。

## 初心者

QMK を使い始めたばかりの場合は、全てを単純にしたいでしょう。レイヤーをセットアップする時は、これらのガイドラインに従ってください:

* デフォルトの "base" レイヤーとして、layer 0 をセットアップします。これは通常の入力レイヤーであり、任意のレイアウト (qwerty、dvorak、colemak など)にすることができます。通常はキーボードのキーのほとんどまたは全てが定義されているため、これを最下位のレイヤーとして設定することが重要です。そうすることで、もしそれが他のレイヤーの上 (つまりレイヤー番号が大きい)にある場合の影響を防ぎます。
* layer 0 をルートとして、レイヤーを "ツリー" レイアウトに配置します。他の複数のレイヤーから同じレイヤーに行こうとしないでください。
* 各レイヤーのキーマップでは、より高い番号のレイヤーのみを参照します。レイヤーは最大の番号(最上位)のアクティブレイヤーから処理されるため、下位レイヤーの状態を変更するのは難しくエラーが発生しやすくなります。

## 中級ユーザ

複数の基本レイヤーが必要な場合があります。例えば、QWERTY と Dvorak を切り替える場合、国ごとに異なるレイアウトを切り替える場合、あるいは異なるビデオゲームごとにレイアウトを切り替える場合などです。基本レイヤーは常に最小の番号のレイヤーである必要があります。複数の基本レイヤーがある場合、常にそれらを相互排他的に扱う必要があります。1つの基本レイヤーがオンの場合、他をオフにします。

## 上級ユーザ

レイヤーがどのように動作し、何ができるかを理解したら、より創造的になります。初心者のセクションで列挙されている規則は、幾つかの巧妙な詳細を回避するのに役立ちますが、特に超コンパクトなキーボードのユーザにとって制約になる場合があります。レイヤーの仕組みを理解することで、レイヤーをより高度な方法で使うことができます。

レイヤーは番号順に上に積み重なっています。キーの押下の動作を決定する時に、QMK は上から順にレイヤーを走査し、`KC_TRNS` に設定されていない最初のアクティブなレイヤーに到達すると停止します。結果として、現在のレイヤーよりも数値的に低いレイヤーをアクティブにし、現在のレイヤー(あるいはアクティブでターゲットレイヤーよりも高い別のレイヤー)に `KC_TRNS` 以外のものがある場合、それが送信されるキーであり、アクティブ化したばかりのレイヤー上のキーではありません。これが、ほとんどの人の "なぜレイヤーが切り替わらないのか" 問題の原因です。

場合によっては、マクロ内あるいはタップダンスルーチンの一部としてレイヤーを切り替えほうが良いかもしれません。`layer_on` はレイヤーをアクティブにし、`layer_off` はそれを非アクティブにします。もっと多くのレイヤーに関する関数は、[action_layer.h](https://github.com/qmk/qmk_firmware/blob/master/tmk_core/common/action_layer.h) で見つけることができます。

# 修飾キー :id=modifier-keys

以下のようにキーコードとモディファイアを組み合わせることができます。押すと、モディファイアのキーダウンイベントが送信され、次に `kc` のキーダウンイベントが送信されます。放すと、`kc` のキーアップイベントが送信され、次にモディファイアのキーアップイベントが送信されます。

| キー | エイリアス | 説明 |
|----------|-------------------------------|----------------------------------------------------|
| `LCTL(kc)` | `C(kc)` | 左 Control を押しながら `kc` を押します。 |
| `LSFT(kc)` | `S(kc)` | 左 Shift を押しながら `kc` を押します。 |
| `LALT(kc)` | `A(kc)` | 左 Alt を押しながら `kc`を押します。 |
| `LGUI(kc)` | `G(kc)`, `LCMD(kc)`, `LWIN(kc)` | 左 GUI を押しながら `kc` を押します。 |
| `RCTL(kc)` |  | 右 Control を押しながら `kc` を押します。 |
| `RSFT(kc)` |  | 右 Shift を押しながら `kc` を押します。 |
| `RALT(kc)` | `ALGR(kc)` | 右 Alt を押しながら `kc` を押します。 |
| `RGUI(kc)` | `RCMD(kc)`, `LWIN(kc)` | 右 GUI を押しながら `kc` を押します。 |
| `SGUI(kc)` | `SCMD(kc)`, `SWIN(kc)` | 左 Shift と左 GUI を押しながら `kc` を押します。 |
| `LCA(kc)` |  | 左 Control と左 Alt を押しながら `kc` を押します。 |
| `LCAG(kc)` |  | 左 Control、左 Alt、左 GUI を押しながら `kc` を押します。 |
| `MEH(kc)` |  | 左 Control、左 Shift、左 Alt を押しながら `kc` を押します。 |
| `HYPR(kc)` |  | 左 Control、左 Shift、左 Alt、左 GUI を押しながら `kc` を押します。 |

また、それらを繋げることができます。例えば、`LCTL(LALT(KC_DEL))` は1回のキー押下で Control+Alt+Delete を送信するキーを作成します。

# 過去の内容

このページには多くの機能が含まれていました。このページを構成していた多くのセクションをそれぞれのページに移動しました。これより下は全て単なるリダイレクトであるため、web上で古いリンクをたどっている人は探しているものを見つけることができます。

## モッドタップ :id=mod-tap

* [モッドタップ](ja/mod_tap.md)

## ワンショットキー :id=one-shot-keys

* [ワンショットキー](ja/one_shot_keys.md)

## タップホールド設定オプション :id=tap-hold-configuration-options

* [タップホールド設定オプション](ja/tap_hold.md)
