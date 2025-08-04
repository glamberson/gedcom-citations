# GEDCOM Citation Source Templates Extension

## Purpose

Many types of sources exist with a common set of elements that can be used to format a citation.
In order to facilitate formatting of consistent citations, it is desirable to allow
specifying such information with a master source.

This extension adds a tag, `_TPLT`, to allow recording the fact that a given master source
has citation elements in common with other similar sources.

This extension also allows associating citation elements with a given source, by use of
new tags that can be used together with the `_TPLT` tag to allow applications to prompt
for, store, and generate citations for various types of sources.

## Tags

### `_TPLT` (Source Template)

A substructure to identify a Source Template that contains additional information on how data should be captured for the associated source and how those data should be combined to create a citation.

#### `_TPLT`

Source Templates allow applications to give users a set of data to capture for a given source type
(such as a grave marker, obituary, will, etc.) and how such data should be formatted in a footnote citation.
Source templates might be published as standard source templates or might be specific to a given application.

The URI for this extension's `_TPLT` tag is `https://gedcom.io/terms/v7/_TPLT`.

The `_TPLT` structure is used as a substructure of the `SOUR` structure as follows:

```
n SOUR @XREF:SOUR@                          {1:1}  g7:record-SOUR
  +1 <<SOUR_DETAIL>>                        {0:1}
     +2 _TPLT <URI>                         {0:1}  https://gedcom.io/terms/v7/_TPLT
        +3 _FIEL <Text>                     {0:M}  https://gedcom.io/terms/v7/_FIEL
           +4 TEXT <Text>                   {0:1}  g7:TEXT
              +5 MIME <MediaType>           {0:1}  g7:MIME
              +5 LANG <Language>            {0:1}  g7:LANG
        +3 SOUR @<XREF:SOUR>@               {0:1}  g7:SOUR
```

The source template's payload is a [URI](https://www.rfc-editor.org/info/rfc3986) where the source template definition can be found.

The `_TPLT` substructures are:

`_FIEL` (Field)
: A field or parameter from the source template. The payload contains the machine-readable variable name from the source template. The name displayed to users may be something else, such as the human-readable label from the source template.

### `SOUR` (Source)

In addition to the `_TPLT` substructure added in this extension, use of a `_TPLT` together with a source collection requires adding support for the following
substructure of the `SOUR` structure:

```
n SOUR @XREF:SOUR@                          {1:1}  g7:record-SOUR
  +1 <<SOUR_DETAIL>>                        {0:1}
     +2 _TPLT <URI>                         {0:1}  https://gedcom.io/terms/v7/_TPLT
        +3 SOUR @<XREF:SOUR>@               {0:1}  g7:SOUR
```

The `SOUR` substructure is used to identify a collection that the `SOUR` is contained in, such as when using a source template that is for a source within a collection.

## Definitions

Master Source
: A `g7:record-SOUR`.

Source Detail
: A `g7:SOUR` substructure of an `INDI` or `FAM` record.

Citation
: Data that would be output when displaying the associated source in a source list entry, first reference note, or short subsequent note.

Citation Element
: A piece of information about a source, such as its author, title, publication date, repository, URL, etc.

Source Template
: A set of information about a class of sources, with a common set of citation elements and rules for formatting citations of sources of that class.

## Extensions to the `SOUR` structure

### Overview

This extension adds the following substructure to an existing `SOUR` structure:

* A `_TPLT` substructure whose payload is the URI of a source template.
* Multiple `_FIEL` substructures of the `_TPLT` that provide values of citation elements for the master source as specified by the source template, if any.
* `_FIEL` substructures of a `g7:SOUR` `g7:PAGE` that provide values of citation elements for the source details as specified by the source template, if any.
* A `SOUR` substructure of a `_TPLT` that provides a cross-reference to a collection that the source is contained in.

### Citation Element Values

Following GEDCOM conventions, citation element values are not empty unless explicitly stated otherwise in the source template.

### Examples

See [Examples of Source Templates](#examples-of-source-templates).

## Source Templates

### Defining Source Templates

A source template is a YAML file identified by its URI, which is preferably a URL that will resolve
to the YAML file.

Source templates associate citation elements with GEDCOM tags and contain Liquid template patterns that
say how citation elements should be formatted when generating a citation.

### Template Patterns

See [Appendix: Template Patterns](#appendix-template-patterns).

### Source Template Schema

It is proposed that we store source templates in YAML files, as is done for GEDCOM tag, record, and enumset specifications at [GEDCOM/README.md at main · FamilySearch/GEDCOM · GitHub](https://github.com/FamilySearch/GEDCOM/blob/main/extracted-files/README.md)
and [GEDCOM-registries/record-INDI.yaml at main · FamilySearch/GEDCOM-registries · GitHub](https://github.com/FamilySearch/GEDCOM-registries/blob/main/standard/record-INDI.yaml).
As with the other YAML file types, source template YAML files might be standard or might be application-specific.  Either way they are identified by a URI, preferably by a URL resolvable to the source template YAML file.

The following [JSON schema](https://datatracker.ietf.org/doc/html/draft-bhutton-json-schema-01#name-meta-schemas) describes the YAML file syntax:

```json
{
  "$schema": "http://json-schema.org/draft/2020-12/schema#",
  "title": "JSON Schema for GEDCOM source YAML templates",
  "type": "object",
  "properties": {
    "lang": {
      "type": "string",
      "description": "IANA language tag of the language of this template.",
    },
    "uri": {
      "type": "string",
      "description": "The URI of this source template.",
    },
    "name": {
      "type": "string",
      "description": "Textual name which could be displayed to users in a list of source templates.",
    },
    "description": {
      "type": "string",
      "description": "Textual description which could be displayed to users in a list of source templates.",
    },
    "collection": {
      "type": "string",
      "description": "URI of the source template for the containing collection.",
    },
    "keywords": {
      "type": "array",
      "description": "A list of keywords for use, in addition to any words in the name, when filtering source templates by a search string.",
      "items": {
        "type": "string"
      }
    },
    "sfields": {
      "type": "array",
      "description": "Array of citation elements for a master source.",
      "items": {
        "type": "object",
        "properties": {
          "variable": {
            "type": "string",
            "description": "Machine-readable variable name of the citation element.",
          },
          "label": {
            "type": "string",
            "description": "Human-readable label associated with the citation element.",
          },
          "required": {
            "type": "boolean",
            "description": "Whether the citation element is required to have a  non-empty value.",
          },
        },
        "required": ["variable", "label"],
        "additionalProperties": false,
      }
    },
    "dfields": {
      "type": "array",
      "description": "Array of citation elements for source details.",
      "items": {
        "type": "object",
        "properties": {
          "variable": {
            "type": "string",
            "description": "Machine-readable variable name of the citation element.",
          },
          "label": {
            "type": "string",
            "description": "Human-readable label associated with the citation element.",
          },
          "required": {
            "type": "boolean",
            "description": "Whether the citation element is required to have a  non-empty value.",
          },
        },
        "required": ["variable", "label"],
        "additionalProperties": false,
      }
    },
    "formats": {
      "type": "array",
      "description": "Array of style guides for formatting citations.",
      "items": {
        "type": "object",
        "properties": {
          "name": {
            "type": "string",
            "description": "The name of the style guide for formatting citations, such as 'Evidence Explained'.",
          },
          "titl": {
            "type": "string",
            "description": "Template pattern for formatting a TITL using the given style guide.",
          },
          "auth": {
            "type": "string",
            "description": "Template pattern for formatting an AUTH using the given style guide.",
          },
          "publ": {
            "type": "string",
            "description": "Template pattern for formatting a PUBL using the given style guide.",
          },
          "scite": {
            "type": "string",
            "description": "Template pattern for formatting the master source portion of a first reference note.",
          },
          "page": {
            "type": "string",
            "description": "Template pattern for formatting a PAGE using the given style guide.",
          },
          "slist": {
            "type": "string",
            "description": "Template pattern for formatting a source list entry using the given style guide.",
          },
          "first": {
            "type": "string",
            "description": "Template pattern for formatting a first reference note using the given style guide.",
          },
          "short": {
            "type": "string",
            "description": "Template pattern for formatting a short subsequent note using the given style guide.",
          },
        },
        "additionalProperties": false,
      },
    },
    "alttitl": {
      "type": "array",
      "description": "Array of patterns that can be matched for an existing TITL.",
      "items": {
        "type": "string"
      }
    },
    "altauth": {
      "type": "array",
      "description": "Array of patterns that can be matched for an existing AUTH.",
      "items": {
        "type": "string"
      }
    },
    "altpubl": {
      "type": "array",
      "description": "Array of patterns that can be matched for an existing PUBL.",
      "items": {
        "type": "string"
      }
    },
    "altpage": {
      "type": "array",
      "description": "Array of patterns that can be matched for an existing PAGE.",
      "items": {
        "type": "string"
      }
    },
    "repository": {
      "type": "boolean",
      "description": "Whether the master source is expected to be in a repository.",
    },
    "quality": {
      "type": "integer",
      "description": "Quality (roughly 0=unreliable, 1=questionable, 2=secondary, 3=primary, 4=original).",
    },
  },
  "required": ["lang", "uri", "name"],
  "additionalProperties": false,
}
```

To dereference a URI specifying a source template:

1. Look for a locally-installed file with a matching `uri` property.
2. Attempt to retrieve the file from the URI directly via HTTPS.

## Using Source Templates

An application can use a source template as follows.

Creating a source from a template:

1. The application lets the user pick or search for a source template.
2. The application prompts the user to enter values for citation elements as specified by the `sfields` array in the source template.
3. The application creates a master source with `TITL`, `AUTH`, and `PUBL` whose values are generated using the corresponding template patterns in the source template.  If a template pattern is not specified in the source template, the GEDCOM tag is omitted.
4. The application creates a `_TPLT` substructure with the URI of the source template.
5. The application creates `_FIEL` substructures of the `_TPLT` with variables and values of any citation elements that (a) are in the source template, and (b) have non-empty values.

Creating a first reference note for a citation to a source with a template:

1. The application identifies the source template (if any) from the `_TPLT` of the master source.
2. The application prompts the user to enter values for citation elements as specified by the `dfields` array in the source template.
3. The application sets the `PAGE` value by generated using the `page` template pattern in the source template.  If a template pattern is not specified in the source template, the GEDCOM tag is omitted.
4. The application generates text for the citation using the `first` template pattern.  If such a pattern is not present, the following template pattern is used instead: `{% if scite %}{{ scite }}{% else %}{{ TITL }}{% if AUTH %}, by {{ AUTH }}{% endif %}{% if PUBL %} ({{ PUBL }}){% endif %}{% endif %}{% if PAGE %}, {{ PAGE }}{% endif %}{% if URL %}; {{ URL }}{% endif %}{% if NOTE %}. {{ NOTE }}{% endif %}.`  When evaluating this template pattern, the application substitutes values as follows:
   * Any field appearing in the source template (whether in the `sfields` or `dfields` arrays) uses the value entered by the user.
   * If the pattern `scite` appears in the source template, it is expanded using the template pattern there.
   * Otherwise variables are expanded using the value of the corresponding GEDCOM tag.
5. The application creates `_FIEL` substructures of the `PAGE` with variables and values of any citation elements that (a) are in the `dfields` of the source template, and (b) have non-empty values.

## Examples of Source Templates

The following examples show the YAML text, which can be placed into `.yaml` files.

### Grave Marker Template

The Grave Marker example is taken from the monument example at *Evidence Explained*, page 247.

#### Grave Marker YAML

```yaml
%YAML 1.2
---
lang: en-US
uri: https://gedcom.io/templates/GraveMarker
name: Grave Marker
description: The Grave Marker model is used for grave markers, cemetery monuments,
  and other forms of cemetery art (see "Evidence Explained", pages 246-247).
keywords:
  - monument
  - headstone
  - cemetery
sfields:
  - variable: Cemetery
    label: Cemetery
    required: true
  - variable: Location
    label: Cemetery Location
dfields:
  - variable: Section
    label: Section, Lot, or Row
formats:
  - name: Evidence Explained
    titl: {{ Cemetery }}
    publ: {{ Location }}
    scite: {{ Cemetery }}{% if Location %} ({{ Location }}){% endif %}
    page: {{ Section }}
```

#### Example GEDCOM

```gedcom
0 @I1@ INDI
1 SOUR @C20004@
2 PAGE Samuel Witter marker
3 _FIEL Section
4 TEXT Samuel Witter marker
0 @C20004@ SOUR
1 TITL Brian Cemetery
1 _TPLT https://gedcom.io/templates/GraveMarker
2 _FIEL Cemetery
3 TEXT Brian Cemetery
2 _FIEL Location
3 TEXT Lawrence County, Illinois
1 PUBL Lawrence County, Illinois
```

#### Example Formatted Citations

**Source List Entry**

Brian Cemetery (Lawrence County, Illinois).

**First Reference Note**

Brian Cemetery (Lawrence County, Illinois), Samuel Witter marker.

### Tax Record Templates

The Tax Roll and Tax Volume examples are taken from the microfilm examples at *Evidence Explained*, page 531.

#### Tax Roll YAML

```yaml
%YAML 1.2
---
lang: en-US
uri: https://gedcom.io/templates/TaxRoll
name: Tax Roll
description: The Tax Roll (Collection) model is used for a series of tax records
  covering a range of years, such as on a microfilm (see "Evidence Explained",
  pages 531-532).
keywords:
  - assessment
  - run
sfields:
  - variable: Jurisdiction
    label: Jurisdiction
    required: true
  - variable: Series
    label: Series
    required: true
formats:
  - name: Evidence Explained
    titl: {{ Jurisdiction }}, {{ Series }}
    publ: {{ Location }}
    scite: {{ Jurisdiction }}, {{ Series }}
    page: {{ Section }}
repository: true
quality: 3
```

#### Tax Volume YAML

```yaml
%YAML 1.2
---
lang: en-US
uri: https://gedcom.io/templates/TaxVolume
name: Tax Volume
description: The Tax Roll in Series model is used for tax records found within a
  source covering a run of years, such as on a microfilm (see "Evidence
  Explained", pages 530-531). For a single tax roll book with its own title, use
  the Tax Roll Book model instead.
collection: https://gedcom.io/templates/TaxRoll
keywords:
  - assessment
sfields:
  - variable: Volume
    label: Specific Volume
    required: true
dfields:
  - variable: Section
    label: Section (if any)
  - variable: Page
    label: Page
  - variable: Item
    label: Item of Interest
formats:
  - name: Evidence Explained
    titl: {{ Jurisdiction }}, {{ Volume }}
    scite: {{ Jurisdiction }}, {{ Volume }}
    page: |
      {% if Section %}{{ Section }}{% if Page %}, {% endif %}{% endif %}
      {%- if Page %}p. {{ Page }}{% endif %}
      {%- if Item %}{% if Page %}, {% endif %}{{ Item }}{% endif %}
altpage:
  - p. {{ Page }}
repository: true
quality: 3
```

#### Example GEDCOM

```gedcom
0 @I1@ INDI
1 SOUR @C20064@
0 @C20034@ SOUR
1 TITL Christian County, Kentucky, Tax Books, 1797-1875
1 _TPLT https://gedcom.io/templates/TaxRoll
2 _FIEL Jurisdiction
3 TEXT Christian County, Kentucky
2 _FIEL Series
3 TEXT Tax Books, 1797-1875
1 REPO @F10004@
2 CALN 7,926-7,929
3 MEDI FILM
1 _QUAY 3
0 @C20064@ SOUR
1 TITL Christian County, Kentucky, 1799 Tax Book
1 _TPLT https://gedcom.io/templates/TaxVolume
2 SOUR @C20034@
2 _FIEL Volume
3 TEXT 1799 Tax Book
1 REPO @F10004@
2 CALN 7,926
3 MEDI FILM
1 _QUAY 3
0 @F10004@ REPO
1 NAME Family History Library
```

#### Example Formatted Citations

**Source List Entries**

- Christian County, Kentucky, 1799 Tax Book; FHL microfilm 7,926.
- Christian County, Kentucky, Tax Books, 1797-1875; FHL microfilm 7,926-7,929.

**First Reference Notes**

- Christian County, Kentucky, 1799 Tax Book; FHL microfilm 7,926.

## Input Matching

For applications that support the `alt*` fields, such fields can be used to assist in converting a non-templated source to a templated source by providing an appropriate default for a user to confirm.
Let's look at an example using the Web Database model from *Evidence Explained*, pages 254 and 438.

We will use the following YAML snippet as an example:

```
    altpubl: {{ Type }}, {{ Creator }}, {{ Website }} ({{ Url }} : accessed {{ Date }})
    altpubl: {{ Type }}, {{ Creator }}, {{ Website }} ({{ Url }})
    altpubl: {{ Type }}, {{ Website }} ({{ Url }})
    altpubl: {{ Website }} ({{ Url }})
    altpubl: {{ Website }}
    publ: {% if Type %}{{ Type }}, {% endif %}{% if Creator %}{{ Creator }}, {% endif %}{{ Website }} ({{ Url }}{% if Date %} : accessed {{ Date }}{% endif %})
```

Above gives a number of patterns that can be recognized.  In this case, the `publ` pattern
used for output is not particularly usable for input due to the use of conditionals, and the
fact that Type and Creator are both optional and so if only one is present, one could not
reliably tell whether it is the Type or the Creator.  The use of the `altpubl` patterns solves
this by specifying the precedence.

Consider a GEDCOM containing:

- `1 PUBL database, Library and Archives Canada (LAC), Canadian Genealogy Centre (http://www.collectionscanada.ca/genealogy/index-e.html : accessed 21 February 2007)`

	The payload above matches the first pattern.

- `1 PUBL database, Library and Archives Canada (LAC), Canadian Genealogy Centre (http://www.collectionscanada.ca/genealogy/index-e.html)`

	The payload above matches the second pattern.

- `1 PUBL database, Canadian Genealogy Centre (http://www.collectionscanada.ca/genealogy/index-e.html)`

	The payload above matches the third pattern.

- `1 PUBL Canadian Genealogy Centre (http://www.collectionscanada.ca/genealogy/index-e.html)`

	The payload above matches the fourth pattern.

- `1 PUBL Canadian Genealogy Centre`

	The payload above matches the fifth pattern.

## Appendix: Template Patterns

For programmatically generating GEDCOM structures from a given source template, this document
recommends template patterns using [Liquid template syntax](https://shopify.github.io/liquid/).

### Why Liquid?

Liquid was created by Shopify specifically for safe template evaluation with untrusted input. It's battle-tested at scale:
- Used by GitHub Pages (millions of templates daily)
- Used by Jekyll static site generator
- Used by Shopify (billions of template renders)

### Safety Guarantees

Liquid is designed to be safe by default:
- **No infinite loops**: Iteration is limited (default 100, configurable)
- **No arbitrary code execution**: Only pre-registered filters and tags
- **No file system access**: Cannot read/write files
- **No network access**: Cannot make HTTP requests
- **Resource limits**: Memory and CPU limits in implementations

### Parser Availability

Mature, well-maintained parsers exist for all major languages:

| Language | Library | GitHub Stars | URL |
|----------|---------|--------------|-----|
| Ruby | liquid | 10.8k | https://github.com/Shopify/liquid |
| Python | python-liquid | 300+ | https://github.com/jg-rp/liquid |
| JavaScript | liquidjs | 1.5k | https://github.com/harttle/liquidjs |
| Java | Liqp | 240+ | https://github.com/bkiers/Liqp |
| Go | liquid | 270+ | https://github.com/osteele/liquid |
| PHP | liquid-php | 40+ | https://github.com/kalimah-apps/liquid-php |
| C# | DotLiquid | 1k | https://github.com/dotliquid/dotliquid |
| Rust | liquid-rust | 430+ | https://github.com/cobalt-org/liquid-rust |

### Liquid Patterns

`{{ variable }}`
: Replace with the value of the specified variable.

`{{ variable | default: "text" }}`
: If variable is non-empty, replace with its value. Otherwise replace with the specified text.

`{% if variable %}text{% endif %}`
: If variable is empty, do nothing. Otherwise replace with the specified text.

`{{ variable | escape }}`
: Replace with HTML-escaped value of the variable.

### Common Filters

Liquid provides filters for common text transformations:

```liquid
{{ Author | split: ", " | reverse | join: " " }}  # "Smith, John" → "John Smith"
{{ Date | date: "%Y" }}                           # Extract year from date
{{ Title | capitalize }}                          # Proper case
{{ Name | upcase }}                               # Convert to uppercase
```

### Lists

For multiple authors or other repeated elements:

```liquid
{% for author in Authors limit:10 -%}
  {{ author }}{% unless forloop.last %}, {% endunless %}
{%- endfor %}
```

### Complex Conditionals

More readable than nested patterns:

```liquid
{% if Type == "database" %}
  database
{% elsif Type == "digital" %}
  digital image
{% else %}
  {{ Type }}
{% endif %}
```

### Migration from BASH Syntax

The following table shows how to convert from BASH-style patterns to Liquid:

| BASH Pattern | Liquid Pattern |
|--------------|----------------|
| `${variable}` | `{{ variable }}` |
| `${variable:-default}` | `{{ variable \| default: "default" }}` |
| `${variable:+text}` | `{% if variable %}text{% endif %}` |
| `\$` | `$` |

Note: Liquid rarely needs escaping. When needed, use `{{ "{{" }}` to produce `{{`.

### Example Conversions

**BASH:**
```
${Cemetery}${Location:+ (${Location})}
```

**Liquid:**
```liquid
{{ Cemetery }}{% if Location %} ({{ Location }}){% endif %}
```

**BASH:**
```
${Type:+${Type}, }${Creator:+${Creator}, }${Website} (${Url}${Date:+ : accessed ${Date}})
```

**Liquid:**
```liquid
{% if Type %}{{ Type }}, {% endif %}{% if Creator %}{{ Creator }}, {% endif %}{{ Website }} ({{ Url }}{% if Date %} : accessed {{ Date }}{% endif %})
```