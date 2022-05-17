# Simple JSON Parser in C

## Features

 * Simple interface
   * `json_parse`: Parse a JSON string into a `json_object_t *`
   * `json_print`: Print a `json_object_t *` using specified indentation
   * `json_free`: Free a `json_object_t *` from memory
 * Support for maximum data types
   * `String`
   * `Number`
   * `Object`
   * `Array`
   * `Boolean`
   * `Null` _(omitted from end result)_
 * Systematically structured data types
   * Basic JSON types:
     * String: `json_string_t` (Sugar for `const char *`)
     * Number: `json_number_t` (Sugar for `double`)
     * Object: `json_object_t` (Array of `json_entry_t`)
     * Array: `json_array_t` (Typed array of `json_value_t`)
     * Boolean: `json_boolean_t` (Sugar for `unsigned char`)
   * Tag type: `json_type_t` with possible values
     * `JSON_TYPE_STRING`
     * `JSON_TYPE_NUMBER`
     * `JSON_TYPE_OBJECT`
     * `JSON_TYPE_ARRAY`
     * `JSON_TYPE_BOOLEAN`
     * `JSON_TYPE_NULL` _(only as an indicator)_
   * Key-Value Pair: `json_entry_t`
     * `type`: Enum `json_type_t` tag
     * `value`: Union `json_value_t` value
   * Union: `json_value_t` with easy to interpret fields
     * `as_string`: As `json_string_t` value
     * `as_number`: As `json_number_t` value
     * `as_object`: As `json_object_t *` value
     * `as_array`: As `json_array_t *` value
     * `as_boolean`: As `json_boolean_t` value
 * **Value or Error** (Rust like) `result` type used throughout fallible calls
 * Recursive parsing
 * Compile with `-DJSON_SCRAPE_WHITESPACE` to parse non-minified JSON with whitespace in between

## Setup

Copy the following from this repository in your source

 * `json.h`
 * `json_types.h`
 * `json.c`

And in your code

```C
#include "json.h"
```

## Usage

### Example

 1. Clone this repository
 2. Compile the example `clang example.c json.c -o example.out`
 3. Run the binary `./example.out`

### Basic

```C
#include "json.h"

const char * some_json_string = "{\"hello\":\"world\",\"key\":\"value\"}";

int main() {
  json_object_t *json = json_parse(some_json_string);
  printf("Count: %i\n", json->count);
  printf("Key: %s\nValue: %s", json->entries[0].type, json->entries[0].value.as_string);
  return 0;
}
```
Will output

```
Count: 2
Key: hello
Value: world
```

#### Recursive

```C
#include "json.h"

const char * complex_json = "{\"name\":{\"first\":\"John\",\"last\":\"Doe\"},\"age\":21.5}";

int main() {
  json_object_t *json = json_parse(complex_json);
  json_object_t *name_json = json->entires[0].value.as_object;
  printf("First name: %s\nLast name: %s\nAge: %f",
    name_json->entries[0].value.as_string,
    name_json->entries[1].value.as_string,
    json->entries[1].value.as_number);
  return 0;
}
```
Will output

```
First name: John
Last name: Doe
Age: 21.5
```

## FAQs

### How to know the type?

At each Key-Value pair `json_entry_t`, there is a member `type`

```C
#include "json.h"

...

json_object_t *json = json_parse(some_json_string);
json_entry_t entry = json->entries[0];
switch(entry.type) {
  case JSON_TYPE_STRING:
    // `entry.value.as_string` is a `json_string_t`
    break;
  case JSON_TYPE_NUMBER:
    // `entry.value.as_number` is a `json_number_t`
    break;
  case JSON_TYPE_OBJECT:
    // `entry.value.as_object` is a `json_object_t *`
    break;
  case JSON_TYPE_ARRAY:
    // `entry.value.as_array` is a `json_array_t *`
    break;
  case JSON_TYPE_BOOLEAN:
    // `entry.value.as_boolean` is a `json_boolean_t`
    break;
}
```

### How to get the count of number of Key-Value pairs?

In each `json_object_t`, there is a member `count`

```C
#include "json.h"

...

int i;
json_object_t *json = json_parse(some_json_string);
for(i = 0; i < json->count; i++) {
  json_entry_t entry = json->entries[i];

  json_string_t key = entry.key;
  json_type_t type = entry.type;
  json_value_t value = entry.value;
  // Do something with `key`, `type` and `value`
}
```

### How to get the number of elements in an array?

In each `json_array_t`, there is a member `count`

```C
#include "json.h"

...

int i;
json_object_t *json = json_parse(some_json_string);
json_array_t *array = json->entries[0].value.as_array;
json_type_t type = array->type;
for(i = 0; i < array->count; i++) {
  json_value_t value = array->values[i];
  // Do something with `value`
}
```

### What if the JSON is poorly formatted with uneven whitespace

Compile using `-DJSON_SCRAPE_WHITESPACE`

### What if there is error in JSON

That is when `json_parse` returns `NULL`

## If this helped you in any way you can [buy me a beer](https://www.paypal.me/suhelchakraborty)
