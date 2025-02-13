Fields and Widgets
==================

Contents:

.. contents::
    :depth: 2
    :local:
    :backlinks: none

Model fields
------------

.. module:: django_jsonform.models.fields
    :synopsis: Model fields


``JSONField``
~~~~~~~~~~~~~

.. class:: JSONField(schema, encoder=None, decoder=None, **options)
    
.. versionadded:: 2.0

It is basically a subclass of Django's ``JSONField``, but for convenience,
it automatically sets up the JSON editor widget for you.

The only difference is that it takes a ``schema`` argument.

In Django < 3.1, for databases other than Postgres, it uses a ``TextField``
underneath.

.. attribute:: schema
    :type: dict, callable

    A ``dict`` or a callable object specifying the schema for the current field.

    A callable is useful for :ref:`specifying dynamic choices <dynamic choices>`.

    The callable function may optionally receive the current model instance. See:
    :ref:`Accessing model instance in callable schema`.

    .. versionchanged:: 2.1
        The ability to provide a callable was added.

    .. versionchanged:: 2.8
        Callable schema may receive an ``instance`` argument.

Usage:

.. code-block:: python

    from django_jsonform.models.fields import JSONField


    class MyModel(models.Model):
        ITEMS_SCHEMA = {...}

        items = JSONField(schema=ITEMS_SCHEMA)

For details about other parameters, options and attributes of the ``JSONField``, see
`Django's docs <https://docs.djangoproject.com/en/3.2/ref/models/fields/#django.db.models.JSONField>`__.


``ArrayField``
~~~~~~~~~~~~~~

.. class:: ArrayField(base_field, size=None, **options)

.. versionadded:: 2.0

A subclass of Django's ``ArrayField`` except it renders a dynamic form widget.

It takes exactly the same arguments as the original class.

It also supports multiple levels of array nesting.

Usage:

.. code-block:: python

    from django_jsonform.models.fields import ArrayField


    class MyModel(models.Model):
        items = ArrayField(models.CharField(max_length=50), size=10)
        # ...

For more details, see
`Django's docs <https://docs.djangoproject.com/en/stable/ref/contrib/postgres/fields/#arrayfield>`__.



Widgets
-------

.. module:: django_jsonform.widgets
    :synopsis: Widgets


``JSONFormWidget``
~~~~~~~~~~~~~~~~~~

.. class:: JSONFormWidget(schema, model_name='')
    
The widget which renders the editor.

It can be used in a form if you don't want to use the model field.

.. attribute:: schema
    :type: dict, callable

    A ``dict`` or a callable object specifying the schema for the current field.

    A callable is useful for :ref:`specifying dynamic choices <dynamic choices>`.

    The callable function may optionally receive the current model instance. See:
    :ref:`Accessing model instance in callable schema`.

    .. versionchanged:: 2.1
        The ability to provide a callable was added.

    .. versionchanged:: 2.8
        Callable schema may receive an ``instance`` argument.

.. attribute:: model_name
    :type: str

    An optional string. The name of the model. It is passed to the file upload handler
    so that you can identify which model is requesting the file upload.

    See :ref:`file-upload-request-parameters` for more details.

Usage:

.. code-block:: python

    # admin.py

    from django_jsonform.widgets import JSONFormWidget
    from myapp.models import ShoppingList


    class ShoppingListForm(forms.ModelForm):
        model = ShoppingList
        fields = '__all__'
        widgets = {
            'items': JSONFormWidget(schema=ShoppingList.ITEMS_SCHEMA)
        }

    class ShoppingListAdmin(admin.ModelAdmin):
        form = ShoppingListForm

    admin.site.register(ShoppingList, ShoppingListAdmin)


This widget can not be used directly with Django's ``ArrayField`` because Django's
``ArrayField`` converts the value from array to a string before passing it to
the widget whereas it expects a list or a dict.


Accessing model instance in callable schema
-------------------------------------------

.. versionadded:: 2.8

Automatically accessing model instance in a widget is not possible. This is due
the way Django initialises the widgets and form fields.

However, you can bypass this limitation by manually setting an ``instance`` attribute
on the widget.

.. code-block::

    def callable_schema(instance):
        # ... do something ...
        pass

    class MyModel(models.Model):
        json_field = JSONField(schema=callable_schema)


    # create a custom modelform
    class MyModelForm(forms.ModelForm):
        def __init__(self):
            # manually set the current instance on the widget
            self.fields['json_field'].widget.instance = self.instance


    # set the form on the admin class
    class MyAdmin(admin.ModelAdmin):
        form = MyModelForm

Now, the value of the instance will be passed to your callable schema function.
