---
layout: post
title: XSD example
tags: [xsd, xml]
---

XSD (XML Schema Definition) - can be used to validate incoming xml.

Lets assume that we have next assumptions:

1. Xml consists from zero or more items.
2. Each item has required name, photo, tags, diameter, weight, price fields and one non required enum size field.
3. Diameter, weight and price must be numeric, positive values.
4. Price also must be decimal with two numbers after point.
5. Photo field must contain valid url for image.
6. Size field, if present, can be one of: Big, Small, empty.

Here is example of test xml:

```xml
<?xml version="1.0" encoding="utf-8"?>
<items>
    <item>
        <name>Item 1</name>
        <photo>http://example.com/photo1.png</photo>
        <tags>Tag1, Tag2</tags>
        <diameter>32</diameter>
        <weight>540</weight>
        <price>60</price>
        <size>Big</size>
    </item>
    <item>
        <name>Item 2</name>
        <photo>http://example.com/photo2.png</photo>
        <tags>Tag1</tags>
        <diameter>23</diameter>
        <weight>340</weight>
        <price>50</price>
    </item>
</items>
```

And here is commented xsd example for this xml:

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- describing schema for sample xml -->
<xs:schema attributeFormDefault="unqualified" elementFormDefault="qualified" xmlns:xs="http://www.w3.org/2001/XMLSchema">

    <xs:element name="items"> <!-- root element will be <items> -->
        <xs:complexType> <!-- it will be complex (there are complex and simple) -->
            <xs:sequence> <!-- it will consists from sequence of other elements -->
                <xs:element name="item" minOccurs="0" maxOccurs="unbounded"> <!-- there will be zero or more <item> elements in sequence -->
                    <xs:complexType> <!-- they all have complex type -->
                        <xs:sequence> <!-- each of them will have next sequence of elements -->
                            <xs:element name="name" type="NonEmptyString" minOccurs="1"/> <!-- required (minOccurs="1") non empty (look for NonEmptyString at bottom) name field -->
                            <xs:element name="photo" minOccurs="1"> <!-- required photo with given pattern to validate urls for images -->
                                <xs:simpleType> <!-- this is example how to use additional restrictions for elements -->
                                    <xs:restriction base="xs:anyURI">
                                        <xs:minLength value="1" />
                                        <xs:pattern value="http://.*(png|jpg|jpeg|gif)" />
                                    </xs:restriction>
                                </xs:simpleType>
                            </xs:element>
                            <xs:element name="tags" type="NonEmptyString" minOccurs="1"/>
                            <xs:element name="diameter" type="xs:positiveInteger" minOccurs="1"/>
                            <xs:element name="weight" type="xs:positiveInteger" minOccurs="1"/>
                            <xs:element name="price" type="positiveDecimal" minOccurs="1"/>
                            <xs:element name="size" minOccurs="0" default=""> <!-- example of enum field -->
                                <xs:simpleType>
                                    <xs:restriction base="xs:string">
                                        <xs:enumeration value=""/>
                                        <xs:enumeration value="Big"/>
                                        <xs:enumeration value="Small"/>
                                    </xs:restriction>
                                </xs:simpleType>
                            </xs:element>
                        </xs:sequence>
                    </xs:complexType>
                </xs:element>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
    <xs:simpleType name="positiveDecimal">
        <xs:restriction base="xs:decimal">
            <xs:minExclusive value="0"/>
            <xs:fractionDigits value="2"/>
        </xs:restriction>
    </xs:simpleType>
    <xs:simpleType name="NonEmptyString"> <!-- we can describe our types separately to reuse them later -->
        <xs:restriction base="xs:string">
            <xs:minLength value="1" />
            <xs:pattern value=".*[^\s].*" />
        </xs:restriction>
    </xs:simpleType>
</xs:schema>
```

To check xml you can use xmllint tool: `xmllint --noout --schema example.xsd example.xml`

Or online tool like this: http://xsdvalidation.utilities-online.info/

Also there is bunch of tools to automatically create xsd from xml, one of them: http://www.freeformatter.com/xsd-generator.html
