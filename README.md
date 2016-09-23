Streampub
---------

A streaming EPUB3 writer.

## EXAMPLE

```
var Streampub = require('streampub')
var fs = require('fs')
var epub = new Streampub({title: 'My Example'})
epub.setAuthor('Example User')
epub.pipe(fs.createWriteStream('example.epub'))
epub.write({index: 0, chapterName: 'Chapter 1', fileName: 'chapter-1.xhtml', content: '<b>doc content</b>'})
epub.end()
```

## USAGE

### var epub = new Streampub(*opts*)

*opts* is an object that optionally has the following properties:

* *title* - The title of the epub, defaults to "Untitled"
* *author* - The name of the author of the epub
* *modified* - When the epub was last modified, defaults to now. (Date object)
* *published* - When the source material was published. (Date object)
* *source* - The original URL or URN of the source material
* *language* - The 2 digit language code of the material, defaults to "en".
* *description* - A brief description or summary of the material

All of the options can be set after object creation with obvious setters:

### epub.setTitle(title)
### epub.setAuthor(author)
### epub.setModified(modified)
### epub.setPublished(published)
### epub.setSource(source)
### epub.setDescription(description)
### epub.setLanguage(language)

Identifies the language used in the book content. The content has to comply with [RFC 3066](http://www.ietf.org/rfc/rfc3066.txt). [List of language codes](http://www.loc.gov/standards/iso639-2/php/code_list.php).

### The Streampub Object

The Streampub object is a transform stream that takes chapter information as
input and outputs binary chunks of an epub file. It's an ordinary stream so you
can pipe into it or write to it and call `.end()` when you're done.

### var epub.write(*obj*, *callback*)

This is the usual stream write function.

#### Adding a chapter
The object must have the following keys:
* **`chapterName`** - The name of the chapter in the index.
* **`content`** - *(optional)* The HTML content of this chapter.  This will be passed
  through `htmltidy` in order to make it valid XHTML.
* **`sourceFileName`** - *(optional)* Path to the source HTML file with chapter
* **`index`** - *(optional)* Where the chapter should show up in the index. These numbers
  can have gaps and are used for ordering ONLY. Duplicate index values will
  result in the earlier chapter being excluded from the index. If not specified will
  be added after any where it _was_ specified, in the order written.
* **`fileName`** - *(optional)* The filename to use *inside* the epub. This only matters if
  you want to do links between chapters. This should end in `.xhtml`.

(`sourceFilename` or `content` should be set)

#### Adding other type of assets
You can add other types of files to the epub. For example images or stylesheets.

The object must have the following keys:
* **fileName** - The filename to use *inside* the epub.
* **sourceFileName** - Path to the source file.

#### htmltidy options
`htmltidy` options used are:

```
    'output-xhtml': true,
    'doctype': 'html5',
    'add-xml-decl': true,
    'coerce-endtags': true,
    'enclose-block-text': true,
    'drop-proprietary-attributes': true,
    'strict-tags-attributes': true,
    'clean': true,
    'quote-nbsp': false,
    'numeric-entities': true
```

## Cover image

The epub specification does not contain a standarized way to include book covers. There is however a "best practice" that will work in most reader applications. `streampub` has some magic under the hood to correctly add a cover image. The only requirements are that the file needs to be in JPEG format and should be max 1000x1000 pixels.

### Example
```
var Streampub = require('./index')
var fs = require('fs')

var epub = new Streampub({title: 'My Example'})
epub.setAuthor('Example author')
epub.pipe(fs.createWriteStream('example.epub'), {end: false})
epub.write({id: 'cover-image', sourceFileName: 'image.jpg'})
epub.write({chapterName: 'Chapter 1', content: '<h1>Chapter 1</h1><b>doc content</b>'})
epub.write({chapterName: 'Chapter 2', content: '<h1>Chapter 2</h1><b>doc content</b>'})
epub.end()
```

## VALIDATION

This takes care to generate only valid XML using programmatic generators and
not templates

Epubs produced by this have been validated with
[epubcheck](https://github.com/idpf/epubcheck).  No warnings outside of
content warnings should be present.

Content warnings ordinarily only happen if your content contains broken links–usually relative links to resources
that don't exist in the epub.

## PRIOR ART

There are a bunch of epub generators already available.  Many are pre EPUB3.
Most work off of files on disk rather than in memory constructs.  Only one
other provides a stream that I was able to find was
[epub-generator](https://npmjs.com/package/epub-generator) and it only
provides a read stream.  I wanted to be able to build a full pipeline for,
for example, backpressure reasons.  I also very much wanted to be able to
set epub metadata after object construction time.
