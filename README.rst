======
 Phew
======

The best of Peewee with a extensible models and instrumented runtime online
models.

Inspired by Odoo's runtime models, but allowing the Python inheritance and imports.


.. code-block:: python

   class Entity(ModelDefinition, abstract=True):
       ...


   class Person(Entity):
       ...


In other module:

.. code-block:: python

   from .entities import Person as BasePerson

   class Person(BasePerson):
       # Extends this same model with new fields and behavior.


   class User(Person):
       person: Person  # required to be able to instrument the model


Attributes of a model definition
--------------------------------

:model: A string with the name of the model.  It defaults to the name of the
        class.

:table_name: A string with the name of the table in the DB.  It default to a
			 simple transformation of ``CamelCase`` to ``camel_case``.



Instrumentation and attachment
==============================

Instrumentation is the process of the actually materializing the model
definitions so that every Model Definition is "merged" into a single Model and
attached to a database.


.. code-block:: python

   registry =  Registry(
      models_definitions=[Person, User],
   )


Final models
------------

While creating the registry if two model definition classes are to be combined
in a single model, we automatically create single final model definition that
inherit from all the classes in the same order provided to the registry.

The following two registries definitions are equivalent:


.. code-block:: python

   class PersonA(Entity, model='Person'):
       name: str

   class PersonB(Entity, model='Person'):
	   age: int

   registry = Registry(models_definitions=[PersonA, PersonB])

   class Person(PersonA, PersonB):
       pass

   registry = Registry(models_definitions=[Person])


Actually interacting with the database
======================================


.. code-block:: python

   >>> with ProcessEnvironment(db_connection, registry):
   ...     Person.objects.select()


.. code-block:: python

   >>> Person.objects.select()   # doctest: +ELLIPSIS
   Traceback (...)
     ...
   RuntimeError: The model has not being attached to a registry.


The are three specialized classes of environments:

- `ProcessEnvironment`:class: for single-threaded applications

- `ThreadEnvironment`:class: for multi-threaded applications

- `AsyncEnvironment`:class: for applications using async/greenlets.


The actual instance of ``objects`` will depend on the environment.


Pooling the registry
====================

The `Registry`:class: is very lean and so keeping a instance of the `registry`
is not strictly necessary.



Tying environment to requests
=============================

Django
------

Basically, put this in your ``settings.py`` file.

.. code-block:: python

   from phew.contrib.django import setup
   setup()


It should be enough most of the time.  Under the hood, we
