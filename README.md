# Overview

[//]: # (start-badges)

[![version](https://img.shields.io/pypi/v/marshmallow-configparser.svg)](https://pypi.org/project/marshmallow-configparser/)
[![license](https://img.shields.io/pypi/l/marshmallow-configparser.svg)](https://opensource.org/licenses/MIT)
[![wheel](https://img.shields.io/pypi/wheel/marshmallow-configparser.svg)](https://pypi.org/project/marshmallow-configparser/)
[![python_versions](https://img.shields.io/pypi/pyversions/marshmallow-configparser.svg)](https://pypi.org/project/marshmallow-configparser/)
[![python_implementations](https://img.shields.io/pypi/implementation/marshmallow-configparser.svg)](https://pypi.org/project/marshmallow-configparser/)
[![travis](https://travis-ci.org/tadams42/marshmallow_configparser.svg?branch=master)](https://travis-ci.org/tadams42/marshmallow_configparser)
[![docs](https://readthedocs.org/projects/marshmallow-configparser/badge/?style=flat)](http://marshmallow-configparser.readthedocs.io/en/latest/)
[![requirements](https://requires.io/github/tadams42/marshmallow_configparser/requirements.svg?branch=master)](https://requires.io/github/tadams42/marshmallow_configparser/requirements/?branch=master)
[![codacy_grade](https://api.codacy.com/project/badge/Grade/ad3aa55e2fc74a37a1b1ac2fb59f6dc0)](https://www.codacy.com/app/tadams42/marshmallow_configparser?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=tadams42/marshmallow_configparser&amp;utm_campaign=Badge_Grade)
[![codecov](https://codecov.io/gh/tadams42/marshmallow_configparser/branch/development/graph/badge.svg)](https://codecov.io/gh/tadams42/marshmallow_configparser)

[//]: # (end-badges)

Ever wanted to load plain `.ini` config files and then validate loaded config?

Ever wanted to load config from multiple locations (`/etc/appconfig.conf`, `~/.appconfig.conf`) into single object and then validate that?

Worry no more!

Python's [ConfigParser] met [marshmallow] and now they get along just fine - without any JSON in sight to spoil the fun.

## Installation

~~~sh
pip install marshmallow_configparser
~~~

## Example

Having config file `/tmp/example_config.conf` looking like this:

~~~ini
[Section1]
option1 = mandatory string
option2 = optional string
option3 = 42
option4 = 24

[Section2]
option1 = mandatory string
option2 = optional string
option3 = 42
option4 = 24
~~~

And wanting to load it into our config object:

~~~python
class ConfigObject(object):
    MANDATORY_STRING1 = None
    OPTIONAL_STRING1 = None
    MANDATORY_INTEGER1 = None
    OPTIONAL_INTEGER1 = None
    MANDATORY_STRING2 = None
    OPTIONAL_STRING2 = None
    MANDATORY_INTEGER2 = None
    OPTIONAL_INTEGER2 = None
~~~

We can define [marshmallow] schema:

~~~python
from marshmallow.validate import Range

from marshmallow_configparser import (ConfigBoolean, ConfigInteger,
                                        ConfigParserSchema, ConfigString,
                                        IsNotBlank)

class ConfigSchema(ConfigParserSchema):
    class Meta:
        model = ConfigObject

    MANDATORY_STRING1 = ConfigString(
        section='Section1', load_from='option1', dump_to='option1',
        validate=[IsNotBlank()]
    )
    OPTIONAL_STRING1 = ConfigString(
        section='Section1', load_from='option2', dump_to='option2',
    )
    MANDATORY_INTEGER1 = ConfigInteger(
        section='Section1', load_from='option3', dump_to='option3',
        validate=[Range(min=24, max=42)]
    )
    OPTIONAL_INTEGER1 = ConfigInteger(
        section='Section1', load_from='option4', dump_to='option4',
    )

    MANDATORY_STRING2 = ConfigString(
        section='Section2', load_from='option1', dump_to='option1',
        validate=[IsNotBlank()]
    )
    OPTIONAL_STRING2 = ConfigString(
        section='Section2', load_from='option2', dump_to='option2',
    )
    MANDATORY_INTEGER2 = ConfigInteger(
        section='Section2', load_from='option3', dump_to='option3',
        validate=[Range(min=24, max=42)]
    )
    OPTIONAL_INTEGER2 = ConfigInteger(
        section='Section2', load_from='option4', dump_to='option4',
    )
~~~

Which can then load and validate our config:

~~~python
schema = ConfigSchema()
obj, errors = schema.load(['/tmp/example_config.conf'])
~~~

In the end we have:

~~~python
obj.__dict_

{'MANDATORY_INTEGER1': 42,
    'MANDATORY_INTEGER2': 42,
    'MANDATORY_STRING1': 'mandatory string',
    'MANDATORY_STRING2': 'mandatory string',
    'OPTIONAL_INTEGER1': 24,
    'OPTIONAL_INTEGER2': 24,
    'OPTIONAL_STRING1': 'optional string',
    'OPTIONAL_STRING2': 'optional string'}
~~~

Instead of using convenience classes like `ConfigString`, there are also classes in `marshmallow_configparser.fields` module that expose full [marshmallow] API. Check the docs for details.

## Documentation

http://marshmallow-configparser.readthedocs.io/en/latest/index.html

[marshmallow]: https://github.com/marshmallow-code/marshmallow
[ConfigParser]: https://docs.python.org/3/library/configparser.html#configparser.ConfigParser
