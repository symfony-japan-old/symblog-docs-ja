[パート４] - コメントモデル: コメントの追加、 Doctrine リポジトリとマイグレーション
===================================================================================

要約
--------

この章では、前章で定義したブログモデルを構築します。そして、ブログエントリのコメントを扱うコメントモデルを作成します。各ブログは複数のコメントを含むことができるように、モデル間の関連を作成する方法を説明します。データベースからエンティティを検索するのに、Docrine 2 QueryBuilder クラスと Doctrine 2 Repository クラスを使用します。そして、データベースのスキーマの変更をデプロイするに実用的な方法である Doctrine 2 マイグレーション(Doctrine 2 Migration)のコンセプトを説明します。この章の最後では、コメントモデルを作成し、ブログモデルにリンクします。また、ホームページ(homepage)を作成し、ユーザが各ブログエントリにコメントを投稿できるようにします。


ホームページ
------------

まずホームページを構築するところからこの章を始めましょう。一般的なブログの傾向では、ホームページに新しい投稿順に各ブログエントリのスニペットを表示します。完全なブログエントリ内容は、 show ページへのリンクをたどることで見ることができます。ホームページのルーティングルール、コントローラ、ビューは既に作成してありますので、これらを修正していきます。

ブログエントリ一覧を検索する: クエリー実行とモデル
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ブログエントリ一覧を表示するには、データベースへの検索が必要になります。 Doctrine 2 は、 `Doctrine Query Language <http://www.doctrine-project.org/docs/orm/2.1/en/reference/dql-doctrine-query-language.html>`_ (DQL)と、 `QueryBuilder <http://www.doctrine-project.org/docs/orm/2.1/en/reference/query-builder.html>`_ を使用してデータベースの検索を行います(もちろん Doctrine 2 を介して生の SQL を実行することもできますが、 Doctrine 2 のデータベース抽象を使用しないのでお勧めはしません)。 DQL を生成するためのオブジェクト指向的な方法を提供してくれる ``QueryBuilder`` を使用してデータベースへのクエリーを実行します。  ``src/Blogger/BlogBundle/Controller/PageController.php`` にある ``Page`` コントローラの ``index`` アクションを次のように修正してデータベースからブログを検索しましょう。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    class PageController extends Controller
    {
        public function indexAction()
        {
            $em = $this->getDoctrine()
                       ->getEntityManager();
    
            $blogs = $em->createQueryBuilder()
                        ->select('b')
                        ->from('BloggerBlogBundle:Blog',  'b')
                        ->addOrderBy('b.created', 'DESC')
                        ->getQuery()
                        ->getResult();
    
            return $this->render('BloggerBlogBundle:Page:index.html.twig', array(
                'blogs' => $blogs
            ));
        }
        
        // ..
    }

上記を見ると、まず ``EntityManager`` から ``QueryBuilder`` のインスタンスを取得しています。 ``QueryBuilder`` にはメソッドがたくさんあり、これらを使ってクエリーを組み立てることができます。 ``QueryBuilder`` のドキュメントで利用可能なメソッドの一覧を参照することができます。 `ヘルパーメソッド <http://www.doctrine-project.org/docs/orm/2.1/en/reference/query-builder.html#helper-methods>`_ から始めてみてください。ヘルパーメソッドには、 ``select()``, ``form()``, ``addOrderBy()`` などがあります。 Doctrine 2 を使用すると、 ``Blogger\BlogBundle\Entity\Blog`` とフルパスで書かなくても、 ``BloggerBlogBundle:Blog`` として ``Blog`` エンティティを照合することができます。クエリーに条件を特定したら、 ``getQuery()`` メソッドを呼んで ``DQL`` のインスタンスを取得します。 ``QueryBuilder`` オブジェクトでは、検索ができないので、 ``DQL`` インスタンスに変換する必要があります。そして、 ``DQL`` インスタンスが ``getResult()`` メソッドを呼び ``Blog`` のエントリ一覧を検索します。 ``DQL`` インスタンスには `たくさんのメソッド <http://www.doctrine-project.org/docs/orm/2.1/en/reference/dql-doctrine-query-language.html#query-result-formats>`_ があります。 ``getSingleResult()`` や ``getArrayResult()`` メソッドはその一部です。

ビュー
......

これで ``Blog`` エンティティのコレクションが手に入りましたので、その内容を表示しましょう。ホームページのテンプレート ``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig`` を次の内容に修正してください。

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block body %}
        {% for blog in blogs %}
            <article class="blog">
                <div class="date"><time datetime="{{ blog.created|date('c') }}">{{ blog.created|date('l, F j, Y') }}</time></div>
                <header>
                    <h2><a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id }) }}">{{ blog.title }}</a></h2>
                </header>
        
                <img src="{{ asset(['images/', blog.image]|join) }}" />
                <div class="snippet">
                    <p>{{ blog.blog(500) }}</p>
                    <p class="continue"><a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id }) }}">Continue reading...</a></p>
                </div>
        
                <footer class="meta">
                    <p>Comments: -</p>
                    <p>Posted by <span class="highlight">{{blog.author}}</span> at {{ blog.created|date('h:iA') }}</p>
                    <p>Tags: <span class="highlight">{{ blog.tags }}</span></p>
                </footer>
            </article>
        {% else %}
            <p>There are no blog entries for symblog</p>
        {% endfor %}
    {% endblock %}

ここで Twig の制御構造の１つ ``for..else..endfor`` を紹介しましょう。今までテンプレートエンジンを使用していなければ、次のような PHP スニペットを書いていたことでしょう。

.. code-block:: php

    <?php if (count($blogs)): ?>
        <?php foreach ($blogs as $blog): ?>
            <h1><?php echo $blog->getTitle() ?><?h1>
            <!-- rest of content -->
        <?php endforeach ?>
    <?php else: ?>
        <p>There are no blog entries</p>
    <?php endif ?>

Twig の ``for..else..endfor`` 制御構造は、この作業をよりクリーンにすることができます。ホームページテンプレートのコードのほとんどは、 HTML でブログの内容を出力することに携わっています。しかし、いくつか言及しておくことがあります。まず Twig の ``path`` 関数を使用して、ブログの show ページへのルートを生成しています。ブログの show ページは、 URL にブログ ``ID`` を使用していますので、 ``path`` 関数に引数として渡す必要があります。これは、次のようにします。

.. code-block:: html
    
    <h2><a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id }) }}">{{ blog.title }}</a></h2>
    
次に、 ``<p>{{ blog.blog(500) }}</p>`` を使用してブログの内容を出力します。ここで渡した ``500`` という引数で、ブログエントリの内容の長さを指定しています。これを実際に動かすためには、前に Doctrine 2 のタスクで生成した ``getBlog`` メソッドを修正する必要があります。 ``src/Blogger/BlogBundle/Entity/Blog.php`` にある ``Blog`` エンティティの ``getBLog`` を次のように修正してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php
    public function getBlog($length = null)
    {
        if (false === is_null($length) && $length > 0)
            return substr($this->blog, 0, $length);
        else
            return $this->blog;
    }

``getBlog`` メソッドの普段の動作は、ブログエントリの内容を全て返すようにするべきなので、デフォルト値が ``null`` の ``$length`` パラメータをセットしました。 ``null`` 値が渡るとブログエントリの内容を全て返します。

ブラウザで ``http://symblog.dev/app_dev.php/`` にアクセスすると、次のように最新のブログエントリが表示されるはずです。また、ブログのタイトル、または、 'continue reading...' のリンクをクリックすると、各ブログの show ページにナビゲートするようになっているはずです。

.. image:: /_static/images/part_4/homepage.jpg
    :align: center
    :alt: symblog homepage

コントローラ内でエンティティのクエリーを実行することもできますが、これは修正した方が良いでしょう。それは、次の理由からです。

    1. このアプリケーション内の他の場所から同じクエリーの実行を再利用することができません。もしくは、 ``QueryBuilder`` のコードが重複することになります。
    2. ``QueryBuilder`` のコードが重複してしまうと、クエリーを変更する必要がある際に、重複した場所全てを修正する必要があります。
    3. クエリーとコントローラを分離させることによって、コントローラから独立したクエリーのテストができるようになります。

Doctrine 2 のリポジトリ(Repository)クラスを使用すると、クエリーとコントローラの分離を簡単にできます。

Doctrine 2 リポジトリ(Repositories)
-----------------------------------

前章でブログの show ページを作成した際に、 Doctrine 2 のリポジトリクラスを紹介しました。その際は、 ``Doctrine\ORM\EntityRepository`` クラスのデフォルトの実装で、 ``find()`` メソッドを使用してブログエントリをデータベースから検索しました。今回は、カスタムクエリーを作成しますので、カスタムリポジトリを作成する必要があります。 Doctrine 2 は、この作業をアシストしてくれます。 ``src/Blogger/BlogBundle/Entity/Blog.php`` にある ``Blog`` エンティティのメタデータを次のように修正してください。


.. code-block:: php
    
    // src/Blogger/BlogBundle/Entity/Blog.php
    /**
     * @ORM\Entity(repositoryClass="Blogger\BlogBundle\Entity\Repository\BlogRepository")
     * @ORM\Table(name="blog")
     * @ORM\HasLifecycleCallbacks()
     */
    class Blog
    {
        // ..
    }

上記で、このエンティティに関連するリポジトリの ``BlogRepository`` クラスのネームスペースを指定したのに気づいたでしょう。次のようにもう一度 ``doctrine:generate:entities`` タスクを実行してください。

.. code-block:: bash

    $ php app/console doctrine:generate:entities Blogger\BlogBundle
    
Doctrine 2 は、 ``/BlogBundle/Entity/Repository/BlogRepository.php`` に ``BlogRepository`` リポジトリのシェルクラスを作成したはずです。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Repository/BlogRepository.php
    
    namespace Blogger\BlogBundle\Entity\Repository;

    use Doctrine\ORM\EntityRepository;

    /**
     * BlogRepository
     *
     * This class was generated by the Doctrine ORM. Add your own custom
     * repository methods below.
     */
    class BlogRepository extends EntityRepository
    {

    }

``BlogRepository`` クラスは、 ``find()`` メソッドを持つ ``EntityRepository`` クラスを拡張しています。 ``BlogRepository`` クラスを修正して ``QueryBuilder`` のコードを ``Page`` コントローラからこちらに移動しましょう。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Repository/BlogRepository.php

    namespace Blogger\BlogBundle\Entity\Repository;

    use Doctrine\ORM\EntityRepository;

    /**
     * BlogRepository
     *
     * This class was generated by the Doctrine ORM. Add your own custom
     * repository methods below.
     */
    class BlogRepository extends EntityRepository
    {
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
    }

最新のブログエントリを取得するコントローラの ``QueryBuilder`` コードと同じ動作をする ``getLatestBlogs`` メソッドを作成しました。リポジトリクラスでは、 ``createQueryBuilder()`` メソッドを介して ``QueryBuilder`` へ直接アクセスすることができます。また、検索結果の最大値を指定できるように ``$limit`` パラメータも追加しました。クエリーの結果は、コントローラ内のときと同じになります。 ``from()`` メソッドでエンティティを指定する必要がなくなったことに気づいたかもしれません。 ``BlogRepository`` は、 ``Blog`` エンティティと関連付けられているので、必要なくなったのです。 ``EntityRepository`` クラスの ``createQueryBuilder`` メソッドの実装を見てみると、次のように ``from()`` メソッドを呼んでいるのが確認できます。

.. code-block:: php
    
    // Doctrine\ORM\EntityRepository
    public function createQueryBuilder($alias)
    {
        return $this->_em->createQueryBuilder()
            ->select($alias)
            ->from($this->_entityName, $alias);
    }

最後に ``Page`` コントローラの ``index`` アクションで ``BlogRepository`` を使用するように修正しましょう。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    class PageController extends Controller
    {
        public function indexAction()
        {
            $em = $this->getDoctrine()
                       ->getEntityManager();
                       
            $blogs = $em->getRepository('BloggerBlogBundle:Blog')
                        ->getLatestBlogs();
                       
            return $this->render('BloggerBlogBundle:Page:index.html.twig', array(
                'blogs' => $blogs
            ));
        }
        
        // ..
    }

これでホームページを再読み込みすると、前と全く同じ内容が表示されるはずです。ここでやったことは、コードのリファクタリングで、作業とクラスの配置を正しくするようにしたのです。

さらにモデルについて: コメントエンティティの作成
------------------------------------------------

ブログは著者が書くだけのものではないですよね。読者もブログエントリにコメントができるようにするべきです。これらのコメントも永続化する必要があり、また、個々のブログエントリはそれぞれ複数のコメントを含むことになるので、コメントは ``Blog`` エンティティにリンクさせる必要もあります。

``Comment`` エンティティクラスのベースを定義するところから始めましょう。 ``src/Blogger/BlogBundle/Entity/Comment.php`` に新しいファイルを作成して次の内容をペーストしてください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Comment.php

    namespace Blogger\BlogBundle\Entity;

    use Doctrine\ORM\Mapping as ORM;

    /**
     * @ORM\Entity(repositoryClass="Blogger\BlogBundle\Entity\Repository\CommentRepository")
     * @ORM\Table(name="comment")
     * @ORM\HasLifecycleCallbacks
     */
    class Comment
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
        protected $user;

        /**
         * @ORM\Column(type="text")
         */
        protected $comment;

        /**
         * @ORM\Column(type="boolean")
         */
        protected $approved;
        
        /**
         * @ORM\ManyToOne(targetEntity="Blog", inversedBy="comments")
         * @ORM\JoinColumn(name="blog_id", referencedColumnName="id")
         */
        protected $blog;

        /**
         * @ORM\Column(type="datetime")
         */
        protected $created;

        /**
         * @ORM\Column(type="datetime")
         */
        protected $updated;

        public function __construct()
        {
            $this->setCreated(new \DateTime());
            $this->setUpdated(new \DateTime());
            
            $this->setApproved(true);
        }

        /**
         * @ORM\preUpdate
         */
        public function setUpdatedValue()
        {
           $this->setUpdated(new \DateTime());
        }
    }

上記の内容のほとんどは以前の章でカバーしましたが、今回は ``Blog`` エンティティへのリンクをセットアップするメタデータを使用しました。コメントはブログの投稿に対してのものなので、 ``Comment`` エンティティが属する ``Blog`` エンティティにリンクするためのメタデータを使用しました。それを実現するために ``Blog`` エンティティをターゲットとして ``ManyToOne`` リンクを指定しました。また、 ``Blog`` からの逆リンクを ``comments`` を通して想定として、指定しました。逆リンクを作成するには ``Blog`` エンティティを修正して、 Doctrine 2 に個々のブログが複数のコメントを含むことができるようになっているように伝える必要があります。 ``src/Blogger/BlogBundle/Entity/Blog.php`` の ``Blog`` エンティティを修正して、次のようにマッピングを追加してください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Blog.php

    namespace Blogger\BlogBundle\Entity;

    use Doctrine\ORM\Mapping as ORM;
    use Doctrine\Common\Collections\ArrayCollection;

    /**
     * @ORM\Entity(repositoryClass="Blogger\BlogBundle\Entity\Repository\BlogRepository")
     * @ORM\Table(name="blog")
     * @ORM\HasLifecycleCallbacks
     */
    class Blog
    {
        // ..
        
        /**
         * @ORM\OneToMany(targetEntity="Comment", mappedBy="blog")
         */
        protected $comments;
        
        // ..
        
        public function __construct()
        {
            $this->comments = new ArrayCollection();
            
            $this->setCreated(new \DateTime());
            $this->setUpdated(new \DateTime());
        }
        
        // ..
    }

いくつか変更点を説明しましょう。まず、 ``$comments`` メンバーにメタデータを追加しました。前章で Doctrine 2 による永続化をさせたくなかったので、このメンバーには何も追加しなかったことを覚えていますか。もちろんその通りなのですが、このメンバーには Doctrine 2 に関連する ``Comment`` エンティティを格納できるようしたいですよね。それをしてくれるのがこのメタデータです。次に、 Doctrine 2 のために ``$comments`` メンバーを ``ArrayCollection`` オブジェクトにしないといけません。もちろん、その際に ``use`` 命令文で ``ArrayCollection`` クラスをインポートするのを忘れないでください。

``Comment`` エンティティを作成し ``Blog`` エンティティを修正したので、 Doctrine 2 にアクセサを再生成してもらいましょう。次のタスクを実行してください。

.. code-block:: bash

    $ php app/console doctrine:generate:entities Blogger\BlogBundle
    
両方のエンティティがアップデートされ、正しいアクセサメソッドを持つようになりました。また、 ``src/Blogger/BlogBundle/Entity/Repository/CommentRepository.php`` に ``CommentRepository`` が作成されたことに気づいたでしょうか。このファイルは、メタデータで指定していたため生成されたのです。

最後にエンティティに行った変更をデータベースに反映させる必要があります。次のように ``doctrine:schema:update`` タスクを使用することもできますが、ここでは Doctrine 2 マイグレーションを紹介しましょう。

.. code-block:: bash

    $ php app/console doctrine:schema:update --force

Doctrine 2 マイグレーション(Migrations)
---------------------------------------

Doctrine2 マイグレーションエクステンションとバンドルは、 Symfony2 の標準ディストリビューションでは付いてこないので、 DataFixtures エクステンションとバンドルをインストールしたときのように手動でインストールする必要があります。プロジェクトルートの ``composer.json`` ファイルを開いて、次のように Doctrine 2 マイグレーションとバンドルを追加してください。

.. code-block:: php
    
    "require": {
        "doctrine/doctrine-migrations-bundle": "dev-master",
        "doctrine/migrations": "dev-master"
    }

そして、この変更をベンダーに反映させるため、次のタスクを実行してください。

.. code-block:: bash

    $ php composer.phar update

このタスクを実行すると、それぞれの Github リポジトリから最新のバージョンをダウンロードして、正しい場所にインストールします。

.. note::

    Git がインストールされていないマシンを使用しているのであれば、エクステンションとバンドルを手動でダウンロードしインストールする必要があります。

    doctrine-migrations extension: `ダウンロード <http://github.com/doctrine/migrations>`__
    GitHub にあるパッケージの現在のバージョンは、以下の場所に配置されます。
    ``vendor/doctrine-migrations``.

    DoctrineMigrationsBundle: `ダウンロード <http://github.com/symfony/DoctrineMigrationsBundle>`__
    GitHub にあるパッケージの現在のバージョンは、以下の場所に配置されます。
    ``vendor/bundles/Symfony/Bundle/DoctrineMigrationsBundle``.

次に、 ``app/AppKernel.php`` のカーネルにバンドルを登録しましょう。

.. code-block:: php

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            // ...
            new Symfony\Bundle\DoctrineMigrationsBundle\DoctrineMigrationsBundle(),
            // ...
        );
        // ...
    }

.. warning::

    Doctrine 2 マイグレーションライブラリは、まだアルファ版状態なので、現時点で本番サーバに使用するのはお勧めしません。

これでエンティティの変更をデータベースに反映させる準備ができました。これには、２つのステップがあります。まず、 Doctrine 2 のマイグレーションに現在のデータベーススキーマとエンティティ間の違いを調べさせマイグレーションファイルを作成します。これは ``doctrine:migrations:diff`` タスクで行います。次に実際にこのマイグレーションファイルを元にマイグレーションを実行します。これは ``doctrine:migrations:migrate`` タスクで行います。

次の２つのタスクを実行してデータベーススキーマを修正しましょう。

.. code-block:: bash

    $ php app/console doctrine:migrations:diff
    $ php app/console doctrine:migrations:migrate

これでデータベースが最新のエンティティの状態に反映され、新しくコメントテーブルが作成されました。

.. note::

    データベースに新しく ``migration_versions`` という名前のテーブルが作成されたことに気づいたでしょうか。このテーブルにはマイグレーションのバージョンナンバーが格納されます。ですので、データベースの現在のバージョンを調べることができます。
   
.. tip::

    Doctrine 2 マイグレーションは、プログラムで本番データベースの変更をするので、とても良い方法です。つまり、このタスクを開発スクリプトに統合することができるので、アプリケーションの新しいリリースをデプロイする際に自動的にデータベースがアップデートされるのです。また、 Doctrine 2 マイグレーションは全てのマイグレーションで行った変更を ``up`` と ``down`` メソッドを用いてロールバックすることができます。前のバージョンにロールバックするには、ロールバックしたいバージョンナンバーを指定して次のタスクを使用してください。
    
    .. code-block:: bash
    
        $ php app/console doctrine:migrations:migrate 20110806183439
        
データフィクスチャ: 修正版
--------------------------

これで ``Comment`` エンティティが作成されたので、フィクスチャを追加しましょう。エンティティを作成する度にフィクスチャを追加するのは常に良い考えです。メタデータで指定したように、コメントは必ず ``Blog`` エンティティに関連しなければならないので、 ``Comment`` エンティティのフィクスチャを作成するには、 ``Blog`` エンティティを指定する必要があります。 ``Blog`` エンティティのフィクスチャは既に作成しているので、フィクスチャファイルを修正して ``Comment`` エンティティのフィクスチャを追加するだけです。現時点ではエンティティが２つしかないため管理可能ですが、後に users, blog category といったエンティティが追加され、このバンドル内の他のエンティティを全てロードするとしたら、どうなるでしょう？より良い方法は、 ``Comment`` エンティティのフィクスチャのためのファイルを新しく作成することです。ただしこのアプローチの問題は、 ``Comment`` フィクスチャから ``Blog`` エンティティにアクセスする方法です。

幸運にも、これはフィクスチャファイルのオブジェクトにリファレンスをセットすることで簡単に実現することができます。 ``src/Blogger/BlogBundle/DataFixtures/ORM/BlogFixtures.php`` にある ``Blog`` エンティティの ``DataFixtures`` を次のように修正してください。今回の変更の特筆すべきことは、 ``AbstractFixture`` クラスを拡張し、 ``OrderedFixtureInterface`` インタフェースを実装していることです。また、これらのクラスをインポートする ``use`` 命令文も忘れないでください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/DataFixtures/ORM/BlogFixtures.php

    namespace Blogger\BlogBundle\DataFixtures\ORM;

    use Doctrine\Common\DataFixtures\AbstractFixture;
    use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
    use Doctrine\Common\Persistence\ObjectManager;
    use Blogger\BlogBundle\Entity\Blog;

    class BlogFixtures extends AbstractFixture implements OrderedFixtureInterface
    {
        public function load(ObjectManager $manager)
        {
            // ..

            $manager->flush();

            $this->addReference('blog-1', $blog1);
            $this->addReference('blog-2', $blog2);
            $this->addReference('blog-3', $blog3);
            $this->addReference('blog-4', $blog4);
            $this->addReference('blog-5', $blog5);
        }

        public function getOrder()
        {
            return 1;
        }
    }

``addReference()`` メソッドを使用してブログエンティティへのリファレスを追加します。最初のパラメータは、オブジェクトレイヤーを検索することのできるリファレンス識別子です。そして、 ``getOrder()`` メソッドを実装してフィクスチャロード順を指定する必要があります。コメントよりもブログのフィクスチャが早くロードされるべきなので、ここで 1 を返すようにしてあります。

コメントフィクスチャ
~~~~~~~~~~~~~~~~~~~~

次に ``Comment`` エンティティのフィクスチャを定義する準備ができました。フィクスチャファイル ``src/Blogger/BlogBundle/DataFixtures/ORM/CommentFixtures.php`` を新しく作成し、次の内容をペーストしてください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/DataFixtures/ORM/CommentFixtures.php
    
    namespace Blogger\BlogBundle\DataFixtures\ORM;
    
    use Doctrine\Common\DataFixtures\AbstractFixture;
    use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
    use Doctrine\Common\Persistence\ObjectManager;
    use Blogger\BlogBundle\Entity\Comment;
    use Blogger\BlogBundle\Entity\Blog;
    
    class CommentFixtures extends AbstractFixture implements OrderedFixtureInterface
    {
        public function load(ObjectManager $manager)
        {
            $comment = new Comment();
            $comment->setUser('symfony');
            $comment->setComment('To make a long story short. You can\'t go wrong by choosing Symfony! And no one has ever been fired for using Symfony.');
            $comment->setBlog($manager->merge($this->getReference('blog-1')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('David');
            $comment->setComment('To make a long story short. Choosing a framework must not be taken lightly; it is a long-term commitment. Make sure that you make the right selection!');
            $comment->setBlog($manager->merge($this->getReference('blog-1')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Dade');
            $comment->setComment('Anything else, mom? You want me to mow the lawn? Oops! I forgot, New York, No grass.');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Kate');
            $comment->setComment('Are you challenging me? ');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 06:15:20"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Dade');
            $comment->setComment('Name your stakes.');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 06:18:35"));
            $manager->persist($comment);
            
            $comment = new Comment();
            $comment->setUser('Kate');
            $comment->setComment('If I win, you become my slave.');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 06:22:53"));
            $manager->persist($comment);
            
            $comment = new Comment();
            $comment->setUser('Dade');
            $comment->setComment('Your SLAVE?');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 06:25:15"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Kate');
            $comment->setComment('You wish! You\'ll do shitwork, scan, crack copyrights...');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 06:46:08"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Dade');
            $comment->setComment('And if I win?');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 10:22:46"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Kate');
            $comment->setComment('Make it my first-born!');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 11:08:08"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Dade');
            $comment->setComment('Make it our first-date!');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-24 18:56:01"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Kate');
            $comment->setComment('I don\'t DO dates. But I don\'t lose either, so you\'re on!');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-25 22:28:42"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Stanley');
            $comment->setComment('It\'s not gonna end like this.');
            $comment->setBlog($manager->merge($this->getReference('blog-3')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Gabriel');
            $comment->setComment('Oh, come on, Stan. Not everything ends the way you think it should. Besides, audiences love happy endings.');
            $comment->setBlog($manager->merge($this->getReference('blog-3')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Mile');
            $comment->setComment('Doesn\'t Bill Gates have something like that?');
            $comment->setBlog($manager->merge($this->getReference('blog-5')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Gary');
            $comment->setComment('Bill Who?');
            $comment->setBlog($manager->merge($this->getReference('blog-5')));
            $manager->persist($comment);
    
            $manager->flush();
        }
    
        public function getOrder()
        {
            return 2;
        }
    }
        
``BlogFixtures`` クラスにした変更と同じように、 ``CommentFixtures`` クラスも ``AbstractFixture`` クラスを拡張して ``OrderedFixtureInterface`` を実装します。つまり、 ``getOrder()`` メソッドを実装する必要があるということです。このクラスでは、 2 を返すようにして、ブログフィクスチャよりも後に読み込ませるようにします。

予め作成しておいた ``Blog`` エンティティへのリファレンス方法は、次のようになりました。

.. code-block:: php

    $comment->setBlog($manager->merge($this->getReference('blog-2')));

次のタスクを実行して、フィクスチャをデータベースにロードしましょう。

.. code-block:: bash

    $ php app/console doctrine:fixtures:load
    
コメントの表示
--------------

各ブログエントリに関連うるコメントを表示することができるようになりました。 ``CommentRepository`` を修正して、承認されたコメントを新しい順で検索しましょう。

コメントリポジトリ
~~~~~~~~~~~~~~~~~~

``src/Blogger/BlogBundle/Entity/Repository/CommentRepository.php`` にある ``CommentRepository`` クラスを開いて次の内容に入れ替えてください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Repository/CommentRepository.php

    namespace Blogger\BlogBundle\Entity\Repository;

    use Doctrine\ORM\EntityRepository;

    /**
     * CommentRepository
     *
     * This class was generated by the Doctrine ORM. Add your own custom
     * repository methods below.
     */
    class CommentRepository extends EntityRepository
    {
        public function getCommentsForBlog($blogId, $approved = true)
        {
            $qb = $this->createQueryBuilder('c')
                       ->select('c')
                       ->where('c.blog = :blog_id')
                       ->addOrderBy('c.created')
                       ->setParameter('blog_id', $blogId);
            
            if (false === is_null($approved))
                $qb->andWhere('c.approved = :approved')
                   ->setParameter('approved', $approved);
                   
            return $qb->getQuery()
                      ->getResult();
        }
    }
    
このメソッドは、あるブログエントリに関連しているコメントを検索します。クエリーに where 節を加えてください。 where 節は、パラメータ呼び出しを使用しており、 ``setParameter()`` メソッドで指定します。次のようにクエリーに直接値をセットするのではなく、常にパラメータを使用するようにしてください。
    
.. code-block:: php

    ->where('c.blog = ' . blogId)

この例では、 ``$blogId`` の値はサニタイズされていないので、 `SQL インジェクション <http://en.wikipedia.org/wiki/SQL_injection>`_ の危険性を残してしまうことになります。

ブログコントローラ
------------------

次に ``Blog`` コントローラの ``show`` アクションでそのブログエントリのコメントを検索するように、修正する必要があります。 ``src/Blogger/BlogBundle/Controller/BlogController.php`` の ``Blog`` コントローラを次のように修正してください。

.. code-block:: php
    
    // src/Blogger/BlogBundle/Controller/BlogController.php
    
    public function showAction($id)
    {
        // ..

        if (!$blog) {
            throw $this->createNotFoundException('Unable to find Blog post.');
        }
        
        $comments = $em->getRepository('BloggerBlogBundle:Comment')
                       ->getCommentsForBlog($blog->getId());
        
        return $this->render('BloggerBlogBundle:Blog:show.html.twig', array(
            'blog'      => $blog,
            'comments'  => $comments
        ));
    }

承認されたコメントを検索するのに ``CommentRepository`` の新しいメソッドを使用します。テンプレートに ``$comments`` コレクションも渡します。

ブログの show テンプレート
~~~~~~~~~~~~~~~~~~~~~~~~~~

ブログのコメントのリストをテンプレートに渡すようにしましたので、ブログの show テンプレートを修正してコメントを表示することができます。ブログの show テンプレートに直接コメントをレンダリングすることもできますが、コメントは独自のエンティティなので、テンプレートを分離させて、他のテンプレートでレンダリングし、そのテンプレートをインクルードする方が良いでしょう。そうすることによって、このアプリケーションのどこからでもコメントのレンダリングテンプレートを再利用することができるようになります。 ``src/Blogger/BlogBundle/Resources/views/Blog/show.html.twig`` を修正して次の内容を追加してください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Blog/show.html.twig #}
    
    {# .. #}
    
    {% block body %}
        {# .. #}
    
        <section class="comments" id="comments">
            <section class="previous-comments">
                <h3>Comments</h3>
                {% include 'BloggerBlogBundle:Comment:index.html.twig' with { 'comments': comments } %}
            </section>
        </section>
    {% endblock %}
    
上記のテンプレートには、新しい Twig のタグ ``include`` があります。このタグは  ``BloggerBlogBundle:Comment:index.html.twig`` で指定したテンプレートの内容をインクルードします。また、そのテンプレートには何個でも引数を渡すことができます。今回のケースでは、レンダリングすべき ``Comment`` エンティティのコレクションを渡しています。

コメント show テンプレート
~~~~~~~~~~~~~~~~~~~~~~~~~~

上でインクルードしている ``BloggerBlogBundle:Comment:index.html.twig`` ファイルはまだありませんので、作成する必要があります。このファイルはただのテンプレートなので、ルーティングルールやコントローラを作成する必要はありません。テンプレートファイルのみ作成すればいいのです。 ``src/Blogger/BlogBundle/Resources/views/Comment/index.html.twig`` ファイルを新規に作成して次の内容をペーストしてください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Comment/index.html.twig #}
    
    {% for comment in comments %}
        <article class="comment {{ cycle(['odd', 'even'], loop.index0) }}" id="comment-{{ comment.id }}">
            <header>
                <p><span class="highlight">{{ comment.user }}</span> commented <time datetime="{{ comment.created|date('c') }}">{{ comment.created|date('l, F j, Y') }}</time></p>
            </header>
            <p>{{ comment.comment }}</p>
        </article>
    {% else %}
        <p>There are no comments for this post. Be the first to comment...</p>
    {% endfor %}

``Comment`` エンティティのコレクションをイテレートしてコメントを表示しています。 Twig の便利な関数 ``cycle`` を説明します。この関数は、渡した配列の値を巡回し、ループ進行のイテレーションを行います。ループイテレーションの現在値は、 ``loop.index0`` という特別な変数で取得することができます。この変数は、 0 から始まるループイテレーションのカウントを持っています。ループコードブロックで使用できる `特別変数 <http://www.twig-project.org/doc/templates.html#for>`_ はたくさんありますのでドキュメントを参照してください。また、 ``article`` 要素に HTML ID をセットしたことに気づいたでしょうか？これは、後でコメントへのパーマリンクを付ける際に使います。

コメントの show の CSS
~~~~~~~~~~~~~~~~~~~~~~~~

最後にコメントをスタイリッシュに見せるため、 CSS を追加しましょう。 スタイルシート ``src/Blogger/BlogBundle/Resorces/public/css/blog.css`` を修正して次の内容を追加してください。

.. code-block:: css

    /** src/Blogger/BlogBundle/Resorces/public/css/blog.css **/
    .comments { clear: both; }
    .comments .odd { background: #eee; }
    .comments .comment { padding: 20px; }
    .comments .comment p { margin-bottom: 0; }
    .comments h3 { background: #eee; padding: 10px; font-size: 20px; margin-bottom: 20px; clear: both; }
    .comments .previous-comments { margin-bottom: 20px; }

.. note::

    ``web`` フォルダ内へのバンドルのアセットのリファレンス方法にシンボリックリンクを使用していなければ、次のアセットインストールのタスクをもう一度実行して修正した CSS をコピーする必要があります。

    .. code-block:: bash

        $ php app/console assets:install web
        
ブラウザで ``http://symblog.dev/app_dev.php/2`` などのブログの show ページの１つを見てみると、以下のようにブログコメントが出力されているのが確認できるはずです。

.. image:: /_static/images/part_4/comments.jpg
    :align: center
    :alt: symblog show blog comments
    
コメントの追加
--------------

この章の最後では、ユーザがブログにコメントを投稿できるようにします。コメントはブログの show ページのフォームで投稿します。問い合わせフォームを作成したときに既に Symfony2 でのフォーム作成を説明しました。手動でコメントフォームを作るのではなく、 Symfony2 のタスクで作ることができます。次のタスクを実行して ``Comment`` エンティティの ``CommentType`` クラスを生成してください。

.. code-block:: bash
    
    $ php app/console generate:doctrine:form BloggerBlogBundle:Comment
    
上記のように、ここでも ``Comment`` エンティティの指定にショートカットを使うことができます。

.. tip::

    お気づきかもしれませんが、 ``doctrine:generate:form`` も使用可能です。ネームスペースが異なるだけで同じタスクを実行します。
    
フォーム生成タスクは、 ``src/Blogger/BlogBundle/Form/CommentType.php`` に次のような ``CommentType`` クラスを作成します。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Form/CommentType.php
    
    namespace Blogger\BlogBundle\Form;
    
    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;
    
    class CommentType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder
                ->add('user')
                ->add('comment')
                ->add('approved')
                ->add('created')
                ->add('updated')
                ->add('blog')
            ;
        }
    
        public function getName()
        {
            return 'blogger_blogbundle_commenttype';
        }
    }

以前出てきた ``EnquiryType`` クラスを覚えていれば、このクラスが何をするかご存知でしょう。このクラスをカスタマイズするところから始めますが、一旦カスタマイズ前のフォームを表示してみましょう。

コメントフォームを表示する
~~~~~~~~~~~~~~~~~~~~~~~~~~

ブログの show ページにコメントフォームを加えたいので、 ``Blog`` コントローラの ``show`` アクションにフォームを作成して、直接 ``show`` テンプレートにフォームをレンダリングすることもできます。しかし、コメントの表示部分のコードを分離させた方がより良いでしょう。コメントの表示とコメントフォームの表示の違いは、コメントフォームの方は処理が必要であり、コントローラが必要になることです。上でコメントのテンプレートをインクルードしましたが、その方法とは少し異なります。

ルーティング
~~~~~~~~~~~~

サブミットされたフォームの処理をするためのルーティングルールを作成する必要があります。 ``src/Blogger/BlogBundle/Resources/config/routing.yml`` に、次のように新しくルーティングルールを加えてください。

.. code-block:: yaml

    BloggerBlogBundle_comment_create:
        pattern:  /comment/{blog_id}
        defaults: { _controller: BloggerBlogBundle:Comment:create }
        requirements:
            _method:  POST
            blog_id: \d+
        
コントローラ
~~~~~~~~~~~~

次に、上記のルーティングルールで指定した ``Comment`` コントローラを新しく作成する必要があります。 ``src/Blogger/BlogBundle/Controller/CommentController.php`` にファイルを作成し、次の内容をペーストしてください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/CommentController.php
    
    namespace Blogger\BlogBundle\Controller;
    
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Blogger\BlogBundle\Entity\Comment;
    use Blogger\BlogBundle\Form\CommentType;
    
    /**
     * Comment controller.
     */
    class CommentController extends Controller
    {
        public function newAction($blog_id)
        {
            $blog = $this->getBlog($blog_id);
            
            $comment = new Comment();
            $comment->setBlog($blog);
            $form   = $this->createForm(new CommentType(), $comment);
    
            return $this->render('BloggerBlogBundle:Comment:form.html.twig', array(
                'comment' => $comment,
                'form'   => $form->createView()
            ));
        }
    
        public function createAction($blog_id)
        {
            $blog = $this->getBlog($blog_id);
            
            $comment  = new Comment();
            $comment->setBlog($blog);
            $request = $this->getRequest();
            $form    = $this->createForm(new CommentType(), $comment);
            $form->bindRequest($request);
    
            if ($form->isValid()) {
                // TODO: Persist the comment entity
    
                return $this->redirect($this->generateUrl('BloggerBlogBundle_blog_show', array(
                    'id' => $comment->getBlog()->getId())) .
                    '#comment-' . $comment->getId()
                );
            }
    
            return $this->render('BloggerBlogBundle:Comment:create.html.twig', array(
                'comment' => $comment,
                'form'    => $form->createView()
            ));
        }
        
        protected function getBlog($blog_id)
        {
            $em = $this->getDoctrine()
                        ->getEntityManager();
    
            $blog = $em->getRepository('BloggerBlogBundle:Blog')->find($blog_id);
    
            if (!$blog) {
                throw $this->createNotFoundException('Unable to find Blog post.');
            }
            
            return $blog;
        }
       
    }
    
``Comment`` コントローラに２つのアクションを作成しました。 ``new`` と ``create`` です。 ``new`` アクションは、コメントフォームの表示を担当し、 ``create`` アクションは、コメントフォームからのサブミット内容の処理を担当します。今回追加したコードは多いように見えますが、何も新しいものはありません。全ては、パート２の問い合わせフォームを作成したときにカバーしています。しかし、先に進む前に ``Comment`` コントローラで何が起こるのか整理してみましょう。

フォームバリデーション
~~~~~~~~~~~~~~~~~~~~~~

``user`` や ``comment`` の値が空のままブログコメントをサブミットできるようにはしたくありませんね。パート２の問い合わせフォームを作成したときに説明したバリデータを振り返ってみましょう。  ``src/Blogger/BlogBundle/Entity/Comment.php`` にある ``Comment`` エンティティを修正して、次の内容を追加してください。

.. code-block:: php
    
    <?php
    // src/Blogger/BlogBundle/Entity/Comment.php
    
    // ..
    
    use Symfony\Component\Validator\Mapping\ClassMetadata;
    use Symfony\Component\Validator\Constraints\NotBlank;
    
    // ..
    class Comment
    {
        // ..
        
        public static function loadValidatorMetadata(ClassMetadata $metadata)
        {
            $metadata->addPropertyConstraint('user', new NotBlank(array(
                'message' => 'You must enter your name'
            )));
            $metadata->addPropertyConstraint('comment', new NotBlank(array(
                'message' => 'You must enter a comment'
            )));
        }
        
        // ..
    }

この制約は user と comment のメンバーが空にならないように保証しています。また、 ``message`` オプションを両方の制約に追加してデフォルトのメッセージをオーバーライドしています。 ``ClassMetadata`` と ``NotBlank`` を覚えていましたか？

ビュー
~~~~~~

次に、 ``new`` と ``create`` コントローラアクションに対応するテンプレートを作成する必要があります。まず、  ``src/Blogger/BlogBundle/Resources/views/Comment/form.html.twig`` にある ``new`` のファイルに次の内容をペーストしてください。

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/Comment/form.html.twig #}
    
    <form action="{{ path('BloggerBlogBundle_comment_create', { 'blog_id' : comment.blog.id } ) }}" method="post" {{ form_enctype(form) }} class="blogger">
        {{ form_widget(form) }}
        <p>
            <input type="submit" value="Submit">
        </p>
    </form>

このテンプレートの目的はシンプルで、コメントフォームをレンダリングするだけです。フォームの ``action`` が、新しいルーティングルールの ``BloggerBlogBundle_comment_create``.に ``POST`` していることに気づいたでしょうか？

次に ``create`` のビューを作成しましょう。 ``src/Blogger/BlogBundle/Resources/views/Comment/create.html.twig`` に新しくファイルを作成し、次の内容をペーストしてください。

.. code-block:: html

    {% extends 'BloggerBlogBundle::layout.html.twig' %}
    
    {% block title %}Add Comment{% endblock%}
    
    {% block body %}
        <h1>Add comment for blog post "{{ comment.blog.title }}"</h1>
        {% include 'BloggerBlogBundle:Comment:form.html.twig' with { 'form': form } %}    
    {% endblock %}

``Comment`` コントローラの ``create`` アクションは、フォームのサブミットされた値を処理しますが、フォームがエラーになったときのため、表示も必要になります。重複を避けて ``BloggerBlogBundle:Comment:form.html.twig`` を再利用しましょう。

ブログの show テンプレートを修正して、にフォームを追加しましょう。テンプレートファイル ``src/Blogger/BlogBundle/Resources/views/Blog/show.html.twig`` を次のように修正してください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Blog/show.html.twig #}
    
    {# .. #}
    
    {% block body %}
    
        {# .. #}
        
        <section class="comments" id="comments">
            {# .. #}
            
            <h3>Add Comment</h3>
            {% render 'BloggerBlogBundle:Comment:new' with { 'blog_id': blog.id } %}
        </section>
    {% endblock %}

ここで新しい Twig のタグ ``render`` を使用しました。このタグは、コントローラの内容をテンプレートにレンダリングします。今回のケースでは、 ``BloggerBlogBundle:Comment:new`` コントローラアクションの内容をレンダリングしています。

ブラウザで、 ``http://symblog.dev/app_dev.php/2`` を見てみると、次のように Symfony2 は例外を投げていることに気づくでしょう。

.. image:: /_static/images/part_4/to_string_error.jpg
    :align: center
    :alt: toString() Symfony2 Exception
    
この例外は ``BloggerBlogBundle:Blog:show.html.twig`` テンプレートによって投げられています。 ``BloggerBlogBundle:Blog:show.html.twig`` テンプレートの２５行目を見てみると、 ``BloggerBlogBundle:Comment:create`` コントローラを埋め込む処理で問題ががあることがわかります。

.. code-block:: html

    {% render 'BloggerBlogBundle:Comment:create' with { 'blog_id': blog.id } %}
    
例外メッセージをよく見てみると、なぜ例外が投げられたのかといったことがわかります。

    Entities passed to the choice field must have a "__toString()" method defined

この例外メッセージは、レンダリングしようとした ``choice`` フィールドのエンティティに ``__toString()`` メソッドがないため起きたことだと説明してくれています。 ``choice`` フィールドは、ユーザに選択肢が与えられるフォーム要素で、 ``select`` 要素になります。またコメントフォームのどこに ``choice`` フィールドがあるのか疑問に思うかもしれません。コメントフォームのテンプレートをもう一度見てみると、 ``{{ form_widget(form) }}`` という Twit 関数を使用してフォームをレンダリングしていることに気づいたと思います。この関数は、ベーシックなフォームでフォーム全体を出力します。フォームが作成された際のフォームクラス ``CommentType`` を見てみましょう。 ``FormBuilder`` オブジェクトを介してフィールドをフォームに追加しているのが確認できます。そこで ``blog`` フィールドが追加されています。これが ``choice`` フィールドになっているのです。

パート２の内容を覚えていれば、 ``FormBuilder`` がフィールドに関するメタデータを基に出力するフィールドタイプを推測する方法について説明したと思います。 ``Comment`` と ``Blog`` エンティティ間のリレーションをセットアップしましたので、 ``FormBuilder`` は ``blog`` が ``choice`` フィールドとなり、ユーザがコメントに結びつけるブログエントリを指定することができるように推測しました。フォームに ``choice`` フィールドがあり、 Symfony2 例外が投げられたのは、そのためです。次のように ``Blog`` エントリに ``__toString()`` メソッドを実装すれば、この問題を修正することができます。

.. code-block:: php
    
    // src/Blogger/BlogBundle/Entity/Blog.php
    public function __toString()
    {
        return $this->getTitle();
    }

.. tip::

    問題が起きた時に表示される Symfony2 のエラーメッセージの情報量はとても多くて有益です。これらの情報はデバッグを手助けしてくれるので、エラーメッセージを読んでください。また、エラーメッセージは完全なスタックトレースも提供してくれるため、エラーの原因をステップを確認することができます。
    
これでページを再読み込みすると、コメントが表示されるはずです。しかし、 ``approved``, ``created``, ``updated``, ``blog`` といったここで表示されるべきではないフィールドも出てきてしまっているのに気づくと思います。生成した ``CommentType`` クラスをまだカスタマイズしていないからです。

.. tip::

    レンダリングされたフィールドは全て正しいフィールドのタイプで出力されているように見えます。 ``user`` フィールドは ``text`` フィールドとして、 ``comment`` フィールドは ``textarea`` として、 ``created`` と ``updated`` の ``DateTime`` フィールドは日にちと時間を指定することのできる ``select`` フィールドになります。
    
    これは ``FormBuilders`` がレンダリングに必要なメンバーのフィールドのタイプを推測してくれるためです。エンティティに指定したメタデータに基づいています。 ``Comment`` エンティティのメタデータを細かく特定したので、 ``FormBuilder`` はフィールドタイプが正しく推測できました。
    
``src/Blogger/BlogBundle/Form/CommentType.php`` のファイルを修正して必要なフィールドのみ出力するようにしましょう。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Form/CommentType.php
    
    // ..
    class CommentType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder
                ->add('user')
                ->add('comment')
            ;
        }
    
        // ..
    }

ページを再読み込みすると、 user と comment フィールドのみが出力されます。この時点でフォームをサブミットしても、実際にはコメントはデータベースに保存されません。なぜならフォームコントローラは、バリデーションに通っても ``Comment`` エンティティに対して何もしていないからです。では、どうやって ``Comment`` エンティティをデータベースに永続化しましょうか。 ``DataFixtures`` を作成した際にその方法を見たはずです。 ``Comment`` コントローラの ``create`` アクションを修正して ``Comment`` エンティティをデータベースに永続化するようにしましょう。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/CommentController.php
    
    // ..
    class CommentController extends Controller
    {
        public function createAction($blog_id)
        {
            // ..
            
            if ($form->isValid()) {
                $em = $this->getDoctrine()
                           ->getEntityManager();
                $em->persist($comment);
                $em->flush();
                    
                return $this->redirect($this->generateUrl('BloggerBlogBundle_blog_show', array(
                    'id' => $comment->getBlog()->getId())) .
                    '#comment-' . $comment->getId()
                );
            }
        
            // ..
        }
    }

``Comment`` エンティティの永続化はとてもシンプルで、 ``persist()`` と ``flush()`` を呼ぶだけです。フォームは、 PHP オブジェクトを扱うのみで、オブジェクトの管理と永続化は、 Doctrine 2 が行うことを覚えておいてください。フォームの送信と送信されたデータをデータベースに永続化は、直接の関係はないのです。

これでブログにコメントを追加できるようになりました。

.. image:: /_static/images/part_4/add_comments.jpg
    :align: center
    :alt: symblog add blog comments
    
結論
----

この章でかなり前進しましたね。ブログサイトとして機能しはじめてきています。ホームページのベースとコメントエンティティを作成しました。ユーザはブログに対してコメントを投稿することができるようになり、また、他のユーザが投稿したコメントを参照することもできるようになりました。また、複数のフィクスチャファイル間のリファレンス方法を学びました。そして、 Doctrine 2 マイグレーションを使用し、エンティティの変更ごとにデータベースをマイグレートしました。

次章では、タグクラウドと最近のコメントリストを含めたサイドバーを作成します。また、 Twig のカスタムフィルターを作成して Twig を拡張します。最後にアセットライブラリ Assetic を使用してこのブログサイトのアセットを管理してみます。
    
