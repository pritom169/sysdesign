## URL vs. URI vs. URN


### URI (Uniform Resource Identifier)

"The Identifier"

A URI is a string of characters used to identify a resource. It doesn't necessarily tell you how to access the resource or where it is located—it just identifies it.

Function: Identification.

Scope: It is the parent term that covers both URLs and URNs.

Example: urn:isbn:0-486-27777-3 (This identifies a specific book, but doesn't tell you where to buy it).

### URL (Uniform Resource Locator)

"The Address"

This is the most common type of URI. A URL identifies a resource by telling you where it is located and how to retrieve it.

Function: Location + Access Method.

Key Feature: It includes the protocol (scheme) to use, such as https, ftp, or mailto.

Structure: protocol://domain:port/path?query_string#fragment_id

Protocol: https:// (How to talk to the server)

Domain: www.example.com (Where the server is)

Path: /images/logo.png (Where the file is on that server)

Example: https://www.google.com/search?q=cat (This tells your browser to use HTTPS to go to Google's server and look for the search results for "cat").

### URN (Uniform Resource Name)

"The Permanent Name"

A URN identifies a resource by a unique name that remains constant, even if the resource moves to a different location. It does not tell you how to access the resource, only what it is called.

Function: Naming + Uniqueness.

Key Feature: It is persistent. If a file moves from Server A to Server B, its URL changes, but its URN remains the same.

Structure: urn:<namespace>:<specific-string>

Real-world usage: URNs are often used in libraries, digital documents, and standard codes.

Example: urn:isbn:978-0-123-45678-9 (This is the International Standard Book Number. It identifies that specific book regardless of whether it is in a library, a bookstore, or a PDF on your computer).

---

