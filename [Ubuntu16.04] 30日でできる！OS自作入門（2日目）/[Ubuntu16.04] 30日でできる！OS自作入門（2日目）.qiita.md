# [Ubuntu16.04] 30日でできる！OS自作入門（2日目）


前後の記事
- <a herf="https://qiita.com/pollenjp/items/a13cd764cc739a0ca9df">[Ubuntu16.04] 30日でできる！OS自作入門（１日目）</a>

# 目的
"30日でできる！OS自作入門"の内容をUbuntu(Linux)で実行するには本の内容だけでは厳しいので調べた結果をメモ。（リンクと動作確認済みコード・コメント）
本を読む上でLinuxを使う方の参考になればと思っております。
Ubuntu 16.04 LTS

# 参考
先に参考を載せておきます。

- <a href="http://tsurugidake.hatenablog.jp/entry/2017/08/15/202414">Linuxで書くOS自作入門 2日目 - Tsurugidake's diary</a>
- <a href="http://lv4.hateblo.jp/entry/2011/10/15/100453">OS自作入門 onLinux 2日目 - Handwriting</a>
- <a href="http://d.hatena.ne.jp/dorayakitaro/20090128/p1">LinuxでのOS自作入門～Chapter2（2日目）～ - どらや記</a>
- <a href="https://syusui.tumblr.com/post/109777087853/30%E6%97%A5%E3%81%A7%E3%81%A7%E3%81%8D%E3%82%8Bos%E8%87%AA%E4%BD%9C%E5%85%A5%E9%96%80%E3%82%92linux%E3%81%A7%E3%82%84%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B-2%E6%97%A5%E7%9B%AE">Akitsushima Design — 『30日でできる！OS自作入門』をLinuxでやってみる 2日目</a>
- <a href="http://elm-chan.org/docs/fat.html#bpb">ブート セクタとBPB</a>



# helloos3
- INT命令の際の参考サイトは移転していました：http://oswiki.osask.jp/?%28AT%29BIOS
- memory mapの参考サイトも移転：http://oswiki.osask.jp/?%28AT%29memorymap

ブートセクタ説明用のコードはもう少し下で掲載

```:helloos3.asm
; hello-os
; TAB=4

        ORG     0x7c00          ; このプログラムがメモリ上のどこによみこまれるのか

; ディスクのための記述

        JMP     entry
        DB      0x90
        DB      "HELLOIPL"      ; ブートセレクタの名前を自由にかいていよい  (8Byte)
        DW      512             ; 1セクタの大きさ                           (512にしなければならない)
        DB      1               ; クラスタの大きさ                          (1セクタにしなければならない)
        DW      1               ; FATがどこから始まるか                     (普通は1セクタ目からにする)
        DB      2               ; FATの個数                                 (2にしなければならない)
        DW      224             ; ルートディレクトリ領域の大きさ            (普通は224エントリにする)
        DW      2880            ; このドライブの大きさ                      (2880セクタにしなければならない)
        DB      0xf0            ; メディアタイプ                            (0xf0にしなければならない)
        DW      9               ; FAT領域の長さ                             (9セクタにしなければならない)
        DW      18              ; 1トラックにいくつのセクタがあるか         (18にしなければならない)
        DW      2               ; ヘッドの数                                (2にしなければならない)
        DD      0               ; パーティションを使っていないのでここは必ず0
        DD      2880            ; このドライブの大きさをもう一度書く
        DB      0, 0, 0x29      ; よくわからないけどこの値にしておくといいらしい
        DD      0xffffffff      ; たぶんボリュームシリアル番号
        DB      "HELLO-OS   "   ; ディスクの名前                            (11Byte)
        DB      "FAT12   "      ; フォーマットの名前                        (8Byte)
        RESB    18              ; とりあえず18バイト開けておく

; Program Main Body
entry:
        MOV     AX, 0            ; レジスタの初期化
        MOV     SS, AX
        MOV     SP, 0x7c00
        MOV     DS, AX
        MOV     ES, AX

        MOV     SI, msg
putloop:
        MOV     AL, [SI]        ; BYTE (accumulator low)
        ADD     SI, 1           ; increment
        CMP     AL, 0           ; compare (<end msg>)
        JE      fin             ; jump to fin if equal to 0
        MOV     AH, 0x0e        ; AH = 0x0e
        MOV     BX, 15          ; BH = 0, BL = <color code>
        INT     0x10            ; interrupt BIOS
        JMP     putloop
fin:
        HLT
        JMP     fin

msg:
        DB      0x0a, 0x0a
        DB      "hello, world"
        DB      0x0a
        DB      0               ; end msg

        ;RESB    0x7dfe-($-$$)  ; これだとエラーが出た。。。
        RESB    0x7dfe-0x7c00-($-$$)

        DB      0x55, 0xaa

; ブート以外の記述

        DB      0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
        RESB    4600
        DB      0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
        RESB    1469432
```

## helloos3で詰まったところ

```
        ;RESB    0x7dfe-($-$$)  ; これだとエラーが出た。。。
```

サンプルコードでは`RESB    0x7dfe-$`と記述されていたが、そのまま（上のように）`qemu`で実行したところ以下のような挙動が起きた。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/195174/c69753fb-aa55-02ce-e500-b1f142835534.png)

![image.png](https://qiita-image-store.s3.amazonaws.com/0/195174/3cb65855-8d0b-85d5-9966-0843a5ecbb4d.png)

値からさらに`0x7c00`を引いたら治った。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/195174/ecd04104-b648-83b6-66bf-1634e63f3010.png)


# ブートセクタ

ブートセクタについて少し詳しく見ていこうと思います。面倒な方は次の節に飛んでください。

- <a href="http://elm-chan.org/docs/fat.html#bpb">ブート セクタとBPB</a>
- <a herf="http://tsurugidake.hatenablog.jp/entry/2017/08/15/202414">Linuxで書くOS自作入門 2日目 - Tsurugidake's diary</a>

今まで記述してきた`helloos.asm`の中で

```:helloos3.img
        ......
; ディスクのための記述

        JMP     entry
        DB      0x90
        ......
```

という箇所についての解説ですが、<a href="http://elm-chan.org/docs/fat.html#bpb">ブート セクタとBPB</a>の中に出てくる表にまとまっています。
以下のコードのコメントに名前を書いて置きます。

```:helloos3.asm
; hello-os
; TAB=4

        ORG     0x7c00          ; このプログラムがメモリ上のどこによみこまれるのか

; ディスクのための記述
                                                    offset  byte
        JMP     entry           ; BS_JmpBoot        0
        DB      0x90            ; BS_JmpBoot                1
        DB      "HELLOIPL"      ; BS_OEMName        3       8
        DW      512             ; BPB_BytsPerSec    11      2 : バイト単位のセクタ サイズ
        DB      1               ; BPB_SecPerClus    13      1 : アロケーション ユニット(<-クラスタ)(割り当て単位)当たりのセクタ数
        DW      1               ; BPB_RsvdSecCnt    14      2 : 予約領域のセクタ数 (少なくともこのBPBを含むブートセクタそれ自身が存在するため、0であってはならない)
        DB      2               ; BPB_NumFATs       16      1 : FATの個数 (このフィールドの値は常に2に設定すべきである)
        DW      224             ; BPB_RootEntCnt    17      2 : ルートディレクトリに含まれるディレクトリエントリの数を示す
        DW      2880            ; BPB_TotSec16      19      2 : ボリュームの総セクタ数(古い16ビット フィールド)
        DB      0xf0            ; BPB_Media         21      1 : メディアタイプ(区画分けされた固定ディスク ドライブでは0xF8が標準値である。区画分けされないリムーバブル メディアでは0xF0がしばしば使われる)
        DW      9               ; BPB_FATSz16       22      2 : 1個のFATが占めるセクタ数
        DW      18              ; BPB_SecPerTrk     24      2 : トラック当たりのセクタ数
        DW      2               ; BPB_NumHeads      26      2 : ヘッドの数
        DD      0               ; BPB_HiddSec       28      4 : ストレージ上でこのボリュームの手前に存在する隠れた物理セクタの数(ボリュームがストレージの先頭から始まる場合(つまりフロッピー ディスクなど区画分けされていないもの)では常に0であるべきである。)
        DD      2880            ; BPB_TotSec32      32      4 : ボリュームの総セクタ数(新しい32ビット フィールド)


; FAT12/16におけるオフセット36以降のフィールド
        ;DB      0, 0, 0x29      ; 以下の３行に分けて記述
        DB      0x00            ; BS_DrvNum         36      1
        DB      0x00            ; BS_Reserved1      37      1
        DB      0x29            ; BS_BootSig        38      1

        DD      0xffffffff      ; BS_VolID          39      4 : ボリュームシリアル番号
        DB      "HELLO-OS   "   ; BS_VolLab         43      11 : ディスクの名前(ルート ディレクトリに記録される11バイトのボリューム ラベルに一致する)
        DB      "FAT12   "      ; BS_FilSysType     54      8 : フォーマットの名前
        RESB    18              ; とりあえず18バイト開けておく

; START BS_BootCode                                 64      448
; (ブートストラップ プログラム。システム依存フィールドで、未使用時はゼロで埋める。)
entry:
        MOV     AX, 0            ; レジスタの初期化
        MOV     SS, AX
        MOV     SP, 0x7c00
        MOV     DS, AX
        MOV     ES, AX

        MOV     SI, msg
putloop:
        MOV     AL, [SI]        ; BYTE (accumulator low)
        ADD     SI, 1           ; increment
        CMP     AL, 0           ; compare (<end msg>)
        JE      fin             ; jump to fin if equal to 0
        MOV     AH, 0x0e        ; AH = 0x0e
        MOV     BX, 15          ; BH = 0, BL = <color code>
        INT     0x10            ; interrupt BIOS
        JMP     putloop
fin:
        HLT
        JMP     fin

msg:
        DB      0x0a, 0x0a
        DB      "hello, world"
        DB      0x0a
        DB      0               ; end msg

        ;RESB    0x7dfe-($-$$)  ; これだとエラーが出た。。。
        RESB    0x7dfe-0x7c00-($-$$)    ; 現在の場所から0x1fdまで(残りの未使用領域)を0で埋める。
; END BS_BootCode

        DB      0x55, 0xaa      ; BS_BootSign       510     2 : 以下の記述と同様
        ;DW      0xAA55
```

この`Makefile`作成して以下のコマンド実行でエミュレータでいつものHello, Worldが出力される。

```:terminal(入力)
$ make run
```

※makeコマンドはUbuntuの中にデフォルトで入っている。


# Makefile
説明はおおよそ本に書いてあるとおり、一応Ubuntuで最低限動作するコードを載せときます。

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
	qemu-system-i386 helloos.img
```

