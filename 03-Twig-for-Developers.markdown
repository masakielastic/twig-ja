開発者のためのTwig
===================

この章では、Twigのテンプレート言語についてではなくそのAPIについて説明します。
Twigのテンプレートを書く人にとってではなく、アプリケーションへのテンプレートの窓口を実装する人にとって、この章はリファレンスとして最も役立つでしょう。

基本
------

Twigは **環境** と呼ばれるTwig_Environmentクラスのオブジェクトを利用します。
このクラスのインスタンスは、設定やエクステンションを保管したり、ファイルシステムかもしくは他の場所からテンプレートを読み込むのに利用されます。

ほとんどのアプリケーションは、最初にひとつのTwig_Environmentオブジェクトを生成し、テンプレートを読み込むのにそれを使うでしょう。
もし違った設定を複数使うときでも、複数の環境を持つのは容易です。

あなたのアプリケーションでテンプレートを読み込めるようにTwigを設定する最も単純な方法は、こういった感じです:

    [php]
    require_once '/path/to/lib/Twig/Autoloader.php';
    Twig_Autoloader::register();

    $loader = new Twig_Loader_Filesystem('/path/to/templates');
    $twig = new Twig_Environment($loader, array(
      'cache' => '/path/to/compilation_cache',
    ));

このコードはデフォルトの設定を持つ環境と `/path/to/templates/` フォルダの中からテンプレートを探すローダーを生成します。
ローダーは違ったものでもいいですし、もしあなたがデータベースかもしくは他のリソースからテンプレートをロードしたい場合でも自分でローダーを書くことができます。

>**CAUTION**
>Twig 0.9.3 より前では、 `cache` オプションは存在しておらず、キャッシュディレクトリのパスはローダーの二番目の引数に渡されていました。

-

>**NOTE**
>環境の二番目の引数はオプションの配列であることに注目してください。
>`cache` オプションは、コンパイル済みのキャッシュを格納するディレクトリで、
>Twigは続いて起こるリクエストでのパースフェーズを避けるためにコンパイルされたテンプレートをキャッシュします。
>このキャッシュは、あなたが追加したいかもしれない評価済みのテンプレートのキャッシュとは違います。
>そういった必要がある場合は、あなたはどのPHPキャッシュライブラリを使うことができます。
>このことは、すでに評価されたテンプレートをキャッシュすることとは違います。

この環境からテンプレートを読み込むには、 `Twig_Template` インスタンスを返す `loadTemplate()` メソッドを呼び出すだけです:

    [php]
    $template = $twig->loadTemplate('index.html');

いくつかの変数とともにテンプレートを評価するには、 `render()` メソッドを呼び出します:

    [php]
    echo $template->render(array('the' => 'variables', 'go' => 'here'));

>**NOTE**
>`display()` メソッドはテンプレートを直接出力するためのショートカットです。

環境のオプション
-------------------

新しい `Twig_Environment` インスタンスを生成するとき、コンストラクタの二番目の引数にオプションの配列を渡すことができます:

    [php]
    $twig = new Twig_Environment($loader, array('debug' => true));

以下のオプションが利用可能です:

 * `debug`: `true` にセットした時、生成されたテンプレートは生成された構文木を視覚化できるように `__toString()` メソッドを持ちます(デフォルトは `false`)。

 * `trim_blocks`: 次に続く命令が現れたら改行を消すというPHPの振る舞いを真似します(デフォルトは `false`)。

 * `charset`: テンプレートで使われる文字コード (デフォルトは `utf-8`)。

 * `base_template_class`: 生成されたテンプレートの親となるクラス(デフォルトは `Twig_Template`)。

 * `cache`: 三つの値のどれかを取る:

    * `null` (デフォルト): Twigはコンパイルされたテンプレートを置くためにシステムのテンポラリディレクトリの下にディレクトリを生成するでしょう
      (もしあなたの複数のプロジェクトで同じTwigのソースコードを使うならば、二つのプロジェクトの同じ名前を持った二つのテンプレートは同じキャッシュを共有するため、非推奨です)。

    * `false`: コンパイルキャッシュを全くの無効にする(非推奨)。

    * コンパイルされたテンプレートを置く絶対パス。

 * `auto_reload`: Twigを使って開発するとき、テンプレートのソースコードが変わったらコンパイルし直されるのは便利です。
   もし `auto_reload` オプションに値を与えない場合、 このオプションは `debug` の値によって自動的に決定されるでしょう。

>**CAUTION**
>Twig 0.9.3 より前では、 `cache` と `auto_reload` オプションは存在しません。
>これらはそれぞれファイルシステムローダーの二番目と三番目の引数に渡されていました。

ローダー
-------

>**CAUTION**
>このセクションは Twig 0.9.4以上のバージョンで実装されたローダーについて説明しています。

ローダーは、例えばファイルシステムのようなリソースからテンプレートを読み込むことを責務としています。

### コンパイルキャッシュ

全てのテンプレートローダーは、未来の再利用のためにコンパイル済みのテンプレートをファイルシステムにキャッシュすることができます。
テンプレートは一度だけコンパイルされることでTwigはスピードアップします。
さらにAPCのようなPHPアクセラレータを使えばさらにパフォーマンスを押し上げることができます。
より詳しい情報については `Twig_Environment` の `cache` と `auto_reload` オプションを見てください。

### 組み込みのローダー

以下がTwigの提供する組み込みのローダーのリストです:

 * `Twig_Loader_Filesystem`: ファイルシステムからテンプレートを読み込みます。
   このローダーはファイルシステムのフォルダからテンプレートを探すことができ、テンプレートを読み込むのに最もポピュラーな方法です。

        [php]
        $loader = new Twig_Loader_Filesystem($templateDir);

 * `Twig_Loader_String`: 文字列からテンプレートを読み込みます。
   これはソースコードを直接渡すためのダミーのローダーです。 

        [php]
        $loader = new Twig_Loader_String();

 * `Twig_Loader_Array`: PHPの配列からテンプレートを読み込みます。名前をキーとした文字列の配列を渡します。
   このローダーはユニットテストの際に便利です。

        [php]
        $loader = new Twig_Loader_Array($templates);

>**TIP**
>`Array` かもしくは `String` ローダーをキャッシュとともに使う場合、
>テンプレートの中身が変わる度に新しいキャッシュのためのキーが生成されることを知っておいてください(キャッシュキーはテンプレートのソースコードです)。
>もしキャッシュがどんどん増えていくのを見たくない場合は、自分で古いキャッシュファイルを消していく必要があります。

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
       * もしテンプレートがまだ新鮮ならばtrueを返す。
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

`isFresh()` メソッドは、もし現在のキャッシュされたテンプレートが、
与えられた最新の更新時間に対してまだ新鮮ならば `true` を、そうでなければ `false` を返します。

エクステンションを使う
----------------

Twigのエクステンションは、Twigに対して新しい機能を追加するパッケージです。
エクステンションを使うには単に `addExtension()` メソッドを呼び出してください:

    [php]
    $twig->addExtension('Escaper');

Twigは三つのエクステンションを予め持っています:

 * *Core*: Twigの核となる全ての機能を定義しており、あなたが新しい環境を生成した時に自動的に登録されます。

 * *Escaper*: 出力を自動でエスケープする機能とescape/unescapeブロックのコードを使えるようにします。

 * *Sandbox*: Twig環境にサンドボックスモードを付加します。サンドボックスは信頼できないコードの評価を安全にします。

組み込みのエクステンション
-------------------

このセクションでは組み込みのエクステンションについている機能を説明します。

>**TIP**
>自分のエクステンションを作ってTwigを拡張するならこの章を読んでください。

### コアエクステンション

`core` エクステンションはTwigの核となる全ての機能を定義しています:

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

コアエクステンションはTwig環境に登録する必要はありません。というのもデフォルトで登録されるからです。

### エスケープエクステンション

`escaper` エクステンションはTwigに自動的に出力をエスケープする機能を追加します。
このエクステンションは `autoescape` という新しいタグと `safe` という新しいフィルターを定義します。

エスケープエクステンションを作る時、グローバルな出力エスケープ戦略をonかoffかに切り替えられます:

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
      {% var|escape %}   {# 変数は二回エスケープされない #}
    {% endautoescape %}

>**WARNING**
>`autoescape` タグはインクルードされたファイルの中では効きません。

エスケープの規則は以下のように実装されています(Twig 0.9.5以上):

 * 変数やフィルターの引数としてテンプレート内で使われるリテラル(整数, 真偽値, 配列等)は決して自動的にエスケープされません:

        [twig]
        {{ "Twig<br />" }} {# エスケープされない #}

        {% set text as "Twig<br />" %}
        {{ text }} {# エスケープされる #}

 * エスケープはどんな他のフィルターより前に適用されます
   (フィルターによる変換は安全であるべきで、したがってフィルターされる値やその全ての引数はエスケープされるから):
   Escaping is applied before any other filter is applied (the reasoning
   behind this is that filter transformations should be safe, as the filtered
   value and all its arguments are escaped):

        [twig]
        {{ var|nl2br }} {# is equivalent to {{ var|escape|nl2br }} #}

 * `safe` フィルターはフィルターチェインの何処ででも使えます:

        [twig]
        {{ var|upper|nl2br|safe }} {# これと同じ {{ var|safe|upper|nl2br }} #}

 * 自動的なエスケープはフィルターに渡されるリテラル以外の引数に適用されます:

        [twig]
        {{ var|foo("bar") }} {# "bar" はエスケープされない #}
        {{ var|foo(bar) }} {# bar はエスケープされる#}
        {{ var|foo(bar|safe) }} {# bar はエスケープされない #}

 * もしフィルターチェインの中の一つが `is_escaper` オプションを `true` にセットするならば、自動的なエスケープは適用されません
   (例えば組み込みの `escaper`, `safe`, `urlencode` フィルターの場合)。

### サンドボックスエクステンション

このサンドボックスエクステンションは、信頼できないコードを評価するのに使います。
安全ではない属性やメソッドへのアクセスは制限されます。
サンドボックスのセキュリティはポリシーを表現するインスタンスによって管理されます。
デフォルトでは、Twigはひとつのポリシーを表現するクラス、 `Twig_Sandbox_SecurityPolicy` を持っています。
このクラスはホワイトリスト方式でタグやプロパティやメソッドの利用を許可します:

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

上の設定では、セキュリティポリシーは `it` タグと `uppder` フィルターの利用を許可します。
加えて、テンプレートでは `Article` オブジェクトの `getTitle()`, `getBody()` メソッドと、 
`title`, `body` のパブリックなプロパティにだけアクセスできます。
それ以外の全ては許可されず、 `Twig_Sandbox_SecurityError` 例外が投げられます。

ポリシーオブジェクトはサンドボックスのコンストラクタの最初の引数として渡します:

    [php]
    $sandbox = new Twig_Extension_Sandbox($policy);
    $twig->addExtension($sandbox);

デフォルトでは、サンドボックスモードは効いておらず、信頼できないコードをインクルードするときはこれを有効にすべきです:

    [php]
    {% include "user.html" sandboxed %}

このエクステンションのコンストラクタの第二引数に `true` を渡すことで全てのテンプレートでサンドボックスを有効にできます:

    [php]
    $sandbox = new Twig_Extension_Sandbox($policy, true);

例外
----------

Twigは例外を投げることがあります:

 * `Twig_Error`: すべてのテンプレートのエラーのベースとなる例外です。

 * `Twig_SyntaxError`: テンプレートの文法に問題が生じている事をユーザに知らせるために投げられます。

 * `Twig_RuntimeError`: 実行時になんらかのエラーが出現したときに投げられます(例えばフィルターが存在しない時など)。

 * `Twig_Sandbox_SecurityError`: サンドボックス化されたテンプレートで許可されていないタグやフィルター、メソッド等が呼ばれた時に投げられます。
