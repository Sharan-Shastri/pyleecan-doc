##################
Test contribution
##################

The only way to ensure that the implemented code is working and will always do, is to develop validation test. This page will
present how to develop test for PYLEECAN modules

We invite everyone contributing to the project to systematically add tests to all their contributions if possible.

Which test to develop
----------------------

An easy way to find a PYLEECAN part that need to be tested is to use `Coverage <https://coverage.readthedocs.io/en/v4.5.x/>`__.
It is a python package that enables to see which lines in the code are not executed by the existing tests. It runs all the
tests and build a report listing all the code line not tested.

Follow these steps to see the code not covered by test:

- To install coverage:

    ::

            python -m pip install -U coverage

- To launch all the tests:

    ::

            coverage run -m unittest discover ./pyleecan/Tests -t . -b

- To see the coverage report, run the following command and replace "directory_path" with the path of the directory you wish to have the report in.

    ::

            coverage html -d directory_path

Then open the index.html file:
.. image:: _static/coverage_report.png

**With the report, you will see which files and which code lines are not covered and find what to test next.**


.. image:: _static/coverage.png

**For example the Arc3 method discretize is not covered at 100%, there are some lines not covered as line 40 and 42
colored in pink**

.. image:: _static/coverage1.png

In this case, there is no test to check that the discretization can handle strange arguments.

Test development guidelines
----------------------------

The python test package currently used is `unittest <https://docs.python.org/3/library/unittest.html#module-unittest>`__,
but for future evolution we plan to use `pytest <https://docs.pytest.org/en/latest/>`__ package. To test a code, it is
recommended to develop its specific test code for each method and each class.

Class tests
````````````

PYLEECAN classes are generated automatically, all the classes are built in the same way. This fact enables us
to test all the classes with one test file which you can find in **Tests/Classes**. So you won't need to develop test for
the class it self but you will have to develop the tests for the methods defined in the :ref:`class.generation:Class creation`.

All the methods tests are gathered in **Tests/Methods/<subfolder>** with **Subfolder** the type of the class (Geometry,
Machine, Slot, etc.).

How do we write unittest
``````````````````````````

To write unittest, you may need to create a class which inherits to `TestCase <https://docs.python.org/3/library/unittest.html#unittest.TestCase>`__
from `unittest <https://docs.python.org/3/library/unittest.html#module-unittest>`__ python package.

Here is a basic example of unittest

.. code-block:: python

    from unittest import TestCase

    class TestStringMethods(TestCase):

        def test_upper(self):
            self.assertEqual('foo'.upper(), 'FOO')

        def test_isupper(self):
            self.assertTrue('FOO'.isupper())
            self.assertFalse('Foo'.isupper())

        def test_split(self):
            s = 'hello world'
            self.assertEqual(s.split(), ['hello', 'world'])
            # check that s.split fails when the separator is not a string
            with self.assertRaises(TypeError):
                s.split(2)

    if __name__ == '__main__':
        unittest.main()

To go much further and test more cases on a unittest we use the `ddt <https://ddt.readthedocs.io/en/latest/example.html>`__
python package, which is a class decorator to multiply test. It enables to test the same code with different parameters (to
avoid duplicating code).

Here an example of ddt use

.. code-block:: python

    from ddt import ddt, data
    from numpy import exp

    # For AlmostEqual
    DELTA = 1e-6

    comp_length_test = list()
    comp_length_test.append(
        {"begin": 0, "end": 14.1421356237 * exp(1j * pi / 4), "length": 14.1421356237}
    )
    comp_length_test.append({"begin": 0, "end": 10, "length": 10})
    comp_length_test.append({"begin": 1, "end": 10, "length": 9})
    comp_length_test.append({"begin": 1j, "end": 10j, "length": 9})
    comp_length_test.append({"begin": 0, "end": 3 + 4j, "length": 5})

    @ddt
    class Test_Segment_meth(TestCase):
        """unittest for Segment methods"""

        @data(*comp_length_test)
        def test_comp_length(self, test_dict):
            """Check that you the length return by comp_length is correct
            """
            segment = Segment(test_dict["begin"], test_dict["end"])

            self.assertAlmostEqual(segment.comp_length(), test_dict["length"])
