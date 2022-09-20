# dbt-beautiful-snowflake-utils
This package provides various additional utilities for dbt usage on Snowflake, such as macros for utilizing Snowflake-specific features.

# Package Installation

# Package Macro Usage

# Available Macros
# How to add this package to your DBT Project
Simply add the following to your project's `package.yml` file:
```yaml
packages:
  git: "https://github.com/Armalite/dbt-beautiful-snowflake-utils.git"
  revision: 0.0.1
```
Then ensure you run `dbt deps` to install the package

# How to use this package
## Using supplied macros
You can reference the macros supplied in this package by calling the macro name as usual, prefixed with `beautiful_snowflake_utils.`

For example, to call the `hello_beautiful()` macro in your DBT project, you can call it in your SELECT statement as such:
```sql
SELECT beautiful_snowflake_utils.hello_beautiful()
```

# Available Macros

## Macro: hello_beautiful
<details>
<summary> Click to expand</summary>
  
#### Description
This is a demo macro that takes in no parameters and returns a `Hello Beautiful!` message.
  
#### Arguments
None
  
#### Usage
```jinja
beautiful_snowflake_utils.hello_beautiful()
```
</details>

## Macro: apply_snowflake_tag 

<details>
<summary> Click to expand</summary>

#### Description
This applies a Snowflake tag to a Snowflake object. The object can be a table/view/column etc.
 - The Snowflake tag object must already exist in Snowflake, and be in a accessible location by the user/role that DBT will assume when running
 - The user/role/service account that DBT will assume when running, must have APPLY privilege to this tag

#### Arguments
 - `parent_object_type`: The table/view level object. For tagging tables/views, this will be the same value as `target_object_type`. For tagging columns, this field will determine whether the column belongs to a table or view. Accepted values: `TABLE`, `VIEW`
 - `target_object_type`: The type of the Snowflake object that will be tagged. Accepted values: `TABLE`, `VIEW`, `COLUMN`)
 - `target_object_name`: The full name of the Snowflake object. For tables/views this is a 3 part name: DATABASE.SCHEMA.TABLE. For column this must be a 4-part name: DATABASE.SCHEMA.TABLE.COLUMN
 - `full_tag_name`: The fully qualified tag name (e.g. `MYDB.MYSCHEMA.some_tag`)
 - `tag_value`: The case-sensitive value to set for the tag. If the tag has limited allowed values, this will fail if your value does not meet the criteria.

#### Usage
##### Basic macro call for table tagging
```jinja
apply_snowflake_tag('TABLE', 'TABLE', 'MYPRODUCT_SANDBOX.SOME_SCHEMA.A_COOL_TABLE', 'MYDB.MYSCHEMA.some_tag', 'MyTagValue')
```

##### Basic macro call for column tagging
```jinja
apply_snowflake_tag('TABLE', 'COLUMN', 'MYPRODUCT_SANDBOX.SOME_SCHEMA.A_COOL_TABLE.THIS_IS_A_COLUMN', 'MYDB.MYSCHEMA.some_other_tag', 'MyTagValue')
```

##### In a on-run-end hook
You can add a on-run-end hook, in your `dbt_project.yml` file for example, to ensure tables are tagged after creation, using meta configs that you supply in your models:
```jinja
on-run-end:
  - "{% if execute and flags.WHICH in ('run') and target.name in ('sandbox_dev') %} {% for r in results %} {{ apply_snowflake_tag(r.node.database ~ '.' ~ r.node.schema ~ '.' ~ r.node.name, 'TABLE', 'MYDB.MYSCHEMA.some_tag', r.node.config.meta.get('my_tag_meta')) }} {% endfor %} {% endif %}"
```
With the above example, you can have the following config in your `model/` files to supply the meta values:
```jinja
{{ config(
  meta = {
            "my_tag_meta": "MyTagValue",
          }
)
```
</details>
