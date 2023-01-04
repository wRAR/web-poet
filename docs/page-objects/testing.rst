.. _web-poet-testing:

======================
Tests for Page Objects
======================

Page Objects that inherit from :class:`~.ItemPage` can be tested by saving the
dependencies needed to create one and the result of
:meth:`~web_poet.pages.ItemPage.to_item`, recreating the Page Object from the
dependencies, running its :meth:`~web_poet.pages.ItemPage.to_item` and
comparing the result to the saved one. ``web-poet`` provides the following
tools for this:

* dependency serialization into a Python object and into a set of files;
* recreating Page Objects from the serialized dependencies;
* a high-level function to save a test fixture;
* a ``pytest`` plugin that discovers fixtures and runs tests for them.

Serialization
=============

:func:`web_poet.serialization.serialize` can be used to serialize an iterable
of Page Object dependencies to a Python object.
:func:`web_poet.serialization.deserialize` can be used to recreate a Page
Object from this serialized data.

An instance of :class:`web_poet.serialization.SerializedDataFileStorage` can be
used to write the serialized data to a set of files in a given directory and to
read it back.

.. note::
    We only support serializing dependencies, not Page Object instances,
    because the only universal way to recreate a Page Object is from its
    dependencies, not from some saved internal state.

Each dependency is serialized to one or several ``bytes`` objects, each of
which is saved as a single file. :func:`web_poet.serialization.serialize_leaf`
and :func:`web_poet.serialization.deserialize_leaf` are used to convert between
a dependency and this set of ``bytes`` objects. They are implemented using
:func:`functools.singledispatch` and while the types provided by ``web-poet``
are supported out of the box, user-defined types need a pair of implementation
functions that need to be registered using
:func:`web_poet.serialization.register_serialization`.

Fixtures
========

The provided ``pytest`` plugin expects fixtures in a certain layout. A set of
fixtures for a single Page Object should be contained in a directory named as
that Page Object fully qualified class name. Each fixture is a directory inside
it, that contains data for Page Object inputs and output::

    fixtures
    └── my_project.po.MyItemPage
        ├── test-1
        │   ├── inputs
        │   │   ├── HttpResponse-body.html
        │   │   ├── HttpResponse-other.json
        │   │   └── ResponseUrl.txt
        │   ├── meta.json
        │   └── output.json
        └─── test-2
            ├── inputs
            │   ├── HttpResponse-body.html
            │   ├── HttpResponse-other.json
            │   └── ResponseUrl.txt
            ├── meta.json
            └── output.json

:func:`web_poet.testing.Fixture.save` can be used to create a fixture inside a
Page Object directory from an iterable of dependencies, an output item and an
optional metadata dictionary. It can optionally take a name for the fixture
directory. By default it uses incrementing names "test-1", "test-2" etc.

.. note::
    ``output.json`` contains a result of
    ``ItemAdapter(page_object.to_item()).asdict()`` saved as JSON.

After generating a fixture you can edit ``output.json`` to modify expected
field values and add new fields, which is useful when creating tests for code
that isn't written yet or before modifying its behavior.

Running tests
=============

The provided ``pytest`` plugin is automatically registered when ``web-poet`` is
installed, and running ``pytest`` in a directory containing fixtures will
discover them and run tests for them.

.. _web-poet-testing-frozen_time:

Handling time fields
====================

Sometimes output of a page object might depend on the current time. For
example, the item may contain the scraping datetime, or a current timestamp may
be used to build some URLs. When a test runs at a different time it will break.
To avoid this the metadata dictionary can contain a ``frozen_time`` field set
to the time value used when generating the test. This will instruct the test
runner to use the same time value so that field comparisons are still correct.

The value can be any string understood by `dateutil`_. If it doesn't include
timezone information, UTC will be assumed.

.. _dateutil: https://github.com/dateutil/dateutil
