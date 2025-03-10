---
title: Shortcodes
summary: Flexible content embedding
icon: code
---

# Shortcodes

The [ShortcodeParser](api:SilverStripe\View\Parsers\ShortcodeParser) API is simple parser that allows you to map specifically formatted content to a callback to 
transform them into something else. You might know this concept from forum software which don't allow you to insert
direct HTML, instead resorting to a custom syntax. 

In the CMS, authors often want to insert content elements which go beyond standard formatting, at an arbitrary position 
in their WYSIWYG editor. Shortcodes are a semi-technical solution for this. A good example would be embedding a 3D file 
viewer or a Google Map at a certain location. 

```php
$text = "<h1>My Map</h1>[map]"

// Will output
// <h1>My Map</h1><iframe ..></iframe>
```

Here's some syntax variations:

```php
[my_shortcode]
#
[my_shortcode /]
#
[my_shortcode,myparameter="value"]
#
[my_shortcode,myparameter="value"]Enclosed Content[/my_shortcode]
```

Shortcodes are automatically parsed on any database field which is declared as [HTMLValue](api:SilverStripe\View\Parsers\HTMLValue) or [DBHTMLText](api:SilverStripe\ORM\FieldType\DBHTMLText), 
when rendered into a template. This means you can use shortcodes on common fields like `SiteTree.Content`, and any 
other [DataObject::$db](api:SilverStripe\ORM\DataObject::$db) definitions of these types.

Other fields can be manually parsed with shortcodes through the `parse` method.


```php
use SilverStripe\View\Parsers\ShortcodeParser;

$text = "My awesome [my_shortcode] is here.";
ShortcodeParser::get_active()->parse($text);
```

## Defining Custom Shortcodes
 
First we need to define a callback for the shortcode.

**app/src/Page.php**


```php
use SilverStripe\CMS\Model\SiteTree;

class Page extends SiteTree 
{
    private static $casting = [
        'MyShortCodeMethod' => 'HTMLText'
    ];

    public static function MyShortCodeMethod($arguments, $content = null, $parser = null, $tagName = null) 
    {
        return "<em>" . $tagName . "</em> " . $content . "; " . count($arguments) . " arguments.";
    }
}
```

[warning]
Note that the `$arguments` parameter potentially contains any arbitrary key/value pairs the user has chosen to include.
It is strongly recommended that you don't directly convert this array into a list of attributes for your final HTML markup
as that could lead to XSS vulnerabilities in your project.

If you want to use the `$arguments` parameter as a list of attributes for your final HTML markup, it is strongly recommended that you
pass the array through a filter of allowed arguments using [array_filter()](https://www.php.net/manual/en/function.array-filter.php)
or similar.
[/warning]

These parameters are passed to the `MyShortCodeMethod` callback:

 - Any parameters attached to the shortcode as an associative array (keys are lower-case).
 - Any content enclosed within the shortcode (if it is an enclosing shortcode). Note that any content within this
   will not have been parsed, and can optionally be fed back into the parser.
 - The ShortcodeParser instance used to parse the content.
 - The shortcode tag name that was matched within the parsed content.
 - An associative array of extra information about the shortcode being parsed. For example, if the shortcode is
   is inside an attribute, the `element` key contains a reference to the parent `DOMElement`, and the `node`
   key the attribute's `DOMNode`.


To register a shortcode you call the following.

**app/_config.php**


```php
// ShortcodeParser::get('default')->register($shortcode, $callback);

ShortcodeParser::get('default')->register('my_shortcode', ['Page', 'MyShortCodeMethod']);
```

## Built-in Shortcodes

Silverstripe CMS comes with several shortcode parsers already.

### Links

Internal page links keep references to their database IDs rather than the URL, in order to make these links resilient 
against moving the target page to a different location in the page tree. This is done through the `[sitetree_link]` 
shortcode, which takes an `id` parameter.


```php
<a href="[sitetree_link,id=99]">
```

Links to internal `File` database records work exactly the same, but with the `[file_link]` shortcode.


```php
<a href="[file_link,id=99]">
```

### Images

Images inserted through the "Insert Media" form (WYSIWYG editor) need to retain a relationship with
the underlying [Image](api:SilverStripe\Assets\Image) database record. The `[image]` shortcode saves this database reference
instead of hard-linking to the filesystem path of a given image.

```html
[image id="99" alt="My text"]
```

### Media (Photo, Video and Rich Content)

Many media formats can be embedded into websites through the `<object>` tag, but some require plugins like Flash or 
special markup and attributes. OEmbed is a standard to discover these formats based on a simple URL, for example a 
Youtube link pasted into the "Insert Media" form of the CMS.

Since TinyMCE can't represent all these variations, we're showing a placeholder instead, and storing the URL with a 
custom `[embed]` shortcode.

```html
[embed width=480 height=270 class=left thumbnail=https://i1.ytimg.com/vi/lmWeD-vZAMY/hqdefault.jpg?r=8767]
	https://www.youtube.com/watch?v=lmWeD-vZAMY
[/embed]
```

### Attribute and element scope

HTML with unprocessed shortcodes in it is still valid HTML. As a result, shortcodes can be in two places in HTML:

 - In an attribute value, like so: `<a title="[title]">link</a>`
 - In an element's text, like so: `<p>Some text [shortcode] more text</p>`

The first is called "element scope" use, the second "attribute scope"

You may not use shortcodes in any other location. Specifically, you can not use shortcodes to generate attributes or 
change the name of a tag. These usages are forbidden:

```html
<[paragraph]>Some test</[paragraph]>

<a [titleattribute]>link</a>
```

You may need to escape text inside attributes `>` becomes `&gt;`, You can include HTML tags inside a shortcode tag, but 
you need to be careful of nesting to ensure you don't break the output.
  
```html
<!-- Good -->
<div>
    [shortcode]
        <p>Caption</p>
    [/shortcode]
</div>

<!-- Bad: -->

<div>
    [shortcode]
</div>
<p>
    [/shortcode]
</p>
```

### Location

Element scoped shortcodes have a special ability to move the location they are inserted at to comply with HTML lexical 
rules. Take for example this basic paragraph tag:

```html
<p><a href="#">Head [figure,src="assets/a.jpg",caption="caption"] Tail</a></p>
```

When converted naively would become:

```html
<p><a href="#">Head <figure><img src="assets/a.jpg" /><figcaption>caption</figcaption></figure> Tail</a></p>
```

However this is not valid HTML - P elements can not contain other block level elements.

To fix this you can specify a "location" attribute on a shortcode. When the location attribute is "left" or "right"
the inserted content will be moved to immediately before the block tag. The result is this:

```html
<figure><img src="assets/a.jpg" /><figcaption>caption</figcaption></figure><p><a href="#">Head  Tail</a></p>
```

When the location attribute is "leftAlone" or "center" then the DOM is split around the element. The result is this:

```html
<p><a href="#">Head </a></p><figure><img src="assets/a.jpg" /><figcaption>caption</figcaption></figure><p><a href="#"> Tail</a></p>
```

### Parameter values

Here is a summary of the callback parameter values based on some example shortcodes.

```php
public function MyCustomShortCode($arguments, $content = null, $parser = null, $tagName) 
{
    // ..
}
```

```
[my_shortcode]
$attributes     => [];
$content         => null;
$parser         => ShortcodeParser instance,
$tagName         => 'my_shortcode')
```

```
[my_shortcode,attribute="foo",other="bar"]
$attributes      => ['attribute'  => 'foo', 'other'      => 'bar']
$enclosedContent => null
$parser          => ShortcodeParser instance
$tagName         => 'my_shortcode'
```

```
[my_shortcode,attribute="foo"]content[/my_shortcode]
$attributes      => ['attribute' => 'foo']
$enclosedContent => 'content'
$parser          => ShortcodeParser instance
$tagName         => 'my_shortcode'
```

## Limitations

Since the shortcode parser is based on a simple regular expression it cannot properly handle nested shortcodes. For
example the below code will not work as expected:

```html
[shortcode]
[shortcode][/shortcode]
[/shortcode]
```

The parser will raise an error if it can not find a matching opening tag for any particular closing tag

## Related Documentation

 * [Wordpress Implementation](https://codex.wordpress.org/Shortcode_API)
 * [How to Create a Google Maps Shortcode](how_tos/create_a_google_maps_shortcode)

## API Documentation

 * [ShortcodeParser](api:SilverStripe\View\Parsers\ShortcodeParser)
