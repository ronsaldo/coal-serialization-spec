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
|     Name    | utf8_32_16     | The name of the field. Must be unique inside the parent type |
|     Type    | TypeDescriptor | A descriptor of the field type.                              |

### Cluster Descriptions

This section describes the layout of the cluster descriptions:

| Field Name  |               Type            |                                Description                             |
| ----------- | ----------------------------- | ---------------------------------------------------------------------- |
| Name        | utf8_32_16                    | The name of the structure type. Must be unique.                        |
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

| Type Name | Type Descriptor Encoding | Description |
| --------- | ------------------------ | ----------- |
| object | 0x00 | Object reference |
| boolean8 | 0x01  | Single byte boolean. 0 Means false, everything else is true. |
| boolean16 | 0x02  | Two bytes boolean. 0 Means false, everything else is true. Provided for completeness. |
| boolean32 | 0x03  | Four bytes boolean. 0 Means false, everything else is true. Provided for completeness. |
| boolean64 | 0x04  | Eight bytes boolean. 0 Means false, everything else is true. Provided for completeness. |
| uint8 | 0x05 | 8 bit little-endian unsigned integer. |
| uint16 | 0x06 | 16 bit little-endian unsigned integer. |
| uint32 | 0x07 | 32 bit little-endian unsigned integer. |
| uint64 | 0x08 | 64 bit little-endian unsigned integer. |
| uint128 | 0x09 | 128 bit little-endian unsigned integer. |
| int8 | 0x0A | 8 bit little-endian 2-complement signed integer. |
| int16 | 0x0B | 16 bit little-endian 2-complement signed integer. |
| int32 | 0x0C | 32 bit little-endian 2-complement signed integer. |
| int64 | 0x0D | 64 bit little-endian 2-complement signed integer. |
| int128 | 0x0E | 128 bit little-endian 2-complement signed integer. |
| float16 | 0x0F | (Optional) 16 bit IEEE-754 half-precision binary floating point number with little-endian byte ordering. |
| float32 | 0x10 | 32 bit IEEE-754 single-precision binary  floating point number with little-endian byte ordering. |
| float64 | 0x11 | 64 bit IEEE-754 double-precision binary  floating point number with little-endian byte ordering. |
| float128 | 0x12 | (Optional) 128 bit IEEE-754 quadruple-precision binary  floating point number with little-endian byte ordering. |
| float256 | 0x13 | (Optional) 256 bit IEEE-754 octuple-precision binary  floating point number with little-endian byte ordering. |
| decimal32 | 0x14 | (Optional) 32 bit IEEE-754 decimal floating point number with little-endian byte ordering. |
| decimal64 | 0x15 | (Optional) 64 bit IEEE-754 decimal floating point number with little-endian byte ordering. |
| decimal128 | 0x16 | (Optional) 128 bit IEEE-754 decimal floating point number with little-endian byte ordering. |
| binary_32_8 | 0x17 | Sequence of bytes in the Blob. Serialized inline as the (uint32 blobOffset, uint8 size) sequence.  |
| binary_32_16 | 0x18 | Sequence of bytes in the Blob. Serialized inline as the (uint32 blobOffset, uint16 size) sequence. |
| binary_32_32 | 0x19 | Sequence of bytes in the Blob. Serialized inline as the (uint32 blobOffset, uint32 size) sequence. |
| utf8_32_8 | 0x1A | UTF-8 encoded string in the Blob. Serialized inline as the (uint32 blobOffset, uint8 size) sequence.  |
| utf8_32_16 | 0x1B | UTF-8 encoded string in the Blob. Serialized inline as the (uint32 blobOffset, uint16 size) sequence. |
| utf8_32_32 | 0x1C | UTF-8 encoded string in the Blob. Serialized inline as the (uint32 blobOffset, uint32 size) sequence. |
| utf16_32_8 | 0x1D | UTF-16 encoded string in the Blob. Serialized inline as the (uint32 blobOffset, uint8 size) sequence.  |
| utf16_32_16 | 0x1E | UTF-16 encoded string in the Blob. Serialized inline as the (uint32 blobOffset, uint16 size) sequence. |
| utf16_32_32 | 0x1F | UTF-16 encoded string in the Blob. Serialized inline as the (uint32 blobOffset, uint32 size) sequence. |
| utf32_32_8 | 0x20 | UTF-32 encoded string in the Blob. Serialized inline as the (uint32 blobOffset, uint8 size) sequence.  |
| utf32_32_16 | 0x21 | UTF-32 encoded string in the Blob. Serialized inline as the (uint32 blobOffset, uint16 size) sequence. |
| utf32_32_32 | 0x22 | UTF-32 encoded string in the Blob. Serialized inline as the (uint32 blobOffset, uint32 size) sequence. |
| bigint_32_8 | 0x23 | (Optional) Big integer with magnitude encoded in the Blob. Serialized inline as the (uint32 blobOffset, int8 signAndSize) sequence. |
| bigint_32_16 | 0x24 | Big integer with magnitude encoded in the Blob. Serialized inline as the (uint32 blobOffset, int16 signAndSize) sequence. |
| bigint_32_32 | 0x25 | Big integer with magnitude encoded in the Blob. Serialized inline as the (uint32 blobOffset, int32 signAndSize) sequence. |
| char8 | 0x26 | 8 bit little-endian unsigned integer encoding for character code unit. |
| char16 | 0x27 | 16 bit little-endian unsigned integer encoding for character code unit. |
| char32 | 0x28 | 32 bit little-endian unsigned integer encoding for character code unit. |
| struct | 0x80 [uint32 index] | User defined value type. |
| fixedArray | 0x81 [uint32 size] [TypeDescriptor element] | Fixed length array. |
| array8 | 0x82 [TypeDescriptor element] | Array of elements prefixed with an uint8 length. |
| array16 | 0x83 [TypeDescriptor element] | Array of elements prefixed with an uint16 length. |
| array32 | 0x84 [TypeDescriptor element] | Array of elements prefixed with an uint32 length. |
| set8 | 0x85 [TypeDescriptor element] | Set of elements prefixed with an uint8 length. |
| set16 | 0x86 [TypeDescriptor element] | Set of elements prefixed with an uint16 length. |
| set32 | 0x87 [TypeDescriptor element] | Set of elements prefixed with an uint32 length. |
| map8 | 0x88 [TypeDescriptor key] [TypeDescriptor value] | Set of elements prefixed with an uint8 length. |
| map16 | 0x89 [TypeDescriptor key] [TypeDescriptor value] | Set of elements prefixed with an uint16 length. |
| map32 | 0x8A [TypeDescriptor key] [TypeDescriptor value] | Set of elements prefixed with an uint32 length. |
