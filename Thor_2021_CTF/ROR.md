# ROR:Cryptography:pts

A super secure and state of the art new cipher!!! As a test, we have secured a flag with it, try to get the flag.

```py
from random import randrange
from secret import flag 


def encrypt(p):
	l = []
	for i in range(len(p)):
		l.append(ord(p[i]))
	while len(l) % 64 != 0:
		l.append(randrange(33, 125))
	c = len(l) % 64
	for i in range(c+1):
		for j in range(24):
			l[0*(i+1)] = l[0*(i+1)] ^ l[32*(i+1)]
			l[8*(i+1)] = l[8*(i+1)] ^ l[40*(i+1)]
			l[16*(i+1)] = l[16*(i+1)] ^ l[48*(i+1)]
			l[24*(i+1)] = l[24*(i+1)] ^ l[56*(i+1)]
			l_t = l[:]
			for z in range(len(l)):
				l[z] = l_t[z-1]
	return l

print(encrypt(flag))
```

```txt
[98, 6, 16, 84, 28, 59, 8, 27, 41, 99, 52, 68, 54, 100, 78, 103, 39, 88, 79, 49, 127, 7, 8, 24, 36, 125, 54, 106, 41, 25, 28, 92, 20, 51, 114, 115, 49, 110, 103, 95, 82, 44, 102, 27, 117, 85, 62, 15, 20, 114, 125, 105, 85, 38, 101, 77, 119, 63, 117, 62, 111, 98, 78, 111]
```

# Solution

何度も複雑にXORがされている暗号ですが，XORは二度行うと元に戻ることを利用して，本問では8回暗号化すると元に戻る性質がありました．

```py
from random import randrange

enc=[98, 6, 16, 84, 28, 59, 8, 27, 41, 99, 52, 68, 54, 100, 78, 103, 39, 88, 79, 49, 127, 7, 8, 24, 36, 125, 54, 106, 41, 25, 28, 92, 20, 51, 114, 115, 49, 110, 103, 95, 82, 44, 102, 27, 117, 85, 62, 15, 20, 114, 125, 105, 85, 38, 101, 77, 119, 63, 117, 62, 111, 98, 78, 111]

def decrypt(p):
	l = []
	for i in range(len(p)):
		l.append(p[i])
	while len(l) % 64 != 0:
		l.append(randrange(33, 125))
	c = len(l) % 64
	for i in range(c+1):
		for j in range(24):
			l[0*(i+1)] = l[0*(i+1)] ^ l[32*(i+1)]
			l[8*(i+1)] = l[8*(i+1)] ^ l[40*(i+1)]
			l[16*(i+1)] = l[16*(i+1)] ^ l[48*(i+1)]
			l[24*(i+1)] = l[24*(i+1)] ^ l[56*(i+1)]
			l_t = l[:]
			for z in range(len(l)):
				l[z] = l_t[z-1]
	return l

enc=decrypt(enc)
enc=decrypt(enc)
enc=decrypt(enc)
enc=decrypt(enc)
enc=decrypt(enc)
enc=decrypt(enc)
enc=decrypt(enc)

for i in range(len(enc)):
	enc[i] = chr(enc[i])
print(''.join(enc))
```

`SBCTF{R3v3rs1ng_ROR_C1ph3r}iU&eMw?u>obNob5b'-UoD{c4D6dNg'*2X*!mU`

この先頭が答えです．

## SBCTF{R3v3rs1ng_ROR_C1ph3r}