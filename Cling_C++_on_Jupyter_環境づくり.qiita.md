# Cling C++ on Jupyter 環境づくり

# はじめに
　C++がJupyter上で動かせるという話を聞いて実行してみたいと思ったのですが環境構築に手間取ったので書いておきます。（他の記事に書いてあるとおりにやったらエラーが出て断念）
　このやり方は記事を探してもがいていて、手法を組み合わせたらなんとなくできたものなので正規のやり方では無い可能性の方が高いです。そのことを承知の上ご覧ください。
<a href="https://github.com/root-project/cling">Cling - The Interactive C++ Interpreter - GitHub</a>

# 環境
- Ubuntu16.04 LTS
- Python3 (install済み)
- Jupyter (install済み)

# Clingダウンロード
<a href="https://cdn.rawgit.com/root-project/cling/master/www/index.html">Cling's Web Page</a>
上のリンクがClingの公式ページです。HOMEの下の法にDOWNLOADがあるので、<a herf="https://root.cern.ch/download/cling/">そのリンク</a>先から自分のOSに合ったものをダウンロードします。
　自分はcling_2018-03-02_ubuntu16.tar.bz2をホームディレクトリにダウンロードしました。

```
~/cling_2018-03-02_ubuntu16.tar.bz2
```

まず、解凍・展開します。

```
$ cd ~
$ bzip2 -d cling_2018-03-02_ubuntu16.tar.bz2
$ tar xvf cling_2018-03-02_ubuntu16.tar
```

すると、`cling_2018-03-02_ubuntu16/`ができます。

# Cling実行
まずは普通にClingを実行してみます。

```
$ ./cling_2018-03-02_ubuntu16/bin/cling

****************** CLING ******************
* Type C++ code and press enter to run it *
*             Type .q to exit             *
*******************************************
[cling]$ 
```

指示どおり`.q`あるいは`Ctrl+c`を実行すると抜けます。
次にこのパスを通しておきましょう。<username>は自身のユーザー名に置き換えてください。

```
$ export PATH=/home/<username>/cling_2018-03-02_ubuntu16/bin:$PATH
```

　ただ、これではシェルを閉じてしまえばまた実行しなければならなくなるのでログインシェルの一番下あたりに上の一文を追加しておくといいと思います。
　たとえば、bashを使っていれば`.bashrc`ファイルを開いて一番下に追記

# Jupyter-kernel準備
それではJupyter上で実行できるようにします。

```
$ cd cling_2018-03-02_ubuntu16/share/cling/Jupyter/kernel
$ sudo pip3 install -e .
$ jupyter-kernelspec install --user cling-cpp11
$ jupyter-kernelspec install --user cling-cpp14
$ jupyter-kernelspec install --user cling-cpp17
```

# Jupyter実行
```
$ cd
$ jupyter-notebook
# (または jupyter notebook)
```
さて、ここまでやればJupyter上で実行できることが確認できます。
ちゃんとKearnelもReadyになりました。

- C++11
- C++14
- C++17

が確認できればOKです。
![cling-on-jupyter.png](https://qiita-image-store.s3.amazonaws.com/0/195174/c40c2025-f560-bdef-5e59-5b50c806cdd8.png)


![cling-on-jupyter_sample.png](https://qiita-image-store.s3.amazonaws.com/0/195174/98c3b3ed-2487-5512-cec7-822412eca6bc.png)


今回はこんな感じです。
お疲れ様でした。

# 参考
<a href="http://shuvomoy.github.io/blog/programming/2016/08/04/Cpp-kernel-for-Jupyter.html">C++ Kernel for Jupyter Notebook</a>


