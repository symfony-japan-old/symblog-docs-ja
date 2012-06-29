[パート３] - ブログモデル: Doctrine 2 の使用とデータフィクスチャ
================================================================

要約
--------

この章では、ブログモデルを説明していきます。このモデルは、 `Doctrine 2 <http://www.doctrine-project.org/projects/orm>`_ Object Relation Mapper(ORM)を使用して実装されます。 Doctrine 2 は、永続化した PHP オブジェクトを提供します。また Doctrine 2 は、 Doctrine Query Language(DQL)と呼ばれる特殊な SQL の方言を提供します。さらに、 Doctrine 2 の説明に加え、データフィクスチャのコンセプトも説明します。データフィクスチャは、データベースの開発やテストで使用する適当なテストデータを追加するメカニズムです。この章の最後では、ブログモデルを定義し、新しいモデルに反映させたデータベースに修正します。そしてデータフィクスチャを作成します。また、ブログページの基本を構築します。

Doctrine 2: モデル
---------------------

ブログとして機能させるためには、データを永続化する方法が必要となります。 Doctrine 2 は、この目的にぴったりに設計された ORM ライブラリを提供しています。 Doctrine 2 の ORM は、 PHP PDO の抽象化ストレージを提供する強力な `Database Abstraction Layer <http://www.doctrine-project.org/projects/dbal>`_ の上に実装されています。 DBAL を使用することで、 MySQL, PostgreSQL, SQLite などの異なるストレージエンジンを使用することができます。今回のケースでは、 MySQL を使用しますが、もちろん他のストレージエンジンに置き換えることも簡単にできます。

.. tip::

    ORM に関して詳しくない方には、ここで ORM の原理の基本を説明しましょう。 ORM の定義に関しては、 `Wikipedia <http://en.wikipedia.org/wiki/Object-relational_mapping>`_ を参照してくだい。

    コンピュータソフトウェアにおける "Object-relational mapping (ORM, O/RM, または O/R mapping) は、プログラミングのテクニックの１つで、オブジェクト指向プログラミングにおける不適合なシステム間におけるデータ変換を行います。 ORM は、実際にプログラミング言語から使用可能な"仮想的な オブジェクトデータベース"を作成します。"
    
    ORM は、 MySQL などのリレーショナルデータベースのデータを、操作しやすいように PHP オブジェクトに変換する手助けをします。このことによってクラス内でテーブルで必要な機能をカプセル化することができます。ユーザテーブルを考えてみてください。おそらく username, password, first_name, last_name, email などのフィールドがあることでしょう。 ORM を使用すれば、これらは、 username,  password,  first_name などのメンバーを持つクラスになり、 ``getUsername()``, ``setPassword()`` などのメソッドを呼び出すことが可能になります。 ORM の利点は、これだけではありません。関連するテーブルを検索する際に、ユーザオブジェクトの検索と同時、または、遅延して検索、と選択することもできます。例えば、 user には 関連した friends があったとします。フレンドテーブルがあり、ユーザテーブルのプライマリーキーを格納しています。 ORM を使用すると、 ``$user->getFriends()`` メソッドなどを呼び出し、 friends テーブルのオブジェクトを検索することができます。さらに ORM は、データを永続化することができますので、 PHP オブジェクトを作成して ``save()`` メソッド等を呼べば データをデータベースに永続化することができます。今回は、 Doctrine 2 ORM ライブラリを使用しますので、このチュートリアルを通して ORM の理解が進むことでしょう。

.. note::

    このチュートリアルでは、 Doctrine 2 ORM ライブラリを使用していますが、 Doctrine 2 Object Document Mapper (ODM) ライブラリを使用することができます。このライブラリには多くのバリエーションがあり `MongoDB <http://www.mongodb.org/>`_ や `CouchDB <http://couchdb.apache.org/>`_ の実装もあります。詳細は、  `Doctrine Projects <http://www.doctrine-project.org/projects>`_ を参照してください。

    また、 `クックブック <http://symfony.com/doc/current/cookbook/doctrine/mongodb.html>`__ の記事では、 Symfony2 で ODM をセットアップする方法を説明しています。

ブログエンティティ
~~~~~~~~~~~~~~~~~~

``Blog`` エンティティクラスを作成するところから始めましょう。パート２の章で ``Enquiry`` エンティティを作成した際に、エンティティの説明をしました。エンティティの目的は、データを保持することなので、ブログエントリのデータを保持するのにエンティティを使用します。エンティティを定義することと、データベースにマップすることは完全には一致しません。以前の ``Enquiry`` エンティティではデータの保持はしていましたが、これはウェブマスターにメール送信するためだけに使用されていました。

``src/Blogger/BlogBundle/Entity/Blog.php`` ファイルを新しく作成し、次の内容をペーストしてください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Blog.php

    namespace Blogger\BlogBundle\Entity;

    class Blog
    {
        protected $title;

        protected $author;

        protected $blog;

        protected $image;

        protected $tags;

        protected $comments;

        protected $created;

        protected $updated;
    }


この PHP クラスはとてもシンプルです。親クラスを拡張していませんし、アクセサもありません。メンバーは protected で宣言されているので、このクラスのオブジェクトを処理する際にアクセスすることはできません。自分でこれらのメンバーのゲッターとセッターを宣言することもできますが、 Doctrine 2 はこのタスクをしてくれます。実際のところ、アクセサを書くことは、コーディングにおいてあまり楽しいことではありませんから。

このタスクを実行する前に、 Doctrine 2 がどうやって ``Blog`` エンティティをデータベースにマップするかを説明する必要があります。こういった情報は、 Doctrine 2 のマッピングを使用してメタデータとして特定してマップします。メタデータは、 ``YAML``, ``PHP``, ``XML``, ``アノテーション`` といったフォーマットで指定することができます。このチュートリアルでは、 ``アノテーション`` を使用します。重要なこととして、エンティティのメンバーが全て永続化される必要がないということを覚えておいてください。永続化される必要のないメンバーにはメタデータは必要ありません。このことにより、 Doctrine 2 がデータベースにマップさせるメンバーのみを選択することができるので、開発者に柔軟性を与えてくれます。 ``src/Blogger/BlogBundle/Entity/Blog.php`` ファイルの ``Blog`` エンティティの内容を以下の内容に置き換えてください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Blog.php

    namespace Blogger\BlogBundle\Entity;

    use Doctrine\ORM\Mapping as ORM;

    /**
     * @ORM\Entity
     * @ORM\Table(name="blog")
     */
    class Blog
    {
        /**
         * @ORM\Id
         * @ORM\Column(type="integer")
         * @ORM\GeneratedValue(strategy="AUTO")
         */
        protected $id;

        /**
         * @ORM\Column(type="string")
         */
        protected $title;

        /**
         * @ORM\Column(type="string", length=100)
         */
        protected $author;

        /**
         * @ORM\Column(type="text")
         */
        protected $blog;

        /**
         * @ORM\Column(type="string", length="20")
         */
        protected $image;

        /**
         * @ORM\Column(type="text")
         */
        protected $tags;

        protected $comments;

        /**
         * @ORM\Column(type="datetime")
         */
        protected $created;

        /**
         * @ORM\Column(type="datetime")
         */
        protected $updated;
    }


まず、 Doctrine 2 の ORM マッピングのネームスペースをインポートしてエイリアスをしてください。こうすることで、エンティティに ``アノテーション`` を使用してメタデータを定義することができます。このメタデータはメンバーがどうやってデータベースにマップするのか、といった情報を提供します。

.. tip::

    今回は、 Doctrine 2 のマッピングタイプを使用しましたが、ほんの少ししか使用してません。 Doctrine 2 のウェブサイトで `マッピングタイプ <http://www.doctrine-project.org/docs/orm/2.0/en/reference/basic-mapping.html#doctrine-mapping-types>`_ の一覧を参照することができます。他のマッピングタイプは、このチュートリアルの後の方で紹介します。

既に気づいた方もいらっしゃるかもしれませんが、 ``$comments`` メンバーにはメタデータを書いていません。 ``$comments`` は永続化される必要がないからで、 ``$comments`` にはブログ投稿に関連するコメントのコレクションを提供するだけになります。データベースのことを考えなければ、当然のことでしょう。次のスニペットは、このことを説明しています。

.. code-block:: php

    // Create a blog object.
    $blog = new Blog();
    $blog->setTitle("symblog - A Symfony2 Tutorial");
    $blog->setAuthor("dsyph3r");
    $blog->setBlog("symblog is a fully featured blogging website ...");

    // Create a comment and add it to our blog
    $comment = new Comment();
    $comment->setComment("Symfony2 rocks!");
    $blog->addComment($comment);

上記のスニペットは、ブログクラスとコメントクラスで期待した挙動を説明しています。内部的には、 ``$blog->addComment()`` メソッドは次のように実装されています。

.. code-block:: php

    class Blog
    {
        protected $comments = array();

        public function addComment(Comment $comment)
        {
            $this->comments[] = $comment;
        }
    }

``addComment`` メソッドは、ブログの ``$comments`` メンバーに新しいコメントオブジェクトを追加するのみです。次のように、コメントを取得するのもシンプルです。

.. code-block:: php

    class Blog
    {
        protected $comments = array();

        public function getComments()
        {
            return $this->comments;
        }
    }

上記を見ればわかるように ``$comments`` メンバーは ``Comment`` オブジェクトのリストに過ぎません。 Doctrine 2 はこの動作に関しては、何も変更はしません。 Doctrine 2 は、 ``blog`` オブジェクトに関連するオブジェクト ``$comments`` のメンバーを自動的に追加します。

これで Doctrine 2 のエンティティのマップ方法を説明してきましたので、次のタスクでアクセサメソッドを生成しましょう。

.. code-block:: bash

    $ php app/console doctrine:generate:entities Blogger


``Blog`` エンティティにアクセサメソッドが追加されたことに気づくでしょう。エンティティクラスの ORM メタデータを修正する度に、このタスクを実行して追加のアクセサを生成することができます。このタスクは、エンティティに既にアクセサがあれば修正することはありませんので、アクセサの内容がオーバーライドされることはありません。このことは、後に説明するデフォルトのアクセサをカスタマイズしたときに重要になります。

.. tip::

    今回は、 ``アノテーション`` をエンティティに使用しましたが、 ``doctrine:mapping:convert`` タスクを使用すればマッピング情報を他のマッピングフォーマットに変換することができます。例えば、次のタスクでは、上記のエンティティを ``yaml`` フォーマットに変換します。

    .. code-block:: bash

        $ php app/console doctrine:mapping:convert --namespace="Blogger\BlogBundle\Entity\Blog" yaml src/Blogger/BlogBundle/Resources/config/doctrine

    ``src/Blogger/BlogBundle/Resources/config/doctrine/Blogger.BlogBundle.Entity.Blog.orm.yml`` ファイルが作成され、 ``blog`` エンティティのマッピングが ``yaml`` フォーマットで保存されます。

データベース
~~~~~~~~~~~~

データベースを作成する
......................

パート１からこのチュートリアルをしているのであれば、ウェブ設定画面でデータベースの設定をセットしているはずです。まだしていないのであれば、 ``app/config/parameters.ini`` ファイルを修正して ``database_*`` オプションのパラメータに値を設定してください。

次の Doctrine 2 のタスクでデータベースを作成してみましょう。このタスクは、データベースの作成のみを行い、テーブルの作成はしません。既に同名のデータベースが存在している際には、エラーが投げられ、既存のデータベースに変更は何もされません。

.. code-block:: bash

    $ php app/console doctrine:database:create

これでデータベースに ``Blog`` エンティティの表現を作成する準備ができました。２つの方法でこれを実現。１つは、 Doctrine 2 のスキーマタスクでデータベースをアップデートする方法です。もう１つは、さらに強力な Doctrine2 のマイグレーションです。今回は、スキーマタスクを使用します。 Doctrine 2 のマイグレーションは、後の章で説明をします。

ブログテーブルの作成
....................

データベースにブログテーブルを作成するには次の Doctrine 2 のタスクを実行してください。

.. code-block:: bash

    $ php app/console doctrine:schema:create

このタスクは、 ``blog`` エンティティのデータベーススキーマを生成するのに必要な SQL を実行します。タスクに、 ``--dump-sql`` オプションを付けるとデータベースに SQL を実行するのではなく、 SQL 自体をダンプして出力することができます。データベースに、マップしたフィールドを持った blog テーブルが作成されたのを確認してみてください。

.. tip::

    今まで、 Symfony2 のコマンドラインタスクをいくつか使用してきました。 ``--help`` オプションを付けると、正しいコマンドラインタスクのフォーマットのヘルプを見ることができます。 ``doctrine:schema:create`` タスクのヘルプの詳細を確認するために、次を実行してみてください。

    .. code-block:: bash

        $ php app/console doctrine:schema:create --help

    ヘルプ情報に、使用方法と使用可能なオプションが表示されます。ほとんどのタスクは、実行をカスタマイズできるようにオプションが用意されています。

モデルとビューを統合して、ブログエントリを表示する
---------------------------------------------------------

これで ``Blog`` エンティティを作成し、そのエンティティの内容をデータベースにも反映させましたので、モデルとビューを統合しましょう。ブログの show ページを構築するところから始めましょう。

Show ブログルーティングルール
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

まず、ブログの ``show`` アクションのルーティングルールを作成します。ブログエントリは、ユニークな ID で識別するようにしますので、この ID は URL に出てくる必要があります。 ``BloggerBlogBundle`` のルーティングコンフィギュレーションである ``src/Blogger/BlogBundle/Resources/config/routing.yml`` を次のように修正してください。

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_blog_show:
        pattern:  /{id}
        defaults: { _controller: BloggerBlogBundle:Blog:show }
        requirements:
            _method:  GET
            id: \d+

URL に ブログの ID がある必要がありますので、 ``id`` プレースホルダーを指定しました。これは、 ``http://symblog.co.uk/1`` や ``http://symblog.co.uk/my-blog`` がこのルートにマッチすることを意味します。しかし、エンティティのマッピングで定義されているように ID は 整数値である必要があるので、このルーティングルールに ``id`` パラメータが整数値の際のみマッチするように制約を追加しましょう。 ``id:\d+`` を requirements に指定することで、実現できます。これで、先ほどまでマッチしていた ``http://symblog.co.uk/my-blog`` は、このルートにマッチしなくなりました。このルーティングルールにマッチすると、 ``BloggerBlogBundle`` バンドルの ``Blog`` コントローラの ``show`` アクションを実行します。このコントローラは、まだありませんが、すぐ作成します。

Show コントローラアクション
~~~~~~~~~~~~~~~~~~~~~~~~~~~

モデルとビューを統合はコントローラで行いますので、 show ページのコントローラを作成しましょう。既に存在している ``Page`` コントローラに ``show`` アクションを追加することもできますが、このページは、 ``blog`` エンティティを表示することに特化していますので、 ``Blog`` コントローラの方が適切でしょう。

``src/Blogger/BlogBundle/Controller/BlogController.php`` に新しくファイルを作成し、次の内容をペーストしてください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/BlogController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    /**
     * Blog controller.
     */
    class BlogController extends Controller
    {
        /**
         * Show a blog entry
         */
        public function showAction($id)
        {
            $em = $this->getDoctrine()->getEntityManager();

            $blog = $em->getRepository('BloggerBlogBundle:Blog')->find($id);

            if (!$blog) {
                throw $this->createNotFoundException('Unable to find Blog post.');
            }

            return $this->render('BloggerBlogBundle:Blog:show.html.twig', array(
                'blog'      => $blog,
            ));
        }
    }

上記で、 ``Blog`` エンティティに関する新しいコントローラを作成し、 ``show`` アクションを定義しました。 ``BloggerBlogBundle_blog_show`` ルーティングルールの ``id`` パラメータを指定したので、この値は ``showAction`` メソッドの引数として渡されます。ルーティングルールにもっと多くのパラメータを指定しても、それらの値も独立した引数としてアクションメソッドへ渡されます。

.. tip::

   また、コントローラアクションには、パラメータで指定すれば、 ``Symfony\Component\HttpFoundation\Request`` オブジェクトも渡すことができます。このオブジェクトはフォームを処理する際に便利です。フォームに関してはパート２で使用しましたが、以下のように、このオブジェクトを使用せずに ``Symfony\Bundle\FrameworkBundle\Controller\Controller`` のヘルパーメソッドを使用しました。

    .. code-block:: php

        // src/Blogger/BlogBundle/Controller/PageController.php
        public function contactAction()
        {
            // ..
            $request = $this->getRequest();
        }

    上記の代わりに以下のようにすることもできます。

    .. code-block:: php

        // src/Blogger/BlogBundle/Controller/PageController.php

        use Symfony\Component\HttpFoundation\Request;

        public function contactAction(Request $request)
        {
            // ..
        }
    
    両方とも同じことを行います。コントローラが、ヘルパークラスである ``Symfony\Bundle\FrameworkBundle\Controller\Controller`` を拡張しなければ、最初の方法を使用することはできません。

次に、データベースから ``Blog`` エンティティを検索する必要があります。まず ``Symfony\Bundle\FrameworkBundle\Controller\Controller`` クラスのヘルパーメソッドを使用し Doctrine 2 のエンティティマネージャを取得します。 `エンティティマネージャ <http://www.doctrine-project.org/docs/orm/2.0/en/reference/working-with-objects.html>`_ は、データベースからオブジェクトを検索したり、永続化したりする処理を行います。今回は、 ``エンティティマネージャ`` オブジェクトを使用して ``BloggerBlogBundle:Blog`` エンティティの Doctrine 2  ``リポジトリ`` を取得します。この ``BloggerBlogBundle:Blog`` シンタックスは、 Doctrine 2 で使用することのできるショートカットで、 ``Blogger\BlogBundle\Entity\Blog`` のようにエンティティの名前をフルで書く代わりに使用しています。リポジトリオブジェクトでは、 ``$id`` 引数を渡して ``find()`` メソッドを呼び出します。このメソッドは、プライマリーキーでオブジェクトを検索します。

最後に、エンティティが見つかったかチェックし、ビューにそのエンティティを渡します。エンティティが見つからなければ、 ``createNotFoundException`` 例外が投げられます。この例外は最終的には ``404 Not Found`` レスポンスを生成します。

.. tip::

    リポジトリオブジェクトを使用すると、次のメソッドも含めて、多くの便利なヘルパーメソッドが利用可能になります。

    .. code-block:: php

        // Return entities where 'author' matches 'dsyph3r'
        $em->getRepository('BloggerBlogBundle:Blog')->findBy(array('author' => 'dsyph3r'));

        // Return one entity where 'slug' matches 'symblog-tutorial'
        $em->getRepository('BloggerBlogBundle:Blog')->findOneBySlug('symblog-tutorial');

    次の章では、より複雑なクエリーが必要になるので、カスタムリポジトリクラスを作ります。

ビュー
~~~~~~~~

これで ``Blog`` コントローラに ``show`` アクションを組み込みましたので、 ``Blog`` エンティティの表示にフォーカスすることができます。 ``show`` アクションでテンプレートを指定しているので、 ``BloggerBlogBundle:Blog:show.html.twig`` がレンダリングされることになります。 ``src/Blogger/BlogBundle/Resouces/views/Blog/show.html.twig`` ファイルを新しく作成し、次の内容をペーストしてください。

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resouces/views/Blog/show.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block title %}{{ blog.title }}{% endblock %}

    {% block body %}
        <article class="blog">
            <header>
                <div class="date"><time datetime="{{ blog.created|date('c') }}">{{ blog.created|date('l, F j, Y') }}</time></div>
                <h2>{{ blog.title }}</h2>
            </header>
            <img src="{{ asset(['images/', blog.image]|join) }}" alt="{{ blog.title }} image not found" class="large" />
            <div>
                <p>{{ blog.blog }}</p>
            </div>
        </article>
    {% endblock %}

上記のコードでは、 ``BloggerBlogBundle`` のメインレイアウトを拡張するところから始まっています。次にブログのタイトルでページタイトルをオーバーライドしています。こうすることによって、デフォルトのタイトルではなく、ブログのページタイトルが使用されるので、より説明的になり SEO にも便利になりました。最後に body ブロックオーバーライドし、 ``Blog`` エンティティの内容を出力します。ここでは、ブログの画像をレンダリングするのに ``asset`` 関数を使用します。ブログ画像は、 ``web/images`` フォルダに配置するようにしてください。

CSS
...

ブログの show ページをカッコよく見せるめに、スタイルを追加する必要があります。 スタイルシート ``src/Blogger/BlogBundle/Resouces/public/css/blog.css`` に次の内容を加えてください。

.. code-block:: css

    .date { margin-bottom: 20px; border-bottom: 1px solid #ccc; font-size: 24px; color: #666; line-height: 30px }
    .blog { margin-bottom: 20px; }
    .blog img { width: 190px; float: left; padding: 5px; border: 1px solid #ccc; margin: 0 10px 10px 0; }
    .blog .meta { clear: left; margin-bottom: 20px; }
    .blog .snippet p.continue { margin-bottom: 0; text-align: right; }
    .blog .meta { font-style: italic; font-size: 12px; color: #666; }
    .blog .meta p { margin-bottom: 5px; line-height: 1.2em; }
    .blog img.large { width: 300px; min-height: 165px; }

.. note::

    ``web`` フォルダ内へのバンドルのアセットのリファレンス方法にシンボリックリンクを使用していなければ、次のアセットインストールのタスクをもう一度実行して修正した CSS をコピーする必要があります。

    .. code-block:: bash

        $ php app/console assets:install web


これで ``show`` アクションのコントローラとビューを構築したので、 show ページを確認してみましょう。ブラウザで ``http://symblog.dev/app_dev.php/1`` にアクセスしてください。想定したページと違いましたか？

.. image:: /_static/images/part_3/404_not_found.jpg
    :align: center
    :alt: Symfony2 404 Not Found Exception

Symfony2 は、 ``404 Not Found`` のレスポンスを生成しましたね。なぜなら使用するデータベースにデータがないからです。つまり、 ``id`` が１のエンティティが見つからなかったからなのです。

もちろんデータベースのブログテーブルに１列挿入して確認をすることもできますが、データフィクスチャというさらに良い方法を使用します。

データフィクスチャ(Data Fixtures)
---------------------------------

フィクスチャを使用して、サンプルデータをデータベースに格納することができます。そのために、 Doctrine フィクスチャのエクステンションとバンドルを使用します。 Doctrine フィクスチャ の エクステンションとバンドルは、 Symfony2 の標準ディストリビューションには付いてきませんので、手動でインストールする必要があります。幸運にもこれはとても簡単です。プロジェクトのルートディレクトリにある ``deps`` ファイルを開き、以下のように Doctrine フィクスチャのエクステンションとバンドルを追加してください。

.. code-block:: text

    [doctrine-fixtures]
        git=http://github.com/doctrine/data-fixtures.git

    [DoctrineFixturesBundle]
        git=http://github.com/symfony/DoctrineFixturesBundle.git
        target=/bundles/Symfony/Bundle/DoctrineFixturesBundle

次のタスクを実行して、この変更をベンダーに反映させましょう。

.. code-block:: bash

    $ php bin/vendors install

このタスクを実行すると、それぞれの Github リポジトリから最新版をダウンロードして、正しい場所にインストールします。

.. note::

    Git がインストールされていないマシンを使用しているのであれば、エクステンションとバンドルを手動でダウンロードしインストールする必要があります。

    doctrine-fixtures extension: `ダウンロード <https://github.com/doctrine/data-fixtures>`__
    GitHub にあるパッケージの現在のバージョンは、以下の場所に配置されます。
    ``vendor/doctrine-fixtures``.

    DoctrineFixturesBundle: `Download <https://github.com/symfony/DoctrineFixturesBundle>`__
    GitHub にあるパッケージの現在のバージョンは、以下の場所に配置されます。
    ``vendor/bundles/Symfony/Bundle/DoctrineFixturesBundle``.

次に ``app/autoloader.php`` ファイルを修正して新しいネームスペースを登録します。データフィクスチャ(DataFixtures)も ``Doctrine\Common`` ネームスペース内にあるので、既存の ``Doctrine\Common`` をセットしている場所よりも上に新しいパスを指定する必要があります。ネームスペースは上から順に調べられるので、より特定しているネームスペースは、特定されていないネームスペースよりも前に登録する必要があります。

.. code-block:: php

    // app/autoloader.php
    // ...
    $loader->registerNamespaces(array(
    // ...
    'Doctrine\\Common\\DataFixtures'    => __DIR__.'/../vendor/doctrine-fixtures/lib',
    'Doctrine\\Common'                  => __DIR__.'/../vendor/doctrine-common/lib',
    // ...
    ));

次に、 ``app/AppKernel.php`` のカーネルに ``DoctrineFixturesBundle`` を登録しましょう。

.. code-block:: php

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            // ...
            new Symfony\Bundle\DoctrineFixturesBundle\DoctrineFixturesBundle(),
            // ...
        );
        // ...
    }

ブログフィクスチャ
~~~~~~~~~~~~~~~~~~

ブログのフィクスチャを定義する準備ができました。 ``src/Blogger/BlogBundle/DataFixtures/ORM/BlogFixtures.php`` に新しいフィクスチャファイルを作成し、次の内容をペーストしてください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/DataFixtures/ORM/BlogFixtures.php
    
    namespace Blogger\BlogBundle\DataFixtures\ORM;
    
    use Doctrine\Common\DataFixtures\FixtureInterface;
    use Doctrine\Common\Persistence\ObjectManager;
    use Blogger\BlogBundle\Entity\Blog;
    
    class BlogFixtures implements FixtureInterface
    {
        public function load(ObjectManager $manager)
        {
            $blog1 = new Blog();
            $blog1->setTitle('A day with Symfony2');
            $blog1->setBlog('Lorem ipsum dolor sit amet, consectetur adipiscing eletra electrify denim vel ports.\nLorem ipsum dolor sit amet, consectetur adipiscing elit. Morbi ut velocity magna. Etiam vehicula nunc non leo hendrerit commodo. Vestibulum vulputate mauris eget erat congue dapibus imperdiet justo scelerisque. Nulla consectetur tempus nisl vitae viverra. Cras el mauris eget erat congue dapibus imperdiet justo scelerisque. Nulla consectetur tempus nisl vitae viverra. Cras elementum molestie vestibulum. Morbi id quam nisl. Praesent hendrerit, orci sed elementum lobortis, justo mauris lacinia libero, non facilisis purus ipsum non mi. Aliquam sollicitudin, augue id vestibulum iaculis, sem lectus convallis nunc, vel scelerisque lorem tortor ac nunc. Donec pharetra eleifend enim vel porta.');
            $blog1->setImage('beach.jpg');
            $blog1->setAuthor('dsyph3r');
            $blog1->setTags('symfony2, php, paradise, symblog');
            $blog1->setCreated(new \DateTime());
            $blog1->setUpdated($blog1->getCreated());
            $manager->persist($blog1);
    
            $blog2 = new Blog();
            $blog2->setTitle('The pool on the roof must have a leak');
            $blog2->setBlog('Vestibulum vulputate mauris eget erat congue dapibus imperdiet justo scelerisque. Na. Cras elementum molestie vestibulum. Morbi id quam nisl. Praesent hendrerit, orci sed elementum lobortis.');
            $blog2->setImage('pool_leak.jpg');
            $blog2->setAuthor('Zero Cool');
            $blog2->setTags('pool, leaky, hacked, movie, hacking, symblog');
            $blog2->setCreated(new \DateTime("2011-07-23 06:12:33"));
            $blog2->setUpdated($blog2->getCreated());
            $manager->persist($blog2);
    
            $blog3 = new Blog();
            $blog3->setTitle('Misdirection. What the eyes see and the ears hear, the mind believes');
            $blog3->setBlog('Lorem ipsumvehicula nunc non leo hendrerit commodo. Vestibulum vulputate mauris eget erat congue dapibus imperdiet justo scelerisque.');
            $blog3->setImage('misdirection.jpg');
            $blog3->setAuthor('Gabriel');
            $blog3->setTags('misdirection, magic, movie, hacking, symblog');
            $blog3->setCreated(new \DateTime("2011-07-16 16:14:06"));
            $blog3->setUpdated($blog3->getCreated());
            $manager->persist($blog3);
    
            $blog4 = new Blog();
            $blog4->setTitle('The grid - A digital frontier');
            $blog4->setBlog('Lorem commodo. Vestibulum vulputate mauris eget erat congue dapibus imperdiet justo scelerisque. Nulla consectetur tempus nisl vitae viverra.');
            $blog4->setImage('the_grid.jpg');
            $blog4->setAuthor('Kevin Flynn');
            $blog4->setTags('grid, daftpunk, movie, symblog');
            $blog4->setCreated(new \DateTime("2011-06-02 18:54:12"));
            $blog4->setUpdated($blog4->getCreated());
            $manager->persist($blog4);
    
            $blog5 = new Blog();
            $blog5->setTitle('You\'re either a one or a zero. Alive or dead');
            $blog5->setBlog('Lorem ipsum dolor sit amet, consectetur adipiscing elittibulum vulputate mauris eget erat congue dapibus imperdiet justo scelerisque.');
            $blog5->setImage('one_or_zero.jpg');
            $blog5->setAuthor('Gary Winston');
            $blog5->setTags('binary, one, zero, alive, dead, !trusting, movie, symblog');
            $blog5->setCreated(new \DateTime("2011-04-25 15:34:18"));
            $blog5->setUpdated($blog5->getCreated());
            $manager->persist($blog5);
    
            $manager->flush();
        }
    
    }

このフィクスチャファイルは、 Doctrine 2 を使う上で、たくさんの重要な機能を説明します。データベースにエンティティを永続化する方法に関してもです。

ブログエントリを１つ作成してみましょう。

.. code-block:: php

    $blog1 = new Blog();
    $blog1->setTitle('A day in paradise - A day with Symfony2');
    $blog1->setBlog('Lorem ipsum dolor sit d us imperdiet justo scelerisque. Nulla consectetur...');
    $blog1->setImage('beach.jpg');
    $blog1->setAuthor('dsyph3r');
    $blog1->setTags('symfony2, php, paradise, symblog');
    $blog1->setCreated(new \DateTime());
    $blog1->setUpdated($this->getCreated());
    $manager->persist($blog1);
    // ..

    $manager->flush();

``Blog`` のオブジェクトを作成し、そのメンバーに値をセットするところから始めましょう。現時点では、 Doctrine 2 は ``Entity`` オブジェクトについて何も知りません。 ``$manager->persist($blog1)`` を呼び出したときにのみに、 Doctrine 2 にエンティティオブジェクトをマネージするように伝えます。この ``$manager`` オブジェクトは、データーベースからエンティティを検索した際に見た ``EntityManager`` オブジェクトのインスタンスです。これで Doctrine 2 はエンティティオブジェクトを知るようになりましたが、まだデータベースには永続化しません。永続化するには、 ``$manager->flush()`` の呼び出しが必要です。 ``flush`` メソッドは、 Doctrine 2 に実際にデータベースと、マネージしている全てのエンティティのアクションを相互作用させます。パフォーマンス最適化のために、 Doctrine 2 の作業をまとめて、全てのアクションを１度で行うべきです。このようにフィクスチャが永続化されるのです。各エンティティを作成し、 Doctrine 2 にそのエンティティをマネージさせ、そして最後に全ての作業をフラッシュして一度に書き込みます。

.. tip:

    ``created`` と ``updated`` メンバーにもセットしていることに気づきましたか？これらのフィールドは、オブジェクトの作成時、更新時のそれぞれに自動的にセットするべきなので、あまり良い方法ではありません。 Doctrine 2 は、こういった自動的にセットする方法も持っています。これに関しては、少し後で説明します。

フィクスチャのロード
~~~~~~~~~~~~~~~~~~~~

これでデータベースにフィクスチャをロードする準備ができましたので、次のタスクを実行してください。

.. code-block:: bash

    $ php app/console doctrine:fixtures:load

ブラウザで ``http://symblog.dev/app_dev.php/1``  にアクセスし、 show ページを見ると、次のようなブログエントリが表示されるはずです。

.. image:: /_static/images/part_3/blog_show.jpg
    :align: center
    :alt: The symblog blog show page

URL の ``id`` パラメータを 2 に変更してみてください。ブログの次のエントリが表示されるはずです。

``http://symblog.dev/app_dev.php/100`` URL にアクセスしてみると、 ``404 Not Found`` 例外が投げられたという内容のページが表示されます。これは、 ``Blog`` エンティティには ID が 100 のものが存在しないからです。次に、 ``http://symblog.dev/app_dev.php/symfony2-blog`` URL にアクセスしてみてください。なぜ ``404 Not Found`` 例外が表示されないのでしょうか？理由は、 ``sho`` アクションは実行されなかったからです。この URL では、 ``BloggerBlogBundle_blog_show`` ルートには ``\d+`` 制約があるため、どのアプリケーションのルーティングルールにもマッチしません。 ``No route found for "GET /symfony2-blog"`` 例外が表示されるのは、そのためです。

タイムスタンプ
--------------

この章の最後では、 ``Blog`` エンティティの２つのタイムスタンプのメンバーを見ていきましょう。 ``created`` と ``updated`` です。これら２つのメンバーのための機能は、一般的に ``Timestampable`` ビヘイビアとして参照されます。これらのメンバーは、ブログの作成時間や、最終更新時間を保持します。ブログの作成時、更新時に手動でこれらフィールドをセットしたくはありませんので、 Doctrine 2 に手伝ってもらいましょう。

Doctrine 2 には、 `イベントシステム <http://www.doctrine-project.org/docs/orm/2.0/en/reference/events.html>`_ があり、 `ライフサイクルコールバック <http://www.doctrine-project.org/docs/orm/2.0/en/reference/events.html#lifecycle-callbacks>`_ を用意しています。これらのコールバックを使用して、エンティティのライフタイムの間、エンティティにイベントを通知するように登録することができます。通知されるイベントは、エンティティの変更前、保存後、削除後などがあります。エンティティでライフサイクルコールバックを使用するために、エンティティにこれらのコールバックを登録する必要があります。 ``src/Blogger/BlogBundle/Entity/Blog.php`` にある ``Blog`` エンティティを次のように修正してください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Blog.php

    // ..

    /**
     * @ORM\Entity
     * @ORM\Table(name="blog")
     * @ORM\HasLifecycleCallbacks()
     */
    class Blog
    {
        // ..
    }

これで ``Blog`` エンティティにメソッドを追加して ``preUpdate`` イベントを登録するようにしましょう。また、コンストラクタで ``created`` と `updated`` メンバーのデフォルト値もセットしておきます。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Blog.php

    // ..

    /**
     * @ORM\Entity
     * @ORM\Table(name="blog")
     * @ORM\HasLifecycleCallbacks()
     */
    class Blog
    {
        // ..

        public function __construct()
        {
            $this->setCreated(new \DateTime());
            $this->setUpdated(new \DateTime());
        }

        /**
         * @ORM\PreUpdate
         */
        public function setUpdatedValue()
        {
           $this->setUpdated(new \DateTime());
        }

        // ..
    }

``Blog`` エンティティに、 ``perUpdate`` イベントを通知して ``updated`` メンバー値をセットするように登録します。これで、フィクスチャをロードするタスクを再実行すると、 ``created`` と ``updated`` メンバーが自動的にセットされていることに気づくでしょう。

.. tip::

    Timestampable メンバーはエンティティ共通の必須事項なので、これをサポートするバンドルも利用可能です。 `StofDoctrineExtensionsBundle <https://github.com/stof/StofDoctrineExtensionsBundle>`_ は、 Timestampable, Sluggable, Sortable といった便利な Doctrine 2 のエクステンションを提供しています。

    このチュートリアルの後の方で、このバンドルの統合を紹介します。待ち切れない方は、 `クックブック <http://symfony.com/doc/current/cookbook/doctrine/common_extensions.html>`__ を参照してこのトピックに関してチェックすることができます。

結論
----------

この章では、 Doctrine 2 におけるモデル処理するのコンセプトをたくさんカバーしてきました。また、開発やテストの際に、アプリケーションに適当なテストデータを入れる簡単な方法であるデータフィクスチャの定義も説明しました。

次の章では、コメント(comment)エンティティを追加して、さらにモデルを拡張してみましょう。ホームページ(homepage)を構築して、そこで必要なカスタムリポジトリを作成するところから始めましょう。また、 Doctrine Migration のコンセプトの紹介と、フォームと Doctrine 2 のやりとりを説明し、ブログにコメントを投稿できるようにします。
