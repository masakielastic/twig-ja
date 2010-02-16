レシピ
=======

レイアウトの条件分岐を作る
---------------------------

Ajax と連携させることはときには同じ内容がそのまま表示されたり、レイアウトが伴うことがあることを意味します。しかし Twig テンプレートは PHP クラスとしてコンパイルされるので、`extends` タグを `if` タグでラップすると動きません:

    [twig]
    {# this does not work #}

    {% if request.ajax %}
      {% extends "base.html" %}
    {% endif %}

    {% block content %}
      This is the content to be displayed.
    {% endblock %}

この問題を解決する方法の 1 つは 2 つの異なるテンプレートを用意することです:

    [twig]
    {# index.html #}
    {% extends "layout.html" %}

    {% block content %}
      {% include "index_for_ajax.html" %}
    {% endblock %}


    {# index_for_ajax.html #}
    This is the content to be displayed.

テンプレートの内容の 1 つを表示することを決定するのはコントローラーの責務です:

    [php]
    $twig->render($request->isAjax() ? 'index_for_ajax.html' : 'index.html');

インクルードを動的に行う
-----------------------

テンプレートをインクルードするとき、名前は文字列である必要はありません。たとえば、名前は可変変数の値に依存することができます:

    [twig]
    {% include var ~ '_foo.html' %}

`var` がevaluates to `index` に評価される場合、`index_foo.html` テンプレートがレンダリングされます。

当然のことながら、テンプレートの名前には、次のような有効な正規表現が使えます:

    [twig]
    {% include var|default('index') ~ '_foo.html' %}

構文をカスタマイズする
----------------------

Twig はブロック区切り文字の構文のカスタマイズを許可します。テンプレートがあなたのカスタム構文に結びつけられるのでこの機能を使うことはおすすめできません。しかし、特定のプロジェクトに対して、デフォルトを変更することがもっともであることがあります。

ブロック区切り文字を変更するには、独自のレキサーオブジェクトを作る必要があります:

    [php]
    $twig = new Twig_Environment();

    $lexer = new Twig_Lexer($twig, array(
      'tag_comment'  => array('{#', '#}'),
      'tag_block'    => array('{%', '%}'),
      'tag_variable' => array('{{', '}}'),
    ));
    $twig->setLexer($lexer);

ほかのテンプレート構文をシミュレートするコンフィギュレーションの例として次のようなものがあります:

    [php]
    // Ruby erb syntax
    $lexer = new Twig_Lexer($twig, array(
      'tag_comment'  => array('<%#', '%>'),
      'tag_block'    => array('<%', '%>'),
      'tag_variable' => array('<%=', '%>'),
    ));

    // SGML Comment Syntax
    $lexer = new Twig_Lexer($twig, array(
      'tag_comment'  => array('<!--#', '-->'),
      'tag_block'    => array('<!--', '-->'),
      'tag_variable' => array('${', '}'),
    ));

    // Smarty like
    $lexer = new Twig_Lexer($twig, array(
      'tag_comment'  => array('{*', '*}'),
      'tag_block'    => array('{', '}'),
      'tag_variable' => array('{$', '}'),
    ));

動的なオブジェクトプロパティを使う
---------------------------------

Twig が `article.title` のような変数に遭遇するとき、`article` オブジェクトの `title` パブリックプロパティを見つけようとします。

プロパティが存在しないときに `__get()` マジックメソッドのおかげで動的に定義されることで動きます; 次のコードスニペットのような `__isset()` マジックメソッドを実装することも必要です:

    [php]
    class Article
    {
      public function __get($name)
      {
        if ('title' == $name)
        {
          return 'The title';
        }

        // throw some kind of error
      }

      public function __isset($name)
      {
        if ('title' == $name)
        {
          return true;
        }

        return false;
      }
    }

テンプレートにコンテキストを考慮させる
-------------------------------------

ときには、テンプレートをアプリケーションの何らかの「コンテキスト」に対応させたいことがあります。しかしデフォルトでは、コンパイル済みのテンプレートはパラメーターの配列としてのみ渡されます。

テンプレートをレンダリングするとき、コンテキストをオブジェクトに渡すことができますが、あまり現実的ではありません。よりよい解決方法があります。

デフォルトでは、すべてのコンパイル済みのテンプレートは基底クラスであり組み込みクラスの `Twig_Template` を継承します。この基底クラスのコンフィギュレーションは `base_template_class` オプションで調整可能です:

    [php]
    $twig = new Twig_Environment($loader, array('base_template_class' => 'ProjectTemplate'));

すべてのテンプレートは自動的に `ProjectTemplate` カスタムクラスを継承します。`ProjectTemplate` を作りテンプレートとアプリケーションのあいだのコミュニケーションを可能にするゲッター/セッターメソッドをいくつか追加します:

    [php]
    class ProjectTemplate extends Twig_Template
    {
      protected $context = null;

      public function setContext($context)
      {
        $this->context = $context;
      }

      public function getContext()
      {
        return $this->context;
      }
    }

これで、テンプレートを作るときにコンテキストを注入するのにセッターを使い、カスタムノードのなかでゲッターを使うことができます。
