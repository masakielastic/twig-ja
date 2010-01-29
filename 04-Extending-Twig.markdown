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

インターフェース全体を実装する代わりに、エクステンションクラスは、エクステンションをきれいに保つために上記のすべてのメソッドの空の実装を提供する、 `Twig_Extension` クラスを継承することができます。

`getName()` メソッドはかならず、エクステンション毎にユニークな識別子を返すために実装しなければなりません。ここに、作成可能なもっとも基本的なエクステンションを示します:

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

もっとも Twig に追加したいと思う共通の要素はフィルターでしょう。 A filter is
just a regular PHP function or method that takes the left side of the filter
as first argument and the arguments passed to the filter as extra arguments.

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

メソッドをフィルターとして使うことは、グローバルな名前空間を汚さずにフィルターをパッケージングするためのすぐれた方法です。
わずかなオーバーヘッドで、開発者に柔軟性を与えます。

### Environment aware Filters

The `Twig_Filter` classes take options as their last argument. For instance, if
you want access to the current environment instance in your filter, set the
`needs_environment` option to `true`:

    [php]
    $filter = new Twig_Filter_Function('str_rot13', array('needs_environment' => true));

Twig will then pass the current environment as the first argument to the
filter call:

    [php]
    function twig_compute_rot13(Twig_Environment $env, $string)
    {
      // get the current charset for instance
      $charset = $env->getCharset();

      return str_rot13($string);
    }

### Automatic Escaping

If automatic escaping is enabled, the main value passed to the filters is
automatically escaped. If your filter acts as an escaper, you will want the
raw variable value. In such a case, set the `is_escaper` option to `true`:

    [php]
    $filter = new Twig_Filter_Function('urlencode', array('is_escaper' => true));

>**NOTE**
>The parameters passed as extra arguments to the filters are not affected by
>the `is_escaper` option and they are always escaped according to the
>automatic escaping rules.

デフォルトのフィルターを上書きする（オーバーライドする）
-------------------------------------------------------

>**CAUTION**
>このセクションでは Twig 0.9.5 でデフォルトのフィルターをオーバーライドする
>方法について説明します。

もしデフォルトのフィルターがニーズに合わない場合、簡単にそれらを独自のコアエクステンションを作ることによって上書きできます。
もちろん、 Twig のコアエクステンションのコード全体をコピー アンド ペーストする必要はありません。代わりに、それを拡張して `getFilters` メソッドをオーバーライドすることでフィルターを上書きできます:

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

But I can already hear some people wondering how it can work as the Core
extension is loaded by default. That's true, but the trick is that both
extensions share the same unique identifier (`core` - defined in the
`getName()` method). By registering an extension with the same name as an
existing one, you have actually overridden the default one, even if it is
already registered:

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
>`set` タグはコアエクステンションの一部で、常に使用可能です。
>組み込こまれたバージョンのものはもっとパワフルで複数の割り当てをデフォルトで許可します。
>(cf. the template designers chapter for more
>information).

まず、 `Twig_TokenParser` クラスを新しい言語構造を追加するために作成する必要があります:

    [php]
    class Project_Twig_Set_TokenParser extends Twig_TokenParser
    {
      // ...
    }

Of course, we need to register this token parser in our extension class:

    [php]
    class Project_Twig_Extension extends Twig_Extension
    {
      public function getTokenParsers()
      {
        return array(new Project_Twig_Set_TokenParser());
      }

      // ...
    }

Now, let's see the actual code of the token parser class:

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

The `getTag()` method must return the tag we want to parse, here `set`. The
`parse()` method is invoked whenever the parser encounters a `set` tag. It
should return a `Twig_Node` instance that represents the node. The parsing
process is simplified thanks to a bunch of methods you can call from the token
stream (`$this->parser->getStream()`):

 * `test()`: Tests the type and optionally the value of the next token and
   returns it.

 * `expect()`: Expects a token and returns it (like `test()`) or throw a
   syntax error if not found (the second argument is the expected value of the
   token).

 * `look()`: Looks a the next token. This is how you can have a look at the
   next token without consume it.

Parsing expressions is done by calling the `parseExpression()` like we did for
the `set` tag.

>**TIP**
>Reading the existing `TokenParser` classes is the best way to learn all the
>nitty-gritty details of the parsing process.

The `Project_Twig_Set_Node` class itself is rather simple:

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

The compiler implements a fluid interface and provides methods that helps the
developer generate beautiful and readable PHP code:

 * `subcompile()`: Compiles a node.

 * `raw()`: Writes the given string as is.

 * `write()`: Writes the given string by adding indentation at the beginning
   of each line.

 * `string()`: Writes a quoted string.

 * `repr()`: Writes a PHP representation of a given value (see `Twig_Node_For`
   for a usage example).

 * `pushContext()`: Pushes the current context on the stack (see
   `Twig_Node_For` for a usage example).

 * `popContext()`: Pops a context from the stack (see `Twig_Node_For` for a
   usage example).

 * `addDebugInfo()`: Adds the line of the original template file related to
   the current node as a comment.

 * `indent()`: Indents the generated code (see `Twig_Node_Block` for a usage
   example).

 * `outdent()`: Outdents the generated code (see `Twig_Node_Block` for a usage
   example).

Creating a Node Transformer
---------------------------

To be written...
