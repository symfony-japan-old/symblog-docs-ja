[パート２] - ページの作成: バリデータ、フォーム、メール送信
===========================================================

要約
----

パート１でベーシックな HTML テンプレートを適切な場所に作成しましたので、次はページの１つを実用的にしてみましょう。とてもシンプルなページの問い合わせ(Contact)ページを取り組みましょう。この章の最後には問い合わせページでユーザがウェブマスターに問い合わせを送信することができるようにします。そしてこれらの問い合わせはウェブマスターにメールで送信されます。

この章では、次の内容を説明します。

1. バリデータ
2. フォーム
3. バンドルのコンフィギュレーションの値の設定

問い合わせページ
----------------

ルーティング
~~~~~~~~~~~~

パート１の章で作成したアバウトページのように、問い合わせページのルーティングルールを定義するところから始めましょう。 ``BloggerBlogBundle`` のルーティングファイル ``src/Blogger/BlogBundle/Resources/config/routing.yml`` を開き、次のルーティングルールを追加してください。

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_contact:
        pattern:  /contact
        defaults: { _controller: BloggerBlogBundle:Page:contact }
        requirements:
            _method:  GET

ここでは新しいことは何もありません。このルールは、 HTTP ``GET`` メソッドによるリクエストが ``/contact`` パターンにマッチした際に ``BloggerBlogBundle`` にある ``Page`` コントローラの ``contact`` アクションを実行します。

コントローラ
~~~~~~~~~~~~

次に ``BloggerBlogBundle`` の ``Page`` コントローラに問い合わせページのアクションを追加しましょう。このコントローラは、 ``src/Blogger/BlogBundle/Controller/PageController.php`` にあります。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    // ..
    public function contactAction()
    {
        return $this->render('BloggerBlogBundle:Page:contact.html.twig');
    }
    // ..

問い合わせページのビューをレンダリングしているだけですので、現時点ではとてもシンプルです。後にこのコントローラを修正しに戻ってくることにしましょう。

ビュー
~~~~~~

問い合わせページのビューを ``src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig`` に作成して、次の内容を追加してください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block title %}Contact{% endblock%}

    {% block body %}
        <header>
            <h1>Contact symblog</h1>
        </header>

        <p>Want to contact symblog?</p>
    {% endblock %}

このテンプレートもとてもシンプルです。このテンプレートでは、 ``BloggerBlogBundle`` のレイアウトテンプレートを拡張して、 title ブロックをオーバーライドし新たなタイトルをセットしています。また、 body ブロックに新たな内容も定義しています。

ページをリンクする
~~~~~~~~~~~~~~~~~~

最後に ``app/Resources/views/base.html.twig`` にあるアプリケーションテンプレートの問い合わせページへのリンクを修正します。

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    {% block navigation %}
        <nav>
            <ul class="navigation">
                <li><a href="{{ path('BloggerBlogBundle_homepage') }}">Home</a></li>
                <li><a href="{{ path('BloggerBlogBundle_about') }}">About</a></li>
                <li><a href="{{ path('BloggerBlogBundle_contact') }}">Contact</a></li>
            </ul>
        </nav>
    {% endblock %}

ブラウザで ``http://symblog.dev/app_dev.php/`` にアクセスし、ナビゲーションバーにある問い合わせ(Contact)リンクをクリックすると、ベーシックな問い合わせページが表示されるはずです。これで正しくページをセットアップできたので、問い合わせフォームを作ることにしましょう。問い合わせフォームを作成は、バリデータとフォームの２つの異なる部分に分かれます。バリデータとフォームのコンセプトを取り上げる前に、どうやって問い合わせ内容のデータを処理するか考える必要があります。

問い合わせ(Contact)エンティティ
-------------------------------

ユーザからの問い合わせ内容を表現するクラスを作成することから始めましょう。ユーザには、名前(name)、件名(subject)、問い合わせ内容(body)などの基本情報を入れてもらいたいとします。新しいファイルを  ``src/Blogger/BlogBundle/Entity/Enquiry.php`` として作成し、次の内容をペーストしてください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Enquiry.php

    namespace Blogger\BlogBundle\Entity;

    class Enquiry
    {
        protected $name;

        protected $email;

        protected $subject;

        protected $body;

        public function getName()
        {
            return $this->name;
        }

        public function setName($name)
        {
            $this->name = $name;
        }

        public function getEmail()
        {
            return $this->email;
        }

        public function setEmail($email)
        {
            $this->email = $email;
        }

        public function getSubject()
        {
            return $this->subject;
        }

        public function setSubject($subject)
        {
            $this->subject = $subject;
        }

        public function getBody()
        {
            return $this->body;
        }

        public function setBody($body)
        {
            $this->body = $body;
        }
    }

一目瞭然ですが、このクラスは protected なメンバーとそのアクセサを定義しているだけです。メンバーのバリデート方法やメンバーとフォーム要素との関連の定義は何もありません。それに関しては、後に扱うことにしましょう。


.. note::

    Symfony2 のネームスペースの使用について簡単な余談をしましょう。ここで作成したエンティティクラスは、 ``Blogger\BlogBundle\Entity`` のネームスペースをセットしています。 Symfony2 のオートローディングは、 `PSR-0 標準 <http://groups.google.com/group/php-standards/web/psr-0-final-proposal?pli=1>`_ をサポートしているので、ネームスペースは直接バンドルのフォルダストラクチャにマップします。 ``Enquiry`` エンティティクラスが ``src/Blogger/BlogBundle/Entity/Enquiry.php`` に配置されているということは、 Symfony2 が正しくこのクラスをオートロードできるということを保証しています。

    Symfony2 のオートローダーはなぜ ``Blogger`` ネームスペースが ``src`` ディレクトリにあるか知っているのでしょうか？これは ``app/autoloader.php`` のオートローダーで以下のように設定しているからです。

    .. code-block:: php

        // app/autoloader.php
        $loader->registerNamespaceFallbacks(array(
            __DIR__.'/../src',
        ));

    上の命令文は、既に登録されていないネームスペースのフォールバックを登録しています。 ``Blogger`` ネームスペースは登録されていないので、 Symfony2 のオートローダーは、 ``src`` ディレクトリ内の必要なファイルを探すことになります。

    オートローディングとネームスペースは、 Symfony2 においてとても強力なコンセプトです。 PHP がクラスを探すことができないというエラーに遭遇したら、ネームスペースもしくはフォルダ構造にミスがあることがほとんどでしょう。また、上記のようにオートローダーにネームスペースが登録されているか調べてください。この問題を ``解決`` するために PHP の ``require`` や ``include`` 命令は使用しないでください。代わりにオートローディングを使用してください。

フォーム
--------

次にフォームを作成しましょう。 Symfony2 にはとても強力なフォームフレームワークが付いてきます。このフォームフレームワークは、退屈なフォームの扱いを簡単にしてくれます。他の Symfony2 のコンポーネントと同じように、Symfony2 の外部にある自分のプロジェクトで独立して使用することができます。この `フォームコンポーネントのソース <https://github.com/symfony/Form>`_ は Github で入手可能です。 ``AbstractType`` クラスを拡張して、問い合わせフォームを作成していきましょう。このクラスを作成せずに、コントローラ内で直接フォームを作成することもできますが、フォームを独自のクラスに分離させることによってアプリケーション全体でこのフォームを再利用できるようになります。また、よってコントローラを散らかすことを避けることができます。つまり、コントローラはシンプルであるべきなのです。コントローラの目的はモデルとビューをつなげることなのです。

問い合わせタイプ(EnquiryType)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``src/Blogger/BlogBundle/Form/EnquiryType.php`` に新しくファイルを作成し、次の内容をペーストしてください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Form/EnquiryType.php

    namespace Blogger\BlogBundle\Form;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilderInterface;

    class EnquiryType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder->add('name');
            $builder->add('email', 'email');
            $builder->add('subject');
            $builder->add('body', 'textarea');
        }

        public function getName()
        {
            return 'contact';
        }
    }

``EnquiryType`` クラスは ``FormBuilderInterface`` インターフェイスを受け取り、使用します。 このインターフェイスは ``FormBuilder`` クラスで使用されます。 ``FormBuilder`` クラスは、フォーム作成時にとても役に立ちます。フィールドの保持しているメタデータに基づいたフィールドの定義のプロセスを簡単にしてくれます。今回の Enquiry エンティティはとてもシンプルで、まだメタデータを定義していませんので ``FormBuilder`` はデフォルトのテキスト入力をフィールドタイプに使用します。ほとんどのフィールドにテキスト入力は適切ですが、 body フィールドには ``textarea`` を指定したいですし、 email フィールドには HTML5 の email 入力タイプのアドバンテージを享受したいとします。

.. note::

    ``getName`` メソッドはユニークな識別子を返すようにしてください。

コントローラ内にフォームを作成する
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

これで ``Enquiry`` エンティティと ``EnquiryType`` を定義しましたので、contact アクションを修正して、これらのクラスを使用しましょう。 ``src/Blogger/BlogBundle/Controller/PageController.php`` の contact アクションの内容を次のように変更してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    public function contactAction()
    {
        $enquiry = new Enquiry();
        $form = $this->createForm(new EnquiryType(), $enquiry);

        $request = $this->getRequest();
        if ($request->getMethod() == 'POST') {
            $form->bindRequest($request);

            if ($form->isValid()) {
                // Perform some action, such as sending an email

                // Redirect - This is important to prevent users re-posting
                // the form if they refresh the page
                return $this->redirect($this->generateUrl('BloggerBlogBundle_contact'));
            }
        }

        return $this->render('BloggerBlogBundle:Page:contact.html.twig', array(
            'form' => $form->createView()
        ));
    }

``Enquiry`` エンティティのインスタンスを作成するところから始めましょう。 ``Enquiry`` エンティティは、問い合わせ内容のデータを表します。次に、実際のフォームを作成します。先ほど作成した ``EnquiryType`` を指定して、問い合わせ内容オブジェクトを渡します。 ``createForm`` メソッドはフォームを制作するのに必要なこれら２つの青写真(``Enquiry`` と ``EnquiryType``)を使用することができます。

このコントローラアクションは、フォームの表示と送信された内容を処理しますので、 HTTP メソッドをチェックする必要があります。 フォーム送信は一般的に ``POST`` で行われますので、今回のフォームもそれに合わせます。リクエストメソッドが ``POST`` であれば ``bindRequest`` の呼び出しで、送信されたデータを ``$enquiry`` オブジェクトのメンバーに設定します。この時点で ``$enquiry`` オブジェクトは、ユーザが送信した内容を保持していることになります。

次にフォームが有効であったかチェックをします。現時点では何もバリデータに指定していないので、フォームは常に有効になります。

最後にテンプレートをレンダリングします。テンプレートにフォームのビュー内容も渡していることも忘れないでください。このオブジェクトを使用すれば、ビューでフォームをレンダリングすることができます。

コントローラで２つの新しいクラスを使用したので、ネームスペースをインポートする必要があります。 ``src/Blogger/BlogBundle/Controller/PageController.php`` のコントローラファイルを次のように修正してください。既にある ``use`` 命令文の下に、今回使用したネームスペースをインポートするように追加してください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/PageController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    // Import new namespaces
    use Blogger\BlogBundle\Entity\Enquiry;
    use Blogger\BlogBundle\Form\EnquiryType;

    class PageController extends Controller
    // ..

フォームのレンダリング
~~~~~~~~~~~~~~~~~~~~~~

Twig のメソッドを使用したフォームのレンダリングはとてもシンプルです。 Twig は、フォームのレンダリングにレイヤーシステムを用意しており、全てのエンティティを１度でレンダリングしたり、個々のエラーや要素を別々にレンダリングしたりすることができます。これは、カスタマイズが必要なレベルによって適宜使用してください。

Twig のメソッドのパワーを説明するには、次のスニペットでフォーム全体をレンダリングすることが可能であることを見れば十分でしょう。

.. code-block:: html

    <form action="{{ path('BloggerBlogBundle_contact') }}" method="post" {{ form_enctype(form) }}>
        {{ form_widget(form) }}

        <input type="submit" />
    </form>

この方法はプロトタイプやシンプルなフォームにはとても便利ですが、フォームの表示をカスタマイズすることはよくあるので、これでは無理があることがあるでしょう。

今回の問い合わせフォームでは、もう少しカスタマイズしてみましょう。 ``src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig`` のテンプレートコードを次の内容に修正してください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block title %}Contact{% endblock%}

    {% block body %}
        <header>
            <h1>Contact symblog</h1>
        </header>

        <p>Want to contact symblog?</p>

        <form action="{{ path('BloggerBlogBundle_contact') }}" method="post" {{ form_enctype(form) }} class="blogger">
            {{ form_errors(form) }}

            {{ form_row(form.name) }}
            {{ form_row(form.email) }}
            {{ form_row(form.subject) }}
            {{ form_row(form.body) }}

            {{ form_rest(form) }}

            <input type="submit" value="Submit" />
        </form>
    {% endblock %}

上記を見ればわかるように、フォームをレンダリングするのに ４つの新しい Twig のメソッドを使用しています。

最初のメソッド ``form_enctype`` は、フォームのコンテントタイプをセットします。ファイルアップロードを行うフォームの際には必ずセットしてください。今回のフォームではファイルアップロードをしませんが、将来、このフォームにファイルアップロードを追加するかもしれません。そのときのためにも、 ``form_enctype`` を使用するのは常にベストプラクティスです。コンテントタイプを指定し忘れてファイルアップロードを扱うフォームをデバッグするのは本当に大変です。

２番目のメソッド ``form_errors`` は、バリデーションで失敗した際のフォームのエラーを全てレンダリングします。

３番目のメソッド ``form_row`` は、個々のフォームフィールドに関連する要素をひとまとめにして出力します。 ``form_row`` は、フィールドのエラー、ラベル、実際のフィールド要素を含みます。

最後に ``form_rest`` メソッドを使用します。 ``form_rest`` をフォームの最後に使用するのは常に安全策となります。 ``form_rest`` は、 hidden フィールドや Symfony2 の CSRF トークンも含む全てのフィールドをレンダリングしてくれます。

.. note::

    クロスサイトリクエストフォージェリ (CSRF) の詳細は、 Symfony2 のガイドブックの  `Forms chapter <http://symfony.com/doc/current/book/forms.html#csrf-protection>`_ を参照してください。


フォームをスタイルする
~~~~~~~~~~~~~~~~~~~~~~

``http://symblog.dev/app_dev.php/contact`` にアクセスして問い合わせフォームを見てみると、フォームがあまり魅力的ではないと思われるでしょう。見た目を改良するためにスタイルをいくつか加えてみましょう。このスタイルは Blogger バンドルのフォーム特有のものなので、バンドル内に新しくスタイルシートファイルを作成してスタイルします。 ``src/Blogger/BlogBundle/Resources/public/css/blog.css`` に新しくファイルを作成して以下の内容をペーストしてください。

.. code-block:: css

    .blogger-notice { text-align: center; padding: 10px; background: #DFF2BF; border: 1px solid; color: #4F8A10; margin-bottom: 10px; }
    form.blogger { font-size: 16px; }
    form.blogger div { clear: left; margin-bottom: 10px; }
    form.blogger label { float: left; margin-right: 10px; text-align: right; width: 100px; font-weight: bold; vertical-align: top; padding-top: 10px; }
    form.blogger input[type="text"],
    form.blogger input[type="email"]
        { width: 500px; line-height: 26px; font-size: 20px; min-height: 26px; }
    form.blogger textarea { width: 500px; height: 150px; line-height: 26px; font-size: 20px; }
    form.blogger input[type="submit"] { margin-left: 110px; width: 508px; line-height: 26px; font-size: 20px; min-height: 26px; }
    form.blogger ul li { color: #ff0000; margin-bottom: 5px; }


アプリケーションにこのスタイルシートを使用するように知らせる必要があります。問い合わせ(Contact)テンプレート内でこのスタイルシートをインポートすることもできますが、後に他のテンプレートもこのテンプレートを使用することになります。そのため、パート１で作成した ``BloggerBlogBundle`` レイアウトでインポートする方が里にかなっているでしょう。 ``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` ファイルを開いて ``BloggerBlogBundle`` レイアウトを次の内容で置き換えてください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    {% extends '::base.html.twig' %}

    {% block stylesheets %}
        {{ parent() }}
        <link href="{{ asset('bundles/bloggerblog/css/blog.css') }}" type="text/css" rel="stylesheet" />
    {% endblock %}

    {% block sidebar %}
        Sidebar content
    {% endblock %}

stylesheets ブロックを定義して、親テンプレートで定義されているスタイルシートをオーバーライドしているのがわかりますでしょうか？しかし、ここで重要なのは、 ``parent`` メソッドを呼び出していることです。この ``parent`` メソッドは、親テンプレート ``app/Resources/base.html.twig`` の stylesheets ブロックから内容をインポートして、新しいスタイルシートに追加させています。つまり、既にあるスタイルシートを置き換えたいわけではないのです。

``asset`` 関数を正しくリソースにリンクするには、アプリケーションの ``web`` フォルダにバンドルリソースをコピーするかシンボリックリンクをする必要があります。次のタスクを実行してください。

.. code-block:: bash

    $ php app/console assets:install web --symlink

.. note::

    Windwos のようなシンボリックリンクをサポートしていない OS を使用している際には、次のように symlink オプションを付けないでタスクを実行してください。

    .. code-block:: bash

        php app/console assets:install web

    このメソッドは、実際にバンドルの ``public`` フォルダをアプリケーションの ``web`` フォルダにコピーします。ファイルは実際にコピーされるので、バンドルの public のリソースを変更した際には毎回このタスクを実行する必要があります。

これで問い合わせページを再読み込みすると、フォームにスタイルが綺麗に適用されているはずです。

.. image:: /_static/images/part_2/contact.jpg
    :align: center
    :alt: symblog contact form

.. tip::

    ``asset`` 関数はリソースを使用するのに必要な機能を提供しますが、より良い代替手段があります。 `Kris Wallsmith <http://github.com/kriswallsmith>`_ 作の `Assetic <https://github.com/kriswallsmith/assetic>`_ ライブラリが Symfony2 の標準ディストリビューションにデフォルトで付いてきています。このライブラリは、 Symfony2 の機能の標準以上のアセット管理を提供します。 Assetic を使用すると、アセットのフィルターを実行して、自動的に結合させ、gzip で縮小化させることができます。さらに Assetic を使用することによって、 ``assets:install`` タスクを実行しなくても、バンドルの public フォルダを直接リファレンスすることができるのです。後の章で Assetic の使用を探ることになります。

送信の失敗
----------------

このフォームを送信してみると、次のような Symfony2 のエラーに遭遇します

.. image:: /_static/images/part_2/post_error.jpg
    :align: center
    :alt: No route found for "POST /contact": Method Not Allowed (Allow: GET, HEAD)

このエラーは、HTTP POST メソッドで ``/contact`` にマッチするルートが存在しないことを教えてくれます。このルートは GET と HEAD リクエストのみを受け入れています。このルーティングルールで、メソッドを GET を必須指定して設定していたからです。

``src/Blogger/BlogBundle/Resources/config/routing.yml`` の問い合わせ(contact)のルーティングファイルを修正して、 POST リクエストを受け入れるようにしましょう。

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_contact:
        pattern:  /contact
        defaults: { _controller: BloggerBlogBundle:Page:contact }
        requirements:
            _method:  GET|POST

.. tip::

    ルーティングルールに GET のみを指定しているのに HEAD メソッドも受け入れていることに疑問を持ったかもしれません。理由は HEAD メソッドは、実際のところ GET メソッドだからなのです。ただ、 HEAD メソッドには HTTP ヘッダのみが返されます。

これでフォームを送信すると、エラーが表示されなくなります。しかし、このページでは何もおきません。このページはただ問い合わせフォームへリダイレクトし返してくれるのみです。

バリデータ
----------

Symfony2 のバリデータを使用すると、データのバリデーションの作業をさせてくれます。バリデーションは、フォームから受け取ったデータを処理する際の一般的な作業です。また、バリデーションは、データベースに登録される前のデータに実行される必要があります。 Symfony2 のバリデータは、フォームコンポーネントやデータベースコンポーネントなどの他のコンポーネントから、バリデーションロジックを分離させてくれます。このアプローチは、オブジェクト１つにバリデーションのルールのセットが１つあることを意味しています。

``src/Blogger/BlogBundle/Entity/Enquiry.php`` にある ``Enquiry`` エンティティを修正してバリデータを指定するところから始めましょう。次の５つの新しい ``use`` 命令文をファイルの上部に加えるのを忘れないでください。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Enquiry.php

    namespace Blogger\BlogBundle\Entity;

    use Symfony\Component\Validator\Mapping\ClassMetadata;
    use Symfony\Component\Validator\Constraints\NotBlank;
    use Symfony\Component\Validator\Constraints\Email;
    use Symfony\Component\Validator\Constraints\MinLength;
    use Symfony\Component\Validator\Constraints\MaxLength;

    class Enquiry
    {
        // ..

        public static function loadValidatorMetadata(ClassMetadata $metadata)
        {
            $metadata->addPropertyConstraint('name', new NotBlank());

            $metadata->addPropertyConstraint('email', new Email());

            $metadata->addPropertyConstraint('subject', new NotBlank());
            $metadata->addPropertyConstraint('subject', new MaxLength(50));

            $metadata->addPropertyConstraint('body', new MinLength(50));
        }

        // ..

    }

バリデータを定義するのに、静的メソッド ``loadValidatorMetadata`` を必ず実装してください。このメソッドは ``ClassMetadata`` のオブジェクトを引数で受け取っています。この ``ClassMetadata`` オブジェクトを使用して、エンティティのメンバーにプロパティの制約をセットすることができます。上記の最初の命令文では、 ``NotBlank`` 制約を ``name`` メンバーに適用しています。 ``NotBlank`` はとてもシンプルで、値が空で無ければ ``true`` を返すだけです。次に ``email`` メンバーのバリデーションをセットアップしています。 Symfony2 のバリデータサービスは、 MX レコードまでチェックするドメインチェックを行う  `emails <http://symfony.com/doc/current/reference/constraints/Email.html>`_ のバリデーションを用意しています。 ``subject`` メンバーは ``NotBlank`` と ``MaxLength`` 制約をセットします。このようにメンバーに対しバリデータを好きなだけ適用することができます。

`バリデータ制約 <http://symfony.com/doc/current/reference/constraints.html>`_ の一覧は、 Symfony2 のリファレンスドキュメントを参照してください。また、 `カスタムバリデータを作成する方法 <http://symfony.com/doc/current/cookbook/validation/custom_constraint.html>`_ でも参照することが可能です。

これで、問い合わせフォームを送信すると、送信されたデータがバリデーション制約を通るようになりました。無効なメールアドレスを入力してみてください。メールアドレスが無効であると知らせるエラーメッセージが出力されるはずです。各バリデータは、デフォルトのエラーメッセージを持ってり、必要であればオーバーライドすることができます。email のバリデータのエラーメッセージを変更するには、次のようにしてください。

.. code-block:: php

    $metadata->addPropertyConstraint('email', new Email(array(
        'message' => 'symblog does not like invalid emails. Give me a real one!'
    )));

.. tip::

    HTML5 をサポートしているブラウザを使用してれば、特定の制約に HTML5 のエラーメッセージがプロンプトされます。これはクライアントサイドのバリデーションで、 Symfony2 は ``Entity`` メタデータに基づいて、 HTML5 制約を適切にセットしてくれます。email 要素でこの HTML5 の例を見ることができます。出力される HTML は次のようになります。

    .. code-block:: html

        <input type="email" value="" required="required" name="contact[email]" id="contact_email">

    それは、新しい HTML5 の入力タイプの１つ email を使っており、 required 属性を指定しています。クライアントサイドバリデーションは、フォームをバリデートするのにサーバにアクセスすることが無いので、素晴らしいです。しかし、クライアントサイドバリデーションのみを使用しているのは良くありません。クライアントサイドバリデーションを無視するのは簡単なので、必ずサーバサイドでもデータをバリデートしてください。

メール送信
----------

現時点の問い合わせフォームは、ユーザに問い合わせを送信させることができますが、実際のところは何も起こりません。コントローラを修正してブログのウェブマスターにメールを送信するようにしてみましょう。 Symfony2 には、メール送信を行う `Swift Mailer <http://swiftmailer.org/>`_ ライブラリが付いてきます。 Swift Mailer はとても強力なライブラリです。このライブラリが実行できる触りを見てみましょう。

Swift Mailer を設定する
~~~~~~~~~~~~~~~~~~~~~~~

Swift Mailer は、 Symfony2 の標準ディストリビューションで動作するように設定されています。しかし、送信方法と証明書に関する設定を変更する必要があります。 ``app/config/parameters.ini`` ファイルを開いて、 ``mailer_`` 接頭辞の設定を見てください。

.. code-block:: text

    mailer_transport="smtp"
    mailer_host="localhost"
    mailer_user=""
    mailer_password=""

Swift Mailer はメール送信に関してたくさんの方法を用意しています。 SMTP サーバの使用、sendmail のローカルインストールの使用、 GMail アカウントの使用などです。シンプルさのために GMail アカウントを使用しましょう。パラメターを次のように変更して、 username と password を必要に応じて修正してください。

.. code-block:: text

    mailer_transport="gmail"
    mailer_encryption="ssl"
    mailer_auth_mode="login"
    mailer_host="smtp.gmail.com"
    mailer_user="your_username"
    mailer_password="your_password"

.. warning::

    プロジェクトで Git のようなバージョンコントロールシステム(VCS)を使用しているならば、特にパブリックにアクセス可能なリポジトリの際に、次のことに気をつけてください。 GMail のユーザ名とパスワードがこういったリポジトリにコミットされてしまうと、誰もがその内容を見ることができてしまいます。 ``app/config/parameters.ini`` を、使用している VCS の ignore リストに加えていることを確かめてください。この問題への一般的なアプローチは、慎重に扱うべき情報を持つファイルの名前に接尾辞を付けることです。  ``app/config/parameters.ini`` に ``.dist`` を後に付けるなどです。そして、このファイルの設定に問題のないデフォルト値を入れます。 ``app/config/parameters.ini`` のような実際のファイルは VCS の ignore リストに加えてください。これで ``*.dist`` ファイルをプロジェクトにデプロイすることができ、開発者に ``.dist`` 拡張子を取り除き必要は設定に入れ替えさせることができます。

コントローラを修正する
~~~~~~~~~~~~~~~~~~~~~~

``src/Blogger/BlogBundle/Controller/PageController.php`` にある ``Page`` コントローラを次の内容に修正してください。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php

    public function contactAction()
    {
        // ..
        if ($form->isValid()) {

            $message = \Swift_Message::newInstance()
                ->setSubject('Contact enquiry from symblog')
                ->setFrom('enquiries@symblog.co.uk')
                ->setTo('email@email.com')
                ->setBody($this->renderView('BloggerBlogBundle:Page:contactEmail.txt.twig', array('enquiry' => $enquiry)));
            $this->get('mailer')->send($message);

            $this->get('session')->setFlash('blogger-notice', 'Your contact enquiry was successfully sent. Thank you!');

            // Redirect - This is important to prevent users re-posting
            // the form if they refresh the page
            return $this->redirect($this->generateUrl('BloggerBlogBundle_contact'));
        }
        // ..
    }

Swift Mailer ライブラリを使用して ``Swift_Message`` インスタンスが作成すれば、メールを送信することができます。

.. note::

    Swift Mailer のライブラリは、ネームスペースを使用していないので、 Swift Mailer クラスには ``\`` の接頭辞が必要です。このことによって PHP に `global space <http://www.php.net/manual/en/language.namespaces.global.php>`_ にエスケープバックするのを伝えています。ネームスペースを付けていない全てのクラスと関数に ``\`` の接頭辞を付ける必要があります。 ``Swift_Message`` クラスの前にこの接頭辞を付けなければ、 PHP は現在のネームスペース(``Blogger\BlogBundle\Controller``)からこのクラスを探し出し、その場所には存在しないのでエラーが投げられることになります。

また、このセッションで、 ``flash`` メッセージをセットしました。 フラッシュ(flash)メッセージは、１回のリクエストのみ有効なメッセージです。その後は、 Symfony2 によって自動的にクリーンアップされます。 ``flash`` メッセージは、ユーザが問い合わせを送ったことを知らせるために、問い合わせテンプレート内で表示されます。 ``flash`` メッセージは１度のリクエストのみ有効ですので、直前のアクションの成功をユーザに知らせるにはちょうどいい仕組みです。

``falsh`` メッセージを表示するようにするには、  ``src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig`` を修正する必要があります。テンプレートの内容を以下のように修正してください。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig #}

    {# rest of template ... #}
    <header>
        <h1>Contact symblog</h1>
    </header>

    {% if app.session.hasFlash('blogger-notice') %}
        <div class="blogger-notice">
            {{ app.session.flash('blogger-notice') }}
        </div>
    {% endif %}

    <p>Want to contact symblog?</p>

    {# rest of template ... #}

上記は、 `blogger-notice` 識別子の ``flash`` メッセージがセットされているか調べ、セットしていればその内容が出力されます。

ウェブマスターのメールアドレスを登録する
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Symfony2 は、コンフィギュレーションシステムを提供しており、独自の設定を定義することができます。ウェブマスターのメールアドレスを上記のようにハードコードするのではなく、このシステムを使用して設定してみましょう。そうすることによって、コードを重複することなく他の場所でも簡単にこの値を再利用することができるようになります。さらに、このブログが流行ってしまい、たくさんの問い合わせが来てしまうことになっても、メールアドレスの値をアシスタントのメールアドレスに変更するだけでいいのです。 ``src/Blogger/BlogBundle/Resources/config/config.yml`` ファイルを新しく作成し、次の内容をペーストしてください。

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/config.yml
    parameters:
        # Blogger contact email address
        blogger_blog.emails.contact_email: contact@email.com

パラメータを定義する際には、パラメータの名前をいくつかの要素に分けることは良いプラクティスです。最初の部分は、バンドル名の小文字を使用し区切り文字にアンダースコアを使います。今回の例では、 ``BloggerBlogBundle`` を ``blogger_blog`` に変換しています。パラメータ名の残りの部分は、ピリオド文字を区切り文字に使い、好きなだけつなげることができます。このように論理的にパラメータをグループ化することができます。

Symfony2 のアプリケーションでは、新しくパラメータを使用するために、 ``app/config/config.yml`` にあるメインのアプリケーションコンフィギュレーションファイルでこの設定をインポートする必要があります。メインのアプリケーションコンフィギュレーションファイルの上の方にある ``imports`` 命令文を修正して次のようにしてください。

.. code-block:: yaml

    # app/config/config.yml
    imports:
        # .. existing import here
        - { resource: @BloggerBlogBundle/Resources/config/config.yml }

インポートするパスは、ディスク上のファイルの物理的な位置です。 ``@BloggerBlogBundle`` は、 ``BloggerBlogBundle`` のパスになり、実際は ``src/Blogger/BlogBundle`` になります。

最後に、問い合わせ(contact)アクションを修正して、指定したパラメータを使用するようにしましょう。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php

    public function contactAction()
    {
        // ..
        if ($form->isValid()) {

            $message = \Swift_Message::newInstance()
                ->setSubject('Contact enquiry from symblog')
                ->setFrom('enquiries@symblog.co.uk')
                ->setTo($this->container->getParameter('blogger_blog.emails.contact_email'))
                ->setBody($this->renderView('BloggerBlogBundle:Page:contactEmail.txt.twig', array('enquiry' => $enquiry)));
            $this->get('mailer')->send($message);

            // ..
        }
        // ..
    }

.. tip::

    アプリケーションコンフィギュレーションファイルの上部でこのファイルをインポートするようにしたので、アプリケーション内でインポートされたパラメータの全てをオーバーライドすることができます。例えば、 ``app/config/config.yml`` の下部に以下の内容を追加すれば、パラメータの値のバンドル設定値をオーバーライドします。

    .. code-block:: yaml

        # app/config/config.yml
        parameters:
            # Blogger contact email address
            blogger_blog.emails.contact_email: assistant@email.com

    これらのカスタマイズは、後でアプリケーションがオーバーライドできるように、バンドルに安全なデフォルト値を提供できるようにしています。 

.. note::

    この方法を使用してバンドルコンフィギュレーションパラメータを作成することは簡単ですが、 Symfony2 はバンドルに `セマンティックコンフィグレーションを通す <http://symfony.com/doc/current/cookbook/bundles/extension.html>`_ 方法を用意しています。後にこの方法については説明をします。

メールテンプレートを作成する
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

メールの本文にテンプレートをレンダリング結果をセットします。 ``src/Blogger/BlogBundle/Resources/views/Page/contactEmail.txt.twig`` ファイルを作成し、次の内容をペーストしてください。

.. code-block:: text

    {# src/Blogger/BlogBundle/Resources/views/Page/contactEmail.txt.twig #}
    A contact enquiry was made by {{ enquiry.name }} at {{ "now" | date("Y-m-d H:i") }}.

    Reply-To: {{ enquiry.email }}
    Subject: {{ enquiry.subject }}
    Body:
    {{ enquiry.body }}

メールの内容は、ユーザが送信した問い合わせのみです。

このテンプレートの拡張子が今まで作成してきた他のテンプレートと異なるのに気づいたでしょうか？ ``.txt.twig`` 拡張子を使用しています。 拡張子の前の部分の ``.txt`` は、生成するファイルのフォーマットを指定しています。ここでの一般的なフォーマットは、 .txt,  .html, .css, .js, .xml, .json です。拡張子の後ろの部分は使用するテンプレートエンジンを指定しています。今回の場合あ、 Twig です。拡張子に ``.php`` を使用していれば、レダンリングするテンプレートに PHP を使用します。

これで、問い合わせを送信すると、メールが ``blogger_blog.emails.contact_email`` パラメータにセットしたアドレスへ送信されます。

.. tip::

    Symfony2 を使用すれば、 Symfony2 の異なる環境での作業に応じて、 Swift Mailer ライブラリの挙動を設定することができます。 ``test`` 環境の使用で、このことを確認することができます。デフォルトでは、 Symfony2 の標準ディストリビューションは、 ``test`` 環境で実行する際には、 Swift Mailer はメールを送信しないように設定されています。この設定は、 ``app/config/config_test.yml`` コンフィギュレーションファイルにセットされています。

    .. code-block:: yaml

        # app/config/config_test.yml
        swiftmailer:
            disable_delivery: true

    この機能を ``dev`` 環境にコピーしても便利でしょう。そうすれば、開発中に思いがけず間違ったアドレスにメールを送ってしまうことが防ぐことができます。そのような設定をするには、上のcomfyゆレーションの内容を ``dev`` コンフィギュレーションの  ``app/config/config_dev.yml`` に追加してください。

    これで実際のメールアドレスにメールは送信されなくなりましたが、メールが送られたか、メールの内容は大丈夫か、といったことをテストする方法に疑問を持つでしょう。 Symfony2 は、ディベロッパーツールバーを通すことによって、メールの内容を確認することができます。メールが送信されるとメール通知アイコンがツールバーに表示され、 Swift Mailer が送信する予定だったメールに関する全ての情報を確認することができます。

    .. image:: /_static/images/part_2/email_notifications.jpg
        :align: center
        :alt: Symfony2 toolbar show email notifications

    問い合わせフォームのように、メール送信後にリダイレクトを実行するのであれば、ツールバーでメール通知を見るために ``app/config/config_dev.yml`` の ``intercept_redirects`` 設定を ``true`` にセットする必要があります。

    また、次のように ``dev`` 環境の設定ファイル ``app/config/config_dev.yml`` に指定すれば、 ``dev`` 環境で全てのメール送信先を特定のメールアドレスに設定することができます。

    .. code-block:: yaml

        # app/config/config_dev.yml
        swiftmailer:
            delivery_address:  development@symblog.dev

結論
----

全てのウェブサイトの最も基本的な部分であるフォーム作成の背景にあるコンセプトを説明してきました。 Symfony2 は、素晴らしいバリデータとフォームライブラリが付いてくるので、フォームからバリデーションロジックを分離することができます。そして、モデルなどのアプリケーションの他の部分からバリデーションロジックを使用することができます。また、アプリケーションから読むことのできるカスタムコンフィギュレーションの設定方法を紹介しました。

次の章では、このチュートリアルにおいて重要な位置付けのモデルについて見ていきます。 Doctrine2 を紹介し、また、Doctrine2 を使用して、ブログモデルを定義していきます。また、ブログページを作成し、データフィクスチャのコンセプトを説明していきます。
