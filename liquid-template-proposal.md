# Proposal: Adopt Liquid Template Syntax for Citation Templates

## Summary

This proposal suggests adopting [Liquid template syntax](https://shopify.github.io/liquid/) instead of BASH-style syntax for the Source Templates Extension. Liquid provides the safety guarantees needed while maintaining developer-friendly syntax and broad language support.

## Background

The current [templates-extension.md](templates-extension.md) specification uses BASH-style parameter expansion syntax:
- `${variable}`
- `${variable:-default}`
- `${variable:+text}`

As noted in the specification itself (line 316): "there does not seem to already be general open source libraries that simply do what we want". While BASH syntax is familiar, it has no parser implementations outside of shell environments, requiring every GEDCOM implementation to write a custom parser.

## Why Liquid?

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

## Syntax Comparison

### Variables

| Feature | BASH | Liquid |
|---------|------|--------|
| Variable | `${Cemetery}` | `{{ Cemetery }}` |
| HTML escape | Manual | `{{ Cemetery \| escape }}` |

### Conditionals

| Feature | BASH | Liquid |
|---------|------|--------|
| If not empty | `${Location:+ (${Location})}` | `{% if Location %} ({{ Location }}){% endif %}` |
| Default value | `${Publisher:-n.p.}` | `{{ Publisher \| default: "n.p." }}` |

### Real Examples from Current Spec

#### Grave Marker (from line 593)

**Current (BASH):**
```yaml
scite: ${Cemetery}${Location:+ (${Location})}
```

**Proposed (Liquid):**
```yaml
scite: {{ Cemetery }}{% if Location %} ({{ Location }}){% endif %}
```

#### Tax Volume (from line 694)

**Current (BASH):**
```yaml
page: |
  ${Section:+${Section}${Page:+, }}${Page:+p. ${Page}}${Item:+${Page:+, }${Item}}
```

**Proposed (Liquid):**
```yaml
page: |
  {% if Section %}{{ Section }}{% if Page %}, {% endif %}{% endif %}
  {%- if Page %}p. {{ Page }}{% endif %}
  {%- if Item %}{% if Page %}, {% endif %}{{ Item }}{% endif %}
```

Note: The `-` in `{%-` removes preceding whitespace, giving precise control over output formatting.

#### Web Database (from line 751)

**Current (BASH):**
```yaml
publ: ${Type:+${Type}, }${Creator:+${Creator}, }${Website} (${Url}${Date:+ : accessed ${Date}})
```

**Proposed (Liquid):**
```yaml
publ: >-
  {% if Type %}{{ Type }}, {% endif %}
  {%- if Creator %}{{ Creator }}, {% endif %}
  {%- capture url_part %}{{ Url }}{% if Date %} : accessed {{ Date }}{% endif %}{% endcapture %}
  {{- Website }} ({{ url_part }})
```

## Additional Benefits

### 1. Built-in Filters

Liquid provides filters for common text transformations:

```liquid
{{ Author | split: ", " | reverse | join: " " }}  # "Smith, John" → "John Smith"
{{ Date | date: "%Y" }}                           # Extract year from date
{{ Title | capitalize }}                          # Proper case
{{ Name | upcase }}                               # Convert to uppercase
```

### 2. Clean List Handling

For multiple authors or other repeated elements:

```liquid
{% for author in Authors limit:10 -%}
  {{ author }}{% unless forloop.last %}, {% endunless %}
{%- endfor %}
```

### 3. Complex Conditionals

More readable than nested BASH expansions:

```liquid
{% if Type == "database" %}
  database
{% elsif Type == "digital" %}
  digital image
{% else %}
  {{ Type }}
{% endif %}
```

### 4. Capture for Reuse

Build complex strings once:

```liquid
{% capture full_date -%}
  {%- if Day %}{{ Day }} {% endif -%}
  {%- if Month %}{{ Month }} {% endif -%}
  {{- Year -}}
{%- endcapture %}
```

## Implementation Considerations

### 1. Escaping

BASH uses `\$` for literal `$`. Liquid rarely needs escaping, but when needed:
- `{{ "{{" }}` produces `{{`
- Or use `{% raw %}{{ variable }}{% endraw %}` to output literally

### 2. Functions

Current spec (line 338) proposes: `$(function arg1 arg2)`

Liquid approach: `{{ variable | function: arg1, arg2 }}`

Common functions as filters:
- `{{ Date | date: "%Y" }}` - Extract year
- `{{ Name | split: ", " | first }}` - Get surname
- `{{ Text | truncate: 50 }}` - Limit length

### 3. Safety Configuration

Example configuration for maximum safety:

```ruby
# Ruby example
template = Liquid::Template.parse(input, 
  error_mode: :strict,
  strict_variables: true,
  strict_filters: true
)

# Set resource limits
template.resource_limits = {
  render_length_limit: 10_000,
  render_score_limit: 100,
  assign_score_limit: 50
}
```

## Migration Path

1. **Phase 1**: Document Liquid as the recommended syntax
2. **Phase 2**: Provide automated BASH→Liquid converter
3. **Phase 3**: Update all examples in repository
4. **Phase 4**: Implementations MAY support both during transition

## Sample Converter

A simplified Python converter for basic patterns (production converters should handle edge cases like nested braces, escaped quotes, and mixed patterns):

```python
import re

def bash_to_liquid(template):
    # ${var} → {{ var }}
    template = re.sub(r'\$\{([^}:]+)\}', r'{{ \1 }}', template)
    
    # ${var:-default} → {{ var | default: "default" }}
    template = re.sub(r'\$\{([^}:]+):-([^}]+)\}', 
                     r'{{ \1 | default: "\2" }}', template)
    
    # ${var:+text} → {% if var %}text{% endif %}
    template = re.sub(r'\$\{([^}:]+):\+([^}]+)\}', 
                     r'{% if \1 %}\2{% endif %}', template)
    
    return template
```

Note: This handles common cases but doesn't address escaping, nested patterns, or edge cases. A full converter would need more robust parsing.

## Conclusion

Liquid offers:
1. **Proven safety** at massive scale (GitHub, Shopify)
2. **Available parsers** in all major languages
3. **Developer-friendly** syntax that's readable and maintainable
4. **Rich features** for citation formatting needs
5. **Active maintenance** and community support

The syntax change would make GEDCOM citation templates immediately implementable in any language without custom parser development, while providing stronger safety guarantees than the current proposal.

## References

- [Liquid Documentation](https://shopify.github.io/liquid/)
- [Liquid for Programmers](https://github.com/Shopify/liquid/wiki/Liquid-for-Programmers)
- [Liquid Safety Features](https://github.com/Shopify/liquid/wiki/Liquid-for-Programmers#security)
- [LiquidJS (JavaScript implementation)](https://liquidjs.com/)
- [Python Liquid Documentation](https://jg-rp.github.io/liquid/)