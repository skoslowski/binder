Configuration
#############

Binder provide two way to supply configiration options: command-line and config file.



Command line options
====================

`--root-module` specify name of generatred Python root module. This name is also used as prefix for various Binder output
files. Typically the following files will be generated: ``<root-module>.cpp``, ``<root-module>.sources``,
``<root-module>.modules``.


`--max-file-size` specify maximum file size in bytes exiding which Binder will split generated sources into multiple files.


`--prefix` name/path prefix for generated files.


`--bind` list of namespaces that need to binded. Works in conjunction with similar config file directives.


`--skip` list of namespaces that should be skipped. Works in conjunction with similar config file directives.


`--config` specify config file to use.


`--single-file` if specified instruct Binder to put generated sources into single large file. This might be useful for small projects.


`--annotate-includes` [debug] if specified Binder will comment each include with type name which trigger it inclusion.


`--trace` [debug] if specified instruct Binder to add extra debug output before binding each type. This might be useful when debugging generated code that produce seg-faults during python import.



Config file options
===================

Config file is text file containing either comment-line (starts with #) or dirctive line stated with either ``+`` or ``-`` signs
following by a directives name and optional prameters. Some directives will accept only the ``+`` while other could be used with
both prefixes.

Config file directives:
-----------------------

* ``namespace``, specify if functions/classes/enums from particular namespace should be bound. Could be used with both ``+`` and ``-``
  prefixes. This directives work recursivly so for example if you specify ``+namespace root`` and later ``-namespace root::a`` then
  all objects in ``root`` will be bound with exception of ``root::a`` and it decedents.

.. code-block:: bash

  -namespace boost
  +namespace utility


* ``class``, specify if particular class/struct should be bound. Purpose of this directive is to allow developer to cherry-pick
  particular class from other wise binded/skipped namespaces and mark it for binding/skipping.

.. code-block:: bash

  -class utility::pointer::ReferenceCount
  -class std::__weak_ptr


* ``function``, specify if particular function should be bound. This could be used for both template and normal function.

.. code-block:: bash

  -function ObjexxFCL::FArray<std::string>::operator-=
  -function core::id::swap



* ``include``, directive to control C++ include directives. Force Binder to either skip adding particular include into generated
  source files (``-`` prefix) or force Binder to always add some include files into each generated file. Normally Binder could
  automatically determent which C++ header files is needed in orger to specify type/functions but in some cases it might be
  useful to be able to control this process. For exampel forcing some includes is particularly useful when you want to provide
  custom-binder-functions with either ``+binder`` or ``+add_on_binder`` directives.

.. code-block:: bash

  -include <boost/format/internals.hpp>
  +include <python/PyRosetta/binder/stl_binders.hpp>


* ``binder``, specify custom binding function for particular concreate or template class. In the example below all
  specializations of tempalte std::vector will be handled by ``binder::vector_binder`` function. For template classes binder
  function should be a template function taking the same number of types as original type and having the follwoing type
  signature: pybind11 module, then std::string for each template argument provided. So for ``std::vector`` it will be:

.. code-block:: c++

  template <typename T, class Allocator>
  vector_binder(pybind11::module &m, std::string const &name, std::string const & /*allocator name*/) {...}


* ``+add_on_binder``, similar to ``binder``: specify custom binding function for class/struct that will be called `after` Binder
  generated code bound it. This allow developer to create extra bindings for particular type (bind special Python methos,
  operators, etc.)

.. code-block:: bash

  +binder std::vector my_binders::vector_binder
  +binder std::map    my_binders::map_binder

  +add_on_binder numeric::xyzVector rosetta_binders::xyzVector_add_on_binder




* ``default_static_pointer_return_value_policy``, specify return value policy for static functions returning pointer to objects. Default is
  'pybind11::return_value_policy::automatic'.


* ``default_static_lvalue_reference_return_value_policy``, specify return value policy for static functions returning l-value reference. Default
  is 'pybind11::return_value_policy::automatic'.


* ``default_static_rvalue_reference_return_value_policy``, specify return value policy for static functions returning r-value reference. Default
  is 'pybind11::return_value_policy::automatic'.


* ``default_member_pointer_return_value_policy``, specify return value policy for member functions returning pointer to objects. Default is
  'pybind11::return_value_policy::automatic'.


* ``default_member_lvalue_reference_return_value_policy``, specify return value policy for member functions returning l-value reference. Default
  is 'pybind11::return_value_policy::automatic'.


* ``default_member_rvalue_reference_return_value_policy``, specify return value policy for member functions returning r-value reference. Default
  is 'pybind11::return_value_policy::automatic'.





.. code-block:: bash

  +default_pointer_return_value_policy           pybind11::return_value_policy::reference
  +default_lvalue_reference_return_value_policy  pybind11::return_value_policy::reference_internal
  +default_rvalue_reference_return_value_policy  pybind11::return_value_policy::move
