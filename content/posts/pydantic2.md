+++
title = 'Migration to pydantic 2'
date = 2024-09-08T00:00:00+03:00
+++
## What is Pydantic?
Pydantic is the data validation library for Python with JSON Schema generation for data models.

It is used in FastAPI to describe incoming and outgoing data and to generate OpenAPI Schema.

In `Pydantic 1`, there was:
- no strict mod;
- no obvious mechanism to specify a serializer for the model by analogy with a validator;
- quite slow parsing and validation which forced the use of third-party json parsers and serializers, like orjson.

But still it was a widely used and popular tool.

## OpenAPI and JSON Schema
`Open API` uses the `JSON Schema` to describe input and output data.

There are some inconsistencies between `JSON Schema` and `OpenAPI`. For example, `exclusiveMinimum` in `Json Schema` is a number, and in OpenAPI it is a boolean.

Hopefully, these two specifications will synchronize in the future.

## Pydantic 2 architecture
Validation is overwritten in rust and moved to `pydantic-core` package. Python code can be called from rust by writing custom validators and serializers.

Package `pydantic` now contains only python code and its main task now is to generate a scheme for data validation and to pass it the core.

`BaseSettings` is moved to `pydantic-settings` package.

## Pydantic 2 improvements
### Speed
4-50 times faster than version 1, 17 times faster in average ([source](https://docs.pydantic.dev/latest/blog/pydantic-v2/#performance)).

But its more interesting to compare with `orjson` for json parsing and serialization. The following model was used:
```python
class Model(pydantic.BaseModel):
    str: str
    bool: bool
    dt: datetime.datetime
```

1. `model_dump_json` vs `model_dump` + `orjson.dumps`
```python
obj = Model(str="test", bool=True, dt="2021-07-27T16:02:08.070557")
%timeit obj.model_dump_json()
%timeit orjson.dumps(obj.model_dump())

# 1.03 µs ± 1.72 ns per loop (mean ± std. dev. of 7 runs, 1,000,000 loops each)
# 807 ns ± 1.12 ns per loop (mean ± std. dev. of 7 runs, 1,000,000 loops each)
```
2. `model_validate_json` vs `model_validate` + `orjson.loads`
```python
data = '{"str": "test", "bool": true, "dt": "2021-07-27T16:02:08.070557"}'
%timeit Model.model_validate_json(data)
%timeit Model.model_validate(orjson.loads(data))

# 1.11 µs ± 10.1 ns per loop (mean ± std. dev. of 7 runs, 1,000,000 loops each)
# 1.04 µs ± 4.63 ns per loop (mean ± std. dev. of 7 runs, 1,000,000 loops each)
```

3. `pydantic` vs `orjson.dumps`
```python
adapter = pydantic.TypeAdapter(typing.Any)
data = [{"str": "test"}, {"bool": True}, {"dt": "2021-07-27T16:02:08.070557"}]
%timeit adapter.dump_json(data)
%timeit orjson.dumps(data)

# 717 ns ± 2.5 ns per loop (mean ± std. dev. of 7 runs, 1,000,000 loops each)
# 136 ns ± 0.255 ns per loop (mean ± std. dev. of 7 runs, 10,000,000 loops each)
```
4. `pydantic` vs `orjson.loads`
```python
data = '[{"str": "test"}, {"bool": true}, {"dt": "2021-07-27T16:02:08.070557"}]'
adapter = pydantic.TypeAdapter(dict | list)
%timeit adapter.validate_json(data)
%timeit orjson.loads(data)

# 1.01 µs ± 2.86 ns per loop (mean ± std. dev. of 7 runs, 1,000,000 loops each)
# 224 ns ± 0.572 ns per loop (mean ± std. dev. of 7 runs, 1,000,000 loops each)
```
### [Strict mode](https://docs.pydantic.dev/2.0/usage/strict_mode/)
It can be enabled for both the model and the field.

### [Formalised Conversion](https://pydantic.dev/articles/pydantic-v2#formalised-conversion-table)
The main principle is that if a piece of information is lost, an error occurs. For example, float numbers with fractional parts won't be converted to int.

### Other things:
- [serializers](https://docs.pydantic.dev/2.0/usage/serialization/), as methods and using type annotations;
- validators using type annotations;

## Migration tips
1. Try `bump-pydantic` package. It is better to install it globally, rather than in the virtual environment of the project. For me, it was useless.
2. Now `pydantic.BaseSettings` lives in `pydantic-settings` package.
3. Rename all methods for serialization and validation. The old ones are marked as deprecated and will be deleted in pydantic3. For example, `dict` became `model_dump`, etc.
4. You can use v1 directly in v2: `from pydantic.v1 import BaseModel`
5. `parse_raw` and `parse_file` have been removed. Use `model_validate_json` instead
6. `from_orm` method has been removed. Use `model_validate` with the setting `from_attributes=True` in `model_config`. By the way, the model config is now an attribute of the model, not a nested class.
7. `int` is not converted to `str` anymore:
```python
class Test(pydantic.BaseModel):
    n: str
Test(n=1)
# ValidationError: 1 validation error for Test
# n
#   Input should be a valid string [type=string_type, input_value=1, input_type=int]
#     For further information visit https://errors.pydantic.dev/2.1/v/string_type
```
8. `copy.copy` do not work with pydantic1.x-objects. But it is fixed in pydantic2.x, so be careful.
