[パート１] - Symfony2 のコンフィギュレーションとテンプレート
============================================================

要約
----

この章では、 Symfony2 のウェブサイトを作成する際の最初のステップを説明します。 `標準ディストリビューション(Standard Distribution) <http://symfony.com/doc/current/glossary.html#term-distribution>`_ をダウンロードして設定し、ブログバンドルを作成し、メインの HTML テンプレートを組み立てます。この章の最後では、 Symfony2 のウェブサイトを設定し、ローカルのドメイン上で動くようにします。その時点でのウェブサイトはブログのメインの HTML 構造とダミーの内容を含みます。

この章で説明するのは、次の内容です。

    1. Symfony2 アプリケーションのセットアップ
    2. 開発ドメインの設定
    3. Symfony2 のバンドル
    4. ルーティング
    5. コントローラ
    6. Twig を使用したテンプレート

ダウンロードとセットアップ
--------------------------

上記で説明しましたように、ここでは Symfony2 の標準ディストリビューション(Standard Distribution)を使用します。このディストリビューションは、 Symfony2 のコアライブラリとウェブサイトを作成するのに必要な共通のバンドルが付いてきます。 Symfony2 のウェブサイトで `ダウンロード <http://symfony.com/download>`_ することができます。 Symfony2 の素晴らしいドキュメントを繰り返すことはしません。必要条件の詳細は、 `Symfony2 のインストールとコンフィギュレーション <http://symfony.com/doc/current/book/installation.html>`_ の章を参照してください。そのガイドでは、どのパッケージをダウンロードするか、必要なベンダーライブラリのインストールの仕方、フォルダのパーミッションを正しく設定する方法などが説明されています。

.. warning::

    インストールの章の `パーミッションのセットアップ <http://symfony.com/doc/current/book/installation.html#configuration-and-setup>`_ セクションは特に注意してください。そこでは、 ``app/cache`` や ``app/logs`` フォルドのパーミッションのについて説明しており、ウェブサーバユーザとコマンドラインユーザの両方が書き込み権限を持つようにしています。

開発ドメインの作成
------------------

このチュートリアルの目的として、ここではローカルドメインの ``http://symblog.dev/`` を使用しますが、どんなドメインでも結構です。これらの説明は、 `Apache <http://httpd.apache.org/>`_ に関することで、チュートリアルでは、読者がすでに Apache のセットアップをしており、自分のマシンで動作させていることを想定しています。ローカルドメインのセットアップに関する方法を既に知ってたり、 `nginx <http://nginx.net/>`_ などの他のウェブサーバを使用する際には、このセクションを飛ばしてください。

.. note::

    これらのステップは、Linux の Fedora ディストリビューション上で行なっていますので、 オペレーティングシステムによってはパス名などが異なるかもしれません。

Apache のバーチャルホストを作成するところから始めましょう。 Apache のコンフィギュレーションファイルを適切な場所に置き、次の設定を追加して ``DocumentRoot`` と、それに伴う ``Directory`` パスを変更してください。 Apache のコンフィギュレーションの置き先と名前は、 OS によって異なることがあります。 Fedora では、 ``/etc/httpd/conf/httpd.conf`` になります。 ``sudo`` 権限でこのファイルを次のように変更する必要があります。

.. code-block:: text

    # /etc/httpd/conf/httpd.conf

    NameVirtualHost 127.0.0.1

    <VirtualHost 127.0.0.1>
      ServerName symblog.dev
      DocumentRoot "/var/www/html/symblog.dev/web"
      DirectoryIndex app.php
      <Directory "/var/www/html/symblog.dev/web">
        AllowOverride All
        Allow from All
      </Directory>
    </VirtualHost>


次にホストファイル ``/etc/hosts`` の最後に新しく使用するドメインを追加します。このファイルを変更するためにも ``sudo`` 権限が必要になります。

.. code-block:: text

    # /etc/hosts
    127.0.0.1     symblog.dev

最後に Apache のサービスを再起動してください。再起動することで変更したコンフィギュレーションの設定がリロードされます。

.. code-block:: bash

    $ sudo service httpd restart

.. tip::

    毎回自分でバーチャルドメインを作成しているのであれば、 `Dynamic virtual hosts <http://blog.dsyph3r.com/2010/11/apache-dynamic-virtual-hosts.html>`_ を使用して、このプロセスを簡単にすることができます。

これで ``http://symblog.dev/app_dev.php/`` がアクセス可能になったはずです。

.. image:: /_static/images/part_1/welcome.jpg
    :align: center
    :alt: Symfony2 welcome page

今回初めて Symfony2 のウェルカムページを見た方は、デモページ等を見てみてください。各デモページには、各ページがどうやって動作しているかを説明するスニペットが用意されています。

.. note::

    ウェルカムページの一番下にツールバーがあることに気づくでしょう。これはディベロッパーツールバーと呼ばれ、アプリケーションの状態に関する有益な情報を参照することができます。このツールバーでは、ページの実行時間、メモリーの使用量、データベースへのクエリー、認証状態など多くの情報を参照することができます。デフォルトでは、ツールバーは、 ``dev`` 環境で実行しているときのみしか参照することができません。本番環境においては、アプリケーションの内部の情報を晒すことはセキュリティリスクになりますので参照するようにはなっていません。ツールバーへのリファレンスは、このチュートリアルを通して見ていきます。

Symfony の設定: ウェブインタフェース
------------------------------------

Symfony2 は、ウェブインタフェースでデータベースの設定などを行うことができます。今回のプロジェクトではデータベースを使用しますので、コンフィギュレーターを使用してみましょう。

``http://symblog.dev/app_dev.php/`` にアクセスし、 Configure ボタンをクリックしてください。データベースのセットアップする情報を入力し、次に CSRF トークンを生成するページに行きます(今回のチュートリアルでは、 MySQL を使用することを想定していますが、他のデータベースを選択することも可能です)。そうすると、 Symfony2 が生成したパラメータ設定が表示されます。このページの情報に注目してください。そして、 ``app/config/parameters.ini`` ファイルが書き込み不可であれば、この設定内容を ``app/config/parameters.ini`` にコピーペーストする必要があります(これらの設定は、このファイルの現在の設定を置き換えます)。

バンドル: Symfony2 の構成要素
-----------------------------

バンドルは、全ての Symfony2 アプリケーションの基本的な構成要素です。実際のところ Symfony2 フレームワーク自体もバンドルです。バンドルによる管理を行うことで、機能の分離やコード一式の再利用可能性が実現できます。バンドルは、コントローラ、モデル、テンプレート、画像や CSS などのリソースなどに対する全ての必要な物をカプセル化します。今回は、ウェブサイトを作成する際に Blogger というネームスペースでバンドルを作成してみます。 PHP のネームスペースに慣れていない方は、先に調べておいてください。 Symfony2 ではネームスペースを頻繁に使いますし、実際、全てがネームスペースで管理されています。 Symfony2 のオートロードを行う方法の詳細は、 `Symfony2 autoloader <http://symfony.com/doc/current/cookbook/tools/autoloader.html>`_ を参照してください。

.. tip::

    ネームスペースをしっかり理解していると、フォルダ構造がネームスペース構造に正しくマップされていないときといったような、よくある問題を回避することができます。

バンドルの作成
~~~~~~~~~~~~~~

ブログの機能をカプセル化するために、 Blog バンドルを作成します。この Blog バンドルは、他の Symfony2 アプリケーションに簡単に取り込むことができるように、必要なファイルを全て格納します。 Symfony2 は、共通の作業をアシストしてくれるタスクをたくさん用意しています。その１つはバンドルジェネレーターです。

次のタスクを実行してバンドルジェネレーターを動かしてみましょう。そうすると、バンドルをセットアップに関する設定を行うたくさんのプロンプトが表示されます。今回は、全てのプロンプトのデフォルトを使用してください。

.. code-block:: bash

    $ php app/console generate:bundle --namespace=Blogger/BlogBundle --format=yml

このジェネレーターの実行が終わると、 Symfony2 は基本的なバンドルのレイアウトを構築します。ファイルの変更も少しありますので、ここで言及しておきましょう。

.. tip::

    実際は Symfony2 が用意しているジェネレータータスクを使用する必要はありません。これは単にあなたの開発をアシストするためのものです。もちろん手動でバンドルフォルダーや、そこに置くファイルを作成することができます。ジェネレーターを使用するのは、強制ではありませんが、便利な利点があります。バンドルを作成し実行するために必要な全てのタスクを行なってくれるのです。例えば、アプリケーションにバンドルを登録してくれます。

バンドルの登録
..............

今回作成した新しいバンドル ``BloggerBlogBundle`` は ``app/AppKernel.php`` のカーネルに登録されています。 Symfony2 では、アプリケーションが使用する全てのバンドルを登録する必要があります。また、 ``app/AppKernel.php`` を見てみると、バンドルのいくつかは、 ``dev`` や ``test`` 環境のみに登録されていることに気づくでしょう。本番(``prod``)環境でこれらのバンドルを読み込むことは、使用しない機能のオーバーヘッドとなってしまいますので、登録していません。下のスニペットでは、 ``BloggerBlogBundle`` が登録されている状態を確認することができます。

.. code-block:: php

    // app/AppKernel.php
    class AppKernel extends Kernel
    {
        public function registerBundles()
        {
            $bundles = array(
            // ..
                new Blogger\BlogBundle\BloggerBlogBundle(),
            );
            // ..

            return $bundles;
        }

        // ..
    }

ルーティング
............

バンドルルーティングは、以下のようにアプリケーションのメインルーティングファイル ``app/config/routing.yml`` にインポートされます。

.. code-block:: yaml

    # app/config/routing.yml
    BloggerBlogBundle:
        resource: "@BloggerBlogBundle/Resources/config/routing.yml"
        prefix:   /

prefix オプションを使用すれば、 ``BloggerBlogBundle`` 全てのルーティングを prefix の内容でマウントすることができます。今回のケースでは、デフォルトの ``/`` を使用しています。 ``/blogger`` 以下に配置したい場合には、 prefix の値を ``prefix: /blogger`` のようにしてください。

デフォルトの構造
.................

``src`` ディレクトリには、デフォルトのバンドルレイアウトが入ります。まず、 ``Blogger`` ネームスペースをバンドルで直接マップする ``Blogger`` フォルダがトップレベルに配置されます。この ``Blogger`` フォルダ以下に、実際のバンドルを含む ``BlogBundle`` フォルダがあります。この ``BlogBundle`` フォルダの内容は、このチュートリアルを通して見ていきましょう。 MVC フレームワークに詳しい方であれば、 ``BlogBundle`` 以下のフォルダは、見ればすぐわかる内容でしょう。

デフォルトのコントローラ
~~~~~~~~~~~~~~~~~~~~~~~~

バンドルジェネレーターの役割は、 Symfony2 のデフォルトコントローラを作成することです。 ``http://symblog.dev/app_dev.php/hello/symblog`` にアクセスすればこのコントローラーを実行することができます。シンプルな挨拶ページを見ることができるはずです。 アクセスした URL の ``symblog`` の部分をあなたの名前に変えてみてください。では、どうやってこのページが生成されてたのかを見ていきましょう。

ルート
......

``BloggerBlogBundle`` のルーティングファイルは、 ``src/Blogger/BlogBundle/Resources/config/routing.yml`` に配置されており、以下のようなルーティングルールを含んでいます。

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_homepage:
        pattern:  /hello/{name}
        defaults: { _controller: BloggerBlogBundle:Default:index }

このルーティングは、１つのパターンといくつかのデフォルトオプションで構成されています。パターンは URL に対してチェックされ、デフォルトオプションはルートがマッチした際に実行されるコントローラを特定しています。 ``/hello/{name}`` というパターンであれば、特定の必要条件を指定していないので ``{name}`` プレースホルダーにはどんな値もマッチします。また、このルートは言語、フォーマット、 HTTP メソッドの指定もしていません。 HTTP メソッドを指定していませんので、 GET, POST, PUT などのリクエストはパターンにマッチします。

このルートが特定の条件の全てを満たせば、デフォルトの _controller オプションによって実行されます。 _controller オプションは、 Symfony2 がマップすることになるファイルの論理的な名前を特定します。上記の例では、 ``src/Blogger/BlogBundle/Controller/DefaultController.php`` にある ``Default`` コントローラの ``index`` アクションが実行されます。

コントローラ
............

この例のコントローラはとてもシンプルです。 ``DefaultController`` クラスは、いくつかの便利なメソッドを実装している ``Controller`` クラスを拡張しています。後で使用する ``render`` メソッドもその１つです。このルートは、アクションに渡すことになるプレースホルダーを ``$name`` 引数として定義しています。アクションは、 ``BloggerBlogBundle`` のデフォルトのビューフォルダーにある ``index.html.twig`` テンプレートを指定して ``render`` メソッドを呼び出すのみです。テンプレート名のフォーマットは、 ``bundle:controller:template`` になります。今回の例の ``BloggerBlogBundle:Default:index.html.twig`` は ``BloggerBlogBundle`` フォルダ内の ``Default`` のビューフォルダ内にある ``index.html.twig`` にマップされ、実際のファイルは、 ``src/Blogger/BlogBundle/Resources/views/Default/index.html.twig`` にあります。テンプレートフォーマットの異なるバリエーションは、アプリケーションやこのバンドル内の他の場所にあります。これに関しては、この章で後で説明をします。

以下のように ``array`` オプションを通してテンプレートに ``$name`` を渡すことも可能です。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/DefaultController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class DefaultController extends Controller
    {
        public function indexAction($name)
        {
            return $this->render('BloggerBlogBundle:Default:index.html.twig', array('name' => $name));
        }
    }

テンプレート (ビュー)
.....................

テンプレートは、とても簡単な内容になっています。ここでは、コントローラから渡ってきた name の引数を Hello の後に付けて出力しています。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Default/index.html.twig #}
    Hello {{ name }}!

クリーンアップ
~~~~~~~~~~~~~~

ジェネレーターで作成されたデフォルトのファイルのいくつかは今回必要としていないので、これらのファイルをクリーンアップしましょう。

コントローラファイル ``src/Blogger/BlogBundle/Controller/DefaultController.php`` と、これに付随するビューフォルダ ``src/Blogger/BlogBundle/Resources/views/Default/`` を削除することができます。最後に  ``src/Blogger/BlogBundle/Resources/config/routing.yml`` で定義されているルートを削除します。

テンプレート
------------

Symfony2 のテンプレートは、デフォルトでは２つのオプションがあります。 `Twig <http://www.twig-project.org/>`_ と PHP です。もちろん異なるライブラリを使用することもできます。 Symfony2 の `DI コンテナ <http://symfony.com/doc/current/book/service_container.html>`_ を使用すれば、そういったことが可能になります。今回のケースでは、次の理由から Twig を採用します。

1. Twig は高速です - Twig テンプレートは、 PHP クラスにコンパイルされるため Twig テンプレートを使用する際のオーバーヘッドはほとんどありません。
2. Twig は簡潔です - Twig は、少ないコードでテンプレートに必要な機能を実行することができます。 Twig と比べると PHP は冗長です。
3. Twig はテンプレート継承をサポートしています - これは私個人の最も好きな特徴です。テンプレートは、子テンプレートで親テンプレートが用意したデフォルトを書き換えるようにできるようにしておけば、拡張や書き換えをすることがでできます。
4. Twig はセキュアです - Twig は、デフォルトで出力エスケープをしており、インポートされたテンプレートにさえもサンドボックス環境を提供しています。
5. Twig は拡張性があります - Twig は、テンプレートエンジンとして必要なたくさんの共通のコア機能が付いてきますが、特注の機能が必要な際には、簡単に拡張することができます。

Twig の利点はもっとありますが、その詳細は、 Twig の公式サイト  `Twig <http://www.twig-project.org/>`_ を参照してくだい。

レイアウト構造
~~~~~~~~~~~~~~

Twig はテンプレート継承をサポートしていますので、 `３レベル継承 <http://symfony.com/doc/current/book/templating.html#three-level-inheritance>`_ アプローチを使用することにします。このアプローチは、アプリケーションで３つの異なるレベルでビューを変更させます。３レベル継承は、カスタマイズをしやすくさせてくれます。

メインテンプレート - レベル１
.............................

symblog の基本的な要素のテンプレートを作成しましょう。ここでは、テンプレートと CSS の２つのファイルを使用します。 Symfony2 は `HTML5 <http://diveintohtml5.org/>`_ をサポートしていますので、 HTML5 も使用することにしましょう。

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    <!DOCTYPE html>
    <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html" charset=utf-8" />
            <title>{% block title %}symblog{% endblock %} - symblog</title>
            <!--[if lt IE 9]>
                <script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script>
            <![endif]-->
            {% block stylesheets %}
                <link href='http://fonts.googleapis.com/css?family=Irish+Grover' rel='stylesheet' type='text/css'>
                <link href='http://fonts.googleapis.com/css?family=La+Belle+Aurore' rel='stylesheet' type='text/css'>
                <link href="{{ asset('css/screen.css') }}" type="text/css" rel="stylesheet" />
            {% endblock %}
            <link rel="shortcut icon" href="{{ asset('favicon.ico') }}" />
        </head>
        <body>

            <section id="wrapper">
                <header id="header">
                    <div class="top">
                        {% block navigation %}
                            <nav>
                                <ul class="navigation">
                                    <li><a href="#">Home</a></li>
                                    <li><a href="#">About</a></li>
                                    <li><a href="#">Contact</a></li>
                                </ul>
                            </nav>
                        {% endblock %}
                    </div>

                    <hgroup>
                        <h2>{% block blog_title %}<a href="#">symblog</a>{% endblock %}</h2>
                        <h3>{% block blog_tagline %}<a href="#">creating a blog in Symfony2</a>{% endblock %}</h3>
                    </hgroup>
                </header>

                <section class="main-col">
                    {% block body %}{% endblock %}
                </section>
                <aside class="sidebar">
                    {% block sidebar %}{% endblock %}
                </aside>

                <div id="footer">
                    {% block footer %}
                        Symfony2 blog tutorial - created by <a href="https://github.com/dsyph3r">dsyph3r</a>
                    {% endblock %}
                </div>
            </section>

            {% block javascripts %}{% endblock %}
        </body>
    </html>

.. note::

    このテンプレートでは３つ外部ファイルが使用されています。１つの JavaScript ファイルと２つの CSS ファイルです。この JavaScript ファイルは、IE のバージョン９より前のブラウザにおける HTML5 のサポートを行います。２つの CSS は `Google Web font <http://www.google.com/webfonts>`_ からフォントをインポートしています。

このテンプレートは、ブログサイトのメインの構造をマークアップしています。テンプレートのほとんどは HTML から構成されており、加えて、 Twig による命令が少し含まれています。ここで使用されている Twig の命令を説明します。

まず HEAD ドキュメントの中を見てみましょう。タイトルを見てください。

.. code-block:: html

    <title>{% block title %}symblog{% endblock %} - symblog</title>

まず気づくのは、見慣れない ``{%`` タグでしょう。これは HTML ではありませんし、もちろん PHP でもありません。Twig の３つタグのうちの１つです。このタグは、 Twig に ``何かをさせる`` タグで、制御文やブロック要素を定義するのに使用されます。 Twig のドキュメンテーションの `制御構造 <http://www.twig-project.org/doc/templates.html#list-of-control-structures>`_ で詳細を調べることができます。 title で定義した Twig のブロックは、次の２つのことを行います。タイトルのブロック識別子をセットし、block と endblock の間にあるデフォルトの出力を用意します。ブロックを定義することで Twig の継承モデルのアドバンテージを受けることができます。例えば、ブログの記事のタイトルを title に反映させることができます。テンプレートを拡張し、title ブロックをオーバーライドすることでこれが可能になります。

.. code-block:: html

    {% extends '::base.html.twig' %}

    {% block title %}The blog title goes here{% endblock %}

上記の例では、アプリケーションのベース(base) テンプレートを拡張して、 title ブロックを定義しています。テンプレートのフォーマットは、 ``bundle:controller;template`` でしたが、このテンプレートにある ``extends`` 命令に ``BUndle`` や ``Controller`` 部が無いのに気づきましたか。 ``Bundle`` と ``Controller`` 部を除外すれば、 ``app/Resources/views/`` で定義しているアプリケーションレベルのテンプレートを使用することを指定します。

次に、 title ブロックをもう一つ定義し、内容を入れてあります。今回の例ではブログのタイトルになります。親テンプレートでは既に title ブロックを含んでいるので、ここの内容でオーバーライドしています。これでタイトルには、  'The blog title goes here - symblog' が出力されることになりました。 Twig のこの機能はテンプレートを作成する際に、とてもよく使われます。

スタイルシート(stylesheet)ブロックで Twig のタグの２つ目を紹介します。 ``{{`` タグは、 ``何かを出力させる`` タグです。

.. code-block:: html

    <link href="{{ asset('css/screen.css') }}" type="text/css" rel="stylesheet" />

このタグは、変数や式の値を出力するのに使います。上記の例では、 ``asset`` 関数の返り値を出力します。 ``asset`` 関数は、 CSS や JavaScript や画像などのアプリケーションのアセットへのリンクをポータブルにする方法を提供します。

また、 ``{{`` タグは以下のように出力前にその内容を変更するフィルターと一緒に使用することができます。

.. code-block:: html

    {{ blog.created|date("d-m-Y") }}

フィルターの一覧は、 `Twig ドキュメント <http://www.twig-project.org/doc/templates.html#list-of-built-in-filters>`_ を参照してください。

Twig タグの３つ目はこのテンプレートの中には現れていませんが、それはコメントタグ ``{#`` です。これは次のように使います。

.. code-block:: html

    {# The quick brown fox jumps over the lazy dog #}

このテンプレートでは、とりあえず以上になります。これで必要なときにメインのレイアウトをカスタマイズする準備ができました。

次に、スタイルを追加しましょう。スタイルシートを ``web/css/screen.css`` に作成し、次の内容を追加してください。これはメインのテンプレートのスタイルになります。

.. code-block:: css

    html,body,div,span,applet,object,iframe,h1,h2,h3,h4,h5,h6,p,blockquote,pre,a,abbr,acronym,address,big,cite,code,del,dfn,em,img,ins,kbd,q,s,samp,small,strike,strong,sub,sup,tt,var,b,u,i,center,dl,dt,dd,ol,ul,li,fieldset,form,label,legend,table,caption,tbody,tfoot,thead,tr,th,td,article,aside,canvas,details,embed,figure,figcaption,footer,header,hgroup,menu,nav,output,ruby,section,summary,time,mark,audio,video{border:0;font-size:100%;font:inherit;vertical-align:baseline;margin:0;padding:0}article,aside,details,figcaption,figure,footer,header,hgroup,menu,nav,section{display:block}body{line-height:1}ol,ul{list-style:none}blockquote,q{quotes:none}blockquote:before,blockquote:after,q:before,q:after{content:none}table{border-collapse:collapse;border-spacing:0}

    body { line-height: 1;font-family: Arial, Helvetica, sans-serif;font-size: 12px; width: 100%; height: 100%; color: #000; font-size: 14px; }
    .clear { clear: both; }

    #wrapper { margin: 10px auto; width: 1000px; }
    #wrapper a { text-decoration: none; color: #F48A00; }
    #wrapper span.highlight { color: #F48A00; }

    #header { border-bottom: 1px solid #ccc; margin-bottom: 20px; }
    #header .top { border-bottom: 1px solid #ccc; margin-bottom: 10px; }
    #header ul.navigation { list-style: none; text-align: right; }
    #header .navigation li { display: inline }
    #header .navigation li a { display: inline-block; padding: 10px 15px; border-left: 1px solid #ccc; }
    #header h2 { font-family: 'Irish Grover', cursive; font-size: 92px; text-align: center; line-height: 110px; }
    #header h2 a { color: #000; }
    #header h3 { text-align: center; font-family: 'La Belle Aurore', cursive; font-size: 24px; margin-bottom: 20px; font-weight: normal; }

    .main-col { width: 700px; display: inline-block; float: left; border-right: 1px solid #ccc; padding: 20px; margin-bottom: 20px; }
    .sidebar { width: 239px; padding: 10px; display: inline-block; }

    .main-col a { color: #F48A00; }
    .main-col h1,
    .main-col h2
        { line-height: 1.2em; font-size: 32px; margin-bottom: 10px; font-weight: normal; color: #F48A00; }
    .main-col p { line-height: 1.5em; margin-bottom: 20px; }

    #footer { border-top: 1px solid #ccc; clear: both; text-align: center; padding: 10px; color: #aaa; }

バンドルテンプレート - レベル２
...............................

次に ブログバンドルのレイアウトを作成することにしましょう。 ``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` にファイルを作成し、次の内容を入れてください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    {% extends '::base.html.twig' %}

    {% block sidebar %}
        Sidebar content
    {% endblock %}

このテンプレートは、とてもシンプルに見えますが、このシンプルさがキーなのです。まず、先ほど作成したアプリケーションのベーステンプレートを拡張しています。次に、親のサイドバー(sidebar)ブロックをダミーの内容でオーバーライドしています。ブログの全てのページでサイドバーを表示することになるので、このレベルでカスタマイズする必要があります。全てのページでサイドバーを表示するのであれば、なぜアプリケーションのテンプレートでカスタマイズをしなかったのか疑問に思うかもしれません。答えは簡単です。アプリケーションではバンドルについて何も知っていませんし、知るべきではないのです。バンドルが自身の機能全てを含むべきで、サイドバーはそういった機能なのです。では、逆に個々のページのテンプレートにサイドバーを置いてもいいと疑問に思うかもしれません。この答えも簡単です。そうしてしまうと、ページを追加する度にサイドバーを複製する必要が出てきてしまいます。レベル２のテンプレートは、将来全ての子テンプレートが継承することになるカスタマイズを追加するといった柔軟性を与えてくれるのです。そのため、レベル２のテンプレートは、サイドバーの内容をカスタマイズするのに良い場所なのです。

ページテンプレート - レベル３
.............................

最後にコントローラのレイアウトを見てみましょう。このレベルのレイアウトはコントローラアクション共通のレイアウトです。例えば、ブログの show アクションはブログの show テンプレートを持つことになります。

ホームページ(homepage)のコントローラとそのテンプレートを作成しましょう。初めてのページ作成なので、まずコントローラを作成する必要があります。 ``src/Blogger/BlogBundle/Controller/PageController.php`` にコントローラファイルを作成して次の内容を追加してください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/PageController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class PageController extends Controller
    {
        public function indexAction()
        {
            return $this->render('BloggerBlogBundle:Page:index.html.twig');
        }
    }

次にこのアクションのテンプレートを作成します。上記のコントローラアクションを見ればわかるように、 Page の index テンプレートをレンダリングしています。 ``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig`` テンプレートファイルを作成して、次の内容を追加してください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block body %}
        Blog homepage
    {% endblock %}

このテンプレートが、特定することができる最終的なテンプレートです。この例では、テンプレート名の ``Controller`` 部を除外した ``BloggerBlogBundle::layout.html.twig`` テンプレートを拡張しています。 ``Controller`` 部を除外することで ``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` にあるバンドルレベルのテンプレートを特定しています。

ホームページ(homepage)にルーティングルールを追加しましょう。 ``src/Blogger/BlogBundle/Resources/config/routing.yml`` にあるバンドルのルーティング設定を次のように変更してください。

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_homepage:
        pattern:  /
        defaults: { _controller: BloggerBlogBundle:Page:index }
        requirements:
            _method:  GET

最後に Symfony2 のウェルカムスクリーンで使用しているデフォルトのルーティングルールを削除する必要があります。 ``dev`` ルーティングファイルである ``app/config/routing_dev.yml`` の ``_welcom`` ルートを削除してください。

これで Blogger テンプレートを見ることができるようになりました。ブラウザで ``http://symblog.dev/app_dev.php/`` にアクセスしてみてください。

.. image:: /_static/images/part_1/homepage.jpg
    :align: center
    :alt: symblog main template layout

ブログの基本的なレイアウトが見えるはずです。そして、関連するテンプレートでオーバーライドしたブロックに反映されたメインの内容とサイドバーも見えるはずです。

アバウト(About)ページ
---------------------

この章の最後の作業は、静的なページであるアバウトページを作成することです。ここではページ間のリンクについて説明し、さらに３レベル継承アプローチをより強固にしていきます。

ルーティング
~~~~~~~~~~~~

新しくページを作成する際に、最初に行うべきことはそのページのルートを作成することです。 ``src/Blogger/BlogBundle/Resources/config/routing.yml`` にある ``BloggerBlogBundle`` のルーティングファイルを開いて、次のルーティングルールを追加してください。

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_about:
        pattern:  /about
        defaults: { _controller: BloggerBlogBundle:Page:about }
        requirements:
            _method:  GET

コントローラ
~~~~~~~~~~~~

次に ``src/Blogger/BlogBundle/Controller/PageController.php`` にある ``Page`` コントローラを開いて、アバウトページを処理するアクションを追加してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    class PageController extends Controller
    {
        //  ..

        public function aboutAction()
        {
            return $this->render('BloggerBlogBundle:Page:about.html.twig');
        }
    }

ビュー
~~~~~~

ビューに関しては、 ``src/Blogger/BlogBundle/Resources/views/Page/about.html.twig`` を新しく作成し、次の内容をコピーして追加してください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/about.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block title %}About{% endblock%}

    {% block body %}
        <header>
            <h1>About symblog</h1>
        </header>
        <article>
            <p>Donec imperdiet ante sed diam consequat et dictum erat faucibus. Aliquam sit
            amet vehicula leo. Morbi urna dui, tempor ac posuere et, rutrum at dui.
            Curabitur neque quam, ultricies ut imperdiet id, ornare varius arcu. Ut congue
            urna sit amet tellus malesuada nec elementum risus molestie. Donec gravida
            tellus sed tortor adipiscing fringilla. Donec nulla mauris, mollis egestas
            condimentum laoreet, lacinia vel lorem. Morbi vitae justo sit amet felis
            vehicula commodo a placerat lacus. Mauris at est elit, nec vehicula urna. Duis a
            lacus nisl. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices
            posuere cubilia Curae.</p>
        </article>
    {% endblock %}

アバウトページは、特筆すべきものはありません。ダミーの内容のテンプレートファイルをレンダリングするだけです。しかし、リンクに関しては問題があります。

ページをリンクする
~~~~~~~~~~~~~~~~~~

アバウトページができましたので ``http://symblog.dev/app_dev.php/about`` にアクセスしてみてください。ブログのユーザは、直接 URL を入力する以外にこのページにアクセスすることができません。もちろん Symfony2 にはルーティングでリンクさせる仕組みがちゃんと用意されています。その機能を使うことで、今まで見てきたルートにマッチさせることもできますし、それらのルートから URL を生成することもできます。しかし、次のようにアプリケーションに直接 URL を指定することは止めてください。

.. code-block:: html+php

    <a href="/contact">Contact</a>

    <?php $this->redirect("/contact"); ?>

今までの習慣からこのアプローチがなぜ問題なのか疑問に思うかもしれません。しかし、このアプリーチには次の問題があります。

1. ハードリンクを使用しており、 Symfony2 のルーティングシステムを完全に無視しています。将来問い合わせページの位置の変更をしようと思った際には、全てのハードコードのリファレンスを調べ、変更する必要があります。
2. 環境のコントローラを無視することになります。環境に関してはまだ説明はしてませんが、実は今まで使用してきています。フロントコントローラの ``app_dev.php`` は、 ``開発`` 環境でアプリケーションにアクセスしています。 ``app_dev.php`` を ``app.php`` に変更すると、 ``本番`` 環境でアプリケーションを実行することになります。これらの環境は非常に重要ですので後にチュートリアルで説明をします。現時点では、上記のようなハードリンクは URL にフロントコントローラがないので、現在の環境を維持することができないということを覚えておいてください。

リンクを行う正しい方法は、 Twig の ``path`` と ``url`` メソッドです。この２つはとても似てますが、 ``path`` は相対 URL を、 ``url`` メソッドは絶対 URL を提供することで異なります。 ``app/Resources/views/base.html.twig`` にあるアプリケーションテンプレートを変更して、アバウトページとホームページのリンクを次のように修正してください。

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    {% block navigation %}
        <nav>
            <ul class="navigation">
                <li><a href="{{ path('BloggerBlogBundle_homepage') }}">Home</a></li>
                <li><a href="{{ path('BloggerBlogBundle_about') }}">About</a></li>
                <li><a href="#">Contact</a></li>
            </ul>
        </nav>
    {% endblock %}

ページを再読みしてホームとアバウトページのリンクが正しく動くようになったのを確認してください。このページのソースを見ると、 ``/app_dev.php/`` という接頭辞がリンクに付いているのに気づくでしょう。これは、上で説明をしたフロントコントローラで、 ``path`` を使用することでちゃんと受け継ぐことができるようになっています。

最後にホームページに戻るロゴへのリンクを修正してください。 ``app/Resources/views/base.html.twig`` を修正してください。

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    <hgroup>
        <h2>{% block blog_title %}<a href="{{ path('BloggerBlogBundle_homepage') }}">symblog</a>{% endblock %}</h2>
        <h3>{% block blog_tagline %}<a href="{{ path('BloggerBlogBundle_homepage') }}">creating a blog in Symfony2</a>{% endblock %}</h3>
    </hgroup>
    
結論
----

この章では、 Symfony2 のアプリケーションに関する基本的な分野をカバーしてきました。アプリケーションの設定から実際に動くまでです。そして、ルーティング、 Twig テンプレートエンジンといった Symfony2 のアプリケーションの背後にある基本的なコンセプトを見てきました。

次の章では、問い合わせ(Contact)ページを作成することにします。問い合わせページでは、ウェブフォームから問い合わせを送信できるようにするので、アバウト(About)ページよりも多少複雑になります。そして、次の章でバリデーターとフォームの設計概念を説明します。
