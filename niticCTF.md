# niticCTF

URL: [https://ctf.waku-waku-club.com/](https://ctf.waku-waku-club.com/)

|         項目         |       結果       |
| :------------------: | :--------------: |
|       チーム名       | Tsukushi YOSHIMI |
|         順位         |        22         |
| チームで解いたスコア |       3201       |
| チームで解いた問題数 |        14        |
|  個人で解いた問題数  |        3         |
|  個人で解いたスコア  |       700       |

## Crypto

### Caesar Ciphe

ネットでググるといくらでも復号器あります．

### ord_xor

XORをもう一回やって元に戻せばいいです．

```py

with open("./flag", "r") as f:
    enc = f.read()


def xor(c: str, n: int) -> str:
    temp = ord(c)
    for _ in range(n):
        temp ^= n
    return chr(temp)


flag = ""
for i in range(len(enc)):
    flag += xor(enc[i], i)

print(flag)
```

### tanitu_kanji

`format` をもとにフラグを暗号化しているらしいです．`format`は0または1の10文字と小さいので，これを全探索してから暗号化の操作を逆に戻します．

```py
# import os

alphabets = "abcdefghijklmnopqrstuvwxyz0123456789{}_"
after1 = "fl38ztrx6q027k9e5su}dwp{o_bynhm14aicjgv"
after2 = "rho5b3k17pi_eytm2f94ujxsdvgcwl{}a086znq"

with open("./flag", "r") as file:
    enc = file.read()


def renv(s: str, table: str):  # -> str:
    res = ""
    for c in s:
        i = table.index(c)
        res += alphabets[i]
    return res


for bit in range(pow(2, 10)):
    flag = enc
    for i in range(10):
        if bit >> i & 1:
            flag = renv(flag, after1)
        else:
            flag = renv(flag, after2)
    if flag[:10] == "nitic_ctf{":
        print(flag)

```