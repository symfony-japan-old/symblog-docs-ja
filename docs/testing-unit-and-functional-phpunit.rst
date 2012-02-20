[パート６] - テスト: PHPUnit を使用したユニットテストと機能テスト
=================================================================

要約
--------

この章に来るまでに Symfony2 での開発に関するコアのコンセプトの多くの分野を見てきました。さらに機能を追加する前に、ここでテストを説明しておきましょう。ユニットテストで個々の機能をテストする方法、機能テストで複数のコンポーネントの動作が正しいかを確認する方法の両方を見ていきます。 Symfony2 のテストの中心のライブラリである PHP のテストライブラリ `PHPUnit <http://www.phpunit.de/manual/current/en/>`_ を説明します。テストは重要なトピックなので、後の章でも扱う予定です。この章の最後では、ユニットテスト、機能テストの両方による多くのテストを書くことになります。ブラウザをシミュレートしたり、フォームにデータを入力したり、ウェブページが正しく出力されているかレスポンスをチェックしたります。また、アプリケーションのコードベースでテストがどのくらいのカバレッジなのかチェックもします。

Symfony2 におけるテスト
-----------------------

`PHPUnit <http://www.phpunit.de/manual/current/en/>`_ は、 PHP でのテストの "デファクトスタンダード" になっていますので、 PHPUnit について学習することは、全ての PHP プロジェクトにおいて有益です。また、この章で扱う内容のほとんど、つまりテストに関しては、 PHP 独特のものではなく他の言語にも適応可能です。

.. tip::

    Symfony2 のバンドルをオープンソースで開発することを考えていましたら、バンドルのテストがちゃんとされていること(ドキュメントがあることも)が、注目する点になります。 `Symfony2Bundles <http://symfony2bundles.org/>`_ で利用可能な Symfony2 バンドルを見てみてください。

ユニットテスト
~~~~~~~~~~~~~~

ユニットテストは、コードの個々の部分(Unit)を独立して使用した際に、正しく機能することを保証するためのものです。 Symfony2 のようなオブジェクト指向コードベースでは、個々の部分はクラスやメソッドとなります。例えば、 ``Blog`` と ``Comment`` エンティティクラスのテストを書くことができます。ユニットテストを書く際には、テストケースは他のテストケースと分離して書くべきです。例えばケース B の結果は、ケース A の結果に依存してはいけません。外部依存がある関数のユニットテストを簡単にするためにモックオブジェクトを作成することができるのは、とても便利です。モッキングを使用すれば、実際に実行することなく関数をシミュレートすることができます。例として、外部 API をラップするクラスのユニットテストが挙げられます。 API クラスは、外部 API とコミュニケートするためのトランスポートレイヤーを使用するとします。実際に外部 API を叩くのではなく、指定した結果を返すようにトランスポートレイヤーのコミュニケートをモッキングすることができます。ユニットテストは、アプリケーションのコンポーネントを全体として正しく機能するかをテストするものではありません。こういったテストに関しては、次のトピックである機能テストの領域です。

機能テスト
~~~~~~~~~~

機能テストは、ルーティング、コントローラ、ビューといったアプリケーション内の異なるコンポーネントの統合をチェックします。機能テストは、ユーザ自身がブラウザでホームページを開き、リンクをクリックして正しいブログが表示されるかチェックするような手動のテストと似ています。しかし、機能テストは、このプロセスを自動化してくれます。 Symfony2 には、機能テストをアシストする便利なクラスがたくさん付いてきます。例えば、ページにリクエストを投げることのできる ``Client`` や ``Response`` の内容を横断する DOM ``Crawler`` などがあります。

.. tip::

    テスト駆動のソフトウェア開発プロセスも、いくつかあります。例えば、テスト駆動開発(TDD)やビヘイビア駆動開発(BDD)などです。これらの内容はこのチュートリアルの領域外ですが、 `everzet <https://twitter.com/#!/everzet>`_ によって書かれた BDD ライブラリ `Behat <http://behat.org/>`_ を知っておいた方がいいでしょう。また、 Symfony2 には `BehatBundle <http://docs.behat.org/bundle/index.html>`_ があり、 Symfony2 のプロジェクトに Behat を簡単に埋め込むことが可能です。

PHPUnit
-------

上記で説明したように、 Symfony2 のテストは、 PHPUnit を使用して書きます。この章のテストを実行させてテストをするために PHPUnit をインストールしてください。インストールの詳細は、 PHPUnit の公式ドキュメントの `installation instructions <http://www.phpunit.de/manual/current/en/installation.html>`_ を参照してください。 Symfony2 でテストを実行するには、 PHPUnit 3.5.11 以上を使用してください。 PHPUnit はとても大きなライブラリで、公式ドキュメントへのリファレンスにはさらに詳細の情報を見つけることができます。

アサーション
~~~~~~~~~~~~

テストを書くことは、実際のテストの結果と想定されるテストの結果をチェックすることになります。 PHPUnit には多くのアサーションのメソッドがあり、この作業をアシストしてくれます。以下に一般的なアサーションを挙げました。

.. code-block:: php

    // Check 1 === 1 is true
    $this->assertTrue(1 === 1);

    // Check 1 === 2 is false
    $this->assertFalse(1 === 2);

    // Check 'Hello' equals 'Hello'
    $this->assertEquals('Hello', 'Hello');

    // Check array has key 'language'
    $this->assertArrayHasKey('language', array('language' => 'php', 'size' => '1024'));

    // Check array contains value 'php'
    $this->assertContains('php', array('php', 'ruby', 'c++', 'JavaScript'));

PHPUnit のドキュメントで `アサーション <http://www.phpunit.de/manual/current/en/writing-tests-for-phpunit.html#writing-tests-for-phpunit.assertions>`_ のリストを参照できます。

Symfony2 のテストの実行
-----------------------

実際にテストを書き始める前に、 Symfony2 がどうやってテストを実行しているかを見てみましょう。 PHPUnit は、コンフィギュレーションファイルで実行をセットすることができます。 Symfony2 のプロジェクトでは、このファイルは、 ``app/phpunit.xml.dist`` になります。このファイルは、 ``dist`` の接尾辞が付いているので、このファイルを ``app/phpunit.xml`` にコピーする必要があります。

.. tip::

    Git のようなバージョン管理システムを使用しているのであれば、 ``app/phpunit.xml`` を無視リスト(ignore)に加えてください。

PHPUnit のコンフィギュレーションファイルの中を見てみると、次のようになっています。

.. code-block:: xml

    <!-- app/phpunit.xml -->
    
    <testsuites>
        <testsuite name="Project Test Suite">
            <directory>../src/*/*Bundle/Tests</directory>
            <directory>../src/*/Bundle/*Bundle/Tests</directory>
        </testsuite>
    </testsuites>

上ように directory タグで囲んでいる部分でテストスイートを指定します。このディレクトリを参照して PHPUnit はテストを実行します。また、 PHPUnit の実行の際のコマンドラインに追加の引数を渡せば、テストスイートを実行するのではなく、特定のディレクトリのテストも実行できます。このやり方に関しては、本章の後で説明します。

このコンフィギュレーションで ``app/bootstrap.php.cache`` のブートストラップファイルを指定しているのに気づきましたか？このファイルは、 PHPUnit によるテスト環境のセットアップを取得するのに使われます。

.. code-block:: xml

    <!-- app/phpunit.xml -->
    
    <phpunit
        bootstrap                   = "bootstrap.php.cache" >

.. tip::

    XML ファイルでの PHPUnit の設定に関する詳細は、 `PHPUnit ドキュメント <http://www.phpunit.de/manual/current/en/organizing-tests.html#organizing-tests.xml-configuration>`_ を参照してください。

テストの実行
------------

パート１で Symofny2 の生成タスクを使用して ``BloggerBlogBundle`` を作成した際に、 ``DefaultController`` クラスのテストも同時に作成されます。プロジェクトのルートディレクトリから次のタスクを実行してこのテストを実行することができます。 ``-c`` オプションを指定して、 PHPUnit のコンフィギュレーションファイルを ``app`` ディレクトリからロードしています。

.. code-block:: bash

    $ phpunit -c app

テストが終了すると、テストが失敗したという通知を受け取るはずです。 ``src/Blogger/BlogBundle/Tests/Controller/DefaultControllerTest.php`` の ``DefaultControllerTest`` を見てみると次のようになっています。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Controller/DefaultControllerTest.php

    namespace Blogger\BlogBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class DefaultControllerTest extends WebTestCase
    {
        public function testIndex()
        {
            $client = static::createClient();

            $crawler = $client->request('GET', '/hello/Fabien');

            $this->assertTrue($crawler->filter('html:contains("Hello Fabien")')->count() > 0);
        }
    }

これは Symfony2 が生成した ``DefaultController`` クラスの機能テストです。パート１を覚えていれば、このコントローラは、 ``/hello/{name}`` のリクエストを処理するアクションを持っていました。しかし、このコントローラは削除してしまいましたので、上記のテストは失敗してしまいました。ブラウザで ``http://symblog.dev/app_dev.php/hello/Fabien`` にアクセスしてみてください。このルーティングがないというメッセージが表示されるはずです。上記のテストはこの URL へのリクエストとなりますので、同じレスポンスを受け取ることになります。結果、テストが失敗します。機能テストは、この章の大事な位置づけであり、後で詳細をカバーします。

``DefaultController`` クラスは削除されているので、 ``src/Blogger/BlogBundle/Tests/Controller/DefaultControllerTest.php`` の ``DefaultControllerTest`` クラスも削除しましょう。

ユニットテスト
--------------

既に説明をしましたように、ユニットテストは、アプリケーションの個々の単位を独立してテストすることです。ユニットテストを書く際には、バンドルの構造をテストフォルダの下に複製することをお勧めします。例えば、 ``src/Blogger/BlogBundle/Entity/Blog.php`` にある ``Blog`` エンティティクラスをテストするのであれば、テストファイルは、 ``src/Blogger/BlogBundle/Tests/Entity/BlogTest.php`` に配置します。例として、フォルダのレイアウトは次のようになります。

.. code-block:: text

    src/Blogger/BlogBundle/
                    Entity/
                        Blog.php
                        Comment.php
                    Controller/
                        PageController.php
                    Twig/
                        Extensions/
                            BloggerBlogExtension.php
                    Tests/
                        Entity/
                            BlogTest.php
                            CommentTest.php
                        Controller/
                            PageControllerTest.php
                        Twig/
                            Extensions/
                                BloggerBlogExtensionTest.php

全てのテストファイルには、 Test という接尾辞が付いているのを確認してください。

ブログエンティティのテスト - slugify メソッド
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``Blog`` エンティティの slugify メソッドをテストするところから始めましょう。このメソッドが正しく動くかをチェックするテストを書いてみましょう。新しく ``src/Blogger/BlogBundle/Tests/Entity/BlogTest.php`` ファイルを作成し次の内容を追加してください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    namespace Blogger\BlogBundle\Tests\Entity;

    use Blogger\BlogBundle\Entity\Blog;

    class BlogTest extends \PHPUnit_Framework_TestCase
    {

    }

これで ``Blog`` エンティティのテストクラスを作成できました。このファイルが上で言及したフォルダ構造になっているかチェックしてください。 ``BlogTest`` クラスは、 PHPUnit のベースクラス  ``PHPUnit_Framework_TestCase`` を拡張しています。 PHPUnit を使用して書かれたテストは全てこのクラスの子クラスになります。 ``PHPUnit_Framework_TestCase`` クラスはパブリックなネームスペースで定義されているため、前章で説明したように ``PHPUnit_Framework_TestCase`` クラス名の直前にある ``\`` が必要です。

``Blog`` エンティティのテストのスケルトンクラスができましたので、テストを書いてみましょう。 PHPUnit のテストケースは、 ``test`` 接頭辞を持つメソッドになります。例えば、 ``testSlugify()`` のようになります。 ``src/Blogger/BlogBundle/Tests/Entity/BlogTest.php`` にある ``BlogTest`` を次のように修正してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    class BlogTest extends \PHPUnit_Framework_TestCase
    {
        public function testSlugify()
        {
            $blog = new Blog();

            $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        }
    }

上記はとても簡単なテストケースです。新しく ``Blog`` エンティティをインスタンス化し、 ``slugify`` メソッドの結果を ``assertEquals()`` で調べています。 ``assertEquals()`` メソッドは、想定している結果と実際の結果という２つの引数が必要です。３つ目の引数はオプションで、テストが失敗した際に表示されるメッセージを指定できます。

新しいユニットテストを実行してみましょう。次のコマンドラインを実行してください。

.. code-block:: bash

    $ phpunit -c app

次のような出力が確認できます。

.. code-block :: bash

    PHPUnit 3.5.11 by Sebastian Bergmann.

    .

    Time: 1 second, Memory: 4.25Mb

    OK (1 test, 1 assertion)

PHPUnit の出力は、とてもシンプルです。 PHPUnit に関する情報が表示され、次に ``.`` がテスト実行したテストの数だけ表示されます。今回のケースでは、テストは１つだけでしたので、 ``.`` が１つだけ表示されました。最後に、このテストの結果を表示します。 ``BlogTest`` では、１つのアサーションを持つテストを１つだけ実行しました。コマンドラインの出力にカラー出力を有効にしていれば、最後の行はグリーンで表示され、全てのテストの結果が正しかった(OK)ことが視覚的にわかります。では、一旦 ``testSlugify()`` メソッドを修正してテストが失敗するようにしてみましょう。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSlugify()
    {
        $blog = new Blog();

        $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        $this->assertEquals('a day with symfony2', $blog->slugify('A Day With Symfony2'));
    }

前と同じようにテストを実行してください。次のような出力が得られるはずです。

.. code-block :: bash

    PHPUnit 3.5.11 by Sebastian Bergmann.

    F

    Time: 0 seconds, Memory: 4.25Mb

    There was 1 failure:

    1) Blogger\BlogBundle\Tests\Entity\BlogTest::testSlugify
    Failed asserting that two strings are equal.
    --- Expected
    +++ Actual
    @@ @@
    -a day with symfony2
    +a-day-with-symfony2

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Entity/BlogTest.php:15

    FAILURES!
    Tests: 1, Assertions: 2, Failures: 1.

今回の結果は、もう少し詳細です。 ``.`` となっていたのが ``F`` になったのが確認できます。これは、テストが失敗したことを意味しています。テストにエラーがあった際には、ここに ``E`` という文字が表示されます。次に PHPUnit は、失敗の詳細を表示します。今回のケースでは、１つ失敗がありました。想定した結果と実際の結果が異なったため、 ``Blogger\BlogBundle\Tests\Entity\BlogTest::testSlugify`` メソッドが失敗したのが確認できます。コマンドラインの出力にカラー出力を有効にしていれば、最後の行は赤で表示され、テストが失敗したことが視覚的にわかります。 ``testSlugify()`` メソッドを修正して、成功するように戻しましょう。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSlugify()
    {
        $blog = new Blog();

        $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        $this->assertEquals('a-day-with-symfony2', $blog->slugify('A Day With Symfony2'));
    }

次に進む前に ``slugify()`` メソッドのテストをいくつか追加してみましょう。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSlugify()
    {
        $blog = new Blog();

        $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        $this->assertEquals('a-day-with-symfony2', $blog->slugify('A Day With Symfony2'));
        $this->assertEquals('hello-world', $blog->slugify('Hello    world'));
        $this->assertEquals('symblog', $blog->slugify('symblog '));
        $this->assertEquals('symblog', $blog->slugify(' symblog'));
    }

``Blog`` エンティティの ``slugify`` メソッドをテストしましたので、次は、 ``Blog`` の ``$title`` メンバーが修正されると、 ``Blog`` の ``$slug`` メンバーも正しくセットされるか調べてみましょう。  ``src/Blogger/BlogBundle/Tests/Entity/BlogTest.php`` の ``BlogTest`` を次のメソッドを追加してみましょう。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSetSlug()
    {
        $blog = new Blog();

        $blog->setSlug('Symfony2 Blog');
        $this->assertEquals('symfony2-blog', $blog->getSlug());
    }

    public function testSetTitle()
    {
        $blog = new Blog();

        $blog->setTitle('Hello World');
        $this->assertEquals('hello-world', $blog->getSlug());
    }

まず、 ``$slug`` メンバーが正しく修正されたかチェックするため、 ``setSlug`` メソッドからテストしています。次に、 ``Blog`` エンティティで ``setTitle`` メソッドが呼ばれた際に ``$slug`` メンバーが正しいかチェックしています。

``Blog`` エンティティが正しく動作するかテストを実行してみましょう。

Twig エクステンションのテスト
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

前章では、 ``\DateTime`` インスタンスを、経過時間を表示する文字列に変換する Twig エクステンションを作成しました。 ``src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php`` に新しくファイルを作成し、次の内容を加えてください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php

    namespace Blogger\BlogBundle\Tests\Twig\Extensions;

    use Blogger\BlogBundle\Twig\Extensions\BloggerBlogExtension;

    class BloggerBlogExtensionTest extends \PHPUnit_Framework_TestCase
    {
        public function testCreatedAgo()
        {
            $blog = new BloggerBlogExtension();

            $this->assertEquals("0 seconds ago", $blog->createdAgo(new \DateTime()));
            $this->assertEquals("34 seconds ago", $blog->createdAgo($this->getDateTime(-34)));
            $this->assertEquals("1 minute ago", $blog->createdAgo($this->getDateTime(-60)));
            $this->assertEquals("2 minutes ago", $blog->createdAgo($this->getDateTime(-120)));
            $this->assertEquals("1 hour ago", $blog->createdAgo($this->getDateTime(-3600)));
            $this->assertEquals("1 hour ago", $blog->createdAgo($this->getDateTime(-3601)));
            $this->assertEquals("2 hours ago", $blog->createdAgo($this->getDateTime(-7200)));

            // Cannot create time in the future
            $this->setExpectedException('\InvalidArgumentException');
            $blog->createdAgo($this->getDateTime(60));
        }

        protected function getDateTime($delta)
        {
            return new \DateTime(date("Y-m-d H:i:s", time()+$delta));
        }
    }

このクラスの作成方法は前回とほぼ同じで、Twig エクステンションのテストをする ``testCreatedAgo()`` メソッドを作成しています。このテストケースで PHPUnit の ``setExpectedException()`` メソッドを説明しましょう。このメソッドは、例外が投げられるよりも前に呼び出してください。 Twig エクステンションの ``createdAgo`` メソッドは将来の日付を扱えないので、 ``\Exception`` が投げられます。 ``getDateTime()`` メソッドは、単に ``\DateTime`` インスタンスを作成するためのヘルパーメソッドです。このヘルパーメソッドには、 ``test`` 接頭辞がないのに気づきましたか？これで PHPUnit はこのメソッドをテストケースとして実行しようすることはありません。コマンドラインからこのテストを実行させてみてください。以前と同じようにテストを実行することもできますが、 PHPUnit に特定のフォルダやファイルを指定することもできます。次のコマンドを実行してください。

.. code-block:: bash

    $ phpunit -c app src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php

今回は、 ``BloggerBlogExtensionTest`` ファイルのみのテストを実行します。 PHPUnit は、テストが失敗したということを教えてくれます。出力は次のようになったはずです。

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Twig\Extension\BloggerBlogExtensionTest::testCreatedAgo
    Failed asserting that two strings are equal.
    --- Expected
    +++ Actual
    @@ @@
    -0 seconds ago
    +0 second ago

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php:14

最初のアサーションで ``0 seconds ago`` が返ってくると想定していましたが、 second という文字が複数系ではありませんでした。 ``src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php`` にある Twig エクステンションを次のように修正してください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php

    namespace Blogger\BlogBundle\Twig\Extensions;

    class BloggerBlogExtension extends \Twig_Extension
    {
        // ..

        public function createdAgo(\DateTime $dateTime)
        {
            // ..
            if ($delta < 60)
            {
                // Seconds
                $time = $delta;
                $duration = $time . " second" . (($time === 0 || $time > 1) ? "s" : "") . " ago";
            }
            // ..
        }

        // ..
    }

PHPUnit のテストを再実行してください。今度は、最初のアサーションがパスしたのを確認できますね。しかし、またテストケースが失敗してしまいます。次の結果を調べてみましょう。

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Twig\Extension\BloggerBlogExtensionTest::testCreatedAgo
    Failed asserting that two strings are equal.
    --- Expected
    +++ Actual
    @@ @@
    -1 hour ago
    +60 minutes ago

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php:18

アサーションが失敗したファイルの行数が１８行目となっており、２つ目のアサーションが失敗したということが確認できます。テストケースを見てみると、 Twig エクステンションが正しく機能していないことがわかります。 ``1 hour ago`` が返ってくるべきでしたが、 ``60 minutes ago`` が返ってきてしまいました。 Twig エクステンションの ``BloggerBlogExtension`` を調べてみると理由がわかります。分の比較に ``<=`` ではなく、 ``<`` を使用してしまっていました。また、時間のチェックに関しても同じように間違っていました。 ``src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php`` の Twig エクステンションを次のように修正してください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php

    namespace Blogger\BlogBundle\Twig\Extensions;

    class BloggerBlogExtension extends \Twig_Extension
    {
        // ..

        public function createdAgo(\DateTime $dateTime)
        {
            // ..

            else if ($delta < 3600)
            {
                // Mins
                $time = floor($delta / 60);
                $duration = $time . " minute" . (($time > 1) ? "s" : "") . " ago";
            }
            else if ($delta < 86400)
            {
                // Hours
                $time = floor($delta / 3600);
                $duration = $time . " hour" . (($time > 1) ? "s" : "") . " ago";
            }

            // ..
        }

        // ..
    }

次のコマンドを使用して全てのテストを再実行してください。

.. code-block:: bash

    $ phpunit -c app

上のコマンドで全てのテストを実行し、結果、全部パスすることができました。今回はユニットテストを少し書いただけですが、コーディングにおいてテストが強力で重要であるか感じることができたと思います。出くわしたエラーはあまり重要なものではありませんでしたが、エラーはエラーでした。また、新しく機能追加した際に既存の機能を壊していないかをチェックするのにもテストは有効です。今回扱うユニットテストはここまでです。次に続く章でも、さらにユニットテストを見ていきます。まだテストがない機能を自分でユニットテストを追加してみてください。

機能テスト
----------

ユニットテストを書きましたので、次は複数のコンポーネントを結合したテストを見ていきましょう。機能テストのセクションの最初では、ブラウザリクエストをシミュレートしてレスポンスをテストしてみましょう。

アバウトページのテスト
~~~~~~~~~~~~~~~~~~~~~~

``PageController`` クラスのアバウトページのテストから始めましょう。アバウトページはシンプルですので、始めるにはいい場所です。 ``src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php`` に新しくファイルを作成し、次の内容を追加してください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    namespace Blogger\BlogBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class PageControllerTest extends WebTestCase
    {
        public function testAbout()
        {
            $client = static::createClient();

            $crawler = $client->request('GET', '/about');

            $this->assertEquals(1, $crawler->filter('h1:contains("About symblog")')->count());
        }
    }

上の内容とよく似たコントローラのテストを既に見たことがあるはずです。それは、先ほど見た ``DefaultControllerTest`` クラスです。今回の内容は、 symblog のアバウトページのテストで、 生成された HTML 内の ``H1`` タグで囲まれた文字列に ``About symlog`` があるかチェックします。 ``PageControllerTest`` は、ユニットテストのときと違い ``\PHPUnit_Framework_TestCase`` を拡張しません。その代わりに、 ``WebTestCase`` クラスを拡張します。この ``WebTestCase`` クラスは Symfony2 の FrameworkBundle の一部です。


先ほど説明したように PHPUnit のテストケースは、 ``\PHPUnit_Framework_TestCase`` を拡張する必要がありますが、複数のテストケースに渡って共通の機能が必要なときには、独自クラスでカプセル化し、その毒クラスに ``\PHPUnit_Framework_TestCase`` を拡張させると便利です。 ``WebTestCase`` は、まさしくそういた用途で使用しています。このクラスは、 Symofny2 の機能テストを実行する上で便利なメソッドをたくさん実装しています。 ``vendor/symfony/src/Symfony/Bundle/FrameworkBundle/Test/WebTestCase.php`` にある ``WebTestCase`` ファイルを見てみてください。以下のように、このクラスが実際に ``\PHPUnit_Framework_TestCase`` クラスを拡張しているのが確認できるでしょう。

.. code-block:: php

    // vendor/symfony/src/Symfony/Bundle/FrameworkBundle/Test/WebTestCase.php

    abstract class WebTestCase extends \PHPUnit_Framework_TestCase
    {
        // ..
    }

``WebTestCase`` クラスの ``createClient()`` メソッドを見てみると、このメソッドが Symfony2 のカーネルを初期化しているのが確認できます。また、他のメソッドを見てみると、 ``環境`` に ``test`` を指定しているのにも気づくでしょう(``createClient()`` の引数でオーバーライドをしない限り)。これが前章で説明した ``テスト`` 環境です。

今回作成したクラスに戻ってみると、テストをセットアップして実行させる際に ``createClient()`` メソッドがあるのが確認できます。このインスタンスの ``request()`` メソッドを呼び HTTP GET リクエストで ``/about`` URL にリクエストを送るシミュレートをしています(つまり、ブラウザで ``http://symblog.dev/about`` にアクセスすること)。 ``request()`` メソッドを呼び、 ``Response`` を含んだ ``Crawler`` オブジェクトを返り値としてうけとります。 ``Crawler`` インスタンスは、返ってきた HTML を横断的に調べることができるのでとても便利です。そして、 ``Crawler`` インスタンスを使用して返り値の HTML に ``H1`` タグで囲まれた文字 ``About symblog`` があるかチェックします。このクラスでは、 ``WebTestCase`` クラスを拡張していますが、同じようにアサーションのメソッドが使用できることがわかります(結局 ``PageControllerTest`` クラスは、 ``\PHPUnit_Framework_TestCase`` クラスの小クラスにもなっています)。

次のコマンドで ``PageControllerTest`` を実行しましょう。テストを書いているときは、対象としているファイルのテストのみを実行すると良いでしょう。特にテストケースが膨大になってしまったときに、全てのテストを実行するのには時間がかかってしまいますので。

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

``OK (1 test, 1 assertion)`` が出力され、 １つテスト(``testAbout()``)が実行され、１つ(``assertEquals()``)アサーションが調べられました。

``About symblog`` の文字を ``Contact`` に変更して、テストを実行してみてください。 ``Contact`` という文字列はないので ``assertEquals`` は false になります。

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Controller\PageControllerTest::testAbout
    Failed asserting that <boolean:false> is true.

次に行く前に ``About symblog`` に文字列を戻しておいてください。

``Crawler`` インスタンスを使用すれば、 HTML や XML を横断的に調べることができます(つまり、 ``Crawler`` は HTML か XML のレスポンスのみに有効です)。 ``Crawler`` は、 ``filter()``, ``first()``, ``last()``, ``parent()`` のようなメソッドを使用して、レスポンスを調べることができます。 `jQuery <http://jquery.com/>` を使用したことがあればれ、 ``Crawler`` クラスにすぐ慣れるでしょう。 ``Crawler`` の横断検索のメソッドは Symfony2 のガイドブックの `Testing <http://symfony.com/doc/current/book/testing.html#traversing>`_ で参照することができます。これから、さらに ``Crawler`` の機能を見ていきます。

ホームページ
~~~~~~~~~~~~

アバウトページのテストはシンプルでしたが、ウェブページの機能テストの基本原理の要点を学ぶことができたと思います。

 1. クライアントの作成
 2. ページへリクエスト
 3. レスポンスのチェック

これがプロセスのシンプルな概観ですが、実際には、リンクをクリックしたり、フォームに値を入れ送信したりするなど多くステップがあります。

ホームページのテストをするためのメソッドを作成しましょう。ホームページは、 URL ``/`` でアクセスすることができ、最新のブログエントリの一覧が表示されるようになっています。 ``src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php`` にある ``PageControllerTest`` に ``testIndex()`` メソッドを新しく追加してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testIndex()
    {
        $client = static::createClient();

        $crawler = $client->request('GET', '/');

        // Check there are some blog entries on the page
        $this->assertTrue($crawler->filter('article.blog')->count() > 0);
    }

アバウトページのテストと同じステップを踏んでいるのが確認できましたか。以下のようにテストを実行して、想定通りに動作するか確認してください。

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

このテストをさらに充実させていきましょう。機能テストの役割は、ウェブサイトで実際にユーザ行うことの模倣ができるようにすることです。実際のユーザがウェブサイトのページ間を移動する際は、リンクをクリックします。このアクションをシミュレートして、ブログのタイトルがクリックされたとき、ちゃんとブログの show ページへ移動するか、リンクをテストしてみましょう。 ``PageControllerTest`` クラスの ``tetIndex()`` メソッドを以下のように修正してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testIndex()
    {
        // ..

        // Find the first link, get the title, ensure this is loaded on the next page
        $blogLink   = $crawler->filter('article.blog h2 a')->first();
        $blogTitle  = $blogLink->text();
        $crawler    = $client->click($blogLink->link());

        // Check the h2 has the blog title in it
        $this->assertEquals(1, $crawler->filter('h2:contains("' . $blogTitle .'")')->count());
    }

まず、最初のブログのタイトルリンクのテキストを ``Crawler`` を使用して抽出します。これは filter で ``article.blog. h2 a`` とすればできます。この filter は ``article.blog`` の article タグ内にある ``H2`` タグのさらに内にある ``a`` タグを返します。以下のようにブログ一覧を表示するホームページの HTML を見ながら説明しましょう。

.. code-block:: html

    <article class="blog">
        <div class="date"><time datetime="2011-09-05T21:06:19+01:00">Monday, September 5, 2011</time></div>
        <header>
            <h2><a href="/app_dev.php/1/a-day-with-symfony2">A day with Symfony2</a></h2>
        </header>

        <!-- .. -->
    </article>
    <article class="blog">
        <div class="date"><time datetime="2011-09-05T21:06:19+01:00">Monday, September 5, 2011</time></div>
        <header>
            <h2><a href="/app_dev.php/2/the-pool-on-the-roof-must-have-a-leak">The pool on the roof must have a leak</a></h2>
        </header>

        <!-- .. -->
    </article>

ホームページの HTML で filter の ``article.blog h2 a`` 構造が確認できるでしょう。この HTML には、 ``<article class="blog">`` タグが複数あることに気づきましたか？そのため、 ``Crawler`` はコレクションを返すことになります。最初のリンクが欲しいだけなので、このコレクションに ``first()`` メソッドを使用します。そして、リンク内のテキストを抽出するために ``text()`` メソッドを使用します。今回のケースでは、 ``A day with Symfony2`` となります。次にブログタイトルのリンクをクリックしてブログの show ページにナビゲートします。クライアントの ``click()`` メソッドは、リンクオブジェクトを引数に取り、 ``Crawler`` のインスタンスに ``Response`` を返します。ここまで見てきたように機能テストにおいて ``Crawler`` オブジェクトは、重要なキーとなります。

これで ``Crawler`` オブジェクトがブログの show ページのレスポンスを含むようになったので、その中のリンクが正しいページにナビゲートするかテストする必要があります。先ほどレスポンスのタイトルを調べた際に使用した ``$blogTitle`` の値を使用することができます。
We can use the ``$blogTitle`` value we retrieved earlier to check this against the title in the Response.

次のテストを実行して、ホームページとブログの show ページのナビゲートが正しく動作しているか確認してください。

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

機能テストでウェブサイトのページのナビゲート方法が理解できたと思います。次はフォームのテストを見ていきましょう。

問い合わせページのテスト
~~~~~~~~~~~~~~~~~~~~~~~~

symblog のユーザは、問い合わせページ ``http://symblog.dev/contact`` のフォームを入力して送信することことができます。フォームの送信が正しく動作するかテストしてみましょう。まず、正しくフォームが送信されたときの動作を以下のようにまとめてみましょう(「正しくフォームが送信された」というのは、フォームエラーが何も無いときです)。

 1. 問い合わせページが表示される
 2. フォームのフィールドに値が入れられる
 3. フォームを送信する
 4. symblog にメールが送信されたか調べる
 5. HTTP クライアント側のレスポンスに、正しく問い合わせが行われたという通知があるか調べる

今まで見てきたテストの方法では、１と５のステップしか行うことができません。残りの３つのステップのテストの仕方を見て行きましょう。

``src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php`` にある ``PageControllerTest`` クラスに ``testContact()`` メソッドを追加してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testContact()
    {
        $client = static::createClient();

        $crawler = $client->request('GET', '/contact');

        $this->assertEquals(1, $crawler->filter('h1:contains("Contact symblog")')->count());

        // Select based on button value, or id or name for buttons
        $form = $crawler->selectButton('Submit')->form();

        $form['blogger_blogbundle_enquirytype[name]']       = 'name';
        $form['blogger_blogbundle_enquirytype[email]']      = 'email@email.com';
        $form['blogger_blogbundle_enquirytype[subject]']    = 'Subject';
        $form['blogger_blogbundle_enquirytype[body]']       = 'The comment body must be at least 50 characters long as there is a validation constrain on the Enquiry entity';

        $crawler = $client->submit($form);

        $this->assertEquals(1, $crawler->filter('.blogger-notice:contains("Your contact enquiry was successfully sent. Thank you!")')->count());
    }

一般的な方法から始めます。まず ``/contact`` URL にリクエストをします。そしてそのページの ``H1`` のタイトルが正しいかチェックします。次に ``Crawler`` を使用しフォームの送信ボタンを選択します。フォーム自体ではなくボタンを選択する理由は、複数ボタンがある可能性もありますし、別々にクリックできるようにするためです。選択したボタンからフォームを取得します。フォームの値は、配列の表記法 ``[]`` を使用してセットすることができます。そして、フォームをクライアントの ``submit()`` メソッドに私、実際に送信を行います。ほとんどの場合、その返り値として ``Crawler`` インスタンスを受け取ります。 ``Crawler`` を使用してレスポンスにフラッシュメッセージがあるかチェックします。全て正しく機能しているかチェックするためにテストを実行してください。

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

テストは失敗しました。 PHPUnit は以下の内容を出力しています。

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Controller\PageControllerTest::testContact
    Failed asserting that <integer:0> matches expected <integer:1>.

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php:53

    FAILURES!
    Tests: 3, Assertions: 5, Failures: 1.

この出力は、フォーム送信のレスポンス内にフラッシュメッセージが無い、ということを知らせてくれます。それは ``test`` 環境は、リダイレクトをそのままフォローしないからです。フォームが ``PagetController`` クラスを正しく検証した後にリダイレクトをするようにします。つまり、ここでのリダイレクトはすぐにフォローされないので、明示的にフォローするように指示しなければなりません。理由はとても簡単です。リダイレクト前のレスポンスを調べることができるようにするためです。メールが送信されたかを調べる方法は、この後に説明します。まず、 ``PageControllerTest`` クラスを修正してクライアントがリダイレクトをフォローするように指定してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testContact()
    {
        // ..

        $crawler = $client->submit($form);

        // Need to follow redirect
        $crawler = $client->followRedirect();

        $this->assertEquals(1, $crawler->filter('.blogger-notice:contains("Your contact enquiry was successfully sent. Thank you!")')->count());
    }

これで PHPUnit のテストを実行するとテストはパスするはずです。次に問い合わせフォームの送信プロセスをチェックする最終ステップを見て symfony にメールが送られたかをチェックしましょう。次のコンフィギュレーションを設定しているので、 ``test`` 環境ではメールは配送されないことはもう知っていると思います。

.. code-block:: yaml

    # app/config/config_test.yml

    swiftmailer:
        disable_delivery: true

ウェブプロファイラの集める情報を使用して送信したメールをテストすることができます。これは、クライアントがリダイレクトをフォローする前にする必要があります。リダイレクトしてしまうとプロファイラの情報が使用できませんので、リダイレクト前にプロファイラでチェックしましょう。 ``testContact()`` メソッドを次のように修正してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testContact()
    {
        // ..

        $crawler = $client->submit($form);

        // Check email has been sent
        if ($profile = $client->getProfile())
        {
            $swiftMailerProfiler = $profile->getCollector('swiftmailer');

            // Only 1 message should have been sent
            $this->assertEquals(1, $swiftMailerProfiler->getMessageCount());

            // Get the first message
            $messages = $swiftMailerProfiler->getMessages();
            $message  = array_shift($messages);

            $symblogEmail = $client->getContainer()->getParameter('blogger_blog.emails.contact_email');
            // Check message is being sent to correct address
            $this->assertArrayHasKey($symblogEmail, $message->getTo());
        }

        // Need to follow redirect
        $crawler = $client->followRedirect();

        $this->assertTrue($crawler->filter('.blogger-notice:contains("Your contact enquiry was successfully sent. Thank you!")')->count() > 0);
    }

フォーム送信の後にプロファイラが使用可能かチェックしています。現在の環境の設定でプロファイラを無効にしていないか調べるためです。

.. tip::

    テストは、 ``test`` 環境で実行させなければならない、ということではありません。プロファイラの使えない ``本番`` 環境でテストを実行することも可能です。

プロファイラが使用可能であることを調べたら、 ``swiftmailer`` コレクタを取り出します。 ``swiftmailer`` コレクタは、メール送信サービスがどのように行われたかに関する情報を集めます。このコレクタを使用して送信されたメールに関する情報を取得することができます。

次に ``getMessageCount()`` メソッドを使ってメールが一通送信されたことを調べます。これだけでメールが送られたかをチェックするのに十分かもしれませんが、正しい送信先い送られたかなど調べ尽くしてはいません。間違ったメールアドレスにメールを送信してしまうなんてことはあってはなりません。ちゃんと正しいメールアドレスに送られたかを調べます。

これでテストを実行して全て正しく動作したことをチェックしてください。

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

ブログのコメント追加のテスト
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

問い合わせページのテストで得た知識を使ってブログコメントの送信プロセスをテストしましょう。もう一度フォームが正しく送信されたときにどうなるかのアウトラインをまとめましょう。

 1. ブログページへナビゲートする
 2. コメントフォームに値を入力する
 3. フォームを送信する
 4. ブログのコメントリストの最後にコメントが新しく追加されているかチェックする
 5. サイドバーの最新コメントリストの一番上に追加したコメントが表示されるかチェックする

``src/Blogger/BlogBundle/Tests/Controller/BlogControllerTest.php`` に新しくファイルを作成し、次の内容を追加してください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Controller/BlogControllerTest.php

    namespace Blogger\BlogBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class BlogControllerTest extends WebTestCase
    {
        public function testAddBlogComment()
        {
            $client = static::createClient();

            $crawler = $client->request('GET', '/1/a-day-with-symfony');

            $this->assertEquals(1, $crawler->filter('h2:contains("A day with Symfony2")')->count());

            // Select based on button value, or id or name for buttons
            $form = $crawler->selectButton('Submit')->form();

            $crawler = $client->submit($form, array(
                'blogger_blogbundle_commenttype[user]'          => 'name',
                'blogger_blogbundle_commenttype[comment]'       => 'comment',
            ));

            // Need to follow redirect
            $crawler = $client->followRedirect();

            // Check comment is now displaying on page, as the last entry. This ensure comments
            // are posted in order of oldest to newest
            $articleCrawler = $crawler->filter('section .previous-comments article')->last();

            $this->assertEquals('name', $articleCrawler->filter('header span.highlight')->text());
            $this->assertEquals('comment', $articleCrawler->filter('p')->last()->text());

            // Check the sidebar to ensure latest comments are display and there is 10 of them

            $this->assertEquals(10, $crawler->filter('aside.sidebar section')->last()
                                            ->filter('article')->count()
            );

            $this->assertEquals('name', $crawler->filter('aside.sidebar section')->last()
                                                ->filter('article')->first()
                                                ->filter('header span.highlight')->text()
            );
        }
    }

今回は、すぐに全てのテストを行いましょう。コードの詳細を見る前に、テストを実行して全て正しく動作していることを確認してください。

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/BlogControllerTest.php

PHPUnit は、１つのテストの実行が成功したことを通知するでしょう。 ``testAddBlogComment()`` メソッドのコードを見てみると、クライアントを作成し、ページへリクエストを行い、そのページが正しいかどうかチェックする、といったように今までと同じになっています。そして、エントフォームを追加し、フォームを送信しています。フォームの値を入れる方法は、前のやり方と少々異なっています。今回は、 ``submit()`` メソッドの第二引数にフォームの値を渡しています。

.. tip::

    フォームのフィールドに値をセットする際にオブジェクト指向的なインタフェースを使用することもできます。以下のように行います。

    .. code-block:: php

        // Tick a checkbox
        $form['show_emal']->tick();
        
        // Select an option or a radio
        $form['gender']->select('Male');

フォームの送信後に、
After submitting the form, we request the client should follow the redirect so we
can check the response. We use the ``Crawler`` again to get the last blog comment, which
should be the one we just submitted. Finally we check the latest comments in the
sidebar to check the comment is also the first one in the list.

ブログリポジトリ
~~~~~~~~~~~~~~~~

最後に、 Doctrin 2 のリポジトリのテストに関して説明しましょう。 ``src/Blogger/BlogBundle/Tests/Repository/BlogRepositoryTest.php`` に新しくファイルを作成して、次の内容を加えてください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Repository/BlogRepositoryTest.php

    namespace Blogger\BlogBundle\Tests\Repository;

    use Blogger\BlogBundle\Repository\BlogRepository;
    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class BlogRepositoryTest extends WebTestCase
    {
        /**
         * @var \Blogger\BlogBundle\Repository\BlogRepository
         */
        private $blogRepository;

        public function setUp()
        {
            $kernel = static::createKernel();
            $kernel->boot();
            $this->blogRepository = $kernel->getContainer()
                                           ->get('doctrine.orm.entity_manager')
                                           ->getRepository('BloggerBlogBundle:Blog');
        }

        public function testGetTags()
        {
            $tags = $this->blogRepository->getTags();

            $this->assertTrue(count($tags) > 1);
            $this->assertContains('symblog', $tags);
        }

        public function testGetTagWeights()
        {
            $tagsWeight = $this->blogRepository->getTagWeights(
                array('php', 'code', 'code', 'symblog', 'blog')
            );

            $this->assertTrue(count($tagsWeight) > 1);

            // Test case where count is over max weight of 5
            $tagsWeight = $this->blogRepository->getTagWeights(
                array_fill(0, 10, 'php')
            );

            $this->assertTrue(count($tagsWeight) >= 1);

            // Test case with multiple counts over max weight of 5
            $tagsWeight = $this->blogRepository->getTagWeights(
                array_merge(array_fill(0, 10, 'php'), array_fill(0, 2, 'html'), array_fill(0, 6, 'js'))
            );

            $this->assertEquals(5, $tagsWeight['php']);
            $this->assertEquals(3, $tagsWeight['js']);
            $this->assertEquals(1, $tagsWeight['html']);

            // Test empty case
            $tagsWeight = $this->blogRepository->getTagWeights(array());

            $this->assertEmpty($tagsWeight);
        }
    }

テストを実行するのにデータベースへの接続が必要なので、 ``WebTestCase`` を拡張して Symfony2 のカーネルをブートストラップさせます。次のタスクを使用して、テストを実行してください。

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Repository/BlogRepositoryTest.php

コードカバレッジ
----------------

次に行く前に、コードカバレッジについて説明しましょう。コードカバレッジは、テストが実行されたときにコードのどの部分が実行されたかについての情報を教えてくれます。カバレッジを使用することで、まだテストされていないコードを確認することができ、テストを書く必要があるか判断することができます。

アプリケーションのコードカバレッジ分析の出力させるには、次のタスクを走らせるください。

.. code-block:: bash

    $ phpunit --coverage-html ./phpunit-report -c app/

上のタスクは、 ``phpunit-report`` フォルダにコードカバレッジ分析を出力します。 ``index.html`` ファイルをブラウザで開いて分析結果を確認してみてください。

詳細は、 PHPUnit のドキュメントの `コードカバレッジ分析 <http://www.phpunit.de/manual/current/en/code-coverage-analysis.html>`_ の章を参照してください。

結論
----

この章でテストに関する多くの重要な内容をカバーしてきました。ウェブサイトが正しく機能することを保証するためのテストとして、ユニットテストと機能テストの両方を見てきました。ブラウザのリクエストをシミュレートする方法や Symfony2 の ``Crawler`` クラスを使用してこれらのリクエストのレスポンスをチェックする方法を見てきました。

次章では、 Symfony2 のセキュリティコンポーネントについて見ていきます。つまり、ユーザ管理に関してです。また、 symblog の管理をするために、 FOSUserBundle を組み込むことにします。
