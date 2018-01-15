# jsonapi-requests

[![Build Status](https://travis-ci.org/socialwifi/jsonapi-requests.svg?branch=master)](https://travis-ci.org/socialwifi/jsonapi-requests)
[![Coverage Status](https://coveralls.io/repos/github/socialwifi/jsonapi-requests/badge.svg)](https://coveralls.io/github/socialwifi/jsonapi-requests)
[![Latest Version](https://img.shields.io/pypi/v/jsonapi-requests.svg)](https://pypi.python.org/pypi/jsonapi-requests/)
[![Supported Python versions](https://img.shields.io/pypi/pyversions/jsonapi-requests.svg)](https://pypi.python.org/pypi/jsonapi-requests/)
[![Wheel Status](https://img.shields.io/pypi/wheel/jsonapi-requests.svg)](https://pypi.python.org/pypi/jsonapi-requests/)
[![License](https://img.shields.io/pypi/l/jsonapi-requests.svg)](https://github.com/socialwifi/jsonapi-requests/blob/master/LICENSE)

Python client implementation for json api. http://jsonapi.org/

----
## Usage example

```python
import jsonapi_requests

api = jsonapi_requests.Api.config({
    'API_ROOT': 'https://localhost/api/2.0',
    'AUTH': ('basic_auth_login', 'basic_auth_password'),
    'VALIDATE_SSL': False,
    'TIMEOUT': 1,
})

endpoint = api.endpoint('networks/cd9c124a-acc3-4e20-8c02-3a37d460df22/available-profiles')
response = endpoint.get()

for profile in response.data:
    print(profile.attributes['name'])
# Example output: "162 Sushi"

endpoint = api.endpoint('cookies')
endpoint.post(object=jsonapi_requests.JsonApiObject(
    type='cookies',
    attributes={
        'uuid': '09d3a4fff8d64335a1ee9f1d9d054161', 
        'domain': 'some.domain.pl'
    },
))
# Example output: <ApiResponse({'data': {'id': '81', 'attributes': {'uuid': '09d3a4fff8d64335a1ee9f1d9d054161', 'domain': 'some.domain.pl'}, 'type': 'cookies'}})>
```

## Orm example

Lets say we have api endpoint: `https://localhost/api/2.0/car/2`
which returns

```json
{
    "data":{
        "id": 2,
        "type": "car",
        "attributes": {
            "color": "red"
        },
        "relationships": {
            "driver": {
                "data": {
                    "id": 3, 
                    "type": "person"
                }
            }
        }
    },
    "included": [
        {
            "id": 3,
            "type": "person",
            "attributes": {
                "name": "Kowalski"
            },
            "relationships": {
                "married-to": {
                    "data": {
                        "id": 4, 
                        "type": "person"
                    }
                }
            }
        },
        {
            "id": 4,
            "type": "person",
            "attributes": {
                "name": "Kowalska"
            },
            "relationships": {
                "married-to": {
                    "data": {
                        "id": 3, 
                        "type": "person"
                    }
                }
            }
        },
    ]
}
```

Then we can run:

```python
import jsonapi_requests

api = jsonapi_requests.orm.OrmApi.config({
    'API_ROOT': 'https://localhost/api/2.0',
    'AUTH': ('basic_auth_login', 'basic_auth_password'),
    'VALIDATE_SSL': False,
    'TIMEOUT': 1,
})

class Person(jsonapi_requests.orm.ApiModel):
    class Meta:
        type = 'person'
        api = api

    name = jsonapi_requests.orm.AttributeField('name')
    married_to = jsonapi_requests.orm.RelationField('married-to')

class Car(jsonapi_requests.orm.ApiModel):
    class Meta:
        type = 'car'
        api = api

    color = jsonapi_requests.orm.AttributeField('color')
    driver = jsonapi_requests.orm.RelationField('driver')

car = Car.from_id(2)

car.color # request happens here
# Example output: 'red'

car.driver.name
# Example output:  'Kowalski'

car.driver.married_to.name
# Example output: 'Kowalska'

car.driver.married_to.married_to.name
# Example output: 'Kowalski'
```

### Custom Field Types

It's possible to use a custom type for a field by subclassing `jsonapi_requests.orm.AttributeField` and overriding `serialize` and `deserialize`.

Example:

    from uuid import UUID

    from jsonapi_requests import orm


    class UuidField(orm.AttributeField):
        def serialize(self, value):
            # take a UUID object and make it JSON serializable
            return str(value)

        def deserialize(self, raw_value):
            # take the JSON value (a string in this case) and turn it into a UUID
            return UUID(raw_value)


    class Car(jsonapi_requests.orm.ApiModel):
        class Meta:
            type = 'car'

        guid = UuidField('guid')


    car = Car()
    car.guid = UUID('4cbecc77-fc43-4d29-8915-222d87029ec3')
    car.save()


## Authorization HTTP header forwarding in Flask application

When using jsonapi\_requests with Flask, we can set `jsonapi_requests.auth.FlaskForwardAuth()` as `AUTH` configuration option to copy authorization header from current request context.
It can be useful when fetching resources from different microservices.

Installation with flask support:

```bash
pip install jsonapi-requests[flask]
```

Example usage:

```python
import jsonapi_requests

api = jsonapi_requests.Api.config({
    'API_ROOT': 'https://localhost/api/2.0',
    'AUTH': jsonapi_requests.auth.FlaskForwardAuth(),
})
```

## Documentation
For more documentation check our [wiki](https://github.com/socialwifi/jsonapi-requests/wiki).
