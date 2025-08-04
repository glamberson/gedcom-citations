# Liquid Template Conversion Summary

## Overview

This PR implements the conversion from BASH-style parameter expansion to Liquid template syntax in the Source Templates Extension, as discussed in PR #21.

## Changes Made

### 1. Updated `templates-extension.md`

Converted all BASH-style patterns to Liquid syntax throughout the document:

#### Pattern Conversions

| BASH Pattern | Liquid Pattern | Description |
|--------------|----------------|-------------|
| `${variable}` | `{{ variable }}` | Simple variable substitution |
| `${variable:-default}` | `{{ variable \| default: "default" }}` | Variable with default value |
| `${variable:+text}` | `{% if variable %}text{% endif %}` | Conditional text insertion |
| `\$` | `$` | Literal dollar sign (no escaping needed) |

#### Example Conversions

**Grave Marker Template:**
- BASH: `scite: ${Cemetery}${Location:+ (${Location})}`
- Liquid: `scite: {{ Cemetery }}{% if Location %} ({{ Location }}){% endif %}`

**Tax Volume Template:**
- BASH: `page: ${Section:+${Section}${Page:+, }}${Page:+p. ${Page}}${Item:+${Page:+, }${Item}}`
- Liquid: 
  ```liquid
  page: |
    {% if Section %}{{ Section }}{% if Page %}, {% endif %}{% endif %}
    {%- if Page %}p. {{ Page }}{% endif %}
    {%- if Item %}{% if Page %}, {% endif %}{{ Item }}{% endif %}
  ```

**Web Database Template:**
- BASH: `publ: ${Type:+${Type}, }${Creator:+${Creator}, }${Website} (${Url}${Date:+ : accessed ${Date}})`
- Liquid: `publ: {% if Type %}{{ Type }}, {% endif %}{% if Creator %}{{ Creator }}, {% endif %}{{ Website }} ({{ Url }}{% if Date %} : accessed {{ Date }}{% endif %})`

### 2. Added Comprehensive Appendix

Added "Appendix: Template Patterns" section with:
- Explanation of why Liquid was chosen
- Safety guarantees provided by Liquid
- Parser availability table for all major languages
- Migration guide from BASH to Liquid syntax
- Common Liquid filters and patterns
- Example conversions

### 3. Updated README.md

Added note about Liquid template syntax adoption to the Source Templates Extension description.

### 4. Preserved Original

The original BASH-style version is preserved as `templates-extension-bash.md` for reference.

## Benefits

1. **Safety**: Liquid provides built-in protection against infinite loops and arbitrary code execution
2. **Parser Availability**: Mature parsers exist for Ruby, Python, JavaScript, Java, Go, PHP, C#, and Rust
3. **Battle-tested**: Used by GitHub Pages and Shopify, processing millions of templates daily
4. **Familiar Syntax**: Easy migration from BASH patterns
5. **Rich Features**: Built-in filters for text manipulation, date formatting, etc.

## Compatibility

The conversion maintains semantic equivalence with the original BASH patterns while providing a more robust and portable solution for template evaluation across different GEDCOM implementations.