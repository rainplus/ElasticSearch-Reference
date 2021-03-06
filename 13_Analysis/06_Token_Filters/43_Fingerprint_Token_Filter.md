## Fingerprint Token Filter

The `fingerprint` token filter emits a single token which is useful for fingerprinting a body of text, and/or providing a token that can be clustered on. It does this by sorting the tokens, deduplicating and then concatenating them back into a single token.

For example, the tokens `["the", "quick", "quick", "brown", "fox", "was", "very", "brown"]` will be transformed into a single token: `"brown fox quick the very was"`. Notice how the tokens were sorted alphabetically, and there is only one `"quick"`.

The following are settings that can be set for a `fingerprint` token filter type:

Setting | Description  
---|---    
`separator`| Defaults to a space.    
`max_output_size`| Defaults to `255`.  
  
### Maximum token size

Because a field may have many unique tokens, it is important to set a cutoff so that fields do not grow too large. The `max_output_size` setting controls this behavior. If the concatenated fingerprint grows larger than `max_output_size`, the token filter will exit and will not emit a token (e.g. the field will be empty).
