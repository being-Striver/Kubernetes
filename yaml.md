# YAML
--------------------------
YAML's structure relies heavily on indentation to define relationships between data elements. Unlike other markup languages like XML or JSON, YAML is designed to be easily read and written by humans.

Basic components:
-----------------------
- key-value pairs: YAML represents data as key-value pairs. The key is a string, and the value can be a scalar (string, number, boolean), a sequence (list), or another mapping (dictionary).
- Indentation: Indentation is critical in YAML. It defines the hierarchy and relationships between elements. Consistent indentation (usually two spaces) is essential. Mixing tabs and spaces will cause errors.
- Comments: Comments in YAML start with a `#` (hash) symbol. They are ignored by the YAML parser and are used to provide explanations or annotations.
- Data Types: YAML supports several basic data types:
  - Strings: Represented as text. Can be enclosed in single or double quotes, but often quotes are not necessary.
  - Numbers: Integers and floating-point numbers.
  - Booleans: Represented as `true` or `false` (case-insensitive). Also accepts `yes` or `no`.
  - Null: Represented as `null` or `~`.
  - Sequences (Lists): Sequences are ordered collections of items. They are represented using a hyphen `-` followed by a space.
  - Mappings (Dictionaries): Mappings are collections of key-value pairs. They are represented using indentation to show the relationship between the key and the value.
  - Anchors and Aliases: Anchors allow you to reuse parts of your YAML document. An anchor is defined using `&` followed by an identifier. An alias refers back to the anchor using `*` followed by the identifier. This is useful for avoiding repetition.
   
   The `<<:` is a merge key, which merges the properties of the alias into the current mapping while accessing anchors.
  - Multi-line Strings: YAML supports multi-line strings using `>` or `|`.
  - Explicit Typing: While YAML usually infers data types automatically, you can explicitly specify them using double exclamation marks `!!`.