開発者のための Twig
===================

この章では、Twigのテンプレート言語についてではなくそのAPIについて説明します。Twig のテンプレートを書く人にとってではなく、アプリケーションへのテンプレートの窓口を実装する人にとって、この章はリファレンスとして最も役立つでしょう。

基本
------

Twig は **環境** と呼ばれる Twig_Environment クラスのオブジェクトを利用します。このクラスのインスタンスは、設定やエクステンションを保管したり、ファイルシステムかもしくは他の場所からテンプレートを読み込むのに利用されます。

ほとんどのアプリケーションは、最初にひとつの Twig_Environment オブジェクトを生成し、テンプレートを読み込むのにそれを使うでしょう。もし違った設定を複数使うときでも、複数の環境を持つのは容易です。

あなたのアプリケーションでテンプレートを読み込めるように Twig を設定する最も単純な方法は、こういった感じです:

    [php]
    require_once '/path/to/lib/Twig/Autoloader.php';
    Twig_Autoloader::register();

    $loader = new Twig_Loader_Filesystem('/path/to/templates');
    $twig = new Twig_Environment($loader, array(
      'cache' => '/path/to/compilation_cache',
    ));

このコードはデフォルトの設定を持つ環境と `/path/to/templates/` フォルダーの中からテンプレートを探すローダーを生成します。ローダーは違ったものでもいいですし、もしあなたがデータベースかもしくは他のリソースからテンプレートをロードしたい場合でも自分でローダーを書くことができます。

>**CAUTION**
>Twig 0.9.3 より前では、 `cache` オプションは存在しておらず、キャッシュディレクトリのパスはローダーの 2 番目の引数に渡されていました。

-

>**NOTE**
>環境の 2 番目の引数はオプションの配列であることに注目してください。`cache` オプションは、コンパイル済みのキャッシュを格納するディレクトリで、Twig は続いて起こるリクエストでのパースフェーズを避けるためにコンパイルされたテンプレートをキャッシュします。このキャッシュは、あなたが追加したいかもしれない評価済みのテンプレートのキャッシュとは違います。そういった必要がある場合は、あなたは任意の PHP キャッシュライブラリを使うことができます。このことは、すでに評価されたテンプレートをキャッシュすることとは違います。

この環境からテンプレートを読み込むには、 `Twig_Template` インスタンスを返す `loadTemplate()` メソッドを呼び出すだけです:

    [php]
    $template = $twig->loadTemplate('index.html');

いくつかの変数とともにテンプレートを評価するには、`render()` メソッドを呼び出します:

    [php]
    echo $template->render(array('the' => 'variables', 'go' => 'here'));

>**NOTE**
>`display()` メソッドはテンプレートを直接出力するためのショートカットです。

環境のオプション
----------------

新しい `Twig_Environment` インスタンスを生成するとき、コンストラクターの 2 番目の引数にオプションの配列を渡すことができます:

    [php]
    $twig = new Twig_Environment($loader, array('debug' => true));

以下のオプションが利用可能です:

 * `debug`: `true` にセットしたとき、生成されたテンプレートは生成された構文木を可視化できるように `__toString()` メソッドを持ちます (デフォルトは `false`)。

 * `trim_blocks`: 次に続く命令が現れたら改行を消すというPHPのふるまいを真似します (デフォルトは `false`)。

 * `charset`: テンプレートで使われる文字コード (デフォルトは `utf-8`)。

 * `base_template_class`: 生成されたテンプレートの親となるクラス (デフォルトは `Twig_Template`)。

 * `cache`: 3 つの値の内のどれかをとります:

    * `null` (デフォルト): Twig はコンパイルされたテンプレートを置くためにシステムのテンポラリディレクトリの下にディレクトリを生成するでしょう (もしあなたの複数のプロジェクトで同じ Twig のソースコードを使うならば、2 つのプロジェクトの同じ名前を持った 2 つのテンプレートは同じキャッシュを共有するため、非推奨です)。

    * `false`: コンパイルキャッシュを全くの無効にする (非推奨)。

    * コンパイルされたテンプレートを置く絶対パス。

 * `auto_reload`: Twig を使って開発するとき、テンプレートのソースコードが変わったらコンパイルし直されるのは便利です。もし `auto_reload` オプションに値を渡さない場合、 このオプションは `debug` の値によって自動的に決定されるでしょう。

>**CAUTION**
>Twig 0.9.3 より前では、`cache` と `auto_reload` オプションは存在しません。これらはそれぞれファイルシステムローダーの 2 番目と 3 番目の引数に渡されていました。

ローダー
--------

>**CAUTION**
>このセクションは Twig 0.9.4 以上のバージョンで実装されたローダーについて説明しています。

ローダーは、たとえばファイルシステムのようなリソースからテンプレートを読み込むことを責務としています。

### コンパイルキャッシュ

全てのテンプレートローダーは、未来の再利用のためにコンパイル済みのテンプレートをファイルシステムにキャッシュすることができます。テンプレートは一度だけコンパイルされることで Twig はスピードアップします。さらに APC のような PHP アクセラレーターを使えばさらにパフォーマンスを押し上げることができます。より詳しい情報については `Twig_Environment` の `cache` と `auto_reload` オプションを見てください。

### 組み込みのローダー

以下が Twig の提供する組み込みのローダーのリストです:

 * `Twig_Loader_Filesystem`: ファイルシステムからテンプレートを読み込みます。このローダーはファイルシステムのフォルダからテンプレートを探すことができ、テンプレートを読み込むのにもっともポピュラーな方法です。

        [php]
        $loader = new Twig_Loader_Filesystem($templateDir);

 * `Twig_Loader_String`: 文字列からテンプレートを読み込みます。これはソースコードを直接渡すためのダミーのローダーです。 

        [php]
        $loader = new Twig_Loader_String();

 * `Twig_Loader_Array`: PHP の配列からテンプレートを読み込みます。名前をキーとした文字列の配列を渡します。このローダーはユニットテストの際に便利です。

        [php]
        $loader = new Twig_Loader_Array($templates);

>**TIP**
>`Array` かもしくは `String` ローダーをキャッシュとともに使う場合、テンプレートの中身が変わるたびに新しいキャッシュのためのキーが生成されることを知っておいてください (キャッシュキーはテンプレートのソースコードです)。もしキャッシュがどんどん増えていくのを見たくない場合は、自分で古いキャッシュファイルを消していく必要があります。

### 自分のローダーを作る

全てのローダーは `Twig_LoaderInterface` を実装します:

    [php]
    interface Twig_LoaderInterface
    {
      /**
       * 与えられた名前からテンプレートのソースコードを得る。
       *
       * @param  string $name 読み込むテンプレートの名前
       *
       * @return string テンプレートのソースコード
       */
      public function getSource($name);

      /**
       * 与えられたテンプレートの名前からキャッシュのために使うキーを得る。
       *
       * @param  string $name 読み込むテンプレートの名前
       *
       * @return string キャッシュキー
       */
      public function getCacheKey($name);

      /**
       * もしテンプレートがまだ有効期間の範囲にあれば true を返す。
       *
       * @param string    $name テンプレートの名前
       * @param timestamp $time キャッシュされたテンプレートが最後に更新された時間
       */
      public function isFresh($name, $time);
    }

例として `Twig_Loader_String` がどのように実装されているか見てみましょう:

    [php]
    class Twig_Loader_String implements Twig_LoaderInterface
    {
      public function getSource($name)
      {
        return $name;
      }

      public function getCacheKey($name)
      {
        return $name;
      }

      public function isFresh($name, $time)
      {
        return false;
      }
    }

`isFresh()` メソッドは、もし現在のキャッシュされたテンプレートが、与えられた最新の更新時間に対してまだ有効期限にある場場合は `true` を、そうでなければ `false` を返します。

エクステンションを使う
---------------------

Twig のエクステンションは、Twig に新しい機能を追加するパッケージです。エクステンションを使うには単に `addExtension()` メソッドを呼び出すだけです:

    [php]
    $twig->addExtension('Escaper');

Twig にはあらかじめ 3 つのエクステンションが備わっています:

 * *Core*: Twig の核となる全ての機能を定義しており、あなたが新しい環境を生成したときに自動的に登録されます。

 * *Escaper*: 出力を自動でエスケープする機能と escape/unescape ブロックのコードを使えるようにします。

 * *Sandbox*: Twig 環境にサンドボックスモードを付加します。サンドボックスは信頼できないコードの評価を安全にします。

組み込みのエクステンション
-------------------------

このセクションでは組み込みのエクステンションについている機能を説明します。

>**TIP**
>自分のエクステンションを作って Twig を拡張するのであればこの章を読んでください。

### コアエクステンション

`core` エクステンションは Twig の核となる全ての機能を定義しています:

  * タグ:

     * `for`
     * `if`
     * `extends`
     * `include`
     * `block`
     * `parent`
     * `display`
     * `filter`

  * フィルター:

     * `date`
     * `format`
     * `even`
     * `odd`
     * `urlencode`
     * `title`
     * `capitalize`
     * `upper`
     * `lower`
     * `striptags`
     * `join`
     * `reverse`
     * `length`
     * `sort`
     * `default`
     * `keys`
     * `items`
     * `escape`
     * `e`

コアエクステンションは Twig 環境に登録する必要はありません。というのもデフォルトで登録されるからです。

### エスケープエクステンション

`escaper` エクステンションは Twigに 自動的に出力をエスケープする機能を追加します。このエクステンションは `autoescape` という新しいタグと `safe` という新しいフィルターを定義します。

エスケープエクステンションを作るとき、グローバルな出力エスケープ戦略を on か off かに切り替えられます:

    [php]
    $escaper = new Twig_Extension_Escaper(true);
    $twig->addExtension($escaper);

`true` にセットすることで、テンプレート内のすべての変数は `safe` フィルターを使うときを除いてエスケープされます:

    [twig]
    {{ article.to_html|safe }}

`autoescape` タグを使うことでエスケープのモードを局所的に変えることもできます:

    [twig]
    {% autoescape on %}
      {% var %}
      {% var|safe %}     {# 変数はエスケープされない #}
      {% var|escape %}   {# 変数は 2 回エスケープされない #}
    {% endautoescape %}

>**WARNING**
>`autoescape` タグはインクルードされるファイルの中では効きません。

エスケープの規則は以下のように実装されています (Twig 0.9.5 以上):

 * 変数やフィルターの引数としてテンプレート内で使われるリテラル (整数、真偽値、配列等) はけっして自動的にエスケープされません:

        [twig]
        {{ "Twig<br />" }} {# エスケープされない #}

        {% set text as "Twig<br />" %}
        {{ text }} {# エスケープされる #}

 * エスケープはどんな他のフィルターより前に適用されます (フィルターによる変換は安全であるべきで、したがってフィルターされる値やその全ての引数はエスケープされるため):

        [twig]
        {{ var|nl2br }} {# は {{ var|escape|nl2br }} と同じ #}

 * `safe` フィルターはフィルターチェインのどこにでも使えます:

        [twig]
        {{ var|upper|nl2br|safe }} {# これと同じ: {{ var|safe|upper|nl2br }} #}

 * 自動的なエスケープはフィルターに渡されるリテラル以外の引数に適用されます:

        [twig]
        {{ var|foo("bar") }} {# "bar" はエスケープされない #}
        {{ var|foo(bar) }} {# bar はエスケープされる #}
        {{ var|foo(bar|safe) }} {# bar はエスケープされない #}

 * もしフィルターチェインの中の 1 つが `is_escaper` オプションを `true` にセットするならば、自動エスケープは適用されません (たとえば組み込みの `escaper`、`safe`、`urlencode` フィルターの場合)。

### サンドボックスエクステンション

このサンドボックスエクステンションは、信頼できないコードを評価するのに使います。安全ではない属性やメソッドへのアクセスは制限されます。サンドボックスのセキュリティはポリシーを表現するインスタンスによって管理されます。デフォルトでは、Twig はひとつのポリシーを表現するクラス、`Twig_Sandbox_SecurityPolicy` を持っています。このクラスはホワイトリスト方式でタグやプロパティやメソッドの利用を許可します:

    [php]
    $tags = array('if');
    $filters = array('upper');
    $methods = array(
      'Article' => array('getTitle', 'getBody'),
    );
    $properties = array(
      'Article' => array('title', 'body),
    );
    $policy = new Twig_Sandbox_SecurityPolicy($tags, $filters, $methods, $properties);

上の設定では、セキュリティポリシーは `it` タグと `uppder` フィルターの利用を許可します。加えて、テンプレートでは `Article` オブジェクトの `getTitle()`、`getBody()` メソッドと、`title`、`body` のパブリックなプロパティにだけアクセスできます。それ以外の全ては許可されず、`Twig_Sandbox_SecurityError` 例外が投げられます。

ポリシーオブジェクトはサンドボックスのコンストラクターの最初の引数として渡します:

    [php]
    $sandbox = new Twig_Extension_Sandbox($policy);
    $twig->addExtension($sandbox);

デフォルトでは、サンドボックスモードは効いておらず、信頼できないコードをインクルードするときはこれを有効にすべきです:

    [php]
    {% include "user.html" sandboxed %}

このエクステンションのコンストラクターの第 2 引数に `true` を渡すことで全てのテンプレートでサンドボックスを有効にできます:

    [php]
    $sandbox = new Twig_Extension_Sandbox($policy, true);

例外
----

Twig は例外を投げることがあります:

 * `Twig_Error`: すべてのテンプレートのエラーのベースとなる例外です。

 * `Twig_SyntaxError`: テンプレートの構文に問題が生じていることをユーザーに知らせるために投げられます。

 * `Twig_RuntimeError`: 実行時になんらかのエラーが出現したときに投げられます (たとえばフィルターが存在しないときなど)。

 * `Twig_Sandbox_SecurityError`: サンドボックス化されたテンプレートで許可されていないタグやフィルター、メソッド等が呼ばれたときに投げられます。
