# 1. Introduction
## 1.1. General
The document describes the HTTP Request in Editor format designed to provide a simple way to create, execute, and store information about HTTP requests. The specification mostly repeats [RFC 7230](https://tools.ietf.org/html/rfc7230#section-3https://tools.ietf.org/html/rfc7230#section-3) with several extensions intended for easier requests composing and editing.

The differences between [RFC 7230](https://tools.ietf.org/html/rfc7230#section-3https://tools.ietf.org/html/rfc7230#section-3) and the HTTP Request in Editor format are highlighted using a smaller font size, e.g.:<br/>
<small>This text highlights the differences between the base HTTP message protocol and the HTTP Request in Editor format.</small>

The document is intended to serve as a complete and self-consistent specification. All readers are invited to report any errors, typos and inconsistencies.
## 1.2. Example Request
```
###
POST http://example.com/api/add
Content-Type: application/json

{
  “name”: “entity”,
  “value”: “content”
}
```

The above fragment will execute a POST request to http://example.com/api/add with the JSON message body.

## 1.3. Notation
The format is described using context-free grammar and semantics written in natural language. Context-free grammar provides a formal definition, while format semantics explains the purpose and possible use cases for the statements.

Context-free grammar is represented as a set of production rules.
 
The base token can be replaced with the child token enclosed in parentheses.

_base:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘(’ child ‘)’_<br/>

The base token can be replaced with either child-1 or child-2.

_base:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _child-1_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _child-2_<br/>

The base token can be replaced with either child or optional-child followed by child.

_base:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _[optional-child] child_<br/>

The base token can be replaced with child taken one or multiple times.

_base:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(child)+_<br/>

The base token can be replaced with either nothing or child taken one or multiple times.

_base:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(child)*_<br/>

# 2. Lexical structure
## 2.1. Base symbols
_input-character:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _any Unicode character except new-line_<br/><br/>
<small>Unlike the HTTP message format described in [RFC 7230](https://tools.ietf.org/html/rfc7230#section-3), HTTP Request in Editor supports unicode characters as part of request target, path or query.</small>

_alpha:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _ASCII Latin letters A-Z (\u0041- \u005a) or a-z (\u0061-\u007a)_<br/>

_digit:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _0-9 (\u0030-\u0039)_<br/>

_identifier-character:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _alpha_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _digit_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘-’_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘\_’_<br/>

_identifier:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(ident_ifier-character)+_<br/>

## 2.2. Line Terminators
_new-line:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _LF, also known as "newline"_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _CR, also known as "return"_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _CR LF, “return” followed by “newline”_<br/>

_new-line-with-indent:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _new-line required-whitespace_<br/>

_line-tail:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(input-character)* new-line_<br/>

## 2.3. Whitespaces
_whitespace:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _SP, also known as "space"_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _HT, also known as "horizontal tab"_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _FF, also known as "form feed"_<br/>

_optional-whitespace:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(whitespace)*_<br/>

_required-whitespace:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(whitespace)+_<br/>

## 2.4. Comments
Line comments are supported in HTTP Requests. Comments can be used before or after a request, inside the header section, or within the request body. Comments used within the request body must start from the beginning of the line with or without indent.

_line-comment:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘#’ line-tail_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘//’ line-tail_<br/>

__Example:__
```
# request comment
// request comment
```

<small>The HTTP message protocol supports comments only as part of specialized header fields; HTTP Request in Editor supports general line comments.</small>

## 2.5. Request Separators
Multiple requests defined in a single file must be separated from each other with a request separator symbol. A separator may contain comments.

_request-separator:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘###’ line-tail_<br/>

__Example:__
```
###
```
or 
```
### request comment
```

# 3. Grammar structure
## 3.1. Requests file
An HTTP Requests file is a list of HTTP requests separated by a request separator. A file may start or end with multiple request separators.

_requests-file:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(request-separator)* request (request-with-separator)* (request-separator)*_<br/>

_request-with-separator:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(request-separator)+ request_<br/>

__Example:__
```
###
GET http://example.com
###
GET http://example.com
###
```
<small>One of the most important extensions HTTP Request in Editor provides is the support of multiple requests within a single file. This capability was introduced to make the format more flexible.</small>

## 3.2. Request
An HTTP request starts with a request line followed by optional header fields, message body, response handler, and previous response references. Message body must be separated from the request line or header fields with an empty line.

_request:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _request-line new-line headers new-line [message-body] [response-handler] [response-ref]_<br/>

__Example:__
```
POST http://example.com/auth
Content-Type: application/json

< input.json 
> {% client.global.set("auth", response.body.token); %} 
<> previous-response.200.json
```
<small>Compared with the HTTP message format, two new request sections were introduced: response-ref and response-handler.</small>
### 3.2.1 Request line
A request line consists of a request method, target and the HTTP protocol version. If the request method is omitted, ‘GET’ will be used as a default. The HTTP protocol version can be also omitted.

_request-line:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _[method required-whitespace] request-target [required-whitespace http-version]_<br/>

_method:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘GET’_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘HEAD’_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘POST’_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘PUT’_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘DELETE’_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘CONNECT’_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘PATCH’_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘OPTIONS’_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘TRACE’_<br/>

_http-version:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘HTTP/’ (digit)+ ‘.’ (digit)+_<br/>

__Example:__
```
http://example.com
```
or
```
GET http://example.com HTTP/1.1
```
<small>There are two important differences in request-line between the HTTP message and the HTTP Request in Editor formats:</small><br/>
<small>&nbsp;&nbsp;&nbsp;1) The HTTP Request in Editor format pre-defines the list of supported methods; The HTTP message defines the method as any token;</small><br/>
<small>&nbsp;&nbsp;&nbsp;2) The request method and the protocol version are optional in the HTTP Request in Editor format but are required in the HTTP message.</small><br/>
#### 3.2.1.1 Request target
A request target can be an absolute path on the server, a full path to the server resources including the request scheme, host, port, and the path on the server, and asterisk path, which is intended for performing server-wide requests.

If the first request target type is used, the host must be defined in the  ‘Host’ header field; otherwise, the request can’t be executed.

_request-target:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _origin-form_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _absolute-form_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _asterisk-form_<br/>

_origin-form:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _absolute-path [‘?’ query] [‘#’ fragment]_<br/>

__Example:__
```
GET /api/get
Host: example.com
```

If the request target is a full path, it’s necessary to specify the target authority and optionally the request scheme and path on the server. ‘http’ will be used as a default value if scheme is not specified.

_absolute-form:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _[scheme ‘://’] hier-part [‘?’ query] [‘#’ fragment]_<br/>

_scheme:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘http’_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘https’_<br/>

_hier-part:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _authority [absolute-path]_<br/>

__Example:__
```
GET http://example.com/api/get
```

The asterisk form can be used for a server-wide OPTIONS request.

_asterisk-form:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘*’_<br/>

__Example:__
```
OPTIONS * HTTP/1.1
Host: http://example.com:8080
```
<small>Absolute-form and authority-form from the HTTP message format were combined into a simplified absolute-form version.</small>
#### 3.2.1.2. Authority
Authority is a target host and port. A host can be represented by the IPv4 or IPv6 address, or by a host name. 

_authority:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _host [‘:’ port]_<br/>
_port:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(digit)+_<br/>

_host:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘[‘ ipv6-address ‘]’_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _ipv4-or-reg-name_<br/>

_ipv6-address:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(any input-character except ‘/’ and ‘]’)+_<br/>
_ipv4-or-reg-name:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(any input-character except ‘/’, ‘:’, ‘?’ and ‘#’)+_<br/>

__Example:__
```
http://[::1]
```
```
http://127.0.0.1:8080
```
```
http://example.com
```

<small>HTTP Request in Editor supports unicode characters as part of request authority.</small>
#### 3.2.1.3. Resource path
The resources path on the server is represented by a number of segments separated by ‘/’ or line separators. A path segment may contain any unicode symbol except line separator, ‘/‘, ‘?’ and ‘#’. A path ends after a new line without indent or after the ‘?’ or the ‘#’ symbols.

_absolute-path:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘/’_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(path-separator segment)+_<br/>

_path-separator:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘/’_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _new-line-with-indent_<br/>

_segment:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(any input-character except ‘/’, ‘?’ and ‘#’)*_<br/>

<small>HTTP Request in Editor supports unicode characters as part of a resources path. A resources path can be split into several lines for better request readability. Line separators won’t be sent as part of the request during execution.</small>
#### 3.2.1.4. Query and Fragment
A request query may contain any unicode characters except line separators and the ‘#’ symbol.

_query:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(any input-character except ‘#’)* [new-line-with-indent query]_<br/>


__Example:__
```
http://example.com/api/get?id=42
```

A request fragment may contain any unicode characters except line separators and the ‘?’ symbol.
The fragment part of a URL is used only on a client side; therefore, it won’t be sent as part of the request.

_fragment:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(any input-character except ‘?’)* [new-line-with-indent fragment]_<br/>

__Example:__
```
http://example.com/api/get#q=hello+world
```

All non-ASCII symbols in path and query are encoded before sending; the already encoded symbols must not be encoded twice. For example, ‘%20’ must be inserted as ‘%2520’ to be sent as part of the path.

<small>HTTP Request in Editor supports unicode characters as part of the request query and fragment.</small>
### 3.2.2. Headers
Each header field consists of a case-insensitive field name followed by a colon (‘:’), optional leading whitespace, the field value, and optional trailing whitespace.

Header fields are send as is without encoding.

_headers:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(header-field new-line)*_<br/>

_header-field:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _field-name ‘:’ optional-whitespace field-value optional-whitespace_<br/>

_field-name:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(any input-character except ‘:’)+_<br/>

_field-value:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _line-tail [new-line-with-indent field-value]_<br/>

__Example:__
```
GET http://example.com/api/get?id=15
From: user@example.com
```
### 3.2.3. Message body
The message body can be represented as a simple message or a mixed type message (multipart-form-data). A request message can be inserted in-place or from a file.

_message-body:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _messages_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _multipart-form-data_<br/>

_messages:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _message-line [ new-line message-line ]_<br/>

_message-line:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _(any input-character except ‘< ’, ’<> ’ and ‘###’) line-tail_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _input-file-ref_<br/>

_input-file-ref:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘<’ required-whitespace file-path_<br/>

_file-path:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _line-tail_<br/>

__Example:__
```
POST http://example.com/api/add
Content-Type: application/json

{ “key”: “value” }
```
Or 
```
POST http://example.com/api/add
Content-Type: application/json

< ./input.json
```
<small>Unlike the HTTP message format, the HTTP Request in Editor format supports sending the message body from a separate file.</small>

#### 3.2.3.1. Multipart-form-data
Multipart form data is a mixed message body consisting of several multipart fields separated by a boundary. A multipart field consists of optional header fields and a simple request message. Message body type and multipart boundary should be defined in the ‘Content-Type’ request header field.

_multipart-form-data:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _multipart-field [multipart-form-data] boundary_<br/>

_multipart-field:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _boundary (header-field new-line)* new-line [messages]_<br/>

__Example:__
```
POST http://example.com/api/upload
Content-Type: multipart/form-data; boundary=abcd

--abcd
Content-Disposition: form-data; name="text"

Text
--abcd
Content-Disposition: form-data; name="file_to_send"; filename="input.txt"

< ./input.txt
--abcd--
```

### 3.2.4. Response handler
A custom response handler script can be specified at the end of the request to capture and save information from the response to a global variable, or to perform assertions. 
A script can be inserted in-place or by referencing a separate file. An in-place script can’t contain ‘%}’ or request separator (‘###’).


_response-handler:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘>’ required-whitespace ‘{%’ handler-script ‘%}’_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘>’ required-whitespace file-path_<br/>

__Example:__
```
GET http://example.com/auth

> {% client.global.set("auth", response.body.token);%}
```

<small>The response handler section is not defined in the HTTP message format; it was introduced in HTTP Request in Editor.</small>
### 3.2.5. Response reference
A reference to a previously received response is represented with the ‘<> ’ symbol and the path to a file. References are  used to provide easy access to the requests execution history. The section may be useful for comparing different responses.

_response-ref:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘<>’ required-whitespace file-path_<br/>

__Example:__
```
GET http://example.com

<> previous-response.200.json
```
<small>The response reference section is not defined in the HTTP message format; it was introduced in HTTP Request in Editor.</small>

### 3.2.6. Environment variables
Environment variables are used for avoiding unnecessary data duplication in requests or for providing an easy way of switching between the development and production environments. They can be used inside request target, header fields and message body.
Each environment variable is represented by a case-sensitive identifier surrounded by double curly braces. 

_env-variable:_<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _‘{{’ optional-whitespace identifier optional-whitespace ‘}}’_<br/>

__Example:__
```
GET http://{{host}}/api/get?id={{ element-id }}
```

<small>Environment variables are not defined in the HTTP message format; they were introduced in HTTP Request in Editor.</small>

# 4. Execution
__TODO:__ describe request execution process: preprocessing (decoding and encoding, whitespace trimming), variable substitution, execution, response handler (update global variables, perform assertions).
## 4.1. Encoding
### 4.1.1. Host
__TODO:__ specify how non-ASCII symbols in host should be handled. [RFC3986](https://tools.ietf.org/html/rfc3986#section-3.2.2)

### 4.1.2. Resource path and query
All non-ASCII symbols in path and query are encoded before sending; the already encoded symbols must not be encoded twice. For example, ‘%20’ must be inserted as ‘%2520’ to be sent as part of the path.

### 4.1.3. Request body
Request body should be encoded according to the ‘Content-Type’ header field. If it’s not specified, the ‘UTF-8’ encoding should be used.

## 4.2. Whitespaces
### 4.2.1. Resource path and query
Whitespaces around each line of the request path and the query will be trimmed. To send leading or trailing whitespaces, use the encoded version.

__Example:__
```
http://example.com/
    api
    /get
```

The request will be sent as “http://example.com/api/get”.

__Example:__
```
http://example.com/
    %20api%20
    +/get+
```
The request will be sent as “http://example.com/ api / get ”.

### 4.2.2. Request body
If the request body is configured in-place, whitespaces around it will be trimmed. To send leading or trailing whitespaces as part of the request body, send it from a separate file.

__Example:__
```
###
POST http://example.com/add


message-body

###
```

Only “message-body” will be send as the request body.

__Example:__
```
###
POST http://example.com/add

< input.txt
###
```

_input.txt:_
```

message-body

```

The whole content of the ‘input.txt’ will be send as request body, i.e. “\nmessage-body\n”.

## 4.3. Multipart/form-data
Multipart form body part can be sent either as a file or as a plain string body. Which one to use is regulated by the ‘filename’ option in the body part header.

__Example:__<br/>
The first body part will be sent as a string and the second one – as a file.
```
POST http://example.com/api/upload
Content-Type: multipart/form-data; boundary=abcd

--abcd
Content-Disposition: form-data; name="text"

Text
--abcd
Content-Disposition: form-data; name="file_to_send"; filename="input.txt"

< ./input.txt
--abcd--
```
## 4.4 Environment variables
__TODO:__ how to define environment, how value is inserted

## 4.5 Response handler script
Response handler script should be written in JavaScript [ECMAScript 5.1 specification](https://www.ecma-international.org/ecma-262/5.1/)

__TODO:__ describe API available in response handler script
