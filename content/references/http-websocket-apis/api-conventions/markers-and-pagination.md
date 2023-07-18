---
html: markers-and-pagination.html
parent: api-conventions.html
blurb: Convention for paginating large queries into multiple responses.
---
# Markers and Pagination

Some methods return more data than can efficiently fit into one response. When there are more results than contained, the response includes a <span class="code-snippet">marker</span> field. You can use this to retrieve more pages of data across multiple calls. In each request, pass the <span class="code-snippet">marker</span> value from the previous response to resume from the point where you left off. If the <span class="code-snippet">marker</span> is omitted from a response, then you have reached the end of the data set.

The format of the <span class="code-snippet">marker</span> field is intentionally undefined. Each server can define a <span class="code-snippet">marker</span> field as desired, so it may take the form of a string, a nested object, or another type. Different servers, and different methods provided by the same server, can have different <span class="code-snippet">marker</span> definitions. Each <span class="code-snippet">marker</span> is ephemeral, and may not work as expected after 10 minutes.

<!-- MULTICODE_BLOCK_START -->

_Python_

{{ include_code("_code-samples/markers-and-pagination/py/pagination-with-markers.py", language="py") }}

_JavaScript_

{{ include_code("_code-samples/markers-and-pagination/js/pagination-with-markers.js", language="js") }}

<!-- MULTICODE_BLOCK_END -->