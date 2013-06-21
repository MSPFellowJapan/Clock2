ステップ2　時計を作ろう！2
====================
このステップではステップ１に引き続き時計を作成します．このステップではWinJSのライブラリを使用してOOPを試してみます．
このステップでは以下を習得します．

 * [WinJS.Binding](http://msdn.microsoft.com/ja-jp/library/windows/apps/hh700356.aspx)
 * [WinJS.Binding.Template](http://msdn.microsoft.com/ja-jp/library/windows/apps/hh700356.aspx)
 * [WinJS.Class](http://msdn.microsoft.com/ja-jp/library/windows/apps/hh967790.aspx)
 * [WinJS.Promise](http://msdn.microsoft.com/ja-jp/library/windows/apps/hh700334.aspx)

##プロジェクトの作成
--------------------
ファイル->新規作成->プロジェクトを選択してプロジェクトを作成します．
固定レイアウトアプリケーションでプロジェクト名をClock2とし，作成します．

##構造の作成
------------------
ソリューションエクスプローラからdefault.htmlを開きます．

```
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <meta charset="utf-8" />
    <title>Clock2</title>

    <!-- WinJS 参照 -->
    <link href="//Microsoft.WinJS.1.0/css/ui-dark.css" rel="stylesheet" />
    <script src="//Microsoft.WinJS.1.0/js/base.js"></script>
    <script src="//Microsoft.WinJS.1.0/js/ui.js"></script>

    <!-- Clock2 参照 -->
    <link href="/css/default.css" rel="stylesheet" />
    <script src="/js/default.js"></script>
</head>
<body>
    <div data-win-control="WinJS.UI.ViewBox">
        <div class="fixedlayout">
        	<p>コンテンツをここに挿入</p>    
        </div>
    </div>
</body>
</html>

```

`<p>コンテンツをここに挿入</p>`を削除し`<div class="fixedlayout">`に`id="container"`を追加します．

```
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <meta charset="utf-8" />
    <title>Clock2</title>

    <!-- WinJS 参照 -->
    <link href="//Microsoft.WinJS.1.0/css/ui-dark.css" rel="stylesheet" />
    <script src="//Microsoft.WinJS.1.0/js/base.js"></script>
    <script src="//Microsoft.WinJS.1.0/js/ui.js"></script>

    <!-- Clock2 参照 -->
    <link href="/css/default.css" rel="stylesheet" />
    <script src="/js/default.js"></script>
</head>
<body>
    <div data-win-control="WinJS.UI.ViewBox">
        <!--<div class="fixedlayout">-->
        <div id="container" class="fixedlayout">
            
        </div>
    </div>
</body>
</html>
```

次に`Clock`のテンプレートコード作成します．

```
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <meta charset="utf-8" />
    <title>Clock2</title>

    <!-- WinJS 参照 -->
    <link href="//Microsoft.WinJS.1.0/css/ui-dark.css" rel="stylesheet" />
    <script src="//Microsoft.WinJS.1.0/js/base.js"></script>
    <script src="//Microsoft.WinJS.1.0/js/ui.js"></script>

    <!-- Clock2 参照 -->
    <link href="/css/default.css" rel="stylesheet" />
    <script src="/js/default.js"></script>
</head>
<body>
    <!--追加コード-->
    <div id="clockTemplate" data-win-control="WinJS.Binding.Template">
        <p>
            <span data-win-bind="innerText: hours"></span>:
            <span data-win-bind="innerText: minutes"></span>:
            <span data-win-bind="innerText: seconds"></span>
        </p>
    </div>
    <!--追加コード-->
    <div data-win-control="WinJS.UI.ViewBox">
        <!--<div class="fixedlayout">-->
        <div id="container" class="fixedlayout">
            
        </div>
    </div>
</body>
</html>
```

##ロジックの作成
--------------------
ソリューションエクスプローラからjs->default.jsを開き，以下のようにコードを追加します．

```js:default.js
// 固定レイアウト テンプレートの概要については、次のドキュメントを参照してください:
// http://go.microsoft.com/fwlink/?LinkId=232508
(function () {
    "use strict";

    WinJS.Binding.optimizeBindingReferences = true;

    var app = WinJS.Application;
    var activation = Windows.ApplicationModel.Activation;

    /*----追加コード----*/
    var Clock = WinJS.Class.define(function () {
    }, {
        hours: "00",
        minutes: "00",
        seconds: "00",
        update: function (date) {
            var h, m, s;
            h = date.getHours();
            m = date.getMinutes();
            s = date.getSeconds();
            this.hours = "" + h;
            this.minutes = ((m < 10) ? "0" : "") + m;
            this.seconds = ((s < 10) ? "0" : "") + s;
        },
        start: function () {
            var update, that = this;
            update = function () {
                that.update(new Date());
                setTimeout(update, 1000);
            }
            update();
        }
    });
    /*----追加コード----*/

    app.onactivated = function (args) {
        if (args.detail.kind === activation.ActivationKind.launch) {
            if (args.detail.previousExecutionState !== activation.ApplicationExecutionState.terminated) {
                // TODO: このアプリケーションは新しく起動しました。ここでアプリケーションを
                // 初期化します。
            } else {
                // TODO: このアプリケーションは中断状態から再度アクティブ化されました。
                // ここでアプリケーションの状態を復元します。
            }
            //args.setPromise(WinJS.UI.processAll()); //以下に変更
            /*----追加コード----*/
            args.setPromise(WinJS.UI.processAll().done(function () {
                var clock = WinJS.Binding.as(new Clock());
                clockTemplate.winControl.render(clock, container);
                clock.start();
            }));
            /*----追加コード----*/
        }
    };

    app.oncheckpoint = function (args) {
        // TODO: このアプリケーションは中断しようとしています。ここで中断中に
        // 維持する必要のある状態を保存します。中断中に自動的に保存され、
        // 復元される WinJS.Application.sessionState オブジェクトを使用
        // できます。アプリケーションを中断する前に非同期操作を完了する
        // 必要がある場合は、args.setPromise() を呼び出して
        // args.setPromise().
    };

    app.start();
})();

```

これで実行してみましょう．　次のように表示されたでしょうか．

![とりあえず実行](https://dl.dropboxusercontent.com/u/59753988/StoreappSampleThumb/2.png)

小さいですが確かに動いてますね！
それではステップ1同様にスタイルを適用してみましょう.
ソリューションエクスプローラ->css->default.cssを開き，次のように書き足します.

```css:default.css
html, body {
    height: 100%;
    margin: 0;
    padding: 0;
}

body {
    -ms-flex-align: center;
    -ms-flex-direction: column;
    -ms-flex-pack: center;
    display: -ms-flexbox;
}

.fixedlayout {
    -ms-grid-columns: 1fr;
    -ms-grid-rows: 1fr;
    display: -ms-grid;
    height: 768px;
    width: 1024px;
}

.fixedlayout {
    -ms-grid-columns: 1fr;
    -ms-grid-rows: 1fr;
    /*display: -ms-grid;*//*削除*/
    height: 768px;
    width: 1024px;
    /* ----以下追加コード---- */
	display: -ms-flexbox;
	-ms-flex-align:center;
	-ms-flex-pack:center;
    font-size: 200px;
	/* ----以上追加コード---- */
}
@media screen and (-ms-view-state: fullscreen-landscape) {
}

@media screen and (-ms-view-state: filled) {
}

@media screen and (-ms-view-state: snapped) {
}

@media screen and (-ms-view-state: fullscreen-portrait) {
}
```

これで実行するとステップ1と同様な時計が動いてるはずです．

##解説
------------------------------------------
それでは順に追って解説します．
まず，Templateについて解説します．先ほど，HTMLに以下のコードを追加しました．

```
<div id="clockTemplate" data-win-control="WinJS.Binding.Template">
    <p>
        <span data-win-bind="innerText: hours"></span>:
        <span data-win-bind="innerText: minutes"></span>:
        <span data-win-bind="innerText: seconds"></span>
    </p>
</div>
```
ステップ１の`ViewBox`の時と同様にWinJSのライブラリですので`data-win-control`属性に設定します．テンプレートは`WinJS.Binding.Template`です．テンプレート内にある要素が`Template`オブジェクトの`render`メソッドで生成され，生成されるときに与えるデータとバインドすることが可能です．
バインドはバインドしたい要素に`data-win-bind`属性をつけて行います．`innerText`などDOMのプロパティにアクセスできます．ここでは`hours`,`minutes`,`seconds`をそれぞれkeyとしたデータをバインドします．

次にバインドするデータのクラスを作成します．
default.jsに`Clock`クラスを`WinJS.Class.define`を使って作成します．
`WinJS.Class.define`は第一引数がコンストラクタ，第二引数がインスタンスメンバ，第三引数が静的メンバです．

```
var Clock = WinJS.Class.define(function () {
}, {
    hours: "",
    minutes: "",
    seconds: "",
});
```
これで`hours`,`minutes`,`seconds`をプロパティに持ったクラスの完成です．せっかくクラスなのでメソッドを追加してみましょう．

```
var Clock = WinJS.Class.define(function () {
}, {
    hours: "",
    minutes: "",
    seconds: "",
    update: function (date) {
        var h, m, s;
        h = date.getHours();
        m = date.getMinutes();
        s = date.getSeconds();
        this.hours = "" + h;
        this.minutes = ((m < 10) ? "0" : "") + m;
        this.seconds = ((s < 10) ? "0" : "") + s;
    },
    start: function () {
        var update, that = this;
        update = function () {
            that.update(new Date());
            setTimeout(update, 1000);
        }
        update();
    }
});
```

`update`メソッドは`Date`を引数として受け取り`hours`,`minutes`,`seconds`を更新するメソッド，
`start`メソッドは`setTimeout`を用いて`update`メソッドを1000ミリ秒おきに実行し，時計の動作を開始します．

さて，次にこのクラスからインスタンスを生成し，テンプレートと合わせて時計を生成します．
WinJSのライブラリは`WinJS.UI.processAll`によって初期化されるので，テンプレートを使用するときはこれの後に実行する必要があります．
`WinJS.UI.processAll`はプロミスを返すので`done`または`then`を用いてつなぐことができます．
ここでは`done`を使います．

```
args.setPromise(WinJS.UI.processAll().done(function () {
    var clock = WinJS.Binding.as(new Clock());
    clockTemplate.winControl.render(clock, container);
    clock.start();
}));
```

`new Clock()`で`Clock`のインスタンスを生成します．そして，このインスタンスを`WinJS.Binding.as`にかけることによって`Binding`オブジェクトに変換することができます．`Binding`オブジェクトに変換することによってオブジェクトの変更がDOMオブジェクトに伝わるようになります．よくわからない場合は`WinJS.Binding.as`をやめ，`var clock = new Clock()`で実行してみましょう．時計の表示が更新されないはずです．
次にテンプレートのDOMオブジェクトである`clockTemplate`から`render`メソッドを呼び出します．`data-win-control`属性のデータにアクセスするにはDOMオブジェクト内の`winControl`を参照します．
`render`メソッドは第一引数がバインドするオブジェクト，第二引数が展開先です．今回は`container`内に展開します．
最後に`start`メソッドを呼んで時計の動作を開始します．
