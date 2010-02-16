Twig を拡張する
===============

Twig はタグやフィルターを拡張可能で、またパーサー自身でさえノードの変換クラスとともに拡張するエクステンションをサポートしています。
エクステンションを記述するための主要な動機は、国際化をサポートするクラスのような、再利用可能なクラスに頻繁に使用さえるコードを移動するためです。

ほとんどの場合は、 Twig に追加したい特定のタグやフィルタを管理するために、あなたのプロジェクトにひとつの拡張機能を作成すると便利です。

エクステンションの解剖（学？）
-----------------------------

エクステンションは `Twig_ExtensionInterface` を実装したクラスです:

    [php]
    interface Twig_ExtensionInterface
    {
      /**
       * Initializes the runtime environment.
       *
       * This is where you can load some file that contains filter functions for instance.
       */
      public function initRuntime();

      /**
       * Returns the token parser instances to add to the existing list.
       *
       * @return array An array of Twig_TokenParser instances
       */
      public function getTokenParsers();

      /**
       * Returns the node transformer instances to add to the existing list.
       *
       * @return array An array of Twig_NodeTransformer instances
       */
      public function getNodeTransformers();

      /**
       * Returns a list of filters to add to the existing list.
       *
       * @return array An array of filters
       */
      public function getFilters();

      /**
       * Returns the name of the extension.
       *
       * @return string The extension name
       */
      public function getName();
    }

インターフェイス全体を実装する代わりに、エクステンションクラスは、エクステンションをきれいに保つために上記のすべてのメソッドの空の実装を提供する、 `Twig_Extension` クラスを継承することができます。

`getName()` メソッドはかならず、エクステンションごとにユニークな識別子を返すために実装しなければなりません。ここに、作成可能なもっとも基本的なエクステンションを示します:

    [php]
    class Project_Twig_Extension extends Twig_Extension
    {
      public function getName()
      {
        return 'project';
      }
    }

>**TIP**
>バンドルされたエクステンションはエクステンションがどのように動作するかのすばらしい例です。

カスタム(?)エクステンションの追加は、他のエクステンションを追加するときと同様です:

    [php]
    $twig->addExtension(new Project_Twig_Extension());

フィルターを定義する
--------------------

もっとも Twig に追加したいと思う共通の要素はフィルターでしょう。フィルターは単なる通常の PHP 関数もしくはメソッドで火左側のフィルターを最初の引数としてとり、追加の引数としてフィルターに渡される引数をとります。

>**CAUTION**
>このセクションでは Twig 0.9.5 以上でのフィルターの作り方について記述します。

### 関数フィルター

`rot13` という名前の、 [rot13](http://www.php.net/manual/en/function.str-rot13.php) 変換をおこなった文字列を返すフィルターを作成しましょう。

    [twig]
    {{ "Twig"|rot13 }}

    {# Gjvt が表示される #}

このフィルターは `Twig_Filter` クラスのオブジェクトで定義されます。

`Twig_Filter_Function` クラスは関数としてフィルターを定義することができます:

    [php]
    $filter = new Twig_Filter_Function('str_rot13');

最初の引数は関数をコールするための名前で、ここでは PHP のネイティブ関数の `str_rot13` です。

エクステンションでフィルターを登録するということは、 `getFilters()` メソッドの実装をおこなうということです:

    [php]
    class Project_Twig_Extension extends Twig_Extension
    {
      public function getFilters()
      {
        return array(
          'rot13' => new Twig_Filter_Function('str_rot13'),
        );
      }

      public function getName()
      {
        return 'project';
      }
    }

フィルターに渡されるパラメーターは関数コールの際の他の引数として有効です:

    [twig]
    {{ "Twig"|rot13('prefix_') }}

-

    [php]
    function twig_compute_rot13($string, $prefix = '')
    {
      return $prefix.str_rot13($string);
    }

### クラスメソッドフィルター

`Twig_Filter_Function` はスタティックなメソッドもフィルターとして登録することができます:

    [php]
    class Project_Twig_Extension extends Twig_Extension
    {
      public function getFilters()
      {
        return array(
          'rot13' => new Twig_Filter_Function('Project_Twig_Extension::rot13Filter'),
        );
      }

      static public function rot13Filter($string)
      {
        return str_rot13($string);
      }

      public function getName()
      {
        return 'project';
      }
    }

### オブジェクトメソッドフィルター

`Twig_Filter_Method` を使うことで、メソッドをフィルターとして登録することもできます:

    [php]
    class Project_Twig_Extension extends Twig_Extension
    {
      public function getFilters()
      {
        return array(
          'rot13' => new Twig_Filter_Method($this, 'rot13Filter'),
        );
      }

      public function rot13Filter($string)
      {
        return str_rot13($string);
      }

      public function getName()
      {
        return 'project';
      }
    }

メソッドをフィルターとして使うことは、グローバルな名前空間を汚さずにフィルターをパッケージングするためのすぐれた方法です。わずかなオーバーヘッドで、開発者に柔軟性を与えます。

### フィルターを認識する環境

`Twig_Filter` クラスは最後の引数としてオプションをとります。たとえば、フィルターで現在の環境のインスタンスにアクセスしたい場合、`needs_environment` オプションを `true` にセットします:

    [php]
    $filter = new Twig_Filter_Function('str_rot13', array('needs_environment' => true));

Twig はフィルター呼び出しに最初の引数として現在の環境を渡します:

    [php]
    function twig_compute_rot13(Twig_Environment $env, $string)
    {
      // インスタンスの現在の文字セットを得る
      $charset = $env->getCharset();

      return str_rot13($string);
    }

### 自動エスケーピング

自動エスケーピングが有効な場合、フィルターに渡されるメインの値は自動的にエスケープされます。フィルターがエスケーパーとしてふるまう場合、名前の可変変数がほしくなります。そのような場合、`is_escaper` オプションを `true` にセットします:

    [php]
    $filter = new Twig_Filter_Function('urlencode', array('is_escaper' => true));

>**NOTE**
>フィルターに追加引数として渡されるパラメーターは `is_escaper` オプションの影響を受けこれらは自動エスケーピングのルールにしたがってつねにエスケープされます。

デフォルトのフィルターを上書きする（オーバーライドする）
-----------------------------------------------------

>**CAUTION**
>このセクションでは Twig 0.9.5 でデフォルトのフィルターをオーバーライドする方法について説明します。

もしデフォルトのフィルターがニーズに合わない場合、簡単にそれらを独自のコアエクステンションを作ることによって上書きできます。もちろん、 Twig のコアエクステンションのコード全体をコピー アンド ペーストする必要はありません。代わりに、それを拡張して `getFilters` メソッドをオーバーライドすることでフィルターを上書きできます:

    [php]
    class MyCoreExtension extends Twig_Extension_Core
    {
      public function getFilters()
      {
        return array_merge(
          parent::getFilters(),
          array(
            'date' => Twig_Filter_Method($this, 'dateFilter')
          )
        );
      }

      public function dateFilter($timestamp, $format = 'F j, Y H:i')
      {
        return '...'.twig_date_format_filter($timestamp, $format);
      }
    }

ここでは `date` フィルターを拡張されたものに上書きしました。この新しいコアエクステンションを使うには `MyCoreExtension` エクステンションを登録するのと同じくらいシンプルで、 environment のインスタンスの `addExtension()` メソッドを呼ぶことです（※訳が怪しい）:

    [php]
    $twig = new Twig_Environment($loader, array('debug' => true, 'cache' => false));
    $twig->addExtension(new MyCoreExtension());

しかしデフォルトで Core エクステンションがロードされるのと同じようにこれが動いているのでは考えている方々を存じております。これは正しいですが、トリックは両方のエクステンションが同じ一意の識別子 (`core` - `getName()` メソッドで定義される) を共有することです。既存のエクステンションと同じ名前でエクステンションを登録することで、すでに登録されたデフォルトのものを実際にオーバーライドできます:

    [php]
    $twig->addExtension(new Twig_Extension_Core());
    $twig->addExtension(new MyCoreExtension());

新しいタグを定義する
--------------------

Twig のようなテンプレートエンジンのなかでもっともエキサイティングな機能のうちのひとつは、新しい言語構造を定義することです。

テンプレート内の単純な変数を定義することを許す、シンプルな `set` タグを作ってみましょう。
タグは以下のように使用することができます:

    [twig]
    {% set name as "value" %}

    {{ name }}

    {# should output value #}

>**NOTE**
>`set` タグはコアエクステンションの一部で、常に使用可能です。組み込みバージョンのものはもっとパワフルで複数の割り当てをデフォルトで許可します (cf. 詳しい情報はテンプレートデザイナーの章で)。

まず、 `Twig_TokenParser` クラスを新しい言語構造を追加するために作成する必要があります:

    [php]
    class Project_Twig_Set_TokenParser extends Twig_TokenParser
    {
      // ...
    }

もちろん、トークンパーサーを既存のクラスに登録する必要があります:

    [php]
    class Project_Twig_Extension extends Twig_Extension
    {
      public function getTokenParsers()
      {
        return array(new Project_Twig_Set_TokenParser());
      }

      // ...
    }

では、トークンパーサークラスの実際のコードを見てみましょう:

    [php]
    class Project_Twig_Set_TokenParser extends Twig_TokenParser
    {
      public function parse(Twig_Token $token)
      {
        $lineno = $token->getLine();
        $name = $this->parser->getStream()->expect(Twig_Token::NAME_TYPE)->getValue();
        $this->parser->getStream()->expect(Twig_Token::NAME_TYPE, 'as');
        $value = $this->parser->getExpressionParser()->parseExpression();

        $this->parser->getStream()->expect(Twig_Token::BLOCK_END_TYPE);

        return new Project_Twig_Set_Node($name, $value, $lineno, $this->getTag());
      }

      public function getTag()
      {
        return 'set';
      }
    }

`getTag()` メソッドは解析したいタグを返さなければなりません。ここでは `set` です。パーサーが `set` タグに遭遇するたびに `parse()` メソッドが起動します。これはノードを表す `Twig_Node` インスタンスを返します。解析プロセスはトークンストリームから呼び出しできる一連のメソッド (`$this->parser->getStream()`) のおかげで解析プロセスは簡略化されます:

 * `test()`: 型とオプションとしてトークンの値をテストしそれを返します。

 * `expect()`: ( `test()` のように) トークンを要求しそれを返すもしくは見つからない場合構文エラーを返します (2 番目の引数はトークンの値が要求されます)。

 * `look()`: 次のトークンを探します。これは次のトークンを消費せずに探す手段です。

`set` タグに行ったように式の解析は `parseExpression()` を呼び出すことで行われます。

>**TIP**
>既存の `TokenParser` クラスを読むことが解析プロセスの肝心な詳細をすべて学ぶためのベストな方法です。

`Project_Twig_Set_Node` クラス自身はかなりシンプルです:

    [php]
    class Project_Twig_Set_Node extends Twig_Node
    {
      protected $name;
      protected $value;

      public function __construct($name, Twig_Node_Expression $value, $lineno)
      {
        parent::__construct($lineno);

        $this->name = $name;
        $this->value = $value;
      }

      public function compile($compiler)
      {
        $compiler
          ->addDebugInfo($this)
          ->write('$context[\''.$this->name.'\'] = ')
          ->subcompile($this->value)
          ->raw(";\n")
        ;
      }
    }

コンパイラーは流れるようなインターフェイスを実装し開発者が美しくて読みやすい PHP コードを作り出すのを手助けするメソッドを提供します:

 * `subcompile()`: ノードをコンパイルします。

 * `raw()`: 任意の文字列をそのまま書き出します。

 * `write()`: それぞれの行の冒頭にインデントを追加することで任意の文字列を書き出します。

 * `string()`: クォートつきの文字列を書き出します。

 * `repr()`: 任意の値の PHP 表現を書き出します (使い方の例は `Twig_Node_For` を参照)。

 * `pushContext()`: 現在のコンテキストをスタックにプッシュします (使い方の例は `Twig_Node_For` を参照)。

 * `popContext()`: スタックからコンテキストをポップします (使い方の例は `Twig_Node_For` を参照)。

 * `addDebugInfo()`: 現在のノードに関連するオリジナルのテンプレートファイルの行をコメントとして追加します。

 * `indent()`: 生成コードにインデントを追加します (使い方の例は `Twig_Node_Block` を参照)。

 * `outdent()`: 生成コードのインデントを解除します (使い方の例は `Twig_Node_Block` を参照)。

ノードトランスファーを作る
--------------------------

執筆中
