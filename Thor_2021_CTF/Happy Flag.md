# Happy Flag:Forensics:pts

We have many flags. But we need a good flag!

# Solution

```txt
file world_flags.zeep
world_flags.zeep: Zip archive data, at least v1.0 to extract
```

zipファイルが与えられています．`unzip world_flags.zeep`で解凍します．190914個のtxtファイルが入っています．

`CTF`という文字列が入っているファイルを探します．

```txt
grep CTF -rl . 2>&1 /dev/null
./world_flags/15815.txt
```

`cat ./world_flags/15815.txt`

## SBCTF{Cool_flag_!!!}