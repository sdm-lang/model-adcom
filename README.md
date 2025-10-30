# AdCOM v1.0 FINAL in SDML

This repository contains an SDML representation of the [Advertising Common Object Model or AdCOM](https://github.com/InteractiveAdvertisingBureau/AdCOM) v1.0 specification from
[IAB Tech Lab](https://iabtechlab.com/). AdCOM itself is used in the [OpenRTB v3.0](https://github.com/InteractiveAdvertisingBureau/openrtb) specification as well as in the [OpenRTB Ad Management API v1.1](https://github.com/InteractiveAdvertisingBureau/AdManagementAPI/blob/master/Ad%20Management%20API%20v1.md)
specification.

Note that this is not an abstract domain model as-is, it is a lower-level representation for example purposes.

## Repository Content

The `sdml` directory contains a number of files,

``` text
sdml
  ├─ adcom.sdml
  ├─ adcom_context.sdml
  ├─ adcom_media.sdml
  └─ adcom_placement.sdml
```

## Data Types

### Numeric boolean - `YesNo`

Some Object fields are typed as `integer` but the text makes it clear that there are only 2 permissible values, 0 and 1,
usually representing No/Yes respectively. It would not be possible to use the builtin `boolean` type as SDML does not
support type coercion and so the mapping from true/false to 1/0 is not possible.

The following enumeration is used instead. This does require editing of some descriptions that reference the numeric
values directly.

``` sdml
module adcom is

  enum YesNo of
    @owl:equivalentClass = sdml:integer
    Yes is
      @rdf:value = 1
    end
    No is
      @rdf:value = 0
    end
  end

end
```

## Subtype Object Pattern

In a number of places there are object definitions that contain /required/

| Attribute | Type | Definition |
|-----------|------|------------|
| =title= | object; required * | **Asset Subtype Object** that indicates this is a title asset and provides additional detail as such. * Required if no other asset subtype object is specified. |
| =image= | object; required * | **Asset Subtype Object** that indicates this is an image asset and provides additional detail as such. * Required if no other asset subtype object is specified. |

The pattern then brings together all the types referenced in the table such as *title object* (named `TitleAsset` later in
the text) and *image object* (`ImageAsset`) and includes them in a new union type. This new type is the type used in the
parent `Asset` for a field named, by convention, `sub_type`.

``` sdml
module ascom_media is

  structure Asset is
    ;; ...
    sub_type -> AssetSubType
  end

  union AssetSubType of
    TitleAsset
    ImageAsset
    ;; ...
  end

end
```

## Reference List Pattern

A lot of fields in the AdCOM specification are typed as `integer` but with a reference list of values. Where this list
contains only /known/ values this translates into an SDML enumeration type with an OWL `equivalentClass` property and with
an RDF `value` property on each variant. This enumeration replaces the `integer` type for the field.

However, for many of these reference lists additional vendor-specific values are explicitly allowed starting at 500. To
address these a simple pattern is used where all known values in the specification are put into the enumeration as
above, and a new data type is added for values over 500. Finally these two types are combined into a union that becomes
the field type in the SDML structures. This pattern uses the prefix `Known` for the enumeration and `Vendor` for the data
type for consistency.

``` sdml
module example is

  import rdf

  enum KnownThingType of
    @owl:equivalentClass = sdml:integer

    ThingOne is
      @rdf:value = 1
    end

    ThingTwo is
      @rdf:value = 2
    end

  end

  datatype VendorThingType <- Integer is
    @xsd:minValueInclusive = 500
  end

  union ThingType of
    KnownThingType as Known
    VendorThingType as VendorSpecific
  end

end
```

According to OpenRTB:

> An optional attribute may have a default value to be assumed if omitted. If no default is indicated, then by
> convention its absence should be interpreted as *unknown*, unless otherwise specified.

This model does not introduce specific /Unknown/ values as the [AdCOM proto buffer repository](https://github.com/IABTechLab/adcom-proto) does.

- `KnownThingType` : The list of known enumeration values.
- `VendorThingType` : An integer value data type restricted to values greater than or equal to 500.
- `ThingType` : A union of the two types above.

## License(s)

The contents of this repository are made available under the following
licenses:

### Apache-2.0

> ```text
> Copyright 2025 Simon Johnston <johnstonskj@gmail.com>
> 
> Licensed under the Apache License, Version 2.0 (the "License");
> you may not use this file except in compliance with the License.
> You may obtain a copy of the License at
> 
>     http://www.apache.org/licenses/LICENSE-2.0
> 
> Unless required by applicable law or agreed to in writing, software
> distributed under the License is distributed on an "AS IS" BASIS,
> WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
> See the License for the specific language governing permissions and
> limitations under the License.
> ```

See the enclosed file [LICENSE-Apache](https://github.com/sdm-lang/tree-sitter-sdml/blob/main/LICENSE-APACHE).

### MIT

> ```text
> Copyright 2025 Simon Johnston <johnstonskj@gmail.com>
> 
> Permission is hereby granted, free of charge, to any person obtaining a copy
> of this software and associated documentation files (the “Software”), to deal
> in the Software without restriction, including without limitation the rights to
> use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
> the Software, and to permit persons to whom the Software is furnished to do so,
> subject to the following conditions:
> 
> The above copyright notice and this permission notice shall be included in all
> copies or substantial portions of the Software.
> 
> THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED,
> INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A
> PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
> HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
> OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
> SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
> ```

See the enclosed file [LICENSE-MIT](https://github.com/sdm-lang/tree-sitter-sdml/blob/main/LICENSE-MIT).

### Contributions

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
