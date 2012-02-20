[パート５] - ビューのカスタマイズ: Twig エクステンション、サイドバーと Assetic
==============================================================================

要約
--------

この章では、 symblog のフロントエンドの構築を進めていきます。ブログのコメントに関してホームページの表示を微調整し、また、 URL にブログのタイトルを加え SEO 対策を行います。また、サイドバーに共通のブログコンポーネントを２つ作成します。それはタグクラウドと最新のコメントリストです。さらに、 Symfony2 のいろんな環境について説明し、 symblog を本番環境で実行させる方法を学びます。 Twig テンプレートエンジンに新しいフィルタを加え、 Assetic を使用してアセットファイルを管理させる方法を学びます。この章の最後では、これらの技術をホームページに組み込み、コメントの統合、サイドバーへのタグクラウドと最新コメントリストの表示、 Assetic の使用などを統合したホームページを仕上げます。そして実際に本番環境において symblog の実行を確認できるでしょう。

ホームページ - ブログとコメント
---------------------------------

現時点では、ホームページは最新のブログの一覧を表示していますが、ブログのコメントに関しては扱っていません。前章で ``Comment`` エンティティを作成しましたので、ホームページで表示する内容を見なおしましょう。 ``Blog`` と ``Comment`` エンティティのリンクをセットアップしましたので、 Doctrine 2 は、ブログに対するコメントのリストを検索することができるようになりました( ``Blog`` エンティティに ``$comments`` メンバーを加えたのを覚えていますか)。ホームページのテンプレートファイル ``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig`` を次のように修正してください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}

    {# .. #}
    
    <footer class="meta">
        <p>Comments: {{ blog.comments|length }}</p>
        <p>Posted by <span class="highlight">{{ blog.author }}</span> at {{ blog.created|date('h:iA') }}</p>
        <p>Tags: <span class="highlight">{{ blog.tags }}</span></p>
    </footer>
    
    {# .. #}

ブログのコメントを検索するのに ``comments`` ゲッターを使用し、そのコレクションを Twig の ``length`` フィルターに渡しています。ブラウザで ``http://symblog.dev/app_dev.php/`` を確認してみると、各ブログのコメント数を確認することができます。

上で説明したように、 ``$comments`` が ``Blog`` エンティティのメンバーとして ``Comment`` エンティティにマップされていることを Doctrine 2 に伝えてあります。前章で ``src/Blogger/BlogBundle/Entity/Blog.php`` にある ``Blog`` エンティティ内に次のようにメタデータを設定したのを覚えていますか。

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php

    /**
     * @ORM\OneToMany(targetEntity="Comment", mappedBy="blog")
     */
    protected $comments;

Doctrine 2 は、ブログとコメントにおけるリレーションについて知っていますが、どうやって ``$comments`` メンバーを ``Comment`` エンティティに関連させたのでしょうか。 ``BlogRepository`` に作成した以下のメソッドを覚えていますか？ここでは、ホームページへのブログの一覧を取得していますが、 ``Comment`` エンティティへの関連に関しては何も行なっていません。

.. code-block:: php

    // src/Blogger/BlogBundle/Repository/BlogRepository.php
    
    public function getLatestBlogs($limit = null)
    {
        $qb = $this->createQueryBuilder('b')
                   ->select('b')
                   ->addOrderBy('b.created', 'DESC');

        if (false === is_null($limit))
            $qb->setMaxResults($limit);

        return $qb->getQuery()
                  ->getResult();
    }
    
しかし、 Doctrine 2 は遅延ローディングを行い、必要になったときに ``Comment`` エンティティをデータベースから検索します。今回のケースでは、 ``{{ blog.comments|length }}`` が呼ばれたときにです。デベロッパーツールバーを使用して、この処理を説明することができます。既にデベロッパーツールバーの基本は見てきましたが、今回はディベロッパーツールバーの最も便利な機能の１つである Doctrine 2 プロファイラについて説明します。 Doctrine 2 プロファイラは、以下の画像にあるディベロッパーツールバーなる最後のアイコンをクリックすることで確認することができます。このアイコンの隣に表示されている数字は、現在の HTTP リクエストにおいて実行されたデータベースへのクエリー数です。

.. image:: /_static/images/part_5/doctrine_2_toolbar_icon.jpg
    :align: center
    :alt: Developer toolbar - Doctrine 2 icon

Doctrine 2 アイコンをクリックすると、以下のように、現在の HTTP リクエストにおいて Doctrine 2 によって実行されたデータベースのクエリーの情報が表示されます。

.. image:: /_static/images/part_5/doctrine_2_toolbar_queries.jpg
    :align: center
    :alt: Developer toolbar - Doctrine 2 queries

上のスクリーンショットを見ると、ホームページへのリクエストに対して多くのクエリーが実行されていることが確認できます。２つ目のクエリーは、データベースからブログエンティティを検索しており、 ``BlogRepository`` クラスの ``getLatestBlogs()`` メソッドの実行結果によるものです。このクエリーの次にはデータベースから各ブログのコメントを次々に引き出しているクエリーがあるのに気づくでしょう。各クエリーにある ``WHERE t0.blog_id = ?`` でそれが確認できます。 ``?`` は次の行にあるパラメータの値(ブログの ID)に置き換わります。各クエリーは、ホームページテンプレート内の ``{{ blog.comments }}`` 呼び出しの結果になります。この関数が実行される度に Doctrine 2 は ``Blog`` エンティティに関係している ``Comment`` エンティティを遅延ローディングします。

遅延ローディングは、データベースから関連したエンティティを検索するのにとても効果的ですが、今回のケースに関しては、効果的であるとは限りません。 Doctrine 2 は、データベースにクエリーを実行する際に関連するエンティティに ``join`` をすることができます。 ``join`` を使えば、データベースから１度のクエリーで ``Blog`` に関連している ``Comment`` エンティティを引きぬくことができます。 ``src/Blogger/BlogBundle/Repository/BlogRepository.php`` にある ``BlogRepository`` の ``QueryBuilder`` のコードを次のように comments を join するように修正してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Repository/BlogRepository.php

    public function getLatestBlogs($limit = null)
    {
        $qb = $this->createQueryBuilder('b')
                   ->select('b, c')
                   ->leftJoin('b.comments', 'c')
                   ->addOrderBy('b.created', 'DESC');

        if (false === is_null($limit))
            $qb->setMaxResults($limit);

        return $qb->getQuery()
                  ->getResult();
    }

これでホームページを再読み込みして、ディベロッパーツールバーの Doctrine 2 の出力を見てみると、クエリーの数が減ったことに気づくでしょう。 blog テーブルに comment テーブルが join したことも確認できます。

遅延ローディングと関連するエンティティの join は両方とも強力なコンセプトですが、正しく使う必要があります。これら２つの正しいバランスは、アプリケーションがちゃんと効果的に実行されているかを確認するために調べる必要があります。最初は、全ての関連するエンティティを join するのが素晴らしいことのように見え、遅延ローディングを使用する必要はないので、データベースクエリーを最小がになります。しかし、データベースから多くの情報を検索すれば、 Doctrine 2 がエンティティオブジェクトにこれをハイドレートする必要があるので、多くの処理が必要になることを覚えておいてください。多くのデータは、サーバがエンティティオブジェクトに格納するため多くのメモリが使用されることになります。

次の作業の前に、今追加したコメントの数を表示しているホームページテンプレートへにもう一つマイナーな追加をしましょう。 ``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig`` ファイルを修正して、ブログコメントへのリンクを加えてください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}

    {# .. #}
    
    <footer class="meta">
        <p>Comments: <a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id }) }}#comments">{{ blog.comments|length }}</a></p>
        <p>Posted by <span class="highlight">{{ blog.author }}</span> at {{ blog.created|date('h:iA') }}</p>
        <p>Tags: <span class="highlight">{{ blog.tags }}</span></p>
    </footer>
    
    {# .. #}
            
サイドバー
----------

現時点では、 symblog のサイドバーは空のままです。今回は、ここを修正して２つのブログコンポーネントを追加しましょう。それは、タグクラウドと最新コメントの一覧です。

タグクラウド
~~~~~~~~~~~~

タグクラウドは、各ブログに付けられたタグに関するもので、より多く使われているタグの文字を太くして強調して表示します。まず、全てのブログからタグを全て検索する必要があります。 ``BlogRepository`` クラスに新しいメソッドを作成しましょう。 ``src/Blogger/BlogBundle/Repository/BlogRepository.php`` にある ``BlogRepository`` クラスを次のように修正してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Repository/BlogRepository.php

    public function getTags()
    {
        $blogTags = $this->createQueryBuilder('b')
                         ->select('b.tags')
                         ->getQuery()
                         ->getResult();

        $tags = array();
        foreach ($blogTags as $blogTag)
        {
            $tags = array_merge(explode(",", $blogTag['tags']), $tags);
        }

        foreach ($tags as &$tag)
        {
            $tag = trim($tag);
        }

        return $tags;
    }

    public function getTagWeights($tags)
    {
        $tagWeights = array();
        if (empty($tags))
            return $tagWeights;
        
        foreach ($tags as $tag)
        {
            $tagWeights[$tag] = (isset($tagWeights[$tag])) ? $tagWeights[$tag] + 1 : 1;
        }
        // Shuffle the tags
        uksort($tagWeights, function() {
            return rand() > rand();
        });
        
        $max = max($tagWeights);
        
        // Max of 5 weights
        $multiplier = ($max > 5) ? 5 / $max : 1;
        foreach ($tagWeights as &$tag)
        {
            $tag = ceil($tag * $multiplier);
        }
    
        return $tagWeights;
    }

タグはデータベースに CSV 形式で格納されているので、コンマで分割して配列として返すようにする必要があります。 ``getTags()`` メソッドを使ってこれを行なっています。 ``getTagWeights()`` メソッドは、タグの配列から人気の重みを計算します。また、ページで表示する際にはタグの配列の順番はシャッフルされます。

これでタグクラウドが生成できるようになりましたので、表示させましょう。 ``src/Blogger/BlogBundle/Controller/PageController.php`` に、サイドバーを処理する新しいアクションを作成します。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    
    public function sidebarAction()
    {
        $em = $this->getDoctrine()
                   ->getEntityManager();

        $tags = $em->getRepository('BloggerBlogBundle:Blog')
                   ->getTags();

        $tagWeights = $em->getRepository('BloggerBlogBundle:Blog')
                         ->getTagWeights($tags);

        return $this->render('BloggerBlogBundle:Page:sidebar.html.twig', array(
            'tags' => $tagWeights
        ));
    }

アクションはとてもシンプルで、 タグクラウドを作成するために２つの ``BlogRepository`` メソッドを使用しています。そしてタグ変数をビューに渡しています。 ``src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig`` にビューを作成します。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig #}
    
    <section class="section">
        <header>
            <h3>Tag Cloud</h3>
        </header>
        <p class="tags">
            {% for tag, weight in tags %}
                <span class="weight-{{ weight }}">{{ tag }}</span>
            {% else %}
                <p>There are no tags</p>
            {% endfor %}
        </p>
    </section>

テンプレートもとてもシンプルです。タグの配列をイテレーションして span タグのクラスに重みをセットしています。 ``for`` ループを見てみると、配列の ``key`` と ``value`` ペアへのアクセスしています。 ``tag`` がキーになり ``weight`` がバリューとなります。 ``for`` ループの使用方法はバリエーションがあり、詳細は、 `Twig documentation <http://twig.sensiolabs.org/doc/templates.html#for>`_ を参照してください。

``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` にある ``BloggerBlogBundle`` のメインテンプレートを見てみると、 sidebar のブロックのプレースホルダーがあります。これを新しいサイドバーのアクションをレンダリングして入れ替えましょう。前章で出てきた Twig ``render`` メソッドを覚えていますか。 ``render`` メソッドは、コントローラのアクションから内容をレンダリングします。今回のケースでは、 ``Page`` コントローラの ``sidebar`` アクションになります。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}

    {# .. #}

    {% block sidebar %}
        {% render "BloggerBlogBundle:Page:sidebar" %}
    {% endblock %}

最後にタグクラウド用の CSS を追加しましょう。 ``src/Blogger/BlogBundle/Resources/public/css/sidebar.css`` に新しくスタイルシートを作成してください。

.. code-block:: css

    .sidebar .section { margin-bottom: 20px; }
    .sidebar h3 { line-height: 1.2em; font-size: 20px; margin-bottom: 10px; font-weight: normal; background: #eee; padding: 5px;  }
    .sidebar p { line-height: 1.5em; margin-bottom: 20px; }
    .sidebar ul { list-style: none }
    .sidebar ul li { line-height: 1.5em }
    .sidebar .small { font-size: 12px; }
    .sidebar .comment p { margin-bottom: 5px; }
    .sidebar .comment { margin-bottom: 10px; padding-bottom: 10px; }
    .sidebar .tags { font-weight: bold; }
    .sidebar .tags span { color: #000; font-size: 12px; }
    .sidebar .tags .weight-1 { font-size: 12px; }
    .sidebar .tags .weight-2 { font-size: 15px; }
    .sidebar .tags .weight-3 { font-size: 18px; }
    .sidebar .tags .weight-4 { font-size: 21px; }
    .sidebar .tags .weight-5 { font-size: 24px; }

新しくスタイルシートを追加したので、インクルードする必要があります。 ``BloggerBlogBundle`` のメインレイアウトテンプレートの ``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` を次のように修正してください。

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}

    {# .. #}
    
    {% block stylesheets %}
        {{ parent() }}
        <link href="{{ asset('bundles/bloggerblog/css/blog.css') }}" type="text/css" rel="stylesheet" />
        <link href="{{ asset('bundles/bloggerblog/css/sidebar.css') }}" type="text/css" rel="stylesheet" />
    {% endblock %}
    
    {# .. #}

.. note::

    ``web`` フォルダ内へのバンドルのアセットのリファレンス方法にシンボリックリンクを使用していなければ、次のアセットインストールのタスクをもう一度実行して修正した CSS をコピーする必要があります。

    .. code-block:: bash

        $ php app/console assets:install web
        
これで symblog のウェブサイトを再読み込みすれば、サイドバーにタグクラウドがレンダリングされているのが確認できます。異なる重みを付けてタグを取得するには、ブログフィクスチャを変更してタグの使用回数を調整してください。

最近のコメント一覧
~~~~~~~~~~~~~~~~~~

これでサイドバーにタグクラウドが表示されるようになりましたでの、次は最新のコメント一覧のコンポーネントも追加しましょう。

まず、ブログの最新のコメントを検索して取得する必要があります。 ``src/Blogger/BlogBundle/Repository/CommentRepository.php`` にある ``CommentRepository`` に新しいメソッドを追加しましょう。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Repository/CommentRepository.php

    public function getLatestComments($limit = 10)
    {
        $qb = $this->createQueryBuilder('c')
                    ->select('c')
                    ->addOrderBy('c.id', 'DESC');

        if (false === is_null($limit))
            $qb->setMaxResults($limit);

        return $qb->getQuery()
                  ->getResult();
    }

次に ``src/Blogger/BlogBundle/Controller/PageController.php`` にあるサイドバーのアクションを修正して、最新のコメントを取得し、ビューに渡すようにしましょう。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    
    public function sidebarAction()
    {
        // ..

        $commentLimit   = $this->container
                               ->getParameter('blogger_blog.comments.latest_comment_limit');
        $latestComments = $em->getRepository('BloggerBlogBundle:Comment')
                             ->getLatestComments($commentLimit);
    
        return $this->render('BloggerBlogBundle:Page:sidebar.html.twig', array(
            'latestComments'    => $latestComments,
            'tags'              => $tagWeights
        ));
    }

新しいパラメータ ``blogger_blog.comments.latest_comment_limit`` で取得するコメントの数を制限しているのに気づいたでしょう。このパラメータを使用するために、 ``src/Blogger/BlogBundle/Resources/config/config.yml`` にあるコンフィグファイルを次のように修正しましょう。

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/config.yml
    
    parameters:
        # ..

        # Blogger max latest comments
        blogger_blog.comments.latest_comment_limit: 10

最後に、サイドバーに最新のコメントをレンダリングします。 ``src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig`` のテンプレートを次のように修正してください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig #}

    {# .. #}

    <section class="section">
        <header>
            <h3>Latest Comments</h3>
        </header>
        {% for comment in latestComments %}
            <article class="comment">
                <header>
                    <p class="small"><span class="highlight">{{ comment.user }}</span> commented on
                        <a href="{{ path('BloggerBlogBundle_blog_show', { 'id': comment.blog.id }) }}#comment-{{ comment.id }}">
                            {{ comment.blog.title }}
                        </a>
                        [<em><time datetime="{{ comment.created|date('c') }}">{{ comment.created|date('Y-m-d h:iA') }}</time></em>]
                    </p>
                </header>
                <p>{{ comment.comment }}</p>
                </p>
            </article>
        {% else %}
            <p>There are no recent comments</p>
        {% endfor %}
    </section>

これで symblog のウェブサイトを再読み込みすれば、次のようにサイドバーのタグクラウドの下に最新のコメント一覧が表示されているのを確認できます。

.. image:: /_static/images/part_5/sidebar.jpg
    :align: center
    :alt: Sidebar - Tag Cloud and Latest Comments

Twig エクステンション
---------------------

これまで、ブログコメントが投稿された日付を `2011-04-21` のような標準の日付フォーマットで表示してきました。しかし、 ``posted 3 hours ago`` のようにどのくらい前にコメントが投稿されたかを表示する方がより良いアプローチでしょう。これを実現する機能として ``Comment`` エンティティに新しくメソッドを作成し、 ``{{ comment.created|date('Y-m-d h:iA') }}`` のメソッドを入れ替えることもできます。

しかし、この機能を他の場所でも使用したいとすれば、 ``Comment`` エンティティ内で実現するのはあまりいい方法ではありません。日付の変換を行うのはビューレイヤーの仕事なので、 Twig テンプレートエンジンを使用して実現するべきです。Twig のエクステンションインタフェースがあるので、 Twig でこれを実現することができます。

Twig の `エクステンション <http://www.twig-project.org/doc/extensions.html>`_ インタフェースを使用して、デフォルトの機能を拡張することができます。次のように使用できる Twig フィルターエクステンションを新しく作成しましょう。

.. code-block:: html
    
    {{ comment.created|created_ago }}
    
上記の例では、コメントが投稿された日付を `posted 2 days ago` のようなフォーマットで返すようにします。
    
エクステンション
~~~~~~~~~~~~~~~~

``src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogExtension.php`` に Twig エクステンションのファイルを新しく作成し、次の内容をペーストしてください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogExtension.php

    namespace Blogger\BlogBundle\Twig\Extensions;

    class BloggerBlogExtension extends \Twig_Extension
    {
        public function getFilters()
        {
            return array(
                'created_ago' => new \Twig_Filter_Method($this, 'createdAgo'),
            );
        }

        public function createdAgo(\DateTime $dateTime)
        {
            $delta = time() - $dateTime->getTimestamp();
            if ($delta < 0)
                throw new \InvalidArgumentException("createdAgo is unable to handle dates in the future");

            $duration = "";
            if ($delta < 60)
            {
                // Seconds
                $time = $delta;
                $duration = $time . " second" . (($time > 1) ? "s" : "") . " ago";
            }
            else if ($delta <= 3600)
            {
                // Mins
                $time = floor($delta / 60);
                $duration = $time . " minute" . (($time > 1) ? "s" : "") . " ago";
            }
            else if ($delta <= 86400)
            {
                // Hours
                $time = floor($delta / 3600);
                $duration = $time . " hour" . (($time > 1) ? "s" : "") . " ago";
            }
            else
            {
                // Days
                $time = floor($delta / 86400);
                $duration = $time . " day" . (($time > 1) ? "s" : "") . " ago";
            }

            return $duration;
        }

        public function getName()
        {
            return 'blogger_blog_extension';
        }
    }

エクステンションの作成はとてもシンプルです。追加したいフィルターを返す ``getFilters()`` メソッドをオーバーライドします。ここに複数書けば、複数のフィルターを登録できます。今回のケースでは、 ``created_ago`` フィルターを作成することにします。このフィルターは、 ``createdAgo`` メソッドを使用するように登録しており、このメソッドでは、 ``DateTime`` オブジェクトを文字列に変換し、どのくらいの期間が経ったかを返します。

エクステンションの登録
~~~~~~~~~~~~~~~~~~~~~~

Twig エクステンションを利用可能にするには、 ``src/Blogger/BlogBundle/Resources/config/services.yml`` にあるサービスファイルを次のように修正する必要があります。

.. code-block:: yaml

    services:
        blogger_blog.twig.extension:
            class: Blogger\BlogBundle\Twig\Extensions\BloggerBlogExtension
            tags:
                - { name: twig.extension }

上記では、今作成した ``BloggerBlogBundle`` の Twig エクステンションを新しくサービスとして登録しています。

ビューの修正
~~~~~~~~~~~~

これで新しい Twig フィルターが使用可能になりました。サイドバーの最新のコメント一覧に ``created_ago`` フィルターを使用するように修正しましょう。 ``src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig`` のサイドバーのテンプレートを次のように修正してください。


.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig #}

    {# .. #}
    
    <section class="section">
        <header>
            <h3>Latest Comments</h3>
        </header>
        {% for comment in latestComments %}
            {# .. #}
            <em><time datetime="{{ comment.created|date('c') }}">{{ comment.created|created_ago }}</time></em>
            {# .. #}
        {% endfor %}
    </section>

ブラウザで ``http://symblog.dev/app_dev.php/`` にアクセスすると、最新のコメントの日付が Twig フィルターを使用して、コメントの投稿日付から経った期間をレンダリングするようになったのを確認できます。

ブログの show ページのコメント一覧も、このフィルターを使用するように修正しましょう。 ``src/Blogger/BlogBundle/Resources/views/Comment/index.html.twig`` テンプレートを次の内容に置き換えてください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Comment/index.html.twig #}

    {% for comment in comments %}
        <article class="comment {{ cycle(['odd', 'even'], loop.index0) }}" id="comment-{{ comment.id }}">
            <header>
                <p><span class="highlight">{{ comment.user }}</span> commented <time datetime="{{ comment.created|date('c') }}">{{ comment.created|created_ago }}</time></p>
            </header>
            <p>{{ comment.comment }}</p>
        </article>
    {% else %}
        <p>There are no comments for this post. Be the first to comment...</p>
    {% endfor %}

.. tip::

    GitHub には、`Twig-Extensions <https://github.com/fabpot/Twig-extensions>`_  ライブラリがあり、たくさんの便利な Twig エクステンションが利用可能です。便利なエクステンションを作成したら、このリポジトリにプルリクエストを投げると、他人々も巻き込めるでしょう。

URL にスラッグを使用する
------------------------

現時点では、各ブログのページの URL にはブログの id のみが表示されます。機能面から見れば、これで十分ですが、 SEO 面では良くありません。例えば ``http://symblog.dev/1`` の URL ではブログの内容に関する情報が何もありません。 ``http://symblog.dev/1/a-day-with-symfony2`` のような URL の方が良いでしょう。このためにブログタイトルをスラッグ化し、 URL の一部として使用します。タイトルのスラッグ化は、全ての非 ASCII 文字を ``-`` に置き換えることにします。

ルーティングの修正
~~~~~~~~~~~~~~~~~~

まずブログの show ページのルーティングに、ルールをスラッグ部を追加して変更しましょう。 ``src/Blogger/BlogBundle/Resources/config/routing.yml`` のルーティングルールを次のように修正してください。

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    
    BloggerBlogBundle_blog_show:
        pattern:  /{id}/{slug}
        defaults: { _controller: BloggerBlogBundle:Blog:show }
        requirements:
            _method:  GET
            id: \d+

コントローラ
~~~~~~~~~~~~

既に指定している ``id`` 部と同じように、新しく ``slug`` 部がコントローラのアクションに引数として渡されます。次のように ``src/Blogger/BlogBundle/Controller/BlogController.php`` のコントローラを修正して、この変更を反映させてください。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/BlogController.php

    public function showAction($id, $slug)
    {
        // ..
    }

.. tip::

    コントローラアクションに渡す引数の順序は重要ではありませんが、引数の名前は重要です。 Symfony2 はルーティングの引数をパラメータのリストでマッチします。デフォルト部の値についてまだ使用していないので、ついでにここで言及しておきましょう。次のように、このルーティングルールに新しく ``{id}`` や ``{slug}`` のような ``{comment}`` を追加する際に、 ``defaults`` オプションを使用してデフォルト値を特定することができます。

    .. code-block:: yaml

        BloggerBlogBundle_blog_show:
            pattern:  /{id}/{slug}/{comments}
            defaults: { _controller: BloggerBlogBundle:Blog:show, comments: true }
            requirements:
                _method:  GET
                id: \d+

    .. code-block:: php

        public function showAction($id, $slug, $comments)
        {
            // ..
        }

    上の方法ですと ``http://symblog.dev/1/symfony2-blog`` にリクエストがあれば、 ``showAction`` に渡る ``$comments`` はデフォルト値の ``true`` となります。

タイトルのスラッグ化
~~~~~~~~~~~~~~~~~~~~

ブログのタイトルからスラッグを生成するにあたって、スラッグの値を自動生成するようにしましょう。その場でタイトルフィールドに対してこの操作を行うこともできますが、 ``Blog`` エンティティに slug を格納して、データベースに永続化するようにしましょう。

ブログエンティティの修正
~~~~~~~~~~~~~~~~~~~~~~~~

``Blog`` エンティティに slug を格納するための新しいメンバーを追加しましょう。 ``src/Blogger/BlogBundle/Entity/Blog.php`` にある ``Blog`` エンティティを次のように修正してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php

    class Blog
    {
        // ..

        /**
         * @ORM\Column(type="string")
         */
        protected $slug;

        // ..
    }

次に ``$slug`` メンバーのアクセサを生成してください。前と同じように次のタスクを実行してください。

.. code-block:: bash

    $ php app/console doctrine:generate:entities Blogger

次にデータベースのスキーマを修正しましょう。

.. code-block:: bash

    $ php app/console doctrine:migrations:diff
    $ php app/console doctrine:migrations:migrate

slug 値を生成するために、 symfony1 の `Jobeet <http://www.symfony-project.org/jobeet/1_4/Propel/en/08>`_ チュートリアルにあった slugify メソッドを使用することにします。 ``src/Blogger/BlogBundle/Entity/Blog.php`` にある ``Blog`` エンティティに ``slugify`` メソッドを次のように追加してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php

    public function slugify($text)
    {
        // replace non letter or digits by -
        $text = preg_replace('#[^\\pL\d]+#u', '-', $text);

        // trim
        $text = trim($text, '-');

        // transliterate
        if (function_exists('iconv'))
        {
            $text = iconv('utf-8', 'us-ascii//TRANSLIT', $text);
        }

        // lowercase
        $text = strtolower($text);

        // remove unwanted characters
        $text = preg_replace('#[^-\w]+#', '', $text);

        if (empty($text))
        {
            return 'n-a';
        }

        return $text;
    }

タイトルから自動的にスラッグを生成したいので、title がセットされたときに slug を生成するようにします。そのために ``setTitle`` アクセサを修正して、 slug の値もセットするようにしてください。 ``src/Blogger/BlogBundle/Entity/Blog.php`` の ``Blog`` エンティティを次のように修正してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php

    public function setTitle($title)
    {
        $this->title = $title;

        $this->setSlug($this->title);
    }

次に ``setSlug`` メソッドを修正して、セット前にスラッグ化するようにします。

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php

    public function setSlug($slug)
    {
        $this->slug = $this->slugify($slug);
    }

これでデータフィクスチャをリロードして、ブログのスラッグを生成してください。

.. code-block:: bash

    $ php app/console doctrine:fixtures:load

生成されたルートの修正
~~~~~~~~~~~~~~~~~~~~~~

最後に、既存のブログの show ページへのルートの呼び出しも修正する必要があります。修正する箇所がたくさんありますね。

``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig`` のホームページテンプレートを開いて、次のように修正してください。このテンプレートでは、 ``BloggerBlogBundle_blog_show`` のルートが３つあります。修正は、単に Twig の ``path`` 関数にブログの slug を渡すだけです。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}

    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block body %}
        {% for blog in blogs %}
            <article class="blog">
                <div class="date"><time datetime="{{ blog.created|date('c') }}">{{ blog.created|date('l, F j, Y') }}</time></div>
                <header>
                    <h2><a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id, 'slug': blog.slug }) }}">{{ blog.title }}</a></h2>
                </header>
    
                <img src="{{ asset(['images/', blog.image]|join) }}" />
                <div class="snippet">
                    <p>{{ blog.blog(500) }}</p>
                    <p class="continue"><a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id, 'slug': blog.slug }) }}">Continue reading...</a></p>
                </div>
    
                <footer class="meta">
                    <p>Comments: <a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id, 'slug': blog.slug }) }}#comments">{{ blog.comments|length }}</a></p>
                    <p>Posted by <span class="highlight">{{ blog.author }}</span> at {{ blog.created|date('h:iA') }}</p>
                    <p>Tags: <span class="highlight">{{ blog.tags }}</span></p>
                </footer>
            </article>
        {% else %}
            <p>There are no blog entries for symblog</p>
        {% endfor %}
    {% endblock %}

また、 ``src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig`` のサイドバーの最新のコメント一覧にあるリンクも次のように修正してください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig #}

    {# .. #}

    <a href="{{ path('BloggerBlogBundle_blog_show', { 'id': comment.blog.id, 'slug': comment.blog.slug }) }}#comment-{{ comment.id }}">
        {{ comment.blog.title }}
    </a>

    {# .. #}

最後に ``CommentController`` の ``createAction`` を修正する必要があります。コメント投稿が成功した際にブログの show ページにリダイレクトする際の URL の修正です。 ``src/Blogger/BlogBundle/Controller/CommentController.php`` の ``CommentController`` を次のように修正してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/CommentController.php
    
    public function createAction($blog_id)
    {
        // ..

        if ($form->isValid()) {
            // ..
                
            return $this->redirect($this->generateUrl('BloggerBlogBundle_blog_show', array(
                'id'    => $comment->getBlog()->getId(),
                'slug'  => $comment->getBlog()->getSlug())) .
                '#comment-' . $comment->getId()
            );
        }

        // ..
    }

これでブラウザで ``http://symblog.dev/app_dev.php/`` ページにアクセスし、いずれかのブログタイトルをクリックすると、 URL の最後にブログのスラッグが追加されているのを確認することができます。

環境管理
--------

Symfony2 のシンプルな機能である環境管理はとても強力です。気づいていないかもしれませんが、このチュートリアルのパート１から環境を使用してきています。環境管理を使えば、アプリケーションのライフサイクルの間の特定の異なるニーズに応じて、 Symfony2 やアプリケーションのいろんな面を設定することができます。デフォルトでは、 Symfony2 に次の３つの環境があります。

1. ``dev`` - 開発環境
2. ``test`` - テスト環境
3. ``prod`` - 本番環境

これらの環境は、名前の通りの目的を持っているの説明はいりませんが、個々のニーズに応じてどうやって異なる設定をしていしているのかは、説明が必要でしょう。アプリケーションの開発時には、例外やエラーをスクリーンに表示するディベロッパーツールバーは便利です。しかし、本番環境では、ディベロッパーツールバーは必要ありません。実際、この情報が表示されてしまうと、アプリケーションやサーバの内部情報が漏れてしまうので、セキュリティリスクにつながってしまいます。本番環境では、簡単なメッセージと共にカスタマイズされたページが表示され、裏でテキストファイルにログを書き込む方が良いでしょう。また、本番環境では、キャッシュレイヤーを有効にして、アプリケーションのパフォーマンスを上げることができた方が良いでしょう。 ``開発`` 環境でキャッシュレイヤーを有効にしてしまうと、コンフィギュレーションファイルなどに変更をかけた際に毎回キャッシュをクリアしないといけないので、とても大変です。

もう１つ ``テスト`` 環境があります。これは、ユニットテストや機能テストなどアプリケーションのテストを実行する際に使われます。テストに関してはまだ説明していませんが、次章でテストに関する説明を行います。

フロントコントローラ
~~~~~~~~~~~~~~~~~~~~

今まで、このチュートリアルでは ``開発`` 環境のみ使用してきました。 ``http://symblog.dev/app_dev.php/about`` のように symblog にリクエストをする際に、 ``app_dev.php`` フロントコントローラを使用して ``開発`` 環境の実行を指定してきました。 ``web/app_dev.php`` にあるフロントコントローラを見てみると、次の行があります。

.. code-block:: php

    $kernel = new AppKernel('dev', true);

この行で Symfony2 を始動させています。 Symfony2 の ``AppKernel`` のインスタンスを初期化して、環境を ``dev`` にセットしています。

では、次に ``本番`` 環境のフロントコントローラ ``web/app.php`` を見てみると次のようになっています。

.. code-block:: php

    $kernel = new AppKernel('prod', false);

今回は、 ``AppKernel`` に ``prod`` 環境を渡しているのが確認できます。

テスト環境は、ウェブブラウザから実行させるべきではないので、 ``app_test.php`` フロントコントローラはありません。

コンフィギュレーション設定
~~~~~~~~~~~~~~~~~~~~~~~~~~

フロントコントローラがどうやって環境の変更を使用して、アプリケーション実行させているのかを上記で見てきました。次は、各環境で実行する際にどうやって設定を変更しているのかを見ていきます。 ``app/config`` ディレクトリのファイルを見てみると、多くの ``config.yml`` ファイルがあるのに気づくでしょう。メインのファイルは、 ``config.yml`` ファイルで、他に環境の名前が接尾辞に付いた ``config_dev.yml``, ``config_test.yml``, ``config_prod.yml`` があります。現在の環境に応じて、これらのファイルのどれかがロードされます。 ``config_dev.yml`` を見てみると次の行が一番上にあるのに気づくでしょう。

.. code-block:: yaml

    imports:
        - { resource: config.yml }

``imports`` 命令は、 ``config.yml`` ファイルをこのファイルにインクルードさせます。 ``config_test.yml`` や ``config_prod.yml`` ファイルの両方の一番上にも、この ``imports`` 命令があるのが確認できます。 ``config.yml`` にある共通のコンフィギュレーション設定をインクルードすることによって、各環境の特定の設定でオーバーライドすることができます。 ``app/config/config_dev.yml`` にある ``開発`` コンフィギュレーションファイルでは、次のようにデバッグツールバーの使用を設定している行を確認するこができます。

.. code-block:: yaml

    # app/config/config_dev.yml
    
    web_profiler:
        toolbar: true

この設定は、 ``本番`` では、ディベロッパーツールバーを表示させないので、 ``config_prod.yml`` のコンフィギュレーションファイルにはありません。

本番環境での実行
~~~~~~~~~~~~~~~~

ついに ``本番`` 環境での動作を確認するときが来ました。

まず、次の Symfony2 のタスクを使用してキャッシュをクリアする必要があります。

.. code-block:: bash

    $ php app/console cache:clear --env=prod

次にブラウザで、 ``http://symblog.dev/`` にアクセスしてみましょう。 ``app-dev.php`` フロントコントローラを指定しないでアクセスしてみてください。

.. note::
    
    パート１で説明した Dynamic Virtual Host を使用してるのであれば、次の .htaccess ファイルを ``web/.htaccess`` に追加する必要があります。
    
    .. code-block:: text
    
        <IfModule mod_rewrite.c>
            RewriteBase /
            # ..
        </IfModule>
        

サイトの見た目は、ほとんど同じようになっていますね。しかし、多少重要な機能が異なります。ディベロッパーツールバーが無くなり、詳細な例外メッセージが表示されなくなりました。  ``http://symblog.dev/999`` にアクセスして、この状態を確認してください。

.. image:: /_static/images/part_5/production_error.jpg
    :align: center
    :alt: Production - 404 Error
    
詳細な例外メッセージが、ユーザに問題を知らせるシンプルなメッセージに置き換わりました。これらの例外スクリーンは、あなたのアプリケーションの好きなようにカスタマイズすることができます。この章の最後でその方法を見ていきましょう。

さらに、アプリケーションの実行から ``app/log/prod.log`` ファイルにログが書かれたのを確認することができます。 ``production`` 環境のアプリケーションで、問題があった際にエラーや例外はスクリーンに表示されることはないので、便利です。

.. tip::

    ``htp://symblog.dev/`` へのリクエストがどうやって ``app.php`` ファイルに辿り着いたのでしょうか。今まで ``index.html`` や ``index.php`` をサイトインデックスとして使用してきた人も多いでしょうが、 ``app.php`` がサイトインデックスとして使用されるのはなぜでしょうか？ これは、 ``web/.htaccess`` ファイルの RewriteRule によるものです。

    .. code-block:: text

        RewriteRule ^(.*)$ app.php [QSA,L]

    この行はどんなテキストにもマッチする正規表現 ``^(.*)$`` が書かれており、 ``app.php`` に渡していることがわかります。

    ``mod_rewrite.c`` モジュールを有効にしていていない Apache サーバを使用しているかもしれません。その際は、単に ``app.php`` を ``http://symblog.dev/app.php/`` のように加えてください。

``本番`` 環境の基本をカバーしてきましたが、エラーページのカスタマイズや `capifony <http://capifony.org/>`_ などのツールを使用しての ``本番`` サーバへのデプロイをカバーしていません。これらのトピックは、後の章で扱うことにしましょう。

新しく環境を作成する
~~~~~~~~~~~~~~~~~~~~

最後に、 Symfony2 に独自の環境を簡単にセットアップすることができることも言及しておきます。例えば、本番サーバで動くステージング環境が欲しいが、例外などのデバッグ情報も表示したいとします。そうすることによって、本番環境と開発環境のサーバのコンフィギュレーションが異なっても、実際の本番サーバでテストすることが可能になります。

新しく環境を作成することは簡単ですが、このチュートリアルで扱う内容ではありません。 Symfony2 のクックブックにこの方法を説明している `記事 <http://symfony.com/doc/current/cookbook/configuration/environments.html>`_ がありますので参考にしてください。

Assetic
-------

Symfony2 の標準ディストリビューションには、 `Assetic <https://github.com/kriswallsmith/assetic>`_ と呼ばれるアセット管理ライブラリが付いてきます。このライブラリは、 Python のライブラリ  `webassets <http://elsdoerfer.name/files/docs/webassets/>`_ にインスパイアされたもので、 `Kris Wallsmith <https://twitter.com/#!/kriswallsmith>`_ によって開発されました。

Assetic は、画像、スタイルシート、 JavaScript などのアセットの管理と、これらのファイルに適用するフィルターの管理を行います。フィルターは、 CSS や JavaScript の圧縮、 `CoffeeScript <http://jashkenas.github.com/coffee-script/>`_ ファイルのコンパイル、 HTTP リクエストを減らすことのできる複数のアセットファイルの結合など、便利な機能があります。

これまではテンプレートにアセットをインクルードするのに、次のように Twig の ``asset`` 関数を使用してきました。

.. code-block:: html
    
    <link href="{{ asset('bundles/bloggerblog/css/blog.css') }}" type="text/css" rel="stylesheet" />

``asset`` 関数を使用すれば、 Assetic によって置き換えられます。

アセット
~~~~~~~~

Assetic ライブラリは、アセットを次のように扱います。

`Assetic で使用できるアセットは、ロードやダンプのできるフィルターで使用できます。アセットはメタデータを持っていることもあり、フィルターで操ることができるもの、できないものがあります。`

簡単に言えば、アセットはアプリケーションが使用するスタイルシートや画像といったリソースのことです。

スタイルシート
..............

``BloggerBlogBundle`` のメインレイアウトテンプレートのスタイルシートに ``asset`` を使用するように置き換えてみましょう。 ``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` のテンプレートを次のように修正してください。

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    
    {# .. #}

    {% block stylesheets %}
        {{ parent () }}
        
        {% stylesheets 
            '@BloggerBlogBundle/Resources/public/css/*'
        %}
            <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
        {% endstylesheets %}
    {% endblock %}
    
    {# .. #}

CSS ファイルへのリンクを２つ Assetic 関数で置き換えました。 Assetic の ``stylesheets`` を使用して、 ``src/Blogger/BlogBundle/Resources/public/css`` ディレクトリにある全ての CSS ファイルを１つのファイルに結合し、出力しました。ファイルの結合はとても単純ですが、読み込ませるファイルの数を減らすので、効果的な最適化です。ファイルが少なければ少ないほど、 HTTP のリクエストを減らすことができるのです。 ``css`` ディレクトリ内で ``*`` を使用して全てのファイルを指定していますが、次のように個々のファイルをリスト化することもできます。

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    
    {# .. #}

    {% block stylesheets %}
        {{ parent () }}
        
        {% stylesheets 
            '@BloggerBlogBundle/Resources/public/css/blog.css'
            '@BloggerBlogBundle/Resources/public/css/sidebar.css'
        %}
            <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
        {% endstylesheets %}
    {% endblock %}

    {# .. #}

両方のケースの最終結果は同じになります。最初のオプションでは、 ``*`` を使用しているので、新しく CSS ファイルがそのディレクトリに追加された際に、全てのファイルを Assetic によって結合してCSS をインクルードします。ウェブサイトによってはこの動作は望ましいものではないかもしれませんので、上の２つのどちらかをニーズに応じて使い分けてください。
    
``http://symblog.dev/app_dev.php/`` で出力された HTML を見てみると次のようになっています CSS が次のようにインクルードされているのを確認できます(今は ``開発`` 環境で実行させています)。

.. code-block:: html
    
    <link href="/app_dev.php/css/d8f44a4_part_1_blog_1.css" rel="stylesheet" media="screen" />
    <link href="/app_dev.php/css/d8f44a4_part_1_sidebar_2.css" rel="stylesheet" media="screen" />
    
まず、ファイルが２つあることに疑問を持ったかもしれません。上記の説明では、 Assetic が１つの CSS ファイルに結合すると説明しました。これは ``開発`` 環境で symblog を実行しているからです。以下のように debug フラグを false にセットして Assetic にデバッグを使用しないモードで実行させることができます。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    
    {# .. #}
    
    {% stylesheets 
        '@BloggerBlogBundle/Resources/public/css/*'
        debug=false
    %}
        <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
    {% endstylesheets %}

    {# .. #}
    
これで レンダリングされた HTML を見ると、次のようになっているはずです。

.. code-block:: html

    <link href="/app_dev.php/css/3c7da45.css" rel="stylesheet" media="screen" />
    
このファイルのソースを見てみると、２つの CSS ファイル ``blog.css`` と ``sidebar.css`` が１つのファイルに結合されているのが確認できます。作成された CSS ファイルの名前は Assetic によってランダムに生成されます。生成するファイル名を指定したい際には、次のように ``output`` オプションを使用してください。

.. code-block:: html

    {% stylesheets 
        '@BloggerBlogBundle/Resources/public/css/*'
        output='css/blogger.css'
    %}
        <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
    {% endstylesheets %}

先に進む前に、１つ前のスニペットから debug フラグを取り除いて、アセットの動作をデフォルトに戻しておきます。

また、アプリケーションのベーステンプレートの ``app/Resources/views/base.html.twig`` を次のように修正する必要があります。

.. code-block:: html

    {# app/Resources/views/base.html.twig #}
    
    {# .. #}
    
    {% block stylesheets %}
        <link href='http://fonts.googleapis.com/css?family=Irish+Grover' rel='stylesheet' type='text/css'>
        <link href='http://fonts.googleapis.com/css?family=La+Belle+Aurore' rel='stylesheet' type='text/css'>
        {% stylesheets 
            'css/*'
        %}
            <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
        {% endstylesheets %}
    {% endblock %}
    
    {# .. #}
    
JavaScripts
...........

現時点では、アプリケーションに JavaScript を使用していませんが、 Assetic の使用方法は、スタイルシートの際とほとんど同じです。

.. code-block:: html

    {% javascripts 
        '@BloggerBlogBundle/Resources/public/js/*'
    %}
        <script type="text/javascript" src="{{ asset_url }}"></script>
    {% endjavascripts %}

フィルター
~~~~~~~~~~

Assetic の本当に素晴らしい機能は、フィルターです。フィルターは、個々のアセットやアセットのコレクションに適用することができます。以下の共通のフィルターを含め、たくさんのフィルターがコアライブラリとして提供されています。

1. ``CssMinFilter``: CSS の圧縮
2. ``JpegoptimFilter``: JPEG ファイルの最適化
3. ``Yui\CssCompressorFilter``: YUI compressor を使用した CSS の圧縮
4. ``Yui\JsCompressorFilter``: the YUI compressor を使用した JavaScript の圧縮
5. ``CoffeeScriptFilter``: CoffeeScript を JavaScript にコンパイル

利用可能なフィルターの一覧は、 `Assetic Readme <https://github.com/kriswallsmith/assetic/blob/master/README.md>`_  を参照してください。

これらのフィルターのほとんどは、 YUI Compressor のように、実際の作業は外部のプログラムやライブラリに任せています。そのため、フィルターを使用するためのライブラリをインストールして設定する必要があります。

`YUI Compressor <http://yuilibrary.com/download/yuicompressor/>`_ をダウンロードして、圧縮ファイルを解凍し、 ``build`` ディレクトリにあるファイルを ``app/Resources/java/yuicompressor-2.4.6.jar`` にコピーしてください。今回は、 YUI Compressor のバージョン ``2.4.6`` をダウンロードしたと想定しています。もしバージョンが異なれば、適宜変更してください。

次に、 YUI Compressor を使用して CSS を圧縮する Assetic のフィルターを設定しましょう。 ``app/config/config.yml`` にあるアプリケーションコンフィギュレーションを次のように修正してください。

.. code-block:: yaml
    
    # app/config/config.yml
    
    # ..

    assetic:
        filters:
            yui_css:
                jar: %kernel.root_dir%/Resources/java/yuicompressor-2.4.6.jar
    
    # ..
   
``yui_css`` という名前のフィルターを設定して YUI Compressor を実行えきるようにアプリケーションのリソースディレクトリに jar ファイルを配置しました。このフィルターを使用するために、どのアセットにフィルターを適用させたいか指定する必要があります。次のように ``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` のテンプレートを修正して ``yui_css`` フィルターを適用するようにしてください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}

    {# .. #}
    
    {% stylesheets 
        '@BloggerBlogBundle/Resources/public/css/*'
        output='css/blogger.css'
        filter='yui_css'
    %}
        <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
    {% endstylesheets %}

    {# .. #}

symblogの ウェブサイトを再読み込みし、 Assetic によって出力されたファイルを見てみると、ファイルが圧縮されているのに気づくでしょう。圧縮化は本場サーバでは良い方法ですが、デバッグを難しくしてしまします。特に JavaScript の圧縮に関しては大変です。 ``?`` の接頭辞を使用すれば、 ``開発`` 環境で実行する際に圧縮を無効にすることができます。

.. code-block:: html
    
    {% stylesheets 
        '@BloggerBlogBundle/Resources/public/css/*'
        output='css/blogger.css'
        filter='?yui_css'
    %}
        <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
    {% endstylesheets %}

本番環境のためアセットをダンプする
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

本番環境では、 Assetic を使用してアセットファイルをダンプすることができます。そうすることによって実際にディスクに存在するリソースになります。全てのページで Assetic によるアセットファイル作成の処理をすると、サイトがとても遅くなってしまいます。特にアセットにフィルターを適用している際にです。 ``本番`` 環境でアセットをダンプすれば、ウェブサーバからは Assetic の管理によるアセットが使われるのではなく、ディスクに作成されたアセットファイルを直接使うことができるのです。次のタスクを実行して、アセットファイルのダンプを作成しましょう。

.. code-block:: bash

    $ app/console --env=prod assetic:dump

全ての CSS ファイルが結合され、 ``blogger.css`` として ``web/css`` ディレクトリに作成されます。これで ``本番`` 環境の ``http://symblog.dov/`` を見てみると、このフォルダを直接使用しているのが確認できます。

.. note::

    アセットファイルをディスクにダンプした後に、 ``開発`` 環境で戻したい際には、 ``web/`` ディレクトリに作られたアセットファイルをクリーンアップする必要があり、 Assetic にもう一度作成してもらうようにしてください。

さらに詳細を調べる
~~~~~~~~~~~~~~~~~~

今回は Assetic が行うことのできる触りしか紹介しませんでした。オンラインで読むことのできるリソースとして Symfony2 のクックブックのレシピがあります。

`How to Use Assetic for Asset Management <http://symfony.com/doc/current/cookbook/assetic/asset_management.html>`_

`How to Minify JavaScripts and Stylesheets with YUI Compressor <http://symfony.com/doc/current/cookbook/assetic/yuicompressor.html>`_

`How to Use Assetic For Image Optimization with Twig Functions <http://symfony.com/doc/current/cookbook/assetic/jpeg_optimize.html>`_

`How to Apply an Assetic Filter to a Specific File Extension <http://symfony.com/doc/current/cookbook/assetic/apply_to_option.html>`_

また、以下に `Richard Miller <https://twitter.com/#!/mr_r_miller>`_ による多くの記事を挙げておきます。

`Symfony2: Using CoffeeScript with Assetic <http://miller.limethinking.co.uk/2011/05/16/symfony2-using-coffeescript-with-assetic/>`_

`Symfony2: A Few Assetic Notes <http://miller.limethinking.co.uk/2011/06/02/symfony2-a-few-assetic-notes/>`_

`Symfony2: Assetic Twig Functions <http://miller.limethinking.co.uk/2011/06/23/symfony2-assetic-twig-functions/>`_

.. tip::

    Richard Miller は、 DI 、サービス、Assetic ガイドなどの Symfony2 に関する素晴らしい記事を書いていますので、言及しておきましょう。 `symfony2 でタグ付けされた記事 <http://miller.limethinking.co.uk/tag/symfony2/>`_ を読んでみてください。

結論
----

Symfony2 環境や Assetic ライブラリの使用方法など Symfony2 に関する新しい領域をカバーしてきました。また、ホームページを改良し、サイドバーにコンポーネントを追加しました。

次章では、テストを行います。 PHPUnit を使用したユニットテストと機能テストを両方見ていきます。機能テストを書く際にアシストをしてくれるクラスが Symfony2 に組み込まれていますので、それを見ていきます。ウェブリクエストをシミュレートするクラスや、フォームに値を入れリンクをクリックさせてくれるクラス、レスポンスをらべてくれるクラスなどです。
