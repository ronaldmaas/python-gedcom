# Python GEDCOM Parser

[![PyPI](https://img.shields.io/pypi/v/python-gedcom.svg)](https://pypi.org/project/python-gedcom/)
[![GitHub release](https://img.shields.io/github/release/nickreynke/python-gedcom.svg)](https://github.com/nickreynke/python-gedcom/releases)
[![Build Status](https://travis-ci.org/nickreynke/python-gedcom.svg?branch=master)](https://travis-ci.org/nickreynke/python-gedcom)
![](https://img.shields.io/badge/GEDCOM%20format%20version-5.5-yellowgreen.svg)
![](https://img.shields.io/badge/Python%20versions-3.5%20to%203.8-yellowgreen.svg)

A Python module for parsing, analyzing, and manipulating GEDCOM files.

GEDCOM files contain ancestry data. The parser is currently supporting
the GEDCOM 5.5 format which is detailed [here](https://chronoplexsoftware.com/gedcomvalidator/gedcom/gedcom-5.5.pdf).

> For the latest changes please have a look at the [`CHANGELOG.md`](CHANGELOG.md) file.
>
> The current development process can be tracked in the [develop branch](https://github.com/nickreynke/python-gedcom/tree/develop).

## Installation

The module can be installed via [pipenv](https://github.com/pypa/pipenv) or simply [pip](https://pip.pypa.io/).

Run `pip<version> install python-gedcom` to install or `pip<version> install python-gedcom --upgrade`
to upgrade to the newest version uploaded to the [PyPI repository](https://pypi.org/project/python-gedcom/).

If you want to use the latest pre-release of the `python-gedcom` package,
simply append the `--pre` option to `pip`: `pip<version> install python-gedcom --pre`

> Tip: Using [pipenv](https://github.com/pypa/pipenv) simplifies the installation and maintenance of dependencies.

## Changelog

Please have a look at the [`CHANGELOG.md`](CHANGELOG.md) file.

## Example usage

> **For more examples:**
> Please have a look at the test files found in the `tests/` directory.

When successfully installed you may import the `gedcom` package and use it like so:

```python
from gedcom.element.individual import IndividualElement
from gedcom.parser import Parser

# Path to your `.ged` file
file_path = ''

# Initialize the parser
gedcom_parser = Parser()

# Parse your file
gedcom_parser.parse_file(file_path)

root_child_elements = gedcom_parser.get_root_child_elements()

# Iterate through all root child elements
for element in root_child_elements:

    # Is the `element` an actual `IndividualElement`? (Allows usage of extra functions such as `surname_match` and `get_name`.)
    if isinstance(element, IndividualElement):

        # Get all individuals whose surname matches "Doe"
        if element.surname_match('Doe'):

            # Unpack the name tuple
            (first, last) = element.get_name()

            # Print the first and last name of the found individual
            print(first + " " + last)
```

## Strict parsing

Large sites like Ancestry and MyHeritage (among others) don't always produce perfectly formatted GEDCOM files.
If you encounter errors in parsing, you might consider disabling strict parsing which is enabled by default:

```python
from gedcom.parser import Parser

file_path = '' # Path to your `.ged` file

gedcom_parser = Parser()
gedcom_parser.parse_file(file_path, False) # Disable strict parsing
```

Disabling strict parsing will allow the parser to gracefully handle the following quirks:

- Multi-line fields that don't use `CONC` or `CONT`
- Handle the last line not ending in a CRLF (`\r\n`)

## Reference

> **Note**: At a later state the documentation may be outsourced into individual, automatically generated wiki pages.
> (Makes things a little bit easier.)

### `Parser` class

The `Parser` class represents the actual parser. Use this class to parse a GEDCOM file.

> **Note**: May be imported via `from gedcom.parser import Parser`.

Method | Parameters | Returns | Description
-------|------------|---------|------------
`invalidate_cache` | | | Empties the element list and dictionary to cause `get_element_list()` and `get_element_dictionary()` to return updated data
`get_element_list` | | `list` of `Element` | Returns a list containing all elements from within the GEDCOM file
`get_element_dictionary` | | `dict` of `Element` | Returns a dictionary containing all elements, identified by a pointer, from within the GEDCOM file
`get_root_element` | | `RootElement` | Returns a virtual root element containing all logical records as children
`get_root_child_elements` | | `list` of `Element` | Returns a list of logical records in the GEDCOM file
`parse_file` | `str` file_path, `bool` strict | | Opens and parses a file, from the given file path, as GEDCOM 5.5 formatted data
`get_marriages` | `IndividualElement` individual | `tuple`: (`str` date, `str` place) | Returns a list of marriages of an individual formatted as a tuple (`str` date, `str` place)
`get_marriage_years` | `IndividualElement` individual | `list` of `int` | Returns a list of marriage years (as integers) for an individual
`marriage_year_match` | `IndividualElement` individual, `int` year | `bool` | Checks if one of the marriage years of an individual matches the supplied year. Year is an integer.
`marriage_range_match` | `IndividualElement` individual, `int` from_year, `int` to_year | `bool` | Check if one of the marriage years of an individual is in a given range. Years are integers.
`get_families` | `IndividualElement` individual, `str` family_type = `gedcom.tags.GEDCOM_TAG_FAMILY_SPOUSE` | `list` of `FamilyElement` | Return family elements listed for an individual
`get_ancestors` | `IndividualElement` individual, `str` ancestor_type = `"ALL"` | `list` of `Element` | Return elements corresponding to ancestors of an individual
`get_parents` | `IndividualElement` individual, `str` parent_type = `"ALL"` | `list` of `IndividualElement` | Return elements corresponding to parents of an individual
`find_path_to_ancestor` | `IndividualElement` descendant, `IndividualElement` ancestor, `path` = `None` | `object` | Return path from descendant to ancestor
`get_family_members` | `FamilyElement` family, `str` members_type = `FAMILY_MEMBERS_TYPE_ALL` | `list` of `IndividualElement` | Return array of family members: individual, spouse, and children
`print_gedcom` | | | Write GEDCOM data to stdout
`save_gedcom` | `IO` open_file | | Save GEDCOM data to a file

### `Element` class

An element represents a line from within the parsed GEDCOM file.

May be imported via `from gedcom.element.element import Element`.

Method | Parameters | Returns | Description
-------|------------|---------|------------
`get_level` | | `int` | Returns the level of this element from within the GEDCOM file
`get_pointer` | | `str` | Returns the pointer of this element from within the GEDCOM file
`get_tag` | | `str` | Returns the tag of this element from within the GEDCOM file
`get_value` | | `str` | Returns the tag of this element from within the GEDCOM file
`set_value` | `str` value  | `str` | Sets the value of this element
`get_multi_line_value` | | `str` | Returns the value of this element including concatenations or continuations
`set_multi_line_value` | `str` value | `str` | Sets the value of this element, adding concatenation and continuation lines when necessary
`get_child_elements` | | `list` of `Element` | Returns the direct child elements of this element
`new_child_element` | `str` tag, `str` pointer = `""`, `str` value = `""` | `Element` | Creates and returns a new child element of this element
`add_child_element` | `Element` child | `Element` | Adds a child element to this element
`get_parent_element` | | `Element` | Returns the parent element of this element
`set_parent_element` | `Element` parent | | Adds a parent element to this element. There's usually no need to call this method manually, `add_child_element()` calls it automatically.
`get_individual` | | `str` | **DEPRECATED**: As of version `v1.0.0` use `to_gedcom_string()` method instead.
`to_gedcom_string` | `bool` recursive = `False` | `str` | Formats this element and optionally all of its sub-elements into a GEDCOM conform string

> Casting an `Element` to a string will internally call the `to_gedcom_string()` method.

#### `FamilyElement` class (derived from `Element`)

May be imported via `from gedcom.element.family import FamilyElement`.

Method | Parameters | Returns | Description
-------|------------|---------|------------
`is_family` | | `bool` | Checks if this element is an actual family

#### `FileElement` class (derived from `Element`)

May be imported via `from gedcom.element.file import FileElement`.

Method | Parameters | Returns | Description
-------|------------|---------|------------
`is_file` | | `bool` | Checks if this element is an actual file

#### `IndividualElement` class (derived from `Element`)

Represents a person from within the parsed GEDCOM file.

May be imported via `from gedcom.element.individual import IndividualElement`.

Method | Parameters | Returns | Description
-------|------------|---------|------------
`is_individual` | | `bool` | Checks if this element is an actual individual
`is_deceased` | | `bool` | Checks if this individual is deceased
`is_child` | | `bool` | Checks if this element is a child of a family
`is_private` | | `bool` | Checks if this individual is marked private
`get_name` | | `tuple`: (`str` given_name, `str` surname) | Returns an individual's names as a tuple: (`str` given_name, `str` surname)
`surname_match` | `str` surname_to_match | `bool` | Matches a string with the surname of an individual
`given_name_match` | `str` given_name_to_match | `bool` | Matches a string with the given names of an individual
`get_gender` | | `str` | Returns the gender of a person in string format
`get_birth_data` | | `tuple`: (`str` date, `str` place, `list` sources) | Returns the birth data of a person formatted as a tuple: (`str` date, `str` place, `list` sources)
`get_birth_year` | | `int` | Returns the birth year of a person in integer format
`get_death_data` | | `tuple`: (`str` date, `str` place, `list` sources) | Returns the death data of a person formatted as a tuple: (`str` date, `str` place, `list` sources)
`get_death_year` | | `int` | Returns the death year of a person in integer format
`get_burial_data` | | `tuple`: (`str` date, `str` place, `list` sources) | Returns the burial data of a person formatted as a tuple: (`str` date, `str´ place, `list` sources)
`get_census_data` | | `list` of `tuple`: (`str` date, `str` place, `list` sources) | Returns a list of censuses of an individual formatted as tuples: (`str` date, `str´ place, `list` sources)
`get_last_change_date` | | `str` | Returns the date of when the person data was last changed formatted as a string
`get_occupation` | | `str` | Returns the occupation of a person
`birth_year_match` | `int` year | `bool` | Returns `True` if the given year matches the birth year of this person
`birth_range_match` | `int` from_year, `int` to_year | `bool` | Checks if the birth year of an individual lies within the given range
`death_year_match` | `int` year | `bool` | Returns `True` if the given year matches the death year of this person
`death_range_match` | `int` from_year, `int` to_year | `bool` | Returns `True` if the given year matches the death year of this person
`criteria_match` | `str` criteria | `bool` | Checks if this individual matches all of the given criteria. Full format for `criteria`: `surname=[name]:given_name=[given_name]:birth[year]:birth_range=[from_year-to_year]`

#### `ObjectElement` class (derived from `Element`)

May be imported via `from gedcom.element.object import ObjectElement`.

Method | Parameters | Returns | Description
-------|------------|---------|------------
`is_object` | | `bool` | Checks if this element is an actual object

#### `RootElement` class (derived from `Element`)

Virtual GEDCOM root element containing all logical records as children.

## Local development

Local development is done using [pyenv](https://github.com/pyenv/pyenv) and
[pipenv](https://github.com/pypa/pipenv).

### Running tests

1. Run `pipenv install` to install dependencies
1. Run tests with [tox](https://tox.readthedocs.io/en/latest/index.html) (`pipenv run tox` in your console)
    * For Python 3.5 run `pipenv run tox -e py35` (you need to have Python 3.5 installed)
    * For Python 3.6 run `pipenv run tox -e py36` (you need to have Python 3.6 installed)
    * For Python 3.7 run `pipenv run tox -e py37` (you need to have Python 3.7 installed)
    * For Python 3.8 run `pipenv run tox -e py38` (you need to have Python 3.8 installed)

### Uploading a new package to PyPI

1. Run `pipenv install` to install dependencies
1. Run `pipenv run python3 setup.py sdist bdist_wheel` to generate distribution archives
1. Run `pipenv run twine upload --repository-url https://test.pypi.org/legacy/ dist/*` to upload the archives to the Test Python Package Index repository

> When the package is ready to be published to the real Python Package Index
the `repository-url` is `https://upload.pypi.org/legacy/`.
>
> `pipenv run twine upload --repository-url https://upload.pypi.org/legacy/ dist/*`

## History

This module was originally based on a GEDCOM parser written by 
Daniel Zappala at Brigham Young University (Copyright (C) 2005) which
was licensed under the GPL v2 and then continued by
[Mad Price Ball](https://github.com/madprime) in 2012.

The project was taken over by [Nicklas Reincke](https://github.com/nickreynke) in 2018.
Together with [Damon Brodie](https://github.com/nomadyow) a lot of changes were made and the parser was optimized.

## License

Licensed under the [GNU General Public License v2](http://www.gnu.org/licenses/gpl-2.0.html)

**Python GEDCOM Parser**
<br>Copyright (C) 2018 Damon Brodie (damon.brodie at gmail.com)
<br>Copyright (C) 2018-2019 Nicklas Reincke (contact at reynke.com)
<br>Copyright (C) 2016 Andreas Oberritter
<br>Copyright (C) 2012 Madeleine Price Ball
<br>Copyright (C) 2005 Daniel Zappala (zappala at cs.byu.edu)
<br>Copyright (C) 2005 Brigham Young University

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License along
with this program; if not, write to the Free Software Foundation, Inc.,
51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
