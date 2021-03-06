# NAME

[HTML5::DOM](https://metacpan.org/pod/HTML5::DOM) - Super fast html5 DOM library with css selectors (based on Modest/MyHTML)

<div>
    <a href="https://travis-ci.org/Azq2/perl-html5-dom"><img src="https://travis-ci.org/Azq2/perl-html5-dom.svg?branch=master"></a>
    <a href="https://metacpan.org/pod/HTML5::DOM"><img src="https://img.shields.io/cpan/v/HTML5-DOM.svg"></a>
</div>

# SYNOPSIS

```perl
use warnings;
use strict;
use HTML5::DOM;

# create parser object
my $parser = HTML5::DOM->new;

# parse some html
my $tree = $parser->parse('
 <label>Some list of OS:</lbnel>
 <ul class="list" data-what="os" title="OS list">
    <li>UNIX</li>
    <li>Linux</li>
    <!-- comment -->
    <li>OSX</li>
    <li>Windows</li>
    <li>FreeBSD</li>
 </ul>
');

# find one element by CSS selector
my $ul = $tree->at('ul.list');

# prints tag
print $ul->tag."\n"; # ul

# check if <ul> has class list
print "<ul> has class .list\n" if ($ul->classList->has('list'));

# add some class
$ul->classList->add('os-list');

# prints <ul> classes
print $ul->className."\n"; # list os-list

# prints <ul> attribute title
print $ul->attr("title")."\n"; # OS list

# changing <ul> attribute title
$ul->attr("title", "OS names list");

# find all os names
$ul->find('li')->each(sub {
 my ($node, $index) = @_;
 print "OS #$index: ".$node->text."\n";
});

# we can use precompiled selectors
my $css_parser = HTML5::DOM::CSS->new;
my $selector = $css_parser->parseSelector('li');

# remove OSX from OS
$ul->find($selector)->[2]->remove();

# serialize tree
print $tree->html."\n";

# TODO: more examples in SYNOPSIS
# But you can explore API documentation.
# My lib have simple API, which is intuitively familiar to anyone who used the DOM.
# Sorry, if my english is very bad, in future, i may be fix this. 
# Or you can help me with this, by sending pull request.
```

# DESCRIPTION

[HTML5::DOM](https://metacpan.org/pod/HTML5::DOM) is a fast HTML5 parser and DOM manipulatin library with CSS4 selectors, fully conformant with the HTML5 specification.

It based on  [https://github.com/lexborisov/Modest](https://github.com/lexborisov/Modest) as selector engine and [https://github.com/lexborisov/myhtml](https://github.com/lexborisov/myhtml) as HTML5 parser. 

### Key features

- Really fast HTML parsing, can use threads (by default).
- Supports async parsing.
- Supports parsing by chunks.
- Fully conformant with the HTML5 specification.
- Fast CSS4 selectors.
- Any manipulations using DOM-like API.
- Auto-detect input encoding.
- Fully integration in perl and memory management. You don't care about "free" or "destroy".

# HTML5::DOM

HTML5 parser object.

## new

```perl
# with default options
my $parser = HTML5::DOM->new;

# override some options, if you need
my $parser = HTML5::DOM->new({
   threads                 => 2,
   async                   => 0, 
   ignore_whitespace       => 0, 
   ignore_doctype          => 0, 
   scripts                 => 0, 
   encoding                => "auto", 
   default_encoding        => "UTF-8", 
   encoding_use_meta       => 1, 
   encoding_use_bom        => 1, 
   encoding_prescan_limit  => 1024
});
```

Creates new parser object with options. See ["PARSER OPTIONS"](#parser-options) for details. 

### parse

```perl
my $html = '<div>Hello world!</div>';

# parsing with options defined in HTML5::DOM->new
my $tree = $parser->parse($html);

# parsing with custom options (extends options defined in HTML5::DOM->new)
my $tree = $parser->parse($html, {
    scripts     => 0, 
});
```

Parse html string and return [HTML5::DOM::Tree](#html5domtree) object.

### parseChunkStart

```perl
# start chunked parsing with options defined in HTML5::DOM->new
# call parseChunkStart without options is useless, 
# because first call of parseChunk automatically call parseChunkStart. 
$parser->parseChunkStart();

# start chunked parsing with custom options (extends options defined in HTML5::DOM->new)
$parser->parseChunkStart({
   scripts     => 0, 
});
```

Init chunked parsing. See ["PARSER OPTIONS"](#parser-options) for details. 

### parseChunk

```
$parser->parseChunkStart()->parseChunk('<')->parseChunk('di')->parseChunk('v>');
```

Parse chunk of html stream.

### parseChunkEnd

```perl
# start some chunked parsing
$parser->parseChunk('<')->parseChunk('di')->parseChunk('v>');

# end parsing and get tree
my $tree = $parser->parseChunkEnd();
```

Completes chunked parsing and return [HTML5::DOM::Tree](#html5domtree) object.

# HTML5::DOM::Tree

DOM tree object.

### createElement

```perl
# create new node with tag "div"
my $node = $tree->createElement("div");

# create new node with tag "g" with namespace "svg"
my $node = $tree->createElement("div", "svg");
```

Create new [HTML5::DOM::Element](#html5domelement) with specified tag and namespace.

### createComment

```perl
# create new comment
my $node = $tree->createComment(" ololo ");

print $node->html; # <!-- ololo -->
```

Create new [HTML5::DOM::Comment](#html5domcomment) with specified value.

### createTextNode

```perl
# create new text node
my $node = $tree->createTextNode("psh psh ololo i am driver of ufo >>>");

print $node->html; # psh psh ololo i am driver of ufo &gt;&gt;&gt;
```

Create new [HTML5::DOM::Text](#html5domtext) with specified value.

### parseFragment

```perl
my $fragment = $tree->parseFragment($html);
my $fragment = $tree->parseFragment($html, $context);
my $fragment = $tree->parseFragment($html, $context, $context_ns);
my $fragment = $tree->parseFragment($html, $context, $context_ns, $options);
```

Parse fragment html and create new [HTML5::DOM::Fragment](#html5domfragment).
For more details about fragments: [https://html.spec.whatwg.org/multipage/parsing.html#parsing-html-fragments](https://html.spec.whatwg.org/multipage/parsing.html#parsing-html-fragments)

- `$html` - html fragment string
- `$context` - context tag name, default `div`
- `$context_ns` - context tag namespace, default `html`
- `$options` - parser options

    See ["PARSER OPTIONS"](#parser-options) for details. 

```perl
# simple create new fragment
my $node = $tree->parseFragment("some <b>bold</b> and <i>italic</i> text");

# create new fragment node with custom context tag/namespace and options
my $node = $tree->parseFragment("some <b>bold</b> and <i>italic</i> text", "div", "html", {
   # some options override
   encoding => "windows-1251"
});

print $node->html; # some <b>bold</b> and <i>italic</i> text
```

### document

```perl
my $node = $tree->document;
```

Return [HTML5::DOM::Document](#html5domdocument) node of current tree;

### root

```perl
my $node = $tree->root;
```

Return root node of current tree. (always &lt;html>)

### head

```perl
my $node = $tree->head;
```

Return &lt;head> node of current tree. 

### body

```perl
my $node = $tree->body;
```

Return &lt;body> node of current tree. 

### at

### querySelector

```perl
my $node = $tree->at($selector);
my $node = $tree->querySelector($selector); # alias
```

Find one element node in tree using [CSS Selectors Level 4](https://www.w3.org/TR/selectors-4/)

Return node, or `undef` if not find.

- `$selector` - selector query as plain text or precompiled as [HTML5::DOM::CSS::Selector](#html5domcssselector) or 
[HTML5::DOM::CSS::Selector](#html5domcssselectorentry).

```perl
my $tree = HTML5::DOM->new->parse('<div class="red">red</div><div class="blue">blue</div>')
my $node = $tree->at('body > div.red');
print $node->html; # <div class="red">red</div>
```

### find

### querySelectorAll

```perl
my $collection = $tree->find($selector);
my $collection = $tree->querySelectorAll($selector); # alias
```

Find all element nodes in tree using [CSS Selectors Level 4](https://www.w3.org/TR/selectors-4/)

Return [HTML5::DOM::Collection](#html5domcollection).

- `$selector` - selector query as plain text or precompiled as [HTML5::DOM::CSS::Selector](#html5domcssselector) or 
[HTML5::DOM::CSS::Selector](#html5domcssselectorentry).

```perl
my $tree = HTML5::DOM->new->parse('<div class="red">red</div><div class="blue">blue</div>')
my $collection = $tree->at('body > div.red, body > div.blue');
print $collection->[0]->html; # <div class="red">red</div>
print $collection->[1]->html; # <div class="red">blue</div>
```

### findId

### getElementById

```perl
my $collection = $tree->findId($tag);
my $collection = $tree->getElementById($tag); # alias
```

Find element node with specified id.

Return [HTML5::DOM::Node](#html5domnode) or `undef`.

```perl
my $tree = HTML5::DOM->new->parse('<div class="red">red</div><div class="blue" id="test">blue</div>')
my $node = $tree->findId('test');
print $node->html; # <div class="blue" id="test">blue</div>
```

### findTag

### getElementsByTagName

```perl
my $collection = $tree->findTag($tag);
my $collection = $tree->getElementsByTagName($tag); # alias
```

Find all element nodes in tree with specified tag name.

Return [HTML5::DOM::Collection](#html5domcollection).

```perl
my $tree = HTML5::DOM->new->parse('<div class="red">red</div><div class="blue">blue</div>')
my $collection = $tree->findTag('div');
print $collection->[0]->html; # <div class="red">red</div>
print $collection->[1]->html; # <div class="red">blue</div>
```

### findClass

### getElementsByClassName

```perl
my $collection = $tree->findClass($class);
my $collection = $tree->getElementsByClassName($class); # alias
```

Find all element nodes in tree with specified class name.
This is more fast equivalent to \[class~="value"\] selector.

Return [HTML5::DOM::Collection](#html5domcollection).

```perl
my $tree = HTML5::DOM->new
   ->parse('<div class="red color">red</div><div class="blue color">blue</div>');
my $collection = $tree->findClass('color');
print $collection->[0]->html; # <div class="red color">red</div>
print $collection->[1]->html; # <div class="red color">blue</div>
```

### findAttr

### getElementByAttribute

```perl
# Find all elements with attribute
my $collection = $tree->findAttr($attribute);
my $collection = $tree->getElementByAttribute($attribute); # alias

# Find all elements with attribute and mathcing value
my $collection = $tree->findAttr($attribute, $value, $case = 0, $cmp = '=');
my $collection = $tree->getElementByAttribute($attribute, $value, $case = 0, $cmp = '='); # alias
```

Find all element nodes in tree with specified attribute and optional matching value.

Return [HTML5::DOM::Collection](#html5domcollection).

```perl
my $tree = HTML5::DOM->new
   ->parse('<div class="red color">red</div><div class="blue color">blue</div>');
my $collection = $tree->findAttr('class', 'CoLoR', 1, '~');
print $collection->[0]->html; # <div class="red color">red</div>
print $collection->[1]->html; # <div class="red color">blue</div>

```

CSS selector analogs:

```perl
# [$attribute=$value]
my $collection = $tree->findAttr($attribute, $value, 0, '=');

# [$attribute=$value i]
my $collection = $tree->findAttr($attribute, $value, 1, '=');

# [$attribute~=$value]
my $collection = $tree->findAttr($attribute, $value, 0, '~');

# [$attribute|=$value]
my $collection = $tree->findAttr($attribute, $value, 0, '|');

# [$attribute*=$value]
my $collection = $tree->findAttr($attribute, $value, 0, '*');

# [$attribute^=$value]
my $collection = $tree->findAttr($attribute, $value, 0, '^');

# [$attribute$=$value]
my $collection = $tree->findAttr($attribute, $value, 0, '$');
```

### encoding

### encodingId

```
print "encoding: ".$tree->encoding."\n"; # UTF-8
print "encodingId: ".$tree->encodingId."\n"; # 0
```

Return current tree encoding. See ["ENCODINGS"](#encodings) for details. 

### tag2id

```
print "tag id: ".HTML5::DOM->TAG_A."\n"; # tag id: 4
print "tag id: ".$tree->tag2id("a")."\n"; # tag id: 4
```

Convert tag name to id. Return 0 (HTML5::DOM->TAG\_\_UNDEF), if tag not exists in tree.
See ["TAGS"](#tags) for tag constants list. 

### id2tag

```
print "tag name: ".$tree->id2tag(4)."\n"; # tag name: a
print "tag name: ".$tree->id2tag(HTML5::DOM->TAG_A)."\n"; # tag name: a
```

Convert tag id to name. Return `undef`, if tag id not exists in tree.
See ["TAGS"](#tags) for tag constants list. 

### namespace2id

```
print "ns id: ".HTML5::DOM->NS_HTML."\n"; # ns id: 1
print "ns id: ".$tree->namespace2id("html")."\n"; # ns id: 1
```

Convert namespace name to id. Return 0 (HTML5::DOM->NS\_UNDEF), if namespace not exists in tree.
See ["NAMESPACES"](#namespaces) for namespace constants list. 

### id2namespace

```
print "ns name: ".$tree->id2namespace(1)."\n"; # ns name: html
print "ns name: ".$tree->id2namespace(HTML5::DOM->NS_HTML)."\n"; # ns name: html
```

Convert namespace id to name. Return `undef`, if namespace id not exists.
See ["NAMESPACES"](#namespaces) for namespace constants list. 

### wait

```perl
my $parser = HTML5::DOM->new({async => 1});
my $tree = $parser->parse($some_big_html_file);
# ...some your work...
$tree->wait; # wait before parsing threads done
```

Blocking wait for tree parsing done. Only for async mode.

### parsed

```perl
my $parser = HTML5::DOM->new({async => 1});
my $tree = $parser->parse($some_big_html_file);
# ...some your work...
while (!$tree->parsed); # wait before parsing threads done
```

Non-blocking way for check if tree parsing done. Only for async mode.

### parser

```perl
my $parser = $tree->parser;
```

Return parent [HTML5::DOM](#html5dom).

# HTML5::DOM::Node

DOM node object.

### tag

### nodeName

```perl
my $tag_name = $node->tag;
my $tag_name = $node->nodeName; # uppercase
my $tag_name = $node->tagName;  # uppercase
```

Return node tag name (eg. div or span)

```
$node->tag($tag);
$node->nodeName($tag); # alias
$node->tagName($tag);  # alias
```

Set new node tag name. Allow only for [HTML5::DOM::Element](#html5domelement) nodes.

```
print $node->html; # <div></div>
$node->tag('span');
print $node->html; # <span></span>
print $node->tag; # span
print $node->tag; # SPAN
```

### tagId

```perl
my $tag_id = $node->tagId;
```

Return node tag id. See ["TAGS"](#tags) for tag constants list.

```
$node->tagId($tag_id);
```

Set new node tag id. Allow only for [HTML5::DOM::Element](#html5domelement) nodes.

```
print $node->html; # <div></div>
$node->tagId(HTML5::DOM->TAG_SPAN);
print $node->html; # <span></span>
print $node->tagId; # 117
```

### namespace

```perl
my $tag_ns = $node->namespace;
```

Return node namespace (eg. html or svg)

```
$node->namespace($namespace);
```

Set new node namespace name. Allow only for [HTML5::DOM::Element](#html5domelement) nodes.

```
print $node->namespace; # html
$node->namespace('svg');
print $node->namespace; # svg
```

### namespaceId

```perl
my $tag_ns_id = $node->namespaceId;
```

Return node namespace id. See ["NAMESPACES"](#namespaces) for tag constants list.

```
$node->namespaceId($tag_id);
```

Set new node namespace by id. Allow only for [HTML5::DOM::Element](#html5domelement) nodes.

```
print $node->namespace; # html
$node->namespaceId(HTML5::DOM->NS_SVG);
print $node->namespaceId; # 3
print $node->namespace; # svg
```

### tree

```perl
my $tree = $node->tree;

```

Return parent [HTML5::DOM::Tree](#html5domtree).

### nodeType

```perl
my $type = $node->nodeType;

```

Return node type. All types:

```perl
HTML5::DOM->ELEMENT_NODE                   => 1, 
HTML5::DOM->ATTRIBUTE_NODE                 => 2,   # not supported
HTML5::DOM->TEXT_NODE                      => 3, 
HTML5::DOM->CDATA_SECTION_NODE             => 4,   # not supported
HTML5::DOM->ENTITY_REFERENCE_NODE          => 5,   # not supported
HTML5::DOM->ENTITY_NODE                    => 6,   # not supported
HTML5::DOM->PROCESSING_INSTRUCTION_NODE    => 7,   # not supported
HTML5::DOM->COMMENT_NODE                   => 8, 
HTML5::DOM->DOCUMENT_NODE                  => 9, 
HTML5::DOM->DOCUMENT_TYPE_NODE             => 10, 
HTML5::DOM->DOCUMENT_FRAGMENT_NODE         => 11, 
HTML5::DOM->NOTATION_NODE                  => 12   # not supported
```

Compatible with: [https://developer.mozilla.org/ru/docs/Web/API/Node/nodeType](https://developer.mozilla.org/ru/docs/Web/API/Node/nodeType)

### next

### nextElementSibling

```perl
my $node2 = $node->next;
my $node2 = $node->nextElementSibling; # alias
```

Return next sibling element node

```perl
my $tree = HTML5::DOM->new->parse('
   <ul>
       <li>Linux</li>
       <!-- comment -->
       <li>OSX</li>
       <li>Windows</li>
   </ul>
');
my $li = $tree->at('ul li');
print $li->text;               # Linux
print $li->next->text;         # OSX
print $li->next->next->text;   # Windows
```

### prev

### previousElementSibling

```perl
my $node2 = $node->prev;
my $node2 = $node->previousElementSibling; # alias
```

Return previous sibling element node

```perl
my $tree = HTML5::DOM->new->parse('
   <ul>
       <li>Linux</li>
       <!-- comment -->
       <li>OSX</li>
       <li class="win">Windows</li>
   </ul>
');
my $li = $tree->at('ul li.win');
print $li->text;               # Windows
print $li->prev->text;         # OSX
print $li->prev->prev->text;   # Linux
```

### nextNode

### nextSibling

```perl
my $node2 = $node->nextNode;
my $node2 = $node->nextSibling; # alias
```

Return next sibling node

```perl
my $tree = HTML5::DOM->new->parse('
   <ul>
       <li>Linux</li>
       <!-- comment -->
       <li>OSX</li>
       <li>Windows</li>
   </ul>
');
my $li = $tree->at('ul li');
print $li->text;                       # Linux
print $li->nextNode->text;             # <!-- comment -->
print $li->nextNode->nextNode->text;   # OSX
```

### prevNode

### previousSibling

```perl
my $node2 = $node->prevNode;
my $node2 = $node->previousSibling; # alias
```

Return previous sibling node

```perl
my $tree = HTML5::DOM->new->parse('
   <ul>
       <li>Linux</li>
       <!-- comment -->
       <li>OSX</li>
       <li class="win">Windows</li>
   </ul>
');
my $li = $tree->at('ul li.win');
print $li->text;                       # Windows
print $li->prevNode->text;             # OSX
print $li->prevNode->prevNode->text;   # <!-- comment -->
```

### first

### firstElementChild

```perl
my $node2 = $node->first;
my $node2 = $node->firstElementChild; # alias
```

Return first children element

```perl
my $tree = HTML5::DOM->new->parse('
   <ul>
       <!-- comment -->
       <li>Linux</li>
       <li>OSX</li>
       <li class="win">Windows</li>
   </ul>
');
my $ul = $tree->at('ul');
print $ul->first->text; # Linux
```

### last

### lastElementChild

```perl
my $node2 = $node->last;
my $node2 = $node->lastElementChild; # alias
```

Return last children element

```perl
my $tree = HTML5::DOM->new->parse('
   <ul>
       <li>Linux</li>
       <li>OSX</li>
       <li class="win">Windows</li>
       <!-- comment -->
   </ul>
');
my $ul = $tree->at('ul');
print $ul->last->text; # Windows
```

### firstNode

### firstChild

```perl
my $node2 = $node->firstNode;
my $node2 = $node->firstChild; # alias
```

Return first children node

```perl
my $tree = HTML5::DOM->new->parse('
   <ul>
       <!-- comment -->
       <li>Linux</li>
       <li>OSX</li>
       <li class="win">Windows</li>
   </ul>
');
my $ul = $tree->at('ul');
print $ul->firstNode->html; # <!-- comment -->
```

### lastNode

### lastChild

```perl
my $node2 = $node->lastNode;
my $node2 = $node->lastChild; # alias
```

Return last children node

```perl
my $tree = HTML5::DOM->new->parse('
   <ul>
       <li>Linux</li>
       <li>OSX</li>
       <li class="win">Windows</li>
       <!-- comment -->
   </ul>
');
my $ul = $tree->at('ul');
print $ul->lastNode->html; # <!-- comment -->
```

### html

Universal html serialization and fragment parsing acessor, for single human-friendly api.

```perl
my $html = $node->html();
my $node = $node->html($new_html);
```

- As getter this similar to [outerText](#outertext)
- As setter this similar to [innerText](#innertext)
- As setter for non-element nodes this similar to [nodeValue](#nodevalue)

```perl
my $tree = HTML5::DOM->new->parse('<div id="test">some   text <b>bold</b></div>');

# get text content for element
my $node = $tree->at('#test');
print $node->html;                     # <div id="test">some   text <b>bold</b></div>
$comment->html('<b>new</b>');
print $comment->html;                  # <div id="test"><b>new</b></div>

my $comment = $tree->createComment(" comment text ");
print $comment->html;                  # <!-- comment text -->
$comment->html(' new comment text ');
print $comment->html;                  # <!-- new comment text -->

my $text_node = $tree->createTextNode("plain text >");
print $text_node->html;                # plain text &gt;
$text_node->html('new>plain>text');
print $text_node->html;                # new&gt;plain&gt;text
```

### innerHTML

### outerHTML

- HTML serialization of the node's descendants. 

    ```perl
    my $html = $node->html;
    my $html = $node->outerHTML;
    ```

    Example:

    ```perl
    my $tree = HTML5::DOM->new->parse('<div id="test">some <b>bold</b> test</div>');
    print $tree->outerHTML;                         # <div id="test">some <b>bold</b> test</div>
    print $tree->createComment(' test ')->outerHTML;  # <!-- test -->
    print $tree->createTextNode('test')->outerHTML; # test
    ```

- HTML serialization of the node and its descendants.

    ```perl
    # serialize descendants, without node
    my $html = $node->innerHTML;
    ```

    Example:

    ```perl
    my $tree = HTML5::DOM->new->parse('<div id="test">some <b>bold</b> test</div>');
    print $tree->innerHTML;                         # some <b>bold</b> test
    print $tree->createComment(' test ')->innerHTML;  # <!-- test -->
    print $tree->createTextNode('test')->innerHTML; # test
    ```

- Removes all of the element's descendants and replaces them with nodes constructed by parsing the HTML given in the string **$new\_html**.

    ```perl
    # parse fragment and replace child nodes with it
    my $html = $node->html($new_html);
    my $html = $node->innerHTML($new_html);
    ```

    Example:

    ```perl
    my $tree = HTML5::DOM->new->parse('<div id="test">some <b>bold</b> test</div>');
    print $tree->at('#test')->innerHTML('<i>italic</i>');
    print $tree->body->innerHTML;  # <div id="test"><i>italic</i></div>
    ```

- Replaces the element and all of its descendants with a new DOM tree constructed by parsing the specified **$new\_html**.

    ```perl
    # parse fragment and node in parent node childs with it
    my $html = $node->outerHTML($new_html);
    ```

    Example:

    ```perl
    my $tree = HTML5::DOM->new->parse('<div id="test">some <b>bold</b> test</div>');
    print $tree->at('#test')->outerHTML('<i>italic</i>');
    print $tree->body->innerHTML;  # <i>italic</i>
    ```

See, for more info:

[https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML)

[https://developer.mozilla.org/en-US/docs/Web/API/Element/outerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/outerHTML)

### text

Universal text acessor, for single human-friendly api. 

```perl
my $text = $node->text();
my $node = $node->text($new_text);
```

- For [HTML5::DOM::Text](#html5domtext) is similar to [nodeValue](#nodevalue) (as setter/getter)
- For [HTML5::DOM::Comment](#html5domcomment) is similar to [nodeValue](#nodevalue) (as setter/getter)
- For [HTML5::DOM::DocType](#html5domdoctype) is similar to [nodeValue](#nodevalue) (as setter/getter)
- For [HTML5::DOM::Element](#html5domelement) is similar to [textContent](#textcontent) (as setter/getter)

```perl
my $tree = HTML5::DOM->new->parse('<div id="test">some   text <b>bold</b></div>');

# get text content for element
my $node = $tree->at('#test');
print $node->text;                     # some   text bold
$comment->text('<new node content>');
print $comment->html;                  # &lt;new node conten&gt;

my $comment = $tree->createComment("comment text");
print $comment->text;                  # comment text
$comment->text(' new comment text ');
print $comment->html;                  # <!-- new comment text -->

my $text_node = $tree->createTextNode("plain text");
print $text_node->text;                # plain text
$text_node->text('new>plain>text');
print $text_node->html;                # new&gt;plain&gt;text
```

### innerText

### outerText

### textContent

- Represents the "rendered" text content of a node and its descendants. 
Using default CSS "display" property for tags based on Firefox user-agent style. 

    Only works for elements, for other nodes return `undef`.

    ```perl
    my $text = $node->innerText;
    my $text = $node->outerText; # alias
    ```

    Example:

    ```perl
    my $tree = HTML5::DOM->new->parse('
       <div id="test">
           some       
           <b>      bold     </b>       
           test
           <script>alert()</script>
       </div>
    ');
    print $tree->body->innerText; # some bold test
    ```

    See, for more info: [https://html.spec.whatwg.org/multipage/dom.html#the-innertext-idl-attribute](https://html.spec.whatwg.org/multipage/dom.html#the-innertext-idl-attribute)

- Removes all of its children and replaces them with a text nodes and &lt;br> with the given value.
Only works for elements, for other nodes throws exception.

    - All new line chars (\\r\\n, \\r, \\n) replaces to &lt;br />
    - All other text content replaces to text nodes

    ```perl
    my $node = $node->innerText($text);
    ```

    Example:

    ```perl
    my $tree = HTML5::DOM->new->parse('<div id="test">some text <b>bold</b></div>');
    $tree->at('#test')->innerText("some\nnew\ntext >");
    print $tree->at('#test')->html;    # <div id="test">some<br />new<br />text &gt;</div>
    ```

    See, for more info: [https://html.spec.whatwg.org/multipage/dom.html#the-innertext-idl-attribute](https://html.spec.whatwg.org/multipage/dom.html#the-innertext-idl-attribute)

- Removes the current node and replaces it with the given text.
Only works for elements, for other nodes throws exception.

    - All new line chars (\\r\\n, \\r, \\n) replaces to &lt;br />
    - All other text content replaces to text nodes
    - Similar to innerText($text), but removes current node

    ```perl
    my $node = $node->outerText($text);
    ```

    Example:

    ```perl
    my $tree = HTML5::DOM->new->parse('<div id="test">some text <b>bold</b></div>');
    $tree->at('#test')->outerText("some\nnew\ntext >");
    print $tree->body->html;   # <body>some<br />new<br />text &gt;</body>
    ```

    See, for more info: [https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/outerText](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/outerText)

- Represents the text content of a node and its descendants.

    Only works for elements, for other nodes return `undef`.

    ```perl
    my $text = $node->text;
    my $text = $node->textContent; # alias
    ```

    Example:

    ```perl
    my $tree = HTML5::DOM->new->parse('<b>    test      </b><script>alert()</script>');
    print $tree->body->text; #     test      alert()
    ```

    See, for more info: [https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent)

- Removes all of its children and replaces them with a single text node with the given value.

    ```perl
    my $node = $node->text($new_text);
    my $node = $node->textContent($new_text);
    ```

    Example:

    ```perl
    my $tree = HTML5::DOM->new->parse('<div id="test">some <b>bold</b> test</div>');
    print $tree->at('#test')->text('<bla bla bla>');
    print $tree->at('#test')->html;  # <div id="test">&lt;bla bla bla&gt;</div>
    ```

    See, for more info: [https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent)

### nodeHtml

```perl
my $html = $node->nodeHtml();
```

Serialize to html, without descendants and closing tag.

```perl
my $tree = HTML5::DOM->new->parse('<div id="test">some <b>bold</b> test</div>');
print $tree->at('#test')->nodeHtml(); # <div id="test">
```

### nodeValue

### data

```perl
my $value = $node->nodeValue();
my $value = $node->data(); # alias

my $node = $node->nodeValue($new_value);
my $node = $node->data($new_value); # alias
```

Get or set value of node. Only works for non-element nodes, such as  [HTML5::DOM::Element](#html5domtext),  [HTML5::DOM::Element](#html5domdoctype), 
[HTML5::DOM::Element](#html5domcomment). Return `undef` for other.

```perl
my $tree = HTML5::DOM->new->parse('');
my $comment = $tree->createComment("comment text");
print $comment->nodeValue;                 # comment text
$comment->nodeValue(' new comment text ');
print $comment->html;                      # <!-- new comment text -->
```

### isConnected

```perl
my $flag = $node->isConnected;
```

Return true, if node has parent.

```perl
my $tree = HTML5::DOM->new->parse('
   <div id="test"></div>
');
print $tree->at('#test')->isConnected;             # 1
print $tree->createElement("div")->isConnected;    # 0
```

### parent

### parentElement

```perl
my $node = $node->parent;
my $node = $node->parentElement; # alias
```

Return parent node. Return `undef`, if node detached.

```perl
my $tree = HTML5::DOM->new->parse('
   <div id="test"></div>
');
print $tree->at('#test')->parent->tag; # body
```

### document

### ownerDocument

```perl
my $doc = $node->document;
my $doc = $node->ownerDocument; # alias
```

Return parent [HTML5::DOM::Document](#html5domdocument). 

```perl
my $tree = HTML5::DOM->new->parse('
   <div id="test"></div>
');
print ref($tree->at('#test')->document);   # HTML5::DOM::Document
```

### append

### appendChild

```perl
my $node = $node->append($child);
my $child = $node->appendChild($child); # alias
```

Append node to child nodes.

**append** - returned value is the self node, for chain calls

**appendChild** - returned value is the appended child except when the given child is a [HTML5::DOM::Fragment](#html5domfragment), 
in which case the empty [HTML5::DOM::Fragment](#html5domfragment) is returned.

```perl
my $tree = HTML5::DOM->new->parse('
   <div>some <b>bold</b> text</div>
');
$tree->at('div')
   ->append($tree->createElement('br'))
   ->append($tree->createElement('br'));
print $tree->at('div')->html; # <div>some <b>bold</b> text<br /><br /></div>
```

### prepend

### prependChild

```perl
my $node = $node->prepend($child);
my $child = $node->prependChild($child); # alias
```

Prepend node to child nodes.

**prepend** - returned value is the self node, for chain calls

**prependChild** - returned value is the prepended child except when the given child is a [HTML5::DOM::Fragment](#html5domfragment), 
in which case the empty [HTML5::DOM::Fragment](#html5domfragment) is returned.

```perl
my $tree = HTML5::DOM->new->parse('
   <div>some <b>bold</b> text</div>
');
$tree->at('div')
   ->prepend($tree->createElement('br'))
   ->prepend($tree->createElement('br'));
print $tree->at('div')->html; # <div><br /><br />some <b>bold</b> text</div>
```

### replace

### replaceChild

```perl
my $old_node = $old_node->replace($new_node);
my $old_node = $old_node->parent->replaceChild($new_node, $old_node); # alias
```

Replace node in parent child nodes.

```perl
my $tree = HTML5::DOM->new->parse('
   <div>some <b>bold</b> text</div>
');
my $old = $tree->at('b')->replace($tree->createElement('br'));
print $old->html;              # <b>bold</b>
print $tree->at('div')->html;  # <div>some <br /> text</div>
```

### before

### insertBefore

```perl
my $node = $node->before($new_node);
my $new_node = $node->parent->insertBefore($new_node, $node); # alias
```

Insert new node before current node.

**before** - returned value is the self node, for chain calls

**insertBefore** - returned value is the added child except when the given child is a [HTML5::DOM::Fragment](#html5domfragment), 
in which case the empty [HTML5::DOM::Fragment](#html5domfragment) is returned.

```perl
my $tree = HTML5::DOM->new->parse('
   <div>some <b>bold</b> text</div>
');
$tree->at('b')->before($tree->createElement('br'));
print $tree->at('div')->html; # <div>some <br /><b>bold</b> text</div>
```

### after

### insertAfter

```perl
my $node = $node->after($new_node);
my $new_node = $node->parent->insertAfter($new_node, $node); # alias
```

Insert new node after current node.

**after** - returned value is the self node, for chain calls

**insertAfter** - returned value is the added child except when the given child is a [HTML5::DOM::Fragment](#html5domfragment), 
in which case the empty [HTML5::DOM::Fragment](#html5domfragment) is returned.

```perl
my $tree = HTML5::DOM->new->parse('
   <div>some <b>bold</b> text</div>
');
$tree->at('b')->after($tree->createElement('br'));
print $tree->at('div')->html; # <div>some <b>bold</b><br /> text</div>
```

### remove

### removeChild

```perl
my $node = $node->remove;
my $node = $node->parent->removeChild($node); # alias
```

Remove node from parent. Return removed node.

```perl
my $tree = HTML5::DOM->new->parse('
   <div>some <b>bold</b> text</div>
');
print $tree->at('b')->remove->html;    # <b>bold</b>
print $tree->at('div')->html;          # <div>some  text</div>
```

### clone

### cloneNode

```perl
# clone node to current tree
my $node = $node->clone($deep = 0);
my $node = $node->cloneNode($deep = 0); # alias

# clone node to foreign tree
my $node = $node->clone($deep, $new_tree);
my $node = $node->cloneNode($deep, $new_tree); # alias
```

Clone node. 

**deep** = 0 - only specified node, without childs. 

**deep** = 1 - deep copy with all child nodes.

**new\_tree** - destination tree (if need copy to foreign tree)

```perl
my $tree = HTML5::DOM->new->parse('
   <div>some <b>bold</b> text</div>
');
print $tree->at('b')->clone(0)->html; # <b></b>
print $tree->at('b')->clone(1)->html; # <b>bold</b>
```

### void

```perl
my $flag = $node->void;
```

Return true if node is void. For more details: [http://w3c.github.io/html-reference/syntax.html#void-elements](http://w3c.github.io/html-reference/syntax.html#void-elements)

```
print $tree->createElement('br')->void; # 1
```

### selfClosed

```perl
my $flag = $node->selfClosed;
```

Return true if node self closed. 

```
print $tree->createElement('br')->selfClosed; # 1
```

### position

```perl
my $position = $node->position;
```

Return offsets in input buffer.

```perl
print Dumper($node->position);
# $VAR1 = {'raw_length' => 3, 'raw_begin' => 144, 'element_begin' => 143, 'element_length' => 5}
```

### isSameNode

```perl
my $flag = $node->isSameNode($other_node);
```

Tests whether two nodes are the same, that is if they reference the same object.

```perl
my $tree = HTML5::DOM->new->parse('
   <ul>
       <li>test</li>
       <li>not test</li>
       <li>test</li>
   </ul>
');
my $li = $tree->find('li');
print $li->[0]->isSameNode($li->[0]); # 1
print $li->[0]->isSameNode($li->[1]); # 0
print $li->[0]->isSameNode($li->[2]); # 0
```

### wait

```perl
my $parser = HTML5::DOM->new({async => 1});
my $tree = $parser->parse($some_big_html_file);
# ...some your work...
$tree->body->wait; # wait before parsing for <body> is done
```

Blocking wait for node parsing done. Only for async mode.

### parsed

```perl
my $parser = HTML5::DOM->new({async => 1});
my $tree = $parser->parse($some_big_html_file);
# ...some your work...
while (!$tree->body->parsed); # wait before parsing for <body> is done
```

Non-blocking way for check if node parsing done. Only for async mode.

# HTML5::DOM::Element

DOM node object for elements. Inherit all methods from [HTML5::DOM::Node](#html5domnode).

### children

```perl
my $collection = $node->children;
```

Returns all child elements of current node in [HTML5::DOM::Collection](#html5domcollection).

```perl
my $tree = HTML5::DOM->new->parse('
   <ul>
       <li>Perl</li>
       <!-- comment -->
       <li>PHP</li>
       <li>C++</li>
   </ul>
');
my $collection = $tree->at('ul')->children;
print $collection->[0]->html; # <li>Perl</li>
print $collection->[1]->html; # <li>PHP</li>
print $collection->[2]->html; # <li>C++</li>
```

### childrenNode

### childNodes

```perl
my $collection = $node->childrenNode;
my $collection = $node->childNodes; # alias
```

Returns all child nodes of current node in [HTML5::DOM::Collection](#html5domcollection).

```perl
my $tree = HTML5::DOM->new->parse('
   <ul>
       <li>Perl</li>
       <!-- comment -->
       <li>PHP</li>
       <li>C++</li>
   </ul>
');
my $collection = $tree->at('ul')->childrenNode;
print $collection->[0]->html; # <li>Perl</li>
print $collection->[1]->html; # <!-- comment -->
print $collection->[2]->html; # <li>PHP</li>
print $collection->[3]->html; # <li>C++</li>
```

### attr

### removeAttr

Universal attributes accessor, for single human-friendly api.

```perl
# attribute get
my $value = $node->attr($key);

# attribute set
my $node = $node->attr($key, $value);
my $node = $node->attr($key => $value);

# attribute remove
my $node = $node->attr($key, undef);
my $node = $node->attr($key => undef);
my $node = $node->removeAttr($key);

# bulk attributes set
my $node = $node->attr({$key => $value, $key2 => $value2});

# bulk attributes remove
my $node = $node->attr({$key => undef, $key2 => undef});

# bulk get all attributes in hash
my $hash = $node->attr;
```

Example:

```perl
my $tree = HTML5::DOM->new->parse('
   <div id="test" data-test="test value" data-href="#"></div>
');
my $div = $tree->at('ul');
$div->attr("data-new", "test");
print $div->attr("data-test");     # test value
print $div->{"data-test"};         # test value
print $div->attr->{"data-test"};   # test value

# {id => "test", "data-test" => "test value", "data-href" => "#", "data-new" => "test"}
print $div->attr;

$div->removeAttr("data-test");

# {id => "test", "data-href" => "#", "data-new" => "test"}
print $div->attr;
```

### getAttribute

```perl
my $value = $node->getAttribute($key);
my $value = $node->attr($key); # alias
```

Get attribute value by key.

### setAttribute

```perl
my $node = $node->setAttribute($key, $value);
my $node = $node->attr($key, $value); # alias
```

Set new value or create new attibute. 

### removeAttribute

```perl
my $node = $node->removeAttribute($key);
my $node = $node->removeAttr($key); # alias
```

Remove attribute.

### className

```perl
my $classes = $node->className;
# alias for
my $classes = $node->attr("class");
```

### classList

```perl
my $class_list = $node->classList;

# has class
my $flag = $class_list->has($class_name);
my $flag = $class_list->contains($class_name);

# add class
my $class_list = $class_list->add($class_name);
my $class_list = $class_list->add($class_name, $class_name1, $class_name2, ...);

# add class
my $class_list = $class_list->remove($class_name);
my $class_list = $class_list->remove($class_name, $class_name1, $class_name2, ...);

# toggle class
my $state = $class_list->toggle($class_name);
my $state = $class_list->toggle($class_name, $force_state);
```

Manipulations with classes. Returns [HTML5::DOM::TokenList](#html5domtokenlist).

Similar to [https://developer.mozilla.org/en-US/docs/Web/API/Element/classList](https://developer.mozilla.org/en-US/docs/Web/API/Element/classList)

```perl
my $tree = HTML5::DOM->new->parse('<div class="red">red</div>')
my $node = $tree->body->at('.red');
print $node->has('red');                       # 1
print $node->has('blue');                      # 0
$node->add('blue', 'red', 'yellow', 'orange');
print $node->className;                        # red blue yellow orange
$node->remove('blue', 'orange');
print $node->className;                        # red yellow
print $node->toggle('blue');                   # 1
print $node->className;                        # red yellow blue
print $node->toggle('blue');                   # 0
print $node->className;                        # red yellow
```

### at

### querySelector

```perl
my $node = $node->at($selector);
my $node = $node->at($selector, $combinator);
my $node = $node->querySelector($selector); # alias
my $node = $node->querySelector($selector, $combinator); # alias
```

Find one element node in current node descendants using [CSS Selectors Level 4](https://www.w3.org/TR/selectors-4/)

Return node, or `undef` if not find.

- `$selector` - selector query as plain text or precompiled as [HTML5::DOM::CSS::Selector](#html5domcssselector) or 
[HTML5::DOM::CSS::Selector](#html5domcssselectorentry).
- `$combinator` - custom selector combinator, applies to current node
    - `>>` - descendant selector (default)
    - `>` - child selector
    - `+` - adjacent sibling selector
    - `~` - general sibling selector
    - `||` - column combinator

```perl
my $tree = HTML5::DOM->new->parse('<div class="red">red</div><div class="blue">blue</div>')
my $node = $tree->body->at('body > div.red');
print $node->html; # <div class="red">red</div>
```

### find

### querySelectorAll

```perl
my $collection = $node->find($selector);
my $collection = $node->find($selector, $combinator);
my $collection = $node->querySelectorAll($selector); # alias
my $collection = $node->querySelectorAll($selector, $combinator); # alias
```

Find all element nodes in current node descendants using [CSS Selectors Level 4](https://www.w3.org/TR/selectors-4/)

Return [HTML5::DOM::Collection](#html5domcollection).

- `$selector` - selector query as plain text or precompiled as [HTML5::DOM::CSS::Selector](#html5domcssselector) or 
[HTML5::DOM::CSS::Selector](#html5domcssselectorentry).
- `$combinator` - custom selector combinator, applies to current node
    - `>>` - descendant selector (default)
    - `>` - child selector
    - `+` - adjacent sibling selector
    - `~` - general sibling selector
    - `||` - column combinator

```perl
my $tree = HTML5::DOM->new->parse('<div class="red">red</div><div class="blue">blue</div>')
my $collection = $tree->body->at('body > div.red, body > div.blue');
print $collection->[0]->html; # <div class="red">red</div>
print $collection->[1]->html; # <div class="red">blue</div>
```

### findId

### getElementById

```perl
my $node = $node->findId($tag);
my $node = $node->getElementById($tag); # alias
```

Find element node with specified id in current node descendants.

Return [HTML5::DOM::Node](#html5domnode) or `undef`.

```perl
my $tree = HTML5::DOM->new->parse('<div class="red">red</div><div class="blue" id="test">blue</div>')
my $node = $tree->body->findId('test');
print $node->html; # <div class="blue" id="test">blue</div>
```

### findTag

### getElementsByTagName

```perl
my $node = $node->findTag($tag);
my $node = $node->getElementsByTagName($tag); # alias
```

Find all element nodes in current node descendants with specified tag name.

Return [HTML5::DOM::Collection](#html5domcollection).

```perl
my $tree = HTML5::DOM->new->parse('<div class="red">red</div><div class="blue">blue</div>')
my $collection = $tree->body->findTag('div');
print $collection->[0]->html; # <div class="red">red</div>
print $collection->[1]->html; # <div class="red">blue</div>
```

### findClass

### getElementsByClassName

```perl
my $collection = $node->findClass($class);
my $collection = $node->getElementsByClassName($class); # alias
```

Find all element nodes in current node descendants with specified class name.
This is more fast equivalent to \[class~="value"\] selector.

Return [HTML5::DOM::Collection](#html5domcollection).

```perl
my $tree = HTML5::DOM->new
   ->parse('<div class="red color">red</div><div class="blue color">blue</div>');
my $collection = $tree->body->findClass('color');
print $collection->[0]->html; # <div class="red color">red</div>
print $collection->[1]->html; # <div class="red color">blue</div>
```

### findAttr

### getElementByAttribute

```perl
# Find all elements with attribute
my $collection = $node->findAttr($attribute);
my $collection = $node->getElementByAttribute($attribute); # alias

# Find all elements with attribute and mathcing value
my $collection = $node->findAttr($attribute, $value, $case = 0, $cmp = '=');
my $collection = $node->getElementByAttribute($attribute, $value, $case = 0, $cmp = '='); # alias
```

Find all element nodes in tree with specified attribute and optional matching value.

Return [HTML5::DOM::Collection](#html5domcollection).

```perl
my $tree = HTML5::DOM->new
   ->parse('<div class="red color">red</div><div class="blue color">blue</div>');
my $collection = $tree->body->findAttr('class', 'CoLoR', 1, '~');
print $collection->[0]->html; # <div class="red color">red</div>
print $collection->[1]->html; # <div class="red color">blue</div>
```

CSS selector analogs:

```perl
# [$attribute=$value]
my $collection = $node->findAttr($attribute, $value, 0, '=');

# [$attribute=$value i]
my $collection = $node->findAttr($attribute, $value, 1, '=');

# [$attribute~=$value]
my $collection = $node->findAttr($attribute, $value, 0, '~');

# [$attribute|=$value]
my $collection = $node->findAttr($attribute, $value, 0, '|');

# [$attribute*=$value]
my $collection = $node->findAttr($attribute, $value, 0, '*');

# [$attribute^=$value]
my $collection = $node->findAttr($attribute, $value, 0, '^');

# [$attribute$=$value]
my $collection = $node->findAttr($attribute, $value, 0, '$');
```

### getDefaultBoxType

```perl
my $display = $node->getDefaultBoxType;
```

Get default CSS "display" property for tag (useful for functions like a [innerText](#innertext)).

```perl
my $tree = HTML5::DOM->new
   ->parse('<div class="red color">red</div><script>alert()</script><b>bbb</b>');
print $tree->at('div')->getDefaultBoxType();       # block
print $tree->at('script')->getDefaultBoxType();    # none
print $tree->at('b')->getDefaultBoxType();         # inline
```

# HTML5::DOM::Document

DOM node object for document. Inherit all methods from [HTML5::DOM::Element](#html5domelement).

# HTML5::DOM::Fragment

DOM node object for fragments. Inherit all methods from [HTML5::DOM::Element](#html5domelement).

# HTML5::DOM::Text

DOM node object for text. Inherit all methods from [HTML5::DOM::Node](#html5domnode).

# HTML5::DOM::Comment

DOM node object for comments. Inherit all methods from [HTML5::DOM::Node](#html5domnode).

# HTML5::DOM::DocType

DOM node object for document type. Inherit all methods from [HTML5::DOM::Node](#html5domnode).

# HTML5::DOM::Collection

CSS Parser object

### new

```perl
my $collection = HTML5::DOM::Collection->new($nodes);
```

Creates new collection from `$nodes` (reference to array with [HTML5::DOM::Node](#html5domnode)).

### each

```perl
my $collection = $collection->each(sub {
   my ($node, $index) = @_;
   print "node[$index] is a '$node'\n";
});
```

Forach all nodes in collection.

### map

```perl
my $result = $collection->map(sub {
   my ($token, $index) = @_;
   return $node->tag." => $index";
});
```

Apply callback for each node in collection. Returns new array from results.

```perl
my $result = $collection->map($method, @args);
```

Call method for each node in collection. Returns new [HTML5::DOM::Collection](#html5domcollection) from results.

Example:

```perl
# set text 'test!' for all nodes
my $result = $collection->map('text', 'test!');

# get all tag names as array
my $result = $collection->map('tag');

# remove all nodes in collection
$collection->map('remove');
```

### add

```perl
my $collection = $collection->add($node);
```

Add new item to collection.

### length

```perl
my $length = $collection->length;
```

Items count in collection.

```perl
my $tree = HTML5::DOM->new->parse('
   <ul>
       <li>Linux</li>
       <!-- comment -->
       <li>OSX</li>
       <li>Windows</li>
   </ul>
');
my $collection = $tree->find('ul li');
print $collection->length; # 3
```

### first

### last

```perl
my $node = $collection->first;
my $node = $collection->last;
```

Get first or last item in collection.

```perl
my $tree = HTML5::DOM->new->parse('
   <ul>
       <li>Linux</li>
       <!-- comment -->
       <li>OSX</li>
       <li>Windows</li>
   </ul>
');
my $collection = $tree->find('ul li');
print $collection->first->html;        #  <li>Linux</li>
print $collection->last->html;         #  <li>Windows</li>
```

### item

```perl
my $node = $collection->item($index);
my $node = $collection->[$index];
```

Get item by `$index` in collection.

```perl
my $tree = HTML5::DOM->new->parse('
   <ul>
       <li>Linux</li>
       <!-- comment -->
       <li>OSX</li>
       <li>Windows</li>
   </ul>
');
my $collection = $tree->find('ul li');
print $collection->item(1)->html;      # <li>OSX</li>
print $collection->[1]->html;          # <li>OSX</li>
```

### array

```perl
my $node = $collection->array();
```

Get collection items as array.

### html

```perl
my $html = $collection->html;
```

Concat &lt;outerHTML|/outerHTML> from all items.

### text

```perl
my $text = $collection->text;
```

Concat &lt;textContent|/textContent> from all items.

# HTML5::DOM::TokenList

Similar to [https://developer.mozilla.org/en-US/docs/Web/API/DOMTokenList](https://developer.mozilla.org/en-US/docs/Web/API/DOMTokenList)

### has

### contains

```perl
my $flag = $tokens->has($token);
my $flag = $tokens->contains($token); # alias
```

Check if token contains in current tokens list.

### add

```perl
my $tokens = $tokens->add($token);
my $tokens = $tokens->add($token, $token2, ...);
```

Add new token (or tokens) to current tokens list. Returns self.

### remove

```perl
my $tokens = $tokens->add($token);
my $tokens = $tokens->add($token, $token2, ...);
```

Remove one or more tokens from current tokens list. Returns self.

### toggle

```perl
my $state = $tokens->toggle($token);
my $state = $tokens->toggle($token, $force_state);
```

- `$token` - specified token name
- `$force_state` - optional force state.

    If 1 - similar to [add](https://metacpan.org/pod/add)

    If 0 - similar to [remove](https://metacpan.org/pod/remove)

Toggle specified token in current tokens list.

- If token exists - remove it
- If token not exists - add it

### length

```perl
my $length = $tokens->length;
```

Returns tokens count in current list.

### item

```perl
my $token = $tokens->item($index);
my $token = $tokens->[$index];
```

Return token by index.

### each

```perl
my $token = $tokens->each(sub {
   my ($token, $index) = @_;
   print "tokens[$index] is a '$token'\n";
});
```

Forach all tokens in list.

# HTML5::DOM::CSS

CSS Parser object

### new

```perl
my $css = HTML5::DOM::CSS->new;
```

Create new css parser object.

### parseSelector

```perl
my $selector = HTML5::DOM::CSS->parseSelector($selector_text);
```

Parse `$selector_text` and return [HTML5::DOM::CSS::Selector](#html5domcssselector).

```perl
my $css = HTML5::DOM::CSS->new;
my $selector = $css->parseSelector('body div.red, body span.blue');
```

# HTML5::DOM::CSS::Selector

CSS Selector object (precompiled selector)

### new

```perl
my $selector = HTML5::DOM::CSS::Selector->new($selector_text);
```

Parse `$selector_text` and create new css selector object. 
If your need parse many selectors, more efficient way using
single instance of parser [HTML5::DOM::CSS](#html5domcss) and 
[parseSelector](#parseselector) method.

### text

```perl
my $selector_text = $selector->text;
```

Serialize selector to text.

```perl
my $css = HTML5::DOM::CSS->new;
my $selector = $css->parseSelector('body div.red, body span.blue');
print $selector->text."\n"; # body div.red, body span.blue
```

### ast

```perl
my $ast = $entry->ast;
```

Serialize selector to very simple AST format.

```perl
my $css = HTML5::DOM::CSS->new;
my $selector = $css->parseSelector('div > .red');
print Dumper($selector->ast);

# $VAR1 = [[
#     {
#         'value' => 'div',
#         'type' => 'tag'
#     },
#     {
#         'type'  => 'combinator',
#         'value' => 'child'
#     },
#     {
#         'type' => 'class',
#         'value' => 'red'
#     }
# ]];
```

### length

```perl
my $length = $selector->length;
```

Get selector entries count (selectors separated by "," combinator)

```perl
my $css = HTML5::DOM::CSS->new;
my $selector = $css->parseSelector('body div.red, body span.blue');
print $selector->length."\n"; # 2
```

### entry

```perl
my $entry = $selector->entry($index);
```

Get selector entry by `$index` end return [HTML5::DOM::CSS::Selector::Entry](#html5domcssselectorentry).

```perl
my $css = HTML5::DOM::CSS->new;
my $selector = $css->parseSelector('body div.red, body span.blue');
print $selector->entry(0)->text."\n"; # body div.red
print $selector->entry(1)->text."\n"; # body span.blue
```

# HTML5::DOM::CSS::Selector::Entry

CSS selector entry object (precompiled selector)

### text

```perl
my $selector_text = $entry->text;
```

Serialize entry to text.

```perl
my $css = HTML5::DOM::CSS->new;
my $selector = $css->parseSelector('body div.red, body span.blue');
my $entry = $selector->entry(0);
print $entry->text."\n"; # body div.red
```

### pseudoElement

```perl
my $pseudo_name = $entry->pseudoElement;
```

Return pseudo-element name for entry.

```perl
my $css = HTML5::DOM::CSS->new;
my $selector = $css->parseSelector('div::after');
my $entry = $selector->entry(0);
print $entry->pseudoElement."\n"; # after
```

### ast

```perl
my $ast = $entry->ast;
```

Serialize entry to very simple AST format.

```perl
my $css = HTML5::DOM::CSS->new;
my $selector = $css->parseSelector('div > .red');
my $entry = $selector->entry(0);
print Dumper($entry->ast);

# $VAR1 = [
#     {
#         'value' => 'div',
#         'type' => 'tag'
#     },
#     {
#         'type'  => 'combinator',
#         'value' => 'child'
#     },
#     {
#         'type' => 'class',
#         'value' => 'red'
#     }
# ];
```

### specificity

```perl
my $specificity = $entry->specificity;
```

Get specificity in hash `{a, b, c}`

```perl
my $css = HTML5::DOM::CSS->new;
my $selector = $css->parseSelector('body div.red, body span.blue');
my $entry = $selector->entry(0);
print Dumper($entry->specificity); # {a => 0, b => 1, c => 2}
```

### specificityArray

```perl
my $specificity = $entry->specificityArray;
```

Get specificity in array `[a, b, c]` (ordered by weight)

```perl
my $css = HTML5::DOM::CSS->new;
my $selector = $css->parseSelector('body div.red, body span.blue');
my $entry = $selector->entry(0);
print Dumper($entry->specificityArray); # [0, 1, 2]
```

# HTML5::DOM::Encoding

Encoding detection.

See for available encodings: ["ENCODINGS"](#encodings)

### id2name

```perl
my $encoding = HTML5::DOM::Encoding::id2name($encoding_id);
```

Get encoding name by id.

```
print HTML5::DOM::Encoding::id2name(HTML5::DOM::Encoding->UTF_8); # UTF-8
```

### name2id

```perl
my $encoding_id = HTML5::DOM::Encoding::name2id($encoding);
```

Get id by name.

```
print HTML5::DOM::Encoding->UTF_8;             # 0
print HTML5::DOM::Encoding::id2name("UTF-8");  # 0
```

### detectAuto

```perl
my ($encoding_id, $new_text) = HTML5::DOM::Encoding::detectAuto($text, $max_length = 0);
```

Auto detect text encoding using (in this order):

- [detectByPrescanStream](#detectbyprescanstream)
- [detectBomAndCut](#detectbomandcut)
- [detect](#detect)

Returns array with encoding id and new text without BOM, if success. 

If fail, then encoding id equal HTML5::DOM::Encoding->NOT\_DETERMINED.

```perl
my ($encoding_id, $new_text) = HTML5::DOM::Encoding::detectAuto("ололо");
my $encoding = HTML5::DOM::Encoding::id2name($encoding_id);
print $encoding; # UTF-8
```

### detect

```perl
my $encoding_id = HTML5::DOM::Encoding::detect($text, $max_length = 0);
```

Detect text encoding. Single method for both [detectRussian](#detectrussian) and [detectUnicode](#detectunicode).

Returns encoding id, if success. And returns HTML5::DOM::Encoding->NOT\_DETERMINED if fail.

```perl
my $encoding_id = HTML5::DOM::Encoding::detect("ололо");
my $encoding = HTML5::DOM::Encoding::id2name($encoding_id);
print $encoding; # UTF-8
```

### detectRussian

```perl
my $encoding_id = HTML5::DOM::Encoding::detectRussian($text, $max_length = 0);
```

Detect russian text encoding (using lowercase **trigrams**), such as `windows-1251`, `koi8-r`, `iso-8859-5`, `x-mac-cyrillic`, `ibm866`.

Returns encoding id, if success. And returns HTML5::DOM::Encoding->NOT\_DETERMINED if fail.

### detectUnicode

```perl
my $encoding_id = HTML5::DOM::Encoding::detectRussian($text, $max_length = 0);
```

Detect unicode family text encoding, such as `UTF-8`, `UTF-16LE`, `UTF-16BE`.

Returns encoding id, if success. And returns HTML5::DOM::Encoding->NOT\_DETERMINED if fail.

```perl
# get UTF-16LE data for test
my $str = "ололо";
Encode::from_to($str, "UTF-8", "UTF-16LE");

my $encoding_id = HTML5::DOM::Encoding::detectUnicode($str);
my $encoding = HTML5::DOM::Encoding::id2name($encoding_id);
print $encoding; # UTF-16LE
```

### detectByPrescanStream

```perl
my $encoding_id = HTML5::DOM::Encoding::detectByPrescanStream($text, $max_length = 0);
```

Detect encoding by parsing `<meta>` tags in html.

Returns encoding id, if success. And returns HTML5::DOM::Encoding->NOT\_DETERMINED if fail.

See for more info: [https://html.spec.whatwg.org/multipage/syntax.html#prescan-a-byte-stream-to-determine-its-encoding](https://html.spec.whatwg.org/multipage/syntax.html#prescan-a-byte-stream-to-determine-its-encoding)

```perl
my $encoding_id = HTML5::DOM::Encoding::detectByPrescanStream('
   <meta http-equiv="content-type" content="text/html; charset=windows-1251">
');
my $encoding = HTML5::DOM::Encoding::id2name($encoding_id);
print $encoding; # WINDOWS-1251
```

### detectByCharset

```perl
my $encoding_id = HTML5::DOM::Encoding::detectByCharset($text, $max_length = 0);
```

Extracting character encoding from string. Find "charset=" and see encoding. Return found raw data.

For example: "text/html; charset=windows-1251". Return HTML5::DOM::Encoding->WINDOWS\_1251

And returns HTML5::DOM::Encoding->NOT\_DETERMINED if fail.

See for more info: [https://html.spec.whatwg.org/multipage/infrastructure.html#algorithm-for-extracting-a-character-encoding-from-a-meta-element](https://html.spec.whatwg.org/multipage/infrastructure.html#algorithm-for-extracting-a-character-encoding-from-a-meta-element)

```perl
my $encoding_id = HTML5::DOM::Encoding::detectByPrescanStream('
   <meta http-equiv="content-type" content="text/html; charset=windows-1251">
');
my $encoding = HTML5::DOM::Encoding::id2name($encoding_id);
print $encoding; # WINDOWS-1251
```

### detectBomAndCut

```perl
my ($encoding_id, $new_text) = HTML5::DOM::Encoding::detectBomAndCut($text, $max_length = 0);
```

Returns array with encoding id and new text without BOM. 

If fail, then encoding id equal HTML5::DOM::Encoding->NOT\_DETERMINED.

```perl
my ($encoding_id, $new_text) = HTML5::DOM::Encoding::detectBomAndCut("\xEF\xBB\xBFололо");
my $encoding = HTML5::DOM::Encoding::id2name($encoding_id);
print $encoding; # UTF-8
print $new_text; # ололо
```

# NAMESPACES

### Supported namespace names

```
html, matml, svg, xlink, xml, xmlns
```

### Supported namespace id constants

```
HTML5::DOM->NS_UNDEF
HTML5::DOM->NS_HTML
HTML5::DOM->NS_MATHML
HTML5::DOM->NS_SVG
HTML5::DOM->NS_XLINK
HTML5::DOM->NS_XML
HTML5::DOM->NS_XMLNS
HTML5::DOM->NS_ANY
HTML5::DOM->NS_LAST_ENTRY
```

# TAGS

```
HTML5::DOM->TAG__UNDEF
HTML5::DOM->TAG__TEXT
HTML5::DOM->TAG__COMMENT
HTML5::DOM->TAG__DOCTYPE
HTML5::DOM->TAG_A
HTML5::DOM->TAG_ABBR
HTML5::DOM->TAG_ACRONYM
HTML5::DOM->TAG_ADDRESS
HTML5::DOM->TAG_ANNOTATION_XML
HTML5::DOM->TAG_APPLET
HTML5::DOM->TAG_AREA
HTML5::DOM->TAG_ARTICLE
HTML5::DOM->TAG_ASIDE
HTML5::DOM->TAG_AUDIO
HTML5::DOM->TAG_B
HTML5::DOM->TAG_BASE
HTML5::DOM->TAG_BASEFONT
HTML5::DOM->TAG_BDI
HTML5::DOM->TAG_BDO
HTML5::DOM->TAG_BGSOUND
HTML5::DOM->TAG_BIG
HTML5::DOM->TAG_BLINK
HTML5::DOM->TAG_BLOCKQUOTE
HTML5::DOM->TAG_BODY
HTML5::DOM->TAG_BR
HTML5::DOM->TAG_BUTTON
HTML5::DOM->TAG_CANVAS
HTML5::DOM->TAG_CAPTION
HTML5::DOM->TAG_CENTER
HTML5::DOM->TAG_CITE
HTML5::DOM->TAG_CODE
HTML5::DOM->TAG_COL
HTML5::DOM->TAG_COLGROUP
HTML5::DOM->TAG_COMMAND
HTML5::DOM->TAG_COMMENT
HTML5::DOM->TAG_DATALIST
HTML5::DOM->TAG_DD
HTML5::DOM->TAG_DEL
HTML5::DOM->TAG_DETAILS
HTML5::DOM->TAG_DFN
HTML5::DOM->TAG_DIALOG
HTML5::DOM->TAG_DIR
HTML5::DOM->TAG_DIV
HTML5::DOM->TAG_DL
HTML5::DOM->TAG_DT
HTML5::DOM->TAG_EM
HTML5::DOM->TAG_EMBED
HTML5::DOM->TAG_FIELDSET
HTML5::DOM->TAG_FIGCAPTION
HTML5::DOM->TAG_FIGURE
HTML5::DOM->TAG_FONT
HTML5::DOM->TAG_FOOTER
HTML5::DOM->TAG_FORM
HTML5::DOM->TAG_FRAME
HTML5::DOM->TAG_FRAMESET
HTML5::DOM->TAG_H1
HTML5::DOM->TAG_H2
HTML5::DOM->TAG_H3
HTML5::DOM->TAG_H4
HTML5::DOM->TAG_H5
HTML5::DOM->TAG_H6
HTML5::DOM->TAG_HEAD
HTML5::DOM->TAG_HEADER
HTML5::DOM->TAG_HGROUP
HTML5::DOM->TAG_HR
HTML5::DOM->TAG_HTML
HTML5::DOM->TAG_I
HTML5::DOM->TAG_IFRAME
HTML5::DOM->TAG_IMAGE
HTML5::DOM->TAG_IMG
HTML5::DOM->TAG_INPUT
HTML5::DOM->TAG_INS
HTML5::DOM->TAG_ISINDEX
HTML5::DOM->TAG_KBD
HTML5::DOM->TAG_KEYGEN
HTML5::DOM->TAG_LABEL
HTML5::DOM->TAG_LEGEND
HTML5::DOM->TAG_LI
HTML5::DOM->TAG_LINK
HTML5::DOM->TAG_LISTING
HTML5::DOM->TAG_MAIN
HTML5::DOM->TAG_MAP
HTML5::DOM->TAG_MARK
HTML5::DOM->TAG_MARQUEE
HTML5::DOM->TAG_MENU
HTML5::DOM->TAG_MENUITEM
HTML5::DOM->TAG_META
HTML5::DOM->TAG_METER
HTML5::DOM->TAG_MTEXT
HTML5::DOM->TAG_NAV
HTML5::DOM->TAG_NOBR
HTML5::DOM->TAG_NOEMBED
HTML5::DOM->TAG_NOFRAMES
HTML5::DOM->TAG_NOSCRIPT
HTML5::DOM->TAG_OBJECT
HTML5::DOM->TAG_OL
HTML5::DOM->TAG_OPTGROUP
HTML5::DOM->TAG_OPTION
HTML5::DOM->TAG_OUTPUT
HTML5::DOM->TAG_P
HTML5::DOM->TAG_PARAM
HTML5::DOM->TAG_PLAINTEXT
HTML5::DOM->TAG_PRE
HTML5::DOM->TAG_PROGRESS
HTML5::DOM->TAG_Q
HTML5::DOM->TAG_RB
HTML5::DOM->TAG_RP
HTML5::DOM->TAG_RT
HTML5::DOM->TAG_RTC
HTML5::DOM->TAG_RUBY
HTML5::DOM->TAG_S
HTML5::DOM->TAG_SAMP
HTML5::DOM->TAG_SCRIPT
HTML5::DOM->TAG_SECTION
HTML5::DOM->TAG_SELECT
HTML5::DOM->TAG_SMALL
HTML5::DOM->TAG_SOURCE
HTML5::DOM->TAG_SPAN
HTML5::DOM->TAG_STRIKE
HTML5::DOM->TAG_STRONG
HTML5::DOM->TAG_STYLE
HTML5::DOM->TAG_SUB
HTML5::DOM->TAG_SUMMARY
HTML5::DOM->TAG_SUP
HTML5::DOM->TAG_SVG
HTML5::DOM->TAG_TABLE
HTML5::DOM->TAG_TBODY
HTML5::DOM->TAG_TD
HTML5::DOM->TAG_TEMPLATE
HTML5::DOM->TAG_TEXTAREA
HTML5::DOM->TAG_TFOOT
HTML5::DOM->TAG_TH
HTML5::DOM->TAG_THEAD
HTML5::DOM->TAG_TIME
HTML5::DOM->TAG_TITLE
HTML5::DOM->TAG_TR
HTML5::DOM->TAG_TRACK
HTML5::DOM->TAG_TT
HTML5::DOM->TAG_U
HTML5::DOM->TAG_UL
HTML5::DOM->TAG_VAR
HTML5::DOM->TAG_VIDEO
HTML5::DOM->TAG_WBR
HTML5::DOM->TAG_XMP
HTML5::DOM->TAG_ALTGLYPH
HTML5::DOM->TAG_ALTGLYPHDEF
HTML5::DOM->TAG_ALTGLYPHITEM
HTML5::DOM->TAG_ANIMATE
HTML5::DOM->TAG_ANIMATECOLOR
HTML5::DOM->TAG_ANIMATEMOTION
HTML5::DOM->TAG_ANIMATETRANSFORM
HTML5::DOM->TAG_CIRCLE
HTML5::DOM->TAG_CLIPPATH
HTML5::DOM->TAG_COLOR_PROFILE
HTML5::DOM->TAG_CURSOR
HTML5::DOM->TAG_DEFS
HTML5::DOM->TAG_DESC
HTML5::DOM->TAG_ELLIPSE
HTML5::DOM->TAG_FEBLEND
HTML5::DOM->TAG_FECOLORMATRIX
HTML5::DOM->TAG_FECOMPONENTTRANSFER
HTML5::DOM->TAG_FECOMPOSITE
HTML5::DOM->TAG_FECONVOLVEMATRIX
HTML5::DOM->TAG_FEDIFFUSELIGHTING
HTML5::DOM->TAG_FEDISPLACEMENTMAP
HTML5::DOM->TAG_FEDISTANTLIGHT
HTML5::DOM->TAG_FEDROPSHADOW
HTML5::DOM->TAG_FEFLOOD
HTML5::DOM->TAG_FEFUNCA
HTML5::DOM->TAG_FEFUNCB
HTML5::DOM->TAG_FEFUNCG
HTML5::DOM->TAG_FEFUNCR
HTML5::DOM->TAG_FEGAUSSIANBLUR
HTML5::DOM->TAG_FEIMAGE
HTML5::DOM->TAG_FEMERGE
HTML5::DOM->TAG_FEMERGENODE
HTML5::DOM->TAG_FEMORPHOLOGY
HTML5::DOM->TAG_FEOFFSET
HTML5::DOM->TAG_FEPOINTLIGHT
HTML5::DOM->TAG_FESPECULARLIGHTING
HTML5::DOM->TAG_FESPOTLIGHT
HTML5::DOM->TAG_FETILE
HTML5::DOM->TAG_FETURBULENCE
HTML5::DOM->TAG_FILTER
HTML5::DOM->TAG_FONT_FACE
HTML5::DOM->TAG_FONT_FACE_FORMAT
HTML5::DOM->TAG_FONT_FACE_NAME
HTML5::DOM->TAG_FONT_FACE_SRC
HTML5::DOM->TAG_FONT_FACE_URI
HTML5::DOM->TAG_FOREIGNOBJECT
HTML5::DOM->TAG_G
HTML5::DOM->TAG_GLYPH
HTML5::DOM->TAG_GLYPHREF
HTML5::DOM->TAG_HKERN
HTML5::DOM->TAG_LINE
HTML5::DOM->TAG_LINEARGRADIENT
HTML5::DOM->TAG_MARKER
HTML5::DOM->TAG_MASK
HTML5::DOM->TAG_METADATA
HTML5::DOM->TAG_MISSING_GLYPH
HTML5::DOM->TAG_MPATH
HTML5::DOM->TAG_PATH
HTML5::DOM->TAG_PATTERN
HTML5::DOM->TAG_POLYGON
HTML5::DOM->TAG_POLYLINE
HTML5::DOM->TAG_RADIALGRADIENT
HTML5::DOM->TAG_RECT
HTML5::DOM->TAG_SET
HTML5::DOM->TAG_STOP
HTML5::DOM->TAG_SWITCH
HTML5::DOM->TAG_SYMBOL
HTML5::DOM->TAG_TEXT
HTML5::DOM->TAG_TEXTPATH
HTML5::DOM->TAG_TREF
HTML5::DOM->TAG_TSPAN
HTML5::DOM->TAG_USE
HTML5::DOM->TAG_VIEW
HTML5::DOM->TAG_VKERN
HTML5::DOM->TAG_MATH
HTML5::DOM->TAG_MACTION
HTML5::DOM->TAG_MALIGNGROUP
HTML5::DOM->TAG_MALIGNMARK
HTML5::DOM->TAG_MENCLOSE
HTML5::DOM->TAG_MERROR
HTML5::DOM->TAG_MFENCED
HTML5::DOM->TAG_MFRAC
HTML5::DOM->TAG_MGLYPH
HTML5::DOM->TAG_MI
HTML5::DOM->TAG_MLABELEDTR
HTML5::DOM->TAG_MLONGDIV
HTML5::DOM->TAG_MMULTISCRIPTS
HTML5::DOM->TAG_MN
HTML5::DOM->TAG_MO
HTML5::DOM->TAG_MOVER
HTML5::DOM->TAG_MPADDED
HTML5::DOM->TAG_MPHANTOM
HTML5::DOM->TAG_MROOT
HTML5::DOM->TAG_MROW
HTML5::DOM->TAG_MS
HTML5::DOM->TAG_MSCARRIES
HTML5::DOM->TAG_MSCARRY
HTML5::DOM->TAG_MSGROUP
HTML5::DOM->TAG_MSLINE
HTML5::DOM->TAG_MSPACE
HTML5::DOM->TAG_MSQRT
HTML5::DOM->TAG_MSROW
HTML5::DOM->TAG_MSTACK
HTML5::DOM->TAG_MSTYLE
HTML5::DOM->TAG_MSUB
HTML5::DOM->TAG_MSUP
HTML5::DOM->TAG_MSUBSUP
HTML5::DOM->TAG__END_OF_FILE
HTML5::DOM->TAG_LAST_ENTRY
```

# ENCODINGS

### Supported encoding names

```
AUTO, NOT-DETERMINED, X-USER-DEFINED, 
BIG5, EUC-JP, EUC-KR, GB18030, GBK, IBM866, MACINTOSH, X-MAC-CYRILLIC, SHIFT_JIS, 
ISO-2022-JP, ISO-8859-10, ISO-8859-13, ISO-8859-14, ISO-8859-15, ISO-8859-16, ISO-8859-2, 
ISO-8859-3, ISO-8859-4, ISO-8859-5, ISO-8859-6, ISO-8859-7, ISO-8859-8, ISO-8859-8-I, 
WINDOWS-1250, WINDOWS-1251, WINDOWS-1252, WINDOWS-1253, WINDOWS-1254, 
WINDOWS-1255, WINDOWS-1256, WINDOWS-1257, WINDOWS-1258, WINDOWS-874, 
UTF-8, UTF-16BE, UTF-16LE, KOI8-R, KOI8-U
```

### Supported encoding id consts

```
HTML5::DOM::Encoding->DEFAULT
HTML5::DOM::Encoding->AUTO
HTML5::DOM::Encoding->NOT_DETERMINED
HTML5::DOM::Encoding->UTF_8
HTML5::DOM::Encoding->UTF_16LE
HTML5::DOM::Encoding->UTF_16BE
HTML5::DOM::Encoding->X_USER_DEFINED
HTML5::DOM::Encoding->BIG5
HTML5::DOM::Encoding->EUC_JP
HTML5::DOM::Encoding->EUC_KR
HTML5::DOM::Encoding->GB18030
HTML5::DOM::Encoding->GBK
HTML5::DOM::Encoding->IBM866
HTML5::DOM::Encoding->ISO_2022_JP
HTML5::DOM::Encoding->ISO_8859_10
HTML5::DOM::Encoding->ISO_8859_13
HTML5::DOM::Encoding->ISO_8859_14
HTML5::DOM::Encoding->ISO_8859_15
HTML5::DOM::Encoding->ISO_8859_16
HTML5::DOM::Encoding->ISO_8859_2
HTML5::DOM::Encoding->ISO_8859_3
HTML5::DOM::Encoding->ISO_8859_4
HTML5::DOM::Encoding->ISO_8859_5
HTML5::DOM::Encoding->ISO_8859_6
HTML5::DOM::Encoding->ISO_8859_7
HTML5::DOM::Encoding->ISO_8859_8
HTML5::DOM::Encoding->ISO_8859_8_I
HTML5::DOM::Encoding->KOI8_R
HTML5::DOM::Encoding->KOI8_U
HTML5::DOM::Encoding->MACINTOSH
HTML5::DOM::Encoding->SHIFT_JIS
HTML5::DOM::Encoding->WINDOWS_1250
HTML5::DOM::Encoding->WINDOWS_1251
HTML5::DOM::Encoding->WINDOWS_1252
HTML5::DOM::Encoding->WINDOWS_1253
HTML5::DOM::Encoding->WINDOWS_1254
HTML5::DOM::Encoding->WINDOWS_1255
HTML5::DOM::Encoding->WINDOWS_1256
HTML5::DOM::Encoding->WINDOWS_1257
HTML5::DOM::Encoding->WINDOWS_1258
HTML5::DOM::Encoding->WINDOWS_874
HTML5::DOM::Encoding->X_MAC_CYRILLIC
HTML5::DOM::Encoding->LAST_ENTRY
```

# PARSER OPTIONS

Options for:

- [HTML5::DOM::new](#new)
- [HTML5::DOM::parse](#parse)
- [HTML5::DOM::parseChunkEnd](#parsechunkend)
- [HTML5::DOM::Tree::parseFragment](#parsefragment)

#### threads

Threads count, if 0 - parsing in single mode without threads (default 2)

This option affects only for [HTML5::DOM::new](#new).

#### async

If async 0 (default), then some parsing functions
(such as [HTML5::DOM::parse](#parse), [HTML5::DOM::parseChunkEnd](#parsechunkend), [HTML5::DOM::Tree::parseFragment](#parsefragment))
waiting for parsing done.

If async 1, you must manualy call `$tree->wait` and `$node->wait` for waiting parsing done.

Or use non-blocking `$tree->parsed` and `$node->parsed` to determine parsing state.

This options works only if `threads` > 0

#### ignore\_whitespace

Ignore whitespace tokens (default 0)

#### ignore\_doctype

Do not parse DOCTYPE (default 0)

#### scripts

If 1 - &lt;noscript> contents parsed to single text node (default)

If 0 - &lt;noscript> contents parsed to child nodes

#### encoding

Encoding of input HTML, if `auto` - library can tree to automaticaly determine encoding. (default "auto")

Allowed both encoding name or id. 

#### default\_encoding

Default encoding, this affects only if `encoding` set to `auto` and encoding not determined. (default "UTF-8")

Allowed both encoding name or id. 

See for available encodings: ["ENCODINGS"](#encodings)

#### encoding\_use\_meta

Allow use `<meta>` tags to determine input HTML encoding. (default 1)

See [detectByPrescanStream](#detectbyprescanstream).

#### encoding\_prescan\_limit

Limit string length to determine encoding by `<meta>` tags. (default 1024, from spec)

See [detectByPrescanStream](#detectbyprescanstream).

#### encoding\_use\_bom

Allow use detecding BOM to determine input HTML encoding. (default 1)

See [detectBomAndCut](#detectbomandcut).

# BUGS

[https://github.com/Azq2/perl-html5-dom/issues](https://github.com/Azq2/perl-html5-dom/issues)

# SEE ALSO

- [HTML::MyHTML](https://metacpan.org/pod/HTML::MyHTML) - more low-level myhtml bindings.
- [Mojo::DOM](https://metacpan.org/pod/Mojo::DOM) - pure perl HTML5 DOM library with CSS selectors. 

# AUTHOR

Kirill Zhumarin <kirill.zhumarin@gmail.com>

# LICENSE

- HTML5::DOM - [MIT](https://github.com/Azq2/perl-html5-dom/blob/master/LICENSE)
- Modest - [LGPL 2.1](https://github.com/lexborisov/Modest/blob/master/LICENSE)
- MyHTML - [LGPL 2.1](https://github.com/lexborisov/myhtml/blob/master/LICENSE)
- MyCSS - [LGPL 2.1](https://github.com/lexborisov/mycss/blob/master/LICENSE)
