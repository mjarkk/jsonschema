# `jsonschema` Creates a [json schema](https://json-schema.org/) from a struct

This package can generate a json schema for a struct that is compatible with https://json-schema.org/

```go
type ExampleStruct struct {
  Name string
  Field []int
  SomeOtherStruct OtherStruct
}

func main() {
  defsMap := map[string]jsonschema.Property{}
  schema, err := jsonschema.From(
    ExampleStruct{},
    "#/$defs/",
    func(key string, value jsonschema.Property) {
      defsMap[key] = value
    },
    func(key string) bool {
      _, ok := defsMap[key]
      return ok
    },
    nil,
  )

  if err != nil {
    log.Fatal(err)
  }

  schema.Defs = defs
}
```

### Default applied rules:

- A struct field is labeled as required when the data cannot be nil *so `strings`,`bool`,`int`,`float`,`struct`, etc.. are required and `[]string`, `[8]int`, `*int`, `map[string]string` are not required. You can overwrite this behavior by using `jsonSchema` struct tag*

### Supported struct tags:

- `json:`
  - `"-"` Ignores the field
  - `"other_name"` Renames the field
- `jsonSchema:`
  - `"notRequired"` Set the field are not required *(by default all fields with the exeption of `ptr`, `array`, `slice` and `map` are set as required)*
  - `"required"` Set the field as required
  - `"deprecated"` Mark the field as deprecated
  - `"uniqueItems"` Every array entry must be unique _(Only for arrays)_
  - `"hidden"` Do not expose field in the schema
  - `"min=123"` Set the minimum value or array length for the field
  - `"max=123"` Set the maximum value or array length for the field
- `description:"describe a property here"` Set the description property

You can also chain jsonSchema tags using `,` for example: `jsonSchema:"notRequired,deprecated"`

### Custom (Un)MarshalJSON

Sometimes you might want to define a custom schema for a type that implements the `MarshalJSON` and `UnmarshalJSON` methods.

You can define a custom definition like so:

```go
var PhoneNumber string

func (PhoneNumber) JSONSchemaDescribe() jsonschema.Property {
	minLen := uint(3)
	return jsonschema.Property{
		Title:       "Phone number",
		Description: "This field can contain any kind phone number",
		Type:        jsonschema.PropertyTypeString,
		Examples: []json.RawMessage{
			[]byte("\"06 12345678\""),
			[]byte("\"+31 6 1234 5678\""),
		},
		MinLength: &minLen,
	}
}
```
