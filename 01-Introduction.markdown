はじめに
========

これは柔軟で、速く、セキュアな PHP 言語のテンプレートである Twig のドキュメントです。

Smarty、Django もしくは Jinja などのほかのテキストベースのテンプレート言語に触れる機会があれば、Twig の扱いにすぐ慣れるでしょう。PHP の原則を忠実に守りテンプレート環境に役立つ機能性を追加することでこれはデザイナーと開発者の両方に親しみやすいものになっています。

キーフィーチャは次のとおりです...

 * *速い*: Twig はテンプレートをプレーンな最適化されたプレーンな PHP コードにコンパイルします。PHP コードと比べたオーバーヘッドは最小限に減りました。

 * *安全である*: Twig は信頼できないテンプレートコードを評価するサンドボックスモードを持ちます。このことによってユーザーがテンプレートデザインを修正することが許可されるアプリケーションのテンプレート言語で使えるようになっています。

 * *柔軟である*: Twig は柔軟なレクサーとパーサーによって支えられています。これによって開発者は独自のカスタムタグとフィルターを定義し、また独自の DSL を作ることができます。

前提要件
--------

Twig を実行するには少なくとも **PHP 5.2.4** が必要です。

インストール方法
---------------

Twig をインストールする方法は複数あります。何をすればよいのかわからなければ、タールボールからインストールしてください。

### タールボールリリースから

 1. [ダウンロードページ](http://www.twig-project.org/installation)から最新のタールボールをダウンロードする
 2. タールボールを展開する
 3. ファイルをあなたのプロジェクトのどこかに移動させる

### 開発バージョンをインストールする

 1. Subversion をインストールする
 2. `svn co http://svn.twig-project.org/trunk/ twig`

### PEAR パッケージをインストールする

 1. PEAR をインストールする
 2. pear channel-discover pear.twig-project.org
 3. pear install twig/Twig (もしくは pear install twig/Twig-beta)

基本的な API の使い方
---------------------

この節では Twig の PHP API の手短な説明をします。

Twig を使う最初のステップはオートローダーを登録することです:

    [php]
    require_once '/path/to/lib/Twig/Autoloader.php';
    Twig_Autoloader::register();

`/path/to/lib/` パスを Twig がインストールされているパスに置き換えます。

>**NOTE**
>Twig は PEAR のクラス命名規約に従います。このことは Twig クラスロード機能をあなた独自のオートローダーに簡単に統合できることを意味します。

    [php]
    $loader = new Twig_Loader_String();
    $twig = new Twig_Environment($loader);

    $template = $twig->loadTemplate('Hello {{ name }}!');

    $template->display(array('name' => 'Fabien'));

Twig はテンプレートの位置を割り出すのにローダークラス (`Twig_Loader_String`) を使い、コンフィギュレーションを保存するのに環境クラス (`Twig_Environment`) を使います。

`loadTemplate()` メソッドはテンプレートの位置を割り出しロードし`display()` メソッドでレンダリングするのに最適なテンプレートオブジェクト (`Twig_Template`) を返すためにローダーを使います。

Twig はファイルシステムローダーも備えています:

    [php]
    $loader = new Twig_Loader_Filesystem('/path/to/templates');
    $twig = new Twig_Environment($loader, array(
      'cache' => '/path/to/compilation_cache',
    ));

    $template = $twig->loadTemplate('index.html');
