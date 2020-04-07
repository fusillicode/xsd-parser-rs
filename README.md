# xsd-parser-rs
An xsd/wsdl => rust code generator written in rust. The main target is for generation of the [ONVIF Specifications](https://www.onvif.org/) but should work for other SOAP/XSDL/WSDL needs.

## Work in Progress
This is still a work in progress.  So please feel free to open issues and submit PRs. Please be sure to read and follow our [Code of Conduct](/CODE_OF_CONDUCT.md).

## XSD types mapping

A following mapping used to represent built-in XSD types as rust types:

|XSD               |rust         |
|------------------|-------------|
|hexBinary         |String       | 
|base64Binary      |String       |
|boolean           |bool         |
|integer           |Integer (1)  |
|nonNegativeInteger|Integer (1)  |
|positiveInteger   |Integer (1)  |
|nonPositiveInteger|Integer (1)  |
|negativeInteger   |Integer (1)  |
|long              |i64          |
|int               |i32          |
|short             |i16          |
|byte              |i8           |
|unsignedLong      |u64          |
|unsignedInt       |u32          |
|unsignedShort     |u16          |
|unsignedByte      |u8           |
|decimal           |Decimal (2)  |
|double            |f64          |
|float             |f64          |
|date              |String (3)   |
|time              |String (3)   |
|dateTime          |String (3)   |
|dateTimeStamp     |String (3)   |
|duration          |Duration (4) |
|gDay              |String (5)   |
|gMonth            |String (5)   |
|gMonthDay         |String (5)   |
|gYear             |String (5)   |
|gYearMonth        |String (5)   |
|string            |String       |
|normalizedString  |String       |
|token             |String       |
|language          |String       |
|Name              |String       |
|NCName            |String       |
|ENTITY            |String       |
|ID                |String       |
|IDREF             |String       |
|NMTOKEN           |String       |
|anyURI            |String       |
|QName             |String       |
|NOTATION          |String       |
|ENTITIES          |Vec\<String\>|
|IDREFS            |Vec\<String\>|
|NMTOKENS          |Vec\<String\>|

Notes:

(1) we are using our own type `Integer`, which wraps `num_bigint::Bigintg` and provides 
XML (de)serialization with `yaserde`. You can find `Integer` in `xsd-types/src/types/integer.rs`

(2) we are using our own type `Decimal`, which wraps `bigdecimal::BigDecimal` and provides 
XML (de)serialization with `yaserde`. You can find `Decimal` in `xsd-types/src/types/decimal.rs`

(3) we are going to use types from the chrono crate or its analog in the future

(4) we are using our own type `Duration`, since there is no known implementation
in rust that supports proper month/years holding and literal representation. You can find
`Duration` in `xsd-types/src/types/duration.rs`

(5) we are going to implement types that both provide stored value as integer and
support proper (de)serialization

## `any` elements handling

There are cases when schema allows extensions for the certain type.

```xml
<xs:complexType name="MyType">
    <xs:sequence>
        <xs:element name="Parameters" type="xs:string" />
        <xs:any namespace="##any" processContents="lax" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:anyAttribute namespace="##any" processContents="lax"/>
</xs:complexType>
```

In such cases we don't know in advance what fields must be present in Rust struct so we don't add them to output:

```rust
#[derive(Default, PartialEq, Debug, YaSerialize, YaDeserialize)]
#[yaserde(prefix = "tns", namespace = "tns: http://example.com")]
pub struct MyType {
    #[yaserde(prefix = "tns", rename = "Parameters")]
    pub parameters: String,
}
```

In this unlucky situation to support extensions user can either:
- modify the generated code and add extension fields manually
- modify source XSD and add extension elements there

## License

<sup>
Licensed under either of <a href="LICENSE-APACHE">Apache License, Version
2.0</a> or <a href="LICENSE-MIT">MIT license</a> at your option.
</sup>

<br/>

<sub>
Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in this crate by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
</sub>
