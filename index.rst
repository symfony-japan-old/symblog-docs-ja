Symfony2 でブログを作ろう
=========================

イントロダクション
------------------

このチュートリアルは、 `Symfony2 <http://symfony.com/>`_ を使用して、いろんな機能を備えたブログサイトの作成方法をガイドします。 Symfony2 フレームワークの標準ディストリビューション(Standard Distribution)を使用します。 Symfony2 の標準ディストリビューションは、実際にウェブサイトを作成する際に必要なメインのコンポーネントを含んでいます。このチュートリアルは、いくつかのパートに分かれており、それぞれのパートは Symfony2 フレームワークや Symfony2 コンポーネントに関する内容をカバーしています。このチュートリアルは、 symfony1 の `Jobeet <http://www.symfony-project.org/jobeet/1_4/Doctrine/en/>`_ チュートリアルのようなものを Symfony2 で作成することを意識して執筆されています。

チュートリアルのパート
~~~~~~~~~~~~~~~~~~~~~~

.. toctree::
    :maxdepth: 1

    docs/configuration-and-templating
    docs/validators-and-forms
    docs/doctrine-2-the-blog-model
    docs/extending-the-model-blog-comments
    docs/customising-the-view-more-with-twig
    docs/testing-unit-and-functional-phpunit

デモサイト
----------

symblog のデモサイトは `http://symblog.co.uk <http://symblog.co.uk/>`_ で見ることができます。また、ソースコードは、 `Github <https://github.com/dsyph3r/symblog>`__ にあります。 Github リポジトリのタグには、このチュートリアルの各パートに沿ったソースコードが格納されています。

対象範囲
--------

このチュートリアルは、 Symfony2 を使用してウェブサイトを作成する際に直面する共通のタスクをカバーすることを目的としています。

    1.  バンドル
    2.  コントローラ
    3.  テンプレート(Twig使用)
    4.  モデル - Doctrine 2
    5.  マイグレーション
    6.  データフィクスチャ
    7.  バリデータ
    8.  フォーム
    9.  ルーティング
    10. アセット管理
    11. メール送信
    12. 環境
    13. エラーページのカスタマイズ
    14. セキュリティ
    15. ユーザとセッション
    16. CRUD 生成 
    17. キャッシュ
    18. テスト
    19. デプロイ

Symfony2 はとてもカスタマイズしやすく、多くの異なる方法で同じ作業をすることができます。例えば、コンフィギュレーションでは YAML, XML, PHP, アノテーションを使用することができますし、テンプレートには Twig と PHP を使用することができます。このチュートリアルをシンプルにするために、コンフィギュレーションでは YAML とアノテーションを採用し、テンプレートには Twig を採用することにします。 `Symfony book <http://symfony.com/doc/current/book/index.html>`_ には、この他の方法を使用したたくさんのリソースが用意されています。よりシンプルな方法を作成するのに貢献していただけるのであれば、 `Github <https://github.com/dsyph3r/symblog-docs>`__ をフォークし、プルリクエストを送ってください。 :)

翻訳
----

スペイン語

~~~~~~~~~~

Symblog の `スペイン語版 <http://symblog.site90.net/>`_ は、 `Lisper <https://twitter.com/#!/esymfony>`_ によって翻訳されました。


フランス語
~~~~~~~~~~

Symblog の `フランス語版 <http://keiruaprod.fr/symblog-fr/>`_ は、 `Clement Keirua <https://twitter.com/clemkeirua>`_ によって翻訳されました。

日本語
~~~~~~

Symblog の `日本語版 <http://symblog.ganchiku.com/>`_ は、 `Shin Ohno <https://twitter.com/ganchiku>`_ によって翻訳されました。

著者
------

このチュートリアルは、 `dsyph3r <http://twitter.com/#!/dsyph3r>`_ によって作成されました。

貢献
----

このチュートリアルの `ソース <https://github.com/dsyph3r/symblog-docs>`_ は Github で手に入れることができます。このチュートリアルをシンプルにするための改善や拡張をするには、プロジェクトをフォークし、プルリクエストを送ってください。また、 `GitHub Issue Tracker <https://github.com/dsyph3r/symblog-docs/issues>`_ を使って問題を提起することもできます。よりカッコいいデザインを作成するのに興味があれば、 `連絡 <http://twitter.com/#!/dsyph3r>`_ してください!

クレジット
----------

`Symfony2 の公式ドキュメント <http://symfony.com/doc/current/>`_ の全ての貢献者に感謝します。この公式ドキュメントは素晴らしい情報のリソースです。

フラッグアイコンは、 `famfamfam <http://www.famfamfam.com/lab/icons/flags/>`_ によるものです。

検索
----

特別なトピックを探していますか？ :ref:`search` を使用してください。
