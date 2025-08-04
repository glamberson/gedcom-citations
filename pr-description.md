## Summary

This PR implements the Liquid template syntax conversion as proposed in #21. Dave Thaler expressed enthusiasm for this change during our meeting, and I've completed the full conversion of the templates extension.

## Changes

### Template Syntax Migration

All BASH-style parameter expansion patterns have been converted to Liquid template syntax:

| BASH Pattern | Liquid Pattern |
|--------------|----------------|
| `${variable}` | `{{ variable }}` |
| `${variable:-default}` | `{{ variable \| default: "default" }}` |
| `${variable:+text}` | `{% if variable %}text{% endif %}` |
| `\$` | `$` (rarely needed) |

### Safety Improvements

Liquid provides significant safety advantages over BASH-style patterns:
- **No infinite loops**: Iteration is limited (default 100, configurable)
- **No arbitrary code execution**: Only pre-registered filters and tags
- **No file system access**: Cannot read/write files
- **No network access**: Cannot make HTTP requests
- **Resource limits**: Memory and CPU limits in implementations

### Parser Availability

Mature, well-maintained parsers exist for all major languages, ensuring broad compatibility across genealogy applications.

### Examples Updated

All examples in the specification have been converted, including:
- Grave Marker Template
- Tax Roll Template
- Tax Volume Template
- Input matching patterns

## Implementation Notes

- The conversion maintains exact semantic equivalence with the original BASH patterns
- The syntax is more readable and familiar to developers
- Battle-tested at scale by GitHub Pages, Jekyll, and Shopify
- Comprehensive documentation added in the appendix

## Related Issues

Closes #21