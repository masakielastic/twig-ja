Twig をハックする
=================

Twig はとても拡張しやすく簡単にハックできます。たいていの機能と強化はエクステンションで実現可能なので、コアをハックする前にエクステンションを作ることになるであろうことを念頭においてください。この章は Twig の内部がどのように動いているのか理解したい人にも役立ちます。

Twig はどのように動くのか？
--------------------------

Twig テンプレートのレンダリングは 4 つのキーステップに要約できますs:

 * テンプレートを**ロードします**: テンプレートがすでにコンパイル済みである場合、これをロードして*評価*ステップに向かいます。そうでなければ:

   * 最初に、**レキサー**はテンプレートコードをより簡単に処理できる小さなピースにトークン化します;

   * それから、**パーサー**はトークンストリームをノードの意味あるツリーに変換します (Abstract Syntax Tree);

   * 最後に、*コンパイラー*は AST を PHP コードに変換します;

 * テンプレートを**評価します**: このことは基本的にコンパイル済みテンプレートの `display()` メソッドを呼び出しこれにコンテキストを渡すことを意味します。

レキサー
---------

Twig レキサーはソースコードをトークンストリームにトークン化することを目的としています (それぞれのトークンは `Token` クラスで構成され、ストリームは `Twig_TokenStream` のインスタンスです)。デフォルトのレキサーは 9 つの異なるトークンの型を認識します:

  * `Twig_Token::TEXT_TYPE`
  * `Twig_Token::BLOCK_START_TYPE`
  * `Twig_Token::VAR_START_TYPE`
  * `Twig_Token::BLOCK_END_TYPE`
  * `Twig_Token::VAR_END_TYPE`
  * `Twig_Token::NAME_TYPE`
  * `Twig_Token::NUMBER_TYPE`
  * `Twig_Token::STRING_TYPE`
  * `Twig_Token::OPERATOR_TYPE`
  * `Twig_Token::EOF_TYPE`

環境の `tokenize()` を呼び出すことでソースコードをトークンストリームに手作業で変換できます:

    [php]
    $stream = $twig->tokenize($source, $identifier);

ストリームは `__toString()` メソッドを持つので、オブジェクトを echo することでテキスト表現ができます:

    [php]
    echo $stream."\n";

`Hello {{ name }}` テンプレートの出力です:

    [txt]
    TEXT_TYPE(Hello )
    VAR_START_TYPE()
    NAME_TYPE(name)
    VAR_END_TYPE()
    EOF_TYPE()

`setLexer()` メソッドを呼び出すことで Twig のデフォルトレキサー (`Twig_Lexer`) を変更できます。

    [php]
    $twig->setLexer($lexer);

レキサーは `Twig_LexerInterface` を実装しなければなりません:

    [php]
    interface Twig_LexerInterface
    {
      /**
       * Tokenizes a source code.
       *
       * @param  string $code     The source code
       * @param  string $filename A unique identifier for the source code
       *
       * @return Twig_TokenStream A token stream instance
       */
      public function tokenize($code, $filename = 'n/a');
    }

パーサー
--------

パーサーはトークンストリームを AST (Abstract Syntax Tree)、もしくは (`Twig_Node_Module` クラスの) ノードツリーに変換します。コアエクステンションは `for`、`if` のような基本ノード、式ノードを定義します。

環境の `parse()` メソッドを呼び出すことでトークンストリームをノードツリーに手動で変換できます:

    [php]
    $nodes = $twig->parse($stream);

ノードオブジェクトの echo はツリーのすばらしい表現方法です:

    [php]
    echo $nodes."\n";

`Hello {{ name }}` テンプレートの出力は次の通りです:

    [txt]
    Twig_Node_Module(
      Twig_Node_Text(Hello )
      Twig_Node_Print(
        Twig_Node_Expression_Name(name)
      )
    )

デフォルトのパーサー (`Twig_TokenParser`) は `setParser()` メソッドを呼び出すことで変更することもできます:

    [php]
    $twig->setParser($parser);

すべての Twig パーサーは `Twig_ParserInterface` を実装しなければなりません:

    [php]
    interface Twig_ParserInterface
    {
      /**
       * Converts a token stream to a node tree.
       *
       * @param  Twig_TokenStream $stream A token stream instance
       *
       * @return Twig_Node_Module A node tree
       */
      public function parser(Twig_TokenStream $code);
    }

コンパイラー
------------

最後のステップはコンパイラーによって行われます。これはノードツリーを入力としてとりテンプレートの実行時に利用可能な PHP コードを生成します。デフォルトのコンパイラーはテンプレート継承機能の実装を簡単にするために PHP クラスを生成します。

`compile()` メソッドを渡すことでコンパイラーを呼び出すことができます:

    [php]
    $php = $twig->compile($nodes);

`compile()` メソッドはノードを表す PHP ソースコードを返します。

`Hello {{ name }}` テンプレートのために生成されたテンプレートは次のような内容を読み込みます:

    [php]
    /* Hello {{ name }} */
    class __TwigTemplate_1121b6f109fe93ebe8c6e22e3712bceb extends Twig_Template
    {
      public function display($context)
      {
        $this->env->initRuntime();

        // line 1
        echo "Hello ";
        echo (isset($context['name']) ? $context['name'] : null);
      }
    }

レキサーとパーサーに関しては、デフォルトのコンパイラー (`Twig_Compiler`) は `setCompiler()` メソッドを呼び出すことで変更できます:

    [php]
    $twig->setCompiler($compiler);

すべての Twig コンパイラーは `Twig_CompilerInterface` を実装しなければなりません:

    [php]
    interface Twig_CompilerInterface
    {
      /**
       * Compiles a node.
       *
       * @param  Twig_Node $node The node to compile
       *
       * @return Twig_Compiler The current compiler instance
       */
      public function compile(Twig_Node $node);

      /**
       * Gets the current PHP code after compilation.
       *
       * @return string The PHP code
       */
      public function getSource();
    }
