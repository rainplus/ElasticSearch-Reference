## Pattern Replace Token Filter

The `pattern_replace` token filter allows to easily handle string replacements based on a regular expression. The regular expression is defined using the `pattern` parameter, and the replacement string can be provided using the `replacement` parameter (supporting referencing the original text, as explained [here](http://docs.oracle.com/javase/6/docs/api/java/util/regex/Matcher.html#appendReplacement\(java.lang.StringBuffer,%20java.lang.String\))).

![Warning](/images/icons/warning.png)

### Beware of Pathological Regular Expressions

The pattern replace token filter uses [Java Regular Expressions](http://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html).

A badly written regular expression could run very slowly or even throw a StackOverflowError and cause the node it is running on to exit suddenly.

Read more about [pathological regular expressions and how to avoid them](http://www.regular-expressions.info/catastrophic.html).
