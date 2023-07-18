XRP Ledger APIs generally use strings, rather than native JSON numbers, to represent numeric amounts of currency for both [XRP](xrp.html) and [tokens](tokens.html). This protects against a loss of precision when using JSON parsers, which may automatically try to represent all JSON numbers in a floating-point format. Within the String value, the numbers are serialized in the same way as native JSON numbers:

* Base-10.
* Non-zero-prefaced.
* May contain <span class="code-snippet">.</span> as a decimal point. For example, ½ is represented as <span class="code-snippet">0.5</span>. (American style, not European)
* Negative amounts start with the character <span class="code-snippet">-</span>.
* May contain <span class="code-snippet">E</span> or <span class="code-snippet">e</span> to indicate being raised to a power of 10 (scientific notation). For example, <span class="code-snippet">1.2E5</span> is equivalent to 1.2×10<sup>5</sup>, or <span class="code-snippet">120000</span>. Negative exponents are also possible.
* No comma (<span class="code-snippet">,</span>) characters are used.
