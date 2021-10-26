# Coal Portable Native Object Graph Serialization Format Spec
Language independent self-describing general purpose object-graph serialization format based on clustering.

## Introduction

There are several standardized language independent serialization for tree like data structures, that are also simple to understand and describe. Some of them are text based such as JSON and XML. There are also multiple binary variants of them such as BSON and MessagePack. Unfortunately, there is a lack of a general purpose standardized fast serialization format for general object graphs. There are multiple ad-hoc solutions that are based on performing a depth-first traversal of an object graph, encoding into an existent tree-like serialization format, and replacing back-pointers with references. Unfortunately, these ad-hoc solutions are not as widely standardized as the JSON or XML document formats. Another issue present on several of these document formats, is that they tend to have a strong redundancy on the key names for attribute names. In some cases, this redundancy might account for at least the half of the serialized file format. For all of these reason, we propose the **Coal** serialization format which is described in this document. We name it coal in honor to the existent **Fuel** serialization format.

Our main source of inspiration is the **Fuel** serialization format and framework implemented in Pharo. Fuel and its design is described in *Fuel: A fast general purpose object graph serializer* by M. Dias etal. One of the main limitations of **Fuel** is its lack portability, and the lack of language independance. We propose solving this issue by writing this document with an exact per-byte layout description on the **Coal** serialization format. In addition we also provide direct support for encoding in-line value types in addition to per-object references. The objective of adding support for value-types is to better match the semantics of static-typed languages that have these types with an exact bit-level types, such as the C/C++ primitive types, and the C# structures.

## Value Type Restrictions
To simplify the serialization process of in-line value types, we impose that must comply with:
1. They must be one of the in-line value types that are defined in this spec.
2. User defined value types cannot be recursive, and they cannot hold pointers to other value types. Pointers to objects inside of value types are acceptable.

## Coal Serialization File Layout

The following is the sequence of sections that are always present in a Coal file:

1. File Header
2. Binary Blob
3. Structure Type Layouts
4. Cluster Descriptions
5. Cluster Contents
6. Trailer

### File Header
The following table describes the fields that are present in the file header:

|     Field Name     |  Type  |                  Description                 |
| ------------------ | ------ | -------------------------------------------- |
| Magic Number       | uint32 | 0x4C414F43 Little Endian Encoding for "COAL" |
| Version Major      | uint8  | Expected to be 1.                            |
| Version Minor      | uint8  | Expected to be 0.                            |
| Reserved           | uint16 | Set to 0.                                    |
| Binary Blob Size   | uint32 | The byte length of the binary blob.          |
| Value Type Layouts | uint32 | The number of different value type layouts.  |
| Cluster Count      | uint32 | The number of clusters.                      |
| Object Count       | uint32 | The total of serialized objects.             |


### Binary Blob
After the file header we encode a blob de-duplicated byte sequences. The main purpose of this blob is to reduce the redundancy on string data.

### Structure Type Layouts

This section describes the layout of the user-defined structure types which are in-lined serialized in the different objects. Each user defined structure has the following fields:

| Field Name  |               Type            |                                Description                             |
| ----------- | ----------------------------- | ---------------------------------------------------------------------- |
| Name        | utf8_32_16                    | The name of the structure type. Must be unique.                        |
| Field Count | uint16                        | The number of fields.                                                  |
| Fields      | FieldDescription[Field Count] | The description of the fields that are serialized in sequential order. |

### Field Description

Each field in a structure or a cluster is described with the following fields:

| Field Name  |       Type     |                           Description                        |
| ----------- | -------------- | ------------------------------------------------------------ |
|     Name    | utf8_32_16    | The name of the field. Must be unique inside the parent type |
|     Type    | TypeDescriptor | A descriptor of the field type.                              |

### Cluster Descriptions

This section describes the layout of the cluster descriptions:

| Field Name  |               Type            |                                Description                             |
| ----------- | ----------------------------- | ---------------------------------------------------------------------- |
| Name        | utf8_32_16                   | The name of the structure type. Must be unique.                        |
| Field Count | uint16                        | The number of fields.                                                  |
| Instances   | uint32                        | The number of instances that belong to this cluster.                   |
| Fields      | FieldDescription[Field Count] | The description of the fields that are serialized in sequential order. |

### Cluster Contents

This section contains a sequential serialization of each object instance that is described in the previous section.

### Trailer

This is the final file section. The following is the layout of this second:

|     Field Name    |       Type     |                           Description                        |
| ----------------- | -------------- | ------------------------------------------------------------ |
| Root Object Index |     uint32     | Zero based index of the root object in the graph.            |

## Type Descriptors

This section describes the different types, and the encoding for their descriptors that are allowed by this serialization format:


TODO: Primitive types
- uint8
- uint16
- uint32
- uint64
- uint128
- int8
- int16
- int32
- int64
- int128
- float32
- float64

TODO: Special type
- large integer

TODO: String types
- utf8_32_8
- utf8_32_16
- utf8_32_32
- utf16_32_8
- utf16_32_16
- utf16_32_32
- utf32_32_8
- utf32_32_16
- utf32_32_32

TODO: Blob types
- binary_32_8
- binary_32_16
- binary_32_32

TODO: Derived Types
- Fixed Array
- Array8 - Element Type Descriptor
- Array16 - Element Type Descriptor
- Array32 - Element Type Descriptor

- Set8 - Element Type Descriptor
- Set16 - Element Type Descriptor
- Set32 - Element Type Descriptor

- Map8 - Key Type Descriptor - Value Type Descriptor
- Map16 - Key Type Descriptor - Value Type Descriptor
- Map32 - Key Type Descriptor - Value Type Descriptor

TODO: User Defined Types
- Value Type - Value Type Index
- Object Reference
