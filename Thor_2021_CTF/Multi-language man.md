# Multi-language man:Cryptography:pts

```txt
ыисеаХершы_шы_фадфп_ащк_адфп_дщмукыЪ
```

# Solution

換字式暗号です．先頭が`SBCTF{`で末尾が`}`であると仮定して変換してみます．

`SBCTF{tршs_шs_фfдфп_fщк_fдфп_дщмукs}`

`tршs_шs`が`this_is`であると仮定します．

`SBCTF{this_is_фfдфп_fщк_fдфп_дщмукs}`

Google 翻訳で`дщмукыЪ`を翻訳すると，「もしかして:`lovers}`」と出力されましたが，現在私はうまく再現できませんでした．

`SBCTF{this_is_фflфп_for_flфп_lovers}`

`flфп`が2回出てきます．`flag`ではないかと仮定します．

`SBCTF{this_is_фflag_for_flag_lovers}`

`фflag`なんて単語はないので，作問者の`a_flag`の間違いだと推測し，提出するとフラグであるとわかります．

## SBCTF{this_is_a_flag_for_flag_lovers}