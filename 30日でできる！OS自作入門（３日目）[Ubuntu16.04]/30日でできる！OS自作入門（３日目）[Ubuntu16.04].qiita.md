前後の記事
- <a herf="https://qiita.com/pollenjp/items/a13cd764cc739a0ca9df">[Ubuntu16.04] 30日でできる！OS自作入門（１日目）</a>

# 目的
"30日でできる！OS自作入門"の内容をUbuntu(Linux)で実行するには本の内容だけでは厳しいので調べた結果をメモ。（リンクと動作確認済みコード・コメント）
本を読む上でLinuxを使う方の参考になればと思っております。
Ubuntu 16.04 LTS

# 参考
先に参考を載せておきます。

- <a href="https://wiki.osdev.org/BIOS#BIOS_functions">BIOS_functions - BIOS - OSDev Wiki</a>
- <a href="http://oswiki.osask.jp/?%28AT%29BIOS#q5006ed6">INT(0x13); ディスク関係 - (AT)BIOS - os-wiki</a>
- <a href="http://tsurugidake.hatenablog.jp/entry/2017/08/26/003114">Linuxで書くOS自作入門 3日目(前半) - Tsurugidake's diary</a>
- <a href="https://syusui.tumblr.com/post/109884535088/30%E6%97%A5%E3%81%A7%E3%81%A7%E3%81%8D%E3%82%8Bos%E8%87%AA%E4%BD%9C%E5%85%A5%E9%96%80%E3%82%92linux%E3%81%A7%E3%82%84%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B-3%E6%97%A5%E7%9B%AE">『30日でできる！OS自作入門』をLinuxでやってみる 3日目</a>
- <a href="https://syusui.tumblr.com/post/110352077473/30%E6%97%A5%E3%81%A7%E3%81%A7%E3%81%8D%E3%82%8Bos%E8%87%AA%E4%BD%9C%E5%85%A5%E9%96%80%E3%82%92linux%E3%81%A7%E3%82%84%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B-31%E6%97%A5%E7%9B%AE?is_related_post=1">『30日でできる！OS自作入門』をLinuxでやってみる 3.1日目</a>
- <a href="https://syusui.tumblr.com/post/110447016498/30%E6%97%A5%E3%81%A7%E3%81%A7%E3%81%8D%E3%82%8Bos%E8%87%AA%E4%BD%9C%E5%85%A5%E9%96%80%E3%82%92linux%E3%81%A7%E3%82%84%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B-32%E6%97%A5%E7%9B%AE?is_related_post=1">『30日でできる！OS自作入門』をLinuxでやってみる 3.2日目</a>
- <a href="http://bttb.s1.valueserver.jp/wordpress/blog/2017/11/25/makeos-3-1/">OS自作入門 3日目-1 【Linux】| 64bit環境での苦悩</a>
- <a href="http://bttb.s1.valueserver.jp/wordpress/blog/2017/12/06/makeos-3-2/">OS自作入門 3日目-2 【Linux】| デバッグして実行順序を追ってみる</a>
- <a href="http://takeisamemo.blogspot.jp/2014/09/os30os-3-4.html">[OS作成]30日でできる！OS自作入門 3日目 (4) </a>
- <a herf="http://lv4.hateblo.jp/entry/2011/10/21/222521">OS自作入門 onLinux 3日目</a>


# INT 0x13

- <a href="https://wiki.osdev.org/BIOS#BIOS_functions">BIOS_functions - BIOS - OSDev Wiki</a>
- <a href="http://oswiki.osask.jp/?%28AT%29BIOS#q5006ed6">INT(0x13); ディスク関係 - (AT)BIOS - os-wiki</a>

追記(p.49)したコードを以下に表示

```:harib00a/ipl.asm
; haribote-ipl
; TAB=4
        ORG     0x7c00          ; read start

; discription for floppy disk
        JMP     entry           ; BS_JmpBoot
        DB      0x90            ; BS_JmpBoot
        DB      "HARIBOTE"      ; BS_OEMName    8B
        DW      512             ; BPB_BytsPerSec
        DB      1               ; BPB_SecPerClu
        DW      1               ; BPB_RevdSecCnt    : このBPBを含むブートセクタのみ
        DB      2               ; BPB_NumFATs       : FATの個数 (このフィールドの値は常に2に設定すべきである)
        DW      224             ; BPB_RootEntCnt
        DW      2880            ; BPB_TotSec16
        DB      0xf0            ; BPB_Media
        DW      9               ; BPB_FATSz16
        DW      18              ; BPB_SecPerTrk
        DW      2               ; BPB_NumHeads
        DD      0               ; BPB_HiddSec
        DD      2880            ; BPB_TotSec32

        ; FAT12/16におけるオフセット36以降のフィールド
        DB      0x00            ; BS_DrvNum
        DB      0x00            ; BS_Reserved1
        DB      0x29            ; BS_BootSig

        DD      0xffffffff      ; BS_VolID
        DB      "HARIBOTEOS "   ; BS_VolLab     11B
        DB      "FAT12   "      ; BS_FilSysType 8B
        RESB    18              ; とりあえず18バイト開けておく


; START BS_BootCode 64(0x14)   448(0x1C0)
entry:
        MOV     AX, 0           ; initialize Accumulator(resister)
        MOV     SS, AX          ; Stack Segment
        MOV     SP, 0x7c00      ; Stack Pointer
        MOV     DS, AX          ; Data Segment      : 番地指定のとき重要
        ;MOV     ES, AX          ; Extra Segment

        ;MOV     SI, msg         ; Source Index

; load disk
        MOV     AX, 0x0820
        MOV     ES, AX          ; extra segment :  buffer address       0x0820
        MOV     CH, 0           ; counter high  : cylinder  0
        MOV     DH, 0           ; data high     : head      0
        MOV     CL, 2           ; counter low   : sector    2

        MOV     AH, 0x02        ; acumulator high   : read disk
        MOV     AL, 1           ; acumulator low    : sector    1
        MOV     BX, 0           ; base              : buffer address    0x0000
        MOV     DL, 0x00        ; data low          : drive number
        INT     0x13            ; BIOS call
        JC      error           ; CARRY FLAG

fin:
        HLT
        JMP     fin             ; 無限ループ

error:
        MOV     SI, msg

putloop:
        MOV     AL, [SI]        ; BYTE (accumulator low)
        ADD     SI, 1           ; increment
        CMP     AL, 0           ; 終了条件
        JE      fin             ; jump to fin if equal to 0

        MOV     AH, 0x0e
        MOV     BX, 15
        INT     0x10            ; interrupt BIOS
        JMP     putloop

msg:
        DB      0x0a, 0x0a
        DB      "load error"
        DB      0x0a
        DB      0               ; end msg

        RESB    0x7dfe-0x7c00-($-$$)    ; 現在の場所から0x1fdまで(残りの未使用領域)を0で埋める。
; END BS_BootCode

        DB      0x55, 0xaa      ; BS_BootSign
```

# 実行
- <a href="http://tsurugidake.hatenablog.jp/entry/2017/08/26/003114">Linuxで書くOS自作入門 3日目(前半) - Tsurugidake's diary</a>


`-fda`オプションでフロッピーディスクであることを明示

```:terminal（入力）
$ qemu-system-i386 -fda helloos.img
```

このために修正したMakefileが以下のとおり

```:Makefile
ipl.bin : ipl.asm Makefile
    nasm ipl.asm -o ipl.bin -l ipl.lst

helloos.img : ipl.bin Makefile
    cat ipl.bin > helloos.img

asm :
    make -r ipl.bin

img :
    make -r helloos.img

run :
    make img
    qemu-system-i386 -fda helloos.img
```


# 3. 18セクタまで読む(p.55)

追加したコードは以下

```:ipl.asm
; haribote-ipl
; TAB=4
        ORG     0x7c00          ; read start

; discription for floppy disk
        JMP     entry           ; BS_JmpBoot
        DB      0x90            ; BS_JmpBoot
        DB      "HARIBOTE"      ; BS_OEMName    8B
        DW      512             ; BPB_BytsPerSec
        DB      1               ; BPB_SecPerClu
        DW      1               ; BPB_RevdSecCnt    : このBPBを含むブートセクタのみ
        DB      2               ; BPB_NumFATs       : FATの個数 (このフィールドの値は常に2に設定すべきである)
        DW      224             ; BPB_RootEntCnt
        DW      2880            ; BPB_TotSec16
        DB      0xf0            ; BPB_Media
        DW      9               ; BPB_FATSz16
        DW      18              ; BPB_SecPerTrk
        DW      2               ; BPB_NumHeads
        DD      0               ; BPB_HiddSec
        DD      2880            ; BPB_TotSec32

        ; FAT12/16におけるオフセット36以降のフィールド
        DB      0x00            ; BS_DrvNum
        DB      0x00            ; BS_Reserved1
        DB      0x29            ; BS_BootSig

        DD      0xffffffff      ; BS_VolID
        DB      "HARIBOTEOS "   ; BS_VolLab     11B
        DB      "FAT12   "      ; BS_FilSysType 8B
        RESB    18              ; とりあえず18バイト開けておく


; START BS_BootCode 64(0x14)   448(0x1C0)
entry:
        MOV     AX, 0           ; initialize Accumulator(resister)
        MOV     SS, AX          ; Stack Segment
        MOV     SP, 0x7c00      ; Stack Pointer
        MOV     DS, AX          ; Data Segment      : 番地指定のとき重要

        ;MOV     SI, msg         ; Source Index

; load disk
        MOV     AX, 0x0820
        MOV     ES, AX          ; extra segment :  buffer address       0x0820
        MOV     CH, 0           ; counter high  : cylinder  0
        MOV     DH, 0           ; data high     : head      0
        MOV     CL, 2           ; counter low   : sector    2

readloop:
        MOV     SI, 0           ; 失敗回数を数えるレジスタ

retry:
        MOV     AH, 0x02        ; acumulator high   : 0x02 - read disk
        MOV     AL, 1           ; acumulator low    : sector    1
        MOV     BX, 0           ; buffer address    0x0000
                                ; ES:BX, ESは代入済み
        MOV     DL, 0x00        ; data low          : drive number
        INT     0x13            ; BIOS call
        JNC     next            ; jump if not carry

        ADD     SI, 1           ; increment SI
        CMP     SI, 5
        JAE     error           ; SI >= 5 then jump to error

        MOV     AH, 0x00        ; 0x00 - reset
        MOV     DL, 0x00        ; A drive
        INT     0x13            ; reset drive
        JMP     retry

next:
        ; add 0x20 to ES
        ; 代わりにBXに512を足してもよい
        MOV     AX, ES          ; 0x20だけアドレスを進める
        ADD     AX, 0x0020      ; 512 / 16 = 0x20
        MOV     ES, AX
        ; increment CL (sector number)
        ADD     CL, 1
        CMP     CL, 18
        JBE     readloop

fin:
        HLT
        JMP     fin             ; 無限ループ

error:
        MOV     SI, msg

putloop:
        MOV     AL, [SI]        ; BYTE (accumulator low)
        ADD     SI, 1           ; increment
        CMP     AL, 0           ; 終了条件
        JE      fin             ; jump to fin if equal to 0

        MOV     AH, 0x0e        ; 1 char-function
        MOV     BX, 15          ; color code
        INT     0x10            ; interrupt, call BIOS
        JMP     putloop

msg:
        DB      0x0a, 0x0a
        DB      "load error"
        DB      0x0a
        DB      0               ; end point

        RESB    0x7dfe-0x7c00-($-$$)    ; 現在の場所から0x1fdまで(残りの未使用領域)を0で埋める。
                                        ; 0x7c00スタートなのでその分を引いている
; END BS_BootCode

        DB      0x55, 0xaa      ; BS_BootSign, boot signature
```


```:Makefile
# ファイル生成規則

ipl.bin : ipl.asm Makefile
    nasm ipl.asm -o ipl.bin -l ipl.lst

#helloos.img : ipl.bin tail.bin Makefile
helloos.img : ipl.bin Makefile
    #cat ipl.bin tail.bin > helloos.img
    cat ipl.bin > helloos.img

asm :
    make -r ipl.bin

img :
    make -r helloos.img

run :
    make img
    qemu-system-i386 -fda helloos.img   # "-fda" for floppy disk
```

実行してみましょう

```
$ make run
```

実行結果
![image.png](https://qiita-image-store.s3.amazonaws.com/0/195174/494956f7-df74-3322-2020-85233044f700.png)

# 10シリンダ分を読む(p.56)
追加したコードは以下

```:ipl.asm
; haribote-ipl
; TAB=4

CYLS    EQU     10              ; どこまで読み込むか (CYLinderS)

        ORG     0x7c00          ; このプログラムがメモリ上のどこに読み込まれるか

; discription for floppy disk
        JMP     entry           ; BS_JmpBoot
        DB      0x90            ; BS_JmpBoot
        DB      "HARIBOTE"      ; BS_OEMName    8B
        DW      512             ; BPB_BytsPerSec
        DB      1               ; BPB_SecPerClu
        DW      1               ; BPB_RevdSecCnt    : このBPBを含むブートセクタのみ
        DB      2               ; BPB_NumFATs       : FATの個数 (このフィールドの値は常に2に設定すべきである)
        DW      224             ; BPB_RootEntCnt
        DW      2880            ; BPB_TotSec16
        DB      0xf0            ; BPB_Media
        DW      9               ; BPB_FATSz16
        DW      18              ; BPB_SecPerTrk
        DW      2               ; BPB_NumHeads
        DD      0               ; BPB_HiddSec
        DD      2880            ; BPB_TotSec32

        ; FAT12/16におけるオフセット36以降のフィールド
        DB      0x00            ; BS_DrvNum
        DB      0x00            ; BS_Reserved1
        DB      0x29            ; BS_BootSig

        DD      0xffffffff      ; BS_VolID
        DB      "HARIBOTEOS "   ; BS_VolLab     11B
        DB      "FAT12   "      ; BS_FilSysType 8B
        RESB    18              ; とりあえず18バイト開けておく


; START BS_BootCode 64(0x14)   448(0x1C0)
entry:
        MOV     AX, 0           ; initialize Accumulator(resister)
        MOV     SS, AX          ; Stack Segment
        MOV     SP, 0x7c00      ; Stack Pointer
        MOV     DS, AX          ; Data Segment      : 番地指定のとき重要

        ;MOV     SI, msg         ; Source Index

; load disk
        MOV     AX, 0x0820
        MOV     ES, AX          ; extra segment :  buffer address       0x0820
        MOV     CH, 0           ; cylinder  0
        MOV     DH, 0           ; head      0
        MOV     CL, 2           ; sector    2

readloop:
        MOV     SI, 0           ; 失敗回数を数えるレジスタ

retry:
        MOV     AH, 0x02        ; acumulator high   : 0x02 - read disk
        MOV     AL, 1           ; acumulator low    : sector    1
        MOV     BX, 0           ; buffer address    0x0000
                                ; ES:BX, ESは代入済み
        MOV     DL, 0x00        ; data low          : drive number
        INT     0x13            ; BIOS call
        JNC     next            ; jump if not carry

        ADD     SI, 1           ; increment SI
        CMP     SI, 5
        JAE     error           ; SI >= 5 then jump to error

        MOV     AH, 0x00        ; 0x00 - reset
        MOV     DL, 0x00        ; A drive
        INT     0x13            ; reset drive
        JMP     retry

next:
        ; add 0x20 to ES
        ; 代わりにBXに512を足してもよい
        MOV     AX, ES          ; 0x20だけアドレスを進める
        ADD     AX, 0x0020      ; 512 / 16 = 0x20
        MOV     ES, AX

        ; increment CL (sector number)
        ADD     CL, 1
        CMP     CL, 18
        JBE     readloop

        ; ディスクのウラ面
        MOV     CL, 1           ; reset sector
        ADD     DH, 1           ; reverse HEAD
        CMP     DH, 2
        JB      readloop

        ; next Cylinder
        mov     DH, 0           ; reset HEAd
        ADD     CH, 1           ; cylinder += 1
        CMP     CH, CYLS
        JB      readloop

fin:
        HLT
        JMP     fin             ; 無限ループ

error:
        MOV     SI, msg

putloop:
        MOV     AL, [SI]        ; BYTE (accumulator low)
        ADD     SI, 1           ; increment
        CMP     AL, 0           ; 終了条件
        JE      fin             ; jump to fin if equal to 0

        MOV     AH, 0x0e        ; 1 char-function
        MOV     BX, 15          ; color code
        INT     0x10            ; interrupt, call BIOS
        JMP     putloop

msg:
        DB      0x0a, 0x0a
        DB      "load error"
        DB      0x0a
        DB      0               ; end point

        RESB    0x7dfe - 0x7c00 - ($ - $$)  ; 現在の場所から0x1fdまで(残りの未使用領域)を0で埋める。
                                            ; 0x7c00スタートなのでその分を引いている
; END BS_BootCode
        DB      0x55, 0xaa      ; BS_BootSign, boot signature
```

Makefileはおそらく変更ナシ（一応載せる）

```:Makefile
# ファイル生成規則

ipl.bin : ipl.asm Makefile
    nasm ipl.asm -o ipl.bin -l ipl.lst

#helloos.img : ipl.bin tail.bin Makefile
helloos.img : ipl.bin Makefile
    #cat ipl.bin tail.bin > helloos.img
    cat ipl.bin > helloos.img

asm :
    make -r ipl.bin

img :
    make -r helloos.img

run :
    make img
    qemu-system-i386 -fda haribote.img  # "-da" for floppy disk
```


# OS本体を書き始める(p.57)
ここでLinux/Ubuntu上でedimg.exeをどのように動かせば良いのかわからなかったのでかなり苦戦しました。

まずは`haribote.sys`からです。

```:haribote.asm
fin:
        HLT
        JMP     fin
```

次にこれをディスクイメージ`haribote.img`に保存するのですが、ここで`edimg.exe`が必要なようです。しかし、Linux/Ubuntu上で実行したい自分はいろんな記事を探したり、`edimg.c`の内容を読んで理解しょうとしたりしましたが、以下の記事を見つけて解決しました！（作成者さんありがとうございます！）
- <a href="http://takeisamemo.blogspot.jp/2014/09/os30os-3-4.html">[OS作成]30日でできる！OS自作入門 3日目 (4) </a>
- <a herf="http://lv4.hateblo.jp/entry/2011/10/21/222521">OS自作入門 onLinux 3日目</a>

では、動くファイルを作成していきましょう！

```:Makefile
default:
    make img

ipl.bin : ipl.asm Makefile
    nasm ipl.asm -o ipl.bin -l ipl.lst

haribote.sys : haribote.asm Makefile
    nasm haribote.asm -o haribote.sys -l haribote.lst

haribote.img : ipl.bin haribote.sys Makefile
    mformat -f 1440 -C -B ipl.bin -i haribote.img ::
    mcopy haribote.sys -i haribote.img ::

asm :
    make -r ipl.bin

img :
    make -r haribote.img

run :
    make img
    qemu-system-i386 -fda helloos.img   # "-fda" for floppy disk
```

何をやっているのかは自分にもまだよくわかっていませんが

```
mformat -f 1440 -C -B ipl.bin -i haribote.img ::
mcopy haribote.sys -i haribote.img ::
```

この２行によって保存しているようです。

では、`make img`を実行しましょう。

```:terminal(入力)
$ make img
```

次に`haribote.sys`の機械語との対応を見ましょう。

```:haribote.lst
     1                                  fin:
     2 00000000 F4                              HLT
     3 00000001 EBFD                            JMP     fin

```

`F4`と`EBFD`が確認できますね。


では次にghexを立ち上げてどの箇所に書き込まれているのかをチェックします。

```:terminal(入力)
$ ghex haribote.img
```

ファイルを開いた状態で`ctrl+f`を押すとファイル内文字列検索できるので、その箇所にテキストに乗っているように`48 41 52 ...`（最初の３つくらいで十分）と入力して`Enter`(find next)を押しましょう。（逆説的な説明ではありますが、確認することが目的なのでHARIBOTESYS
の文字を探す必要は無いでしょう。）

２回くらい押すと以下のように`0x2600`にHARIBOTESYSの文字が見つかります。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/195174/826fc5f8-9940-94c9-15f0-90e3c88f9527.png)

`F4 EB FD`と入力した場合も同様です。offset:0x4200（写真左下）が確認できると思います。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/195174/95618f38-a023-0dc7-6f42-13dd3e812a76.png)


これらのことからテキストにかかれていた以下のことが言えます。
>空の状態のディスクに対してファイルを普通に保存すると、
>- (1)ファイル名波0x002600以降に入るらしい
>- (2)ファイルの中身は0x004200以降に入るらしい

良かった。確認できた！
※これ確認するために１周間くらいかけました(0_0)


# ブートセクタからOS本体を実行(p.59) - harib00f
`haribote.asm`と`ipl.asm`に変更を加える。

```:haribote.asm
        ORG     0xc200      ; 0xc200 <- 0x8000 + 0x4200
fin:
        HLT
        JMP     fin
```

```:ipl.asm
; haribote-ipl
; TAB=4

CYLS    EQU     10              ; どこまで読み込むか (CYLinderS)

        ORG     0x7c00          ; このプログラムがメモリ上のどこに読み込まれるか

; discription for floppy disk
        JMP     entry           ; BS_JmpBoot
        DB      0x90            ; BS_JmpBoot
        DB      "HARIBOTE"      ; BS_OEMName    8B
        DW      512             ; BPB_BytsPerSec
        DB      1               ; BPB_SecPerClu
        DW      1               ; BPB_RevdSecCnt    : このBPBを含むブートセクタのみ
        DB      2               ; BPB_NumFATs       : FATの個数 (このフィールドの値は常に2に設定すべきである)
        DW      224             ; BPB_RootEntCnt
        DW      2880            ; BPB_TotSec16
        DB      0xf0            ; BPB_Media
        DW      9               ; BPB_FATSz16
        DW      18              ; BPB_SecPerTrk
        DW      2               ; BPB_NumHeads
        DD      0               ; BPB_HiddSec
        DD      2880            ; BPB_TotSec32

        ; FAT12/16におけるオフセット36以降のフィールド
        DB      0x00            ; BS_DrvNum
        DB      0x00            ; BS_Reserved1
        DB      0x29            ; BS_BootSig

        DD      0xffffffff      ; BS_VolID
        DB      "HARIBOTEOS "   ; BS_VolLab     11B
        DB      "FAT12   "      ; BS_FilSysType 8B
        RESB    18              ; とりあえず18バイト開けておく


; START BS_BootCode 64(0x14)   448(0x1C0)
entry:
        MOV     AX, 0           ; initialize Accumulator(resister)
        MOV     SS, AX          ; Stack Segment
        MOV     SP, 0x7c00      ; Stack Pointer
        MOV     DS, AX          ; Data Segment      : 番地指定のとき重要


; load disk
        MOV     AX, 0x0820
        MOV     ES, AX          ; buffer address       0x0820
        MOV     CH, 0           ; cylinder  0
        MOV     DH, 0           ; head      0
        MOV     CL, 2           ; sector    2

readloop:
        MOV     SI, 0           ; 失敗回数を数えるレジスタ

retry:
        MOV     AH, 0x02        ; acumulator high   : 0x02 - read disk
        MOV     AL, 1           ; acumulator low    : sector    1
        MOV     BX, 0           ; buffer address    0x0000
                                ; ES:BX, ESは代入済み
        MOV     DL, 0x00        ; data low          : drive number
        INT     0x13            ; BIOS call
        JNC     next            ; jump if not carry

        ADD     SI, 1           ; increment SI
        CMP     SI, 5
        JAE     error           ; SI >= 5 then jump to error

        MOV     AH, 0x00        ; 0x00 - reset
        MOV     DL, 0x00        ; A drive
        INT     0x13            ; reset drive
        JMP     retry

next:
        ; add 0x20 to ES
        ; 代わりにBXに512を足してもよい
        MOV     AX, ES          ; 0x20だけアドレスを進める
        ADD     AX, 0x0020      ; 512 / 16 = 0x20
        MOV     ES, AX

        ; increment CL (sector number)
        ADD     CL, 1
        CMP     CL, 18
        JBE     readloop

        ; ディスクのウラ面
        MOV     CL, 1           ; reset sector
        ADD     DH, 1           ; reverse HEAD
        CMP     DH, 2
        JB      readloop

        ; next Cylinder
        mov     DH, 0           ; reset HEAd
        ADD     CH, 1           ; cylinder += 1
        CMP     CH, CYLS
        JB      readloop

; ブートセクタの読み込みが終わったのでOS本体を実行
        JMP     0xc200

error:
        MOV     SI, msg

putloop:
        MOV     AL, [SI]        ; BYTE (accumulator low)
        ADD     SI, 1           ; increment
        CMP     AL, 0           ; 終了条件
        JE      fin             ; jump to fin if equal to 0

        MOV     AH, 0x0e        ; 1 char-function
        MOV     BX, 15          ; color code
        INT     0x10            ; interrupt, call BIOS
        JMP     putloop

fin:
        HLT                     ; 何かあるまでCPUを停止させる
        JMP     fin             ; 無限ループ

msg:
        DB      0x0a, 0x0a
        DB      "load error"
        DB      0x0a
        DB      0               ; end point

        RESB    0x7dfe - 0x7c00 - ($ - $$)  ; 現在の場所から0x1fdまで(残りの未使用領域)を0で埋める。
                                            ; 0x7c00スタートなのでその分を引いている
; END BS_BootCode
        DB      0x55, 0xaa      ; BS_BootSign, boot signature
```

実行

```:terminal(入力)
$ make run
```

実行結果
![image.png](https://qiita-image-store.s3.amazonaws.com/0/195174/edeca5ee-40e5-805b-519b-e4f9c53dd391.png)

これでは上手く読み込まれたのかどうかがわからないので次で確認しましょう。


# OS本体の動作を確認(p.59) - harib00g
説明はテキストで十分かと思うのでここではコードだけ。

```:haribote.asm
; haribote-os
; TAB=4
        ORG     0xc200      ; 0xc200 <- 0x8000 + 0x4200
                            ; Where on memory this program will be loaded

        MOV     AL, 0x13    ; VGA graphics, 320x200x8bit
        MOV     AH, 0x00
        INT     0x10

fin:
        HLT
        JMP     fin
```

`ipl.asm`は１０セクタだけ読み込むコードであることを明示するために`ipl10.asm`に名前変更（内容に変更はないが一応載せておく）

```:ipl10.asm
; haribote-ipl
; TAB=4

CYLS    EQU     10              ; どこまで読み込むか (CYLinderS)

        ORG     0x7c00          ; このプログラムがメモリ上のどこに読み込まれるか

; discription for floppy disk
        JMP     entry           ; BS_JmpBoot
        DB      0x90            ; BS_JmpBoot
        DB      "HARIBOTE"      ; BS_OEMName    8B
        DW      512             ; BPB_BytsPerSec
        DB      1               ; BPB_SecPerClu
        DW      1               ; BPB_RevdSecCnt    : このBPBを含むブートセクタのみ
        DB      2               ; BPB_NumFATs       : FATの個数 (このフィールドの値は常に2に設定すべきである)
        DW      224             ; BPB_RootEntCnt
        DW      2880            ; BPB_TotSec16
        DB      0xf0            ; BPB_Media
        DW      9               ; BPB_FATSz16
        DW      18              ; BPB_SecPerTrk
        DW      2               ; BPB_NumHeads
        DD      0               ; BPB_HiddSec
        DD      2880            ; BPB_TotSec32

        ; FAT12/16におけるオフセット36以降のフィールド
        DB      0x00            ; BS_DrvNum
        DB      0x00            ; BS_Reserved1
        DB      0x29            ; BS_BootSig

        DD      0xffffffff      ; BS_VolID
        DB      "HARIBOTEOS "   ; BS_VolLab     11B
        DB      "FAT12   "      ; BS_FilSysType 8B
        RESB    18              ; とりあえず18バイト開けておく


; START BS_BootCode 64(0x14)   448(0x1C0)
entry:
        MOV     AX, 0           ; initialize Accumulator(resister)
        MOV     SS, AX          ; Stack Segment
        MOV     SP, 0x7c00      ; Stack Pointer
        MOV     DS, AX          ; Data Segment      : 番地指定のとき重要


; load disk
        MOV     AX, 0x0820
        MOV     ES, AX          ; buffer address       0x0820
        MOV     CH, 0           ; cylinder  0
        MOV     DH, 0           ; head      0
        MOV     CL, 2           ; sector    2

readloop:
        MOV     SI, 0           ; 失敗回数を数えるレジスタ

retry:
        MOV     AH, 0x02        ; acumulator high   : 0x02 - read disk
        MOV     AL, 1           ; acumulator low    : sector    1
        MOV     BX, 0           ; buffer address    0x0000
                                ; ES:BX, ESは代入済み
        MOV     DL, 0x00        ; data low          : drive number
        INT     0x13            ; BIOS call
        JNC     next            ; jump if not carry

        ADD     SI, 1           ; increment SI
        CMP     SI, 5
        JAE     error           ; SI >= 5 then jump to error

        MOV     AH, 0x00        ; 0x00 - reset
        MOV     DL, 0x00        ; A drive
        INT     0x13            ; reset drive
        JMP     retry

next:
        ; add 0x20 to ES
        ; 代わりにBXに512を足してもよい
        MOV     AX, ES          ; 0x20だけアドレスを進める
        ADD     AX, 0x0020      ; 512 / 16 = 0x20
        MOV     ES, AX

        ; increment CL (sector number)
        ADD     CL, 1
        CMP     CL, 18
        JBE     readloop

        ; ディスクのウラ面
        MOV     CL, 1           ; reset sector
        ADD     DH, 1           ; reverse HEAD
        CMP     DH, 2
        JB      readloop

        ; next Cylinder
        mov     DH, 0           ; reset HEAd
        ADD     CH, 1           ; cylinder += 1
        CMP     CH, CYLS
        JB      readloop

; ブートセクタの読み込みが終わったのでOS本体を実行
        JMP     0xc200

error:
        MOV     SI, msg

putloop:
        MOV     AL, [SI]        ; BYTE (accumulator low)
        ADD     SI, 1           ; increment
        CMP     AL, 0           ; 終了条件
        JE      fin             ; jump to fin if equal to 0

        MOV     AH, 0x0e        ; 1 char-function
        MOV     BX, 15          ; color code
        INT     0x10            ; interrupt, call BIOS
        JMP     putloop

fin:
        HLT                     ; 何かあるまでCPUを停止させる
        JMP     fin             ; 無限ループ

msg:
        DB      0x0a, 0x0a
        DB      "load error"
        DB      0x0a
        DB      0               ; end point

        RESB    0x7dfe - 0x7c00 - ($ - $$)  ; 現在の場所から0x1fdまで(残りの未使用領域)を0で埋める。
                                            ; 0x7c00スタートなのでその分を引いている
; END BS_BootCode
        DB      0x55, 0xaa      ; BS_BootSign, boot signature
```

`ipl10.asm`に名前を変更したので`Makefile`も名前変更

```:Makefile
default:
    make img

ipl10.bin : ipl10.asm Makefile
    nasm ipl10.asm -o ipl10.bin -l ipl10.lst

haribote.sys : haribote.asm Makefile
    nasm haribote.asm -o haribote.sys -l haribote.lst

haribote.img : ipl10.bin haribote.sys Makefile
    mformat -f 1440 -C -B ipl10.bin -i haribote.img ::
    mcopy haribote.sys -i haribote.img ::

asm :
    make -r ipl10.bin

img :
    make -r haribote.img

run :
    make img
    qemu-system-i386 -fda haribote.img  # "-fda" for floppy disk
```

では実行

```:terminal(入力)
$ make run
```

実行結果
![image.png](https://qiita-image-store.s3.amazonaws.com/0/195174/ee6a9771-e348-e2dd-e0e0-26549092afdb.png)


やったー！真っ黒な画面を出力できた〜！
これで正常に読み込めていることがわかりましたね。んではつぎー。


# ３２ビットモードへ(p.61) - harib00h



