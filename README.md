# linkml-multivalued-default

An experiment in generating an empty list default for multivalued slots in a
LinkML model.

## Background

It is difficult or impossible in LinkML to specify that you want an empty list
as the default value for a slot that is both `required` and `multivalued`. This
repository contains some materials to help demonstrate this, as well as a
workaround.

For more context, see the following issues:
- https://github.com/dandi/dandi-archive/issues/2150
- https://github.com/dandi/dandi-schema/issues/286

## Setup

1. If you are using Nix flakes, run `nix develop` to install dependencies.
   Otherwise, ensure you have a modern Python installed (e.g., Python 3.10).

2. Create a virtual environment with `python -m venv venv` and activate it with
   `. ./venv/bin/activate`.

3. Install the Python dependencies with `pip install -r requirements.txt`.

## Demonstration of non-optional list types with default value.

1. Generate a Pydantic model by running `gen-pydantic personinfo_busted.yaml >personinfo_busted.py`.
   Note that setting both `multivalued=true` and `required=true` correctly
   infers the type of the `aliases3` field as `List[str]` (rather than
   `Optional[List[str]]` as for `aliases2`, which is not required), but that it
   is not possible to set a default value of `[]` directly (the Pydantic
   generator does not have a way to encode list values in the `ifabsent`
   attribute; attempts to do so generate the string `"[]"`).

2. Generate another Pydantic model by running `gen-pydantic
   personinfo_workaround.yaml >personinfo_workaround.py`. Note that this model
   sets a globally unique and incorrectly typed default on `aliases3`.

3. Run `gen-pydantic personinfo_workaround.yaml | sed s/\"aliases3dummy\"/[]/ >personinfo_workaround.py`.
   Note that now the definition of `aliases3` correctly has an empty list
   default value.

4. Fire up Python and run this script:

   ```python
   from personinfo_workaround import Person
   p = Person(name2='Ada Byron', aliases2=['Lady Byron'])
   p
   ```

   Note that `p` contains the expected values for all fields, including a
   default `[]` for `aliases3`.

The need to post-process like this is unfortunate, but seems to be necessary
due to unresolved issues of defaults for multivalued slots in LinkML
([[1]](https://github.com/orgs/linkml/discussions/1975),
[[2]](https://github.com/orgs/linkml/discussions/2314)).

## Conclusion

It *is* possible to generate correct Pydantic models from LinkML schemata
featuring **required**, **multivalued** attributes **with a list-valued
default**, but it currently requires a transformation of the generated code.
