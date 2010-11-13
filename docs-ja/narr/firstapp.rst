.. _firstapp_chapter:

最初の :app:`Pyramid` アプリケーションを作る
=================================================


この章では、最小の `Pyramid` アプリケーションを作る。
作り終わったら、どのようにアプリケーションが動くのかを詳細に説明する。

.. note::

   もしも、あなたが理論派であるなら、コードを追う前に、
   :ref:`contextfinding_chapter` 
   と :ref:`views_chapter` が理解の助けになるだろう。
   しかし、多くのプログラマーがそうであるように、流れに身を任せるなら?、
   これらは必要でない。

.. _helloworld_imperative:

Hello World, Goodbye World
--------------------------


以下は、命令的な設定方法による、
非常にシンプルな :app:`Pyramid` アプリケーションの例である。

.. code-block:: python
   :linenos:

   from pyramid.configuration import Configurator
   from pyramid.response import Response
   from paste.httpserver import serve

   def hello_world(request):
       return Response('Hello world!')

   def goodbye_world(request):
       return Response('Goodbye world!')

   if __name__ == '__main__':
       config = Configurator()
       config.begin()
       config.add_view(hello_world)
       config.add_view(goodbye_world, name='goodbye')
       config.end()
       app = config.make_wsgi_app()
       serve(app, host='0.0.0.0')

このコードを ``helloworld.py`` というPythonスクリプトに保存したら、
:app:`Pyramid` がインストールされている Python インタプリタで実行させよう。
すると、 TCPの8080番ポートでHTTPサーバーが開始される。

.. code-block:: bash

   $ python helloworld.py
   serving on 0.0.0.0:8080 view at http://127.0.0.1:8080


8080番ポートのルートURL (``/``) にブラウザでアクセスしてみると、
サーバーは単純な "Hello world!" というテキストを返す。
さらにブラウザで ``/goodbye`` URLにアクセスすると、
"Goodbye world!" というテキストを返す。

これで、どんなアプリケーションなのか、初歩的な理解を得られたはずだ。
断片をそれぞれ説明していこう。

Imports
~~~~~~~

上記のスクリプトでは以下のような import セットを定義している。:

.. code-block:: python
   :linenos:

   from pyramid.configuration import Configurator
   from pyramid.response import Response
   from paste.httpserver import serve

このスクリプトは、 ``Configurator`` クラスを ``pyramid.configuration`` モジュールからインポートしている。
このクラスは、 :app:`Pyramid` を、特定のアプリケーションに設定するために使われる。
このクラスのインスタンスは、 
:app:`Pyramid` アプリケーション開発のさまざまな設定を行うメソッドを持っている。

そして、 :class:`pyramid.response.Response` クラスを :term:`response` オブジェクト作成のために、後ほど使用する。

他の多くのPython Web フレームワーク と同様、 :app:`Pyramid` も、
サーバーとアプリケーションを接続するために、 :term:`WSGI`
プロトコルを使っている。

The
:mod:`paste.httpserver` server is used in this example as a WSGI server for
convenience, as the ``paste`` package is a dependency of :app:`Pyramid` itself.

この例では、 :mod:`paste.httpserver` サーバーを
WSGIサーバーとして便利に使っている。

View Callable Declarations
~~~~~~~~~~~~~~~~~~~~~~~~~~

上記のスクリプトでは、インポートセットの下に、２つの関数を定義している。
１つは、 ``hello_world`` で、もう１つは、 ``goodbye_world``だ。


.. code-block:: python
   :linenos:

   def hello_world(request):
       return Response('Hello world!')

   def goodbye_world(request):
       return Response('Goodbye world!')

これらの関数には、なにもやっかいなところはない。
どちらの関数も、1つ引数を受け取る(``request``)。

``hello_world`` 関数は、なにもせずに、
``Hello world!`` というボディのレスポンスを返す。
``goodbye_world`` 関数は、 ``Goodbye world!`` というボディのレスポンスを返す。

これらの関数は、 :term:`view callable` として知られている。

訳中の追記：とりあえず、view callable = ビュー関数と訳しておく。

View
callables in a :app:`Pyramid` application accept a single argument,
``request`` and are expected to return a :term:`response` object. 
ビュー関数は :app:`Pyramid` アプリケーションでは、 ``request`` を受け取り、
:term:`response` オブジェクトを返すことになっている。


ビュー関数は、関数である必要はなく、クラスやインスタンスなどで実装することも可能である。
しかし、ここでは、関数でうまく動く。

ビュー関数は通常、 :term:`request` オブジェクトとともに呼び出される。
リクエストオブジェクトは、 :term:`WSGI` サーバーを経由して、 
:app:`Pyramid` に送られてきたHTTPリクエストをあらわしている。


ビュー関数は、 :term:`response` オブジェクトを返すことが求められる。
なぜなら、レスポンスオブジェクトが、実際のHTTPレスポンスを生成するために
必要な情報をすべて持っているからだ。
このオブジェクトは、 :term:`WSGI` サーバーによって、テキストに変換されて、
リクエストしたブラウザに返される。

レスポンスを返すために、それぞれのビュー関数は、 :clas:`pyramid.response.Response` クラスのインスタンスを作成している。
``hello_world`` 関数では、 
``'Hello wrld!'`` 文字列を ``Response`` のコンストラクタに
*body* として、渡している。
``goodbye_world`` 関数では、 ``'Goodbye world!'`` 文字列を渡している。

.. index::
   single: imperative configuration
   single: Configurator
   single: helloworld (imperative)

.. _helloworld_imperative_appconfig:

Application Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~

スクリプト中で、以下のように、すでに説明したインポートや、
関数定義を利用するための、アプリケーションの設定(*configuration*)
``if`` 文の中に書かれている。

.. code-block:: python
   :linenos:

   if __name__ == '__main__':
       config = Configurator()
       config.begin()
       config.add_view(hello_world)
       config.add_view(goodbye_world, name='goodbye')
       config.end()
       app = config.make_wsgi_app()
       serve(app, host='0.0.0.0')

じゃあ、これを細かくかみくだいてみていこう。

Configurator Construction
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python
   :linenos:

   if __name__ == '__main__':
       config = Configurator()

コード中の、``if __name__ == '__main__':`` という行は、
Pythonイディオムになっている。
スクリプトが直接コマンドラインから実行されなかった場合、
if文中のコードは実行されない。
たとえば、このコードが、 ``helloworld.py`` という名前のファイルに
保存されている場合、このコードは、 ``python helloworld.py`` と
オペレーションシステムのコマンドラインで実行した場合のみ、実行される。


この場合、 ``helloworld.py`` は、 Python *モジュール* である。
``if`` の条件は必要 -- または、ベストプラクティス -- である。
なぜなら、 どのPythonモジュールも他の Pythonモジュールから
インポートされるからである。
このイディオムは、
このスクリプトが、モジュールとしてインポートされたときに、
``if`` 文の中身を実行しないことを明示する。
直接、スクリプトとして実行された場合だけ、 ``if`` 文の中身が実行される。

``config = Configurator()`` の行は、 :class:`pyramid.configuration.Configurator`
のインスタンスを作成している。
``config`` オブジェクトは、 特定の :app:`Pyramid` アプリケーションに対する
設定をするためのAPIを実装している。
Configuratorのメソッドが呼び出されると、 
Cornfiguratorは、 :term:`application registory` に、
アプリケーションと関連付けて登録する。

Beginning Configuration
~~~~~~~~~~~~~~~~~~~~~~~

.. ignore-next-block
.. code-block:: python

   config.begin()


:meth:`pyramid.configuration.Configurator.begin` メソッドは、
システムに、アプリケーション設定を開始することを知らせる。
特に、 :term:`application registry` が、このconfigurator を、
"現在の" アプリケーションとして登録する。
これは、 application registry の :term:`thread local` に
このconfiguratorを設定する。
これは明示的に行われる。
なぜならconfiguratorを"current"として、 registoryと関係なく使用するからである。

.. note::

   See :ref:`threadlocals_chapter` for a discussion about what it
   means for an application registry to be "current".

.. _adding_configuration:

Adding Configuration
~~~~~~~~~~~~~~~~~~~~

.. ignore-next-block
.. code-block:: python
   :linenos:

   config.add_view(hello_world)
   config.add_view(goodbye_world, name='goodbye')

各行は、:meth:`pyramid.configuration.Configurator.add_view` メソッド
を呼んでいる。
configurator の ``add_view`` は、
:term:`view configuration` を :term:`application registry` に登録する。
:term:`view configuration` は、 :term:`request` に関する
条件と、それに関連する :term:`view callable` を設定する。
条件は、``add_view`` のキーワード引数で設定する。
それぞれのキーワード引数は、 view configuration :term:`predicate` と呼ばれる。


``config.add_view(hello_world)`` の行は、
``hello_world`` 関数をビュー関数として登録している。
この Configurator の ``add_view`` メソッドは、
ビュー関数か、 :term:`dotted Python name` のどちらかが第一引数でなければならない。
この場合は、 ``hello_world`` が第一引数として渡されている。
この行では、 ``add_view`` の、 :term:`predicate` の ``name`` を 
*デフォルト* 値にしている。
``name`` 条件のデフォルト値は、から文字(``''``) である。
これは、 空文字の :term:`view name` の場合に、
:app:`Pyramid` が、 ``hello_world`` を実行するように設定している。
:term:`view name` は、後の章で詳しく解説しているが、
リクエストの状態が、view名として、空文字となっているということである。
このアプリケーションでいえば、 ``hello_world`` ビュー関数は、
ルートURL ``/`` にブラウザからアクセスした場合に呼び出されることになる。

``config.add_view(goodbye_world, name='goodbye')`` では、
``goodbye_world`` 関数をビュー関数として登録している。
この行では、 ``add_view`` を、第一引数にビュー関数を、
``name`` キーワード引数に、 ``'goodbye'`` を渡して呼び出している。
``name`` 引数は、この :term:`view confguration`` が、
`view name` が、 ``'goodbye'`` のリクエストの場合に ``goodbye_world``
ビュー関数が実行されることをあらわしている。
このアプリケーションでは、 ``/goodbye`` URL
に、ブラウザでアクセスすると、``goodbye_world`` ビュー関数が呼び出されることになる。


それぞれの ``add_view`` メソッドの実行により、
:term:`view configuration` が登録される。
``add_view`` メソッドのキーワード引数で指定される
:term:`predicate` は、ビュー関数を実行するリクエストの条件を追加する。
一般的には、多くの条件を増やすことで、さらにビュー関数を実行する条件を制限できる。
:app:`Pyramid` が、リクエストを処理するときに、
*もっとも詳細な* 条件を指定されている
(view configuration がもっとも条件に多く一致した )
ビュー関数を実行する。


このアプリケーションでは、
:app:`Pyramid` が、もっとも詳細なビュー関数を
view :term:`predicate` を適用して、決定する。
:meth:`pyramid.configuration.Configurator.add_view` の呼び出し順序は
一切関係しない。
``goodbye_world`` を最初に登録して、
``hello_world`` を次に登録できる。
この場合でも、 :app:`Pyramid` は、リクエストに対して、
もっとも詳細な関数を呼び出す。


Ending Configuration
~~~~~~~~~~~~~~~~~~~~

.. ignore-next-block
.. code-block:: python

   config.end()


:meth:`pyramid.configuration.Configurator.end` メソッドは、
システムに、アプリケーション設定が完了したことを知らせsる。
これは、:meth:`pyramid.configuration.Configurator.begin` に対応している。
特に、:term:`application registry` に関連付けられた、
configurator がもはや、"current" のアプリケーションではないことになる。
これにより、:term:`thread local` で取得されるアプリケーションレジストリが、
configuratorと関連したレジストリでなくなることを意味する。

.. note::

   See :ref:`threadlocals_chapter` for a discussion about what it
   means for an application registry to be "current".

.. index::
   single: make_wsgi_app
   single: WSGI application

WSGI Application Creation
~~~~~~~~~~~~~~~~~~~~~~~~~

.. ignore-next-block
.. code-block:: python

   app = config.make_wsgi_app()

After configuring views and ending configuration, the script creates a
WSGI *application* via the
:meth:`pyramid.configuration.Configurator.make_wsgi_app` method.  A
call to ``make_wsgi_app`` implies that all configuration is finished
(meaning all method calls to the configurator which set up views, and
various other configuration settings have been performed).  The
``make_wsgi_app`` method returns a :term:`WSGI` application object
that can be used by any WSGI server to present an application to a
requestor.  :term:`WSGI` is a protocol that allows servers to talk to
Python applications.  We don't discuss :term:`WSGI` in any depth
within this book, however, you can learn more about it by visiting
`wsgi.org <http://wsgi.org>`_.


The :app:`Pyramid` application object, in particular, is an
instance of a class representing a :app:`Pyramid` :term:`router`.
It has a reference to the :term:`application registry` which resulted
from method calls to the configurator used to configure it.  The
:term:`router` consults the registry to obey the policy choices made
by a single application.  These policy choices were informed by method
calls to the :term:`Configurator` made earlier; in our case, the only
policy choices made were implied by two calls to its ``add_view``
method.

WSGI Application Serving
~~~~~~~~~~~~~~~~~~~~~~~~

.. ignore-next-block
.. code-block:: python

   serve(app, host='0.0.0.0')

Finally, we actually serve the application to requestors by starting
up a WSGI server.  We happen to use the :func:`paste.httpserver.serve`
WSGI server runner, passing it the ``app`` object (a :term:`router`)
as the application we wish to serve.  We also pass in an argument
``host=='0.0.0.0'``, meaning "listen on all TCP interfaces."  By
default, the Paste HTTP server listens only on the ``127.0.0.1``
interface, which is problematic if you're running the server on a
remote system and you wish to access it with a web browser from a
local system.  We don't specify a TCP port number to listen on; this
means we want to use the default TCP port, which is 8080.

When this line is invoked, it causes the server to start listening on
TCP port 8080.  It will serve requests forever, or at least until we
stop it by killing the process which runs it.

Conclusion
~~~~~~~~~~

Our hello world application is one of the simplest possible
:app:`Pyramid` applications, configured "imperatively".  We can see
that it's configured imperatively because the full power of Python is
available to us as we perform configuration tasks.

.. note::

  An example of using *declarative* configuration (:term:`ZCML`) instead of
  imperative configuration to create a similar "hello world" is available
  within :ref:`declarative_configuration`.

References
----------

For more information about the API of a :term:`Configurator` object,
see :class:`pyramid.configuration.Configurator` .

For more information about :term:`view configuration`, see
:ref:`views_chapter`.

