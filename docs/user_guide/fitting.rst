Fitting a Model
===============

.. include:: ../macros.hrst

TODO: to fit a model to data, w/ either numpy arrays or pandas DataFrame/Series, etc

Backend graph optimization during fitting
-----------------------------------------

By default, ProbFlow uses
`tf.function <https://www.tensorflow.org/api_docs/python/tf/function>`_
(for TensorFlow) or
`tracing <https://pytorch.org/docs/master/generated/torch.jit.trace.html>`_
(for PyTorch) to optimize the gradient computations during training.  This
generally makes training faster.

.. code-block:: python3

    N = 1024
    D = 7
    randn = lambda *a: np.random.randn(*a).astype('float32')
    x = randn(N, D)
    w = randn(D, 1)
    y = x@w + 0.1*randn(N, 1)

    model = pf.LinearRegression(D)

    model.fit(x, y)
    # takes around 5s

But to disable autograph/tracing and use only eager execution during model
fitting just pass the ``eager=True`` kwarg to ``fit``.  This takes longer but
can be more flexible in certiain situations that autograph/tracing can't
handle.

.. code-block:: python3

    model.fit(x, y, eager=True)
    # takes around 28s

.. warning::

    When inputs are |DataFrames| or |Series| it is not possible to use tracing
    or ``tf.function``, so ProbFlow falls back on eager execution by defualt
    when the input data are |DataFrames| or |Series|

It's much easier to debug models in eager mode, since you can step through your
own code using `pdb <https://docs.python.org/3/library/pdb.html>`_, instead of
trying to step through the tensorflow or pytorch compilation functions.  So, if
you're getting an error when fitting your model and want to debug the problem,
try using ``eager=True`` when calling ``fit``.

However, eager mode is used for all other ProbFlow functionality (e.g.
:meth:`.Model.predict`, :meth:`.Model.predictive_sample`,
:meth:`.Model.metric`, :meth:`.Model.posterior_sample`, etc).  If you want an
optimized version of one of ProbFlow's inference-time methods, for TensorFlow
you can wrap it in a ``tf.function``:

.. code-block:: python3

    #model.fit(...)

    @tf.function
    def fast_predict(X):
        return model.predict(X)

    fast_predict(x_test)

Or for PyTorch, use ``torch.jit.trace``:

.. code-block:: python3

    #model.fit(...)

    def predict_fn(X):
        return model.predict(X)

    fast_predict = torch.jit.trace(predict_fn, (example_x))

    fast_predict(x_test)

TODO: section on using a DataGenerator

TODO: section on Callbacks
