# Scrap files

Adding a new website to Cathode means creating a json file, we call it a **scrap file**.

Each scrap includes the author's contacts, general information about the website and a list of recipes.

Each recipe tells Cathode's engine what to do for a certain section of the website the scrap was made for. For instance you can have a recipe to parse the website's homepage and another one for the article pages. You can have as many recipes per website as you want. The correct recipe is chosen by regexp matching the request with the url field definition.

See below a scrap for Twitter, [twitter.com.json][4]:

```json
{
    "name": "Twitter",
    "author": {
        "name": "Celso Martinho",
        "email": "celso at brpx dot com",
        "github": "celso",
        "twitter": "celso"
    },
    "recipes": [
        {
            "title": "User feed",
            "url": "/.*",
            "cache": 320,
            "scope": ".content",
            "fields": [
                {
                    "title": "p.TweetTextSize@html | spaceout | strip | trim | clean",
                    "embed": "p.TweetTextSize a[data-pre-embedded=\"true\"]@html",
                    "url": "small.time a@href",
                    "date": "small.time span._timestamp@data-time",
                    "thumb": "div.AdaptiveMedia-photoContainer@data-image-url"
                }
            ]
        }
    ]
}
```

The scrap file must be named **[sub.]domain.com.json**.

## Header

The header of a scrap starts with a few attributes:

 * **name** (mandatory) - Name of the scrap. Usually the name of the website.
 * **author/name** (mandatory) - The name of the author of this scrap.
 * **author/email** (optional) - E-mail of the author. In the future we may use the author's contacts to notify him of deprecated of failing scraps.
 * **author/github** (optional) - Github usename
 * **author/twitter** (optional) - Twitter usename (no @)

## Recipe

Each recipe includes the following fields:

 * **title** (mandatory) - Title of the recipe
 * **url** (mandatory) - URL regexp. It can be relative to the scrap's top level domain (as the example shows), protocol agnostic (ex: //twitter.com/.*) or a fqdn (ex: https://twitter.com/.*).
 * **cache** (optional) - Cathode has default cache of 320 seconds for each recipe. You can override this setting here.
 * **scope** (optional) - You can scope the query to a specific element selector that matches a class, id or attribute.
 * **fields** (mandatory) - The fields you'd like to extract with the recipe. Details below.

### Fields

The **fields** is a flat object, or an array with an object, with a list of name/query pairs. If the object is embbeded in an array [ object ], then you're looking for a collection of items that repeats itself with the same pattern across the document.

Common field names you should use if you want multiple output formats (ex: rss) to work as expected:

 * **title** - title of the item
 * **description** or **body** - longer description
 * **url** - item external link
 * **date** - date
 * **image** - image for item

The query pair follows a [jQuery selectors like syntax][3]. Check the examples below to see it in action:

### Query examples

Here's a few examples to give an idea on the kind of queries you can perform on a page.

Let's use the following document structure as the input:

```
<html>
    <head><title>Titly</title></head>
    <body>
        <img src="/imgs/logo.png" class="center">
        <p>Company logo</p>
        <h1>List of items</h1>
        <ul>
            <li>First item</li>
            <li>Second item</li>
        </ul>
        <h2>Story</h2>
        <div class="story">
            <p>Would you tell me, please, which way I ought to go from here?</p>
            <p>That depends a good deal on where you want to get to, said the Cat.</p>
        </div>
        <table>
            <tr>
                <td>John</td>
                <td>Doe</td>
            </tr>
            <tr>
                <td>Mike</td>
                <td>Albert</td>
            </tr>
        </table>
        <div id="disclaimer">Generic disclaimer</div>
    </body>
</html>
```

**Grabbing the title**

```json
{
    "fields": { "title": "head title" }
}
```

```json
[ { "title": "Titly" } ]
```

**Grabbing the items**

```json
{
    "scope": "ul",
    "fields": [ { "items": "li" } ]
}
```

```json
{ "items": [ "First item", "Second item" ] }
```

**Grabbing the image location**

```json
{
    "fields": { "imgsrc": "img@src" }
}
```

```json
[ { "imgsrc": "/imgs/logo.png" } ]
```

**Grabbing the story, cleaning the paragraphs, and joining them in a single string**

```json
{
    "fields": { "text | join": [ "body .story p | clean" ] }
}
```

```json
[ { "text": "Would you tell me, please, which way I ought to go from here? That depends a good deal on where you want to get to, said the Cat." } ]
```

**Grabbing the table rows, using scope**

```json
{
    "scope": "table tr",
    "fields": [ {
        "firstName": "td:nth-child(1)",
        "secondName": "td:nth-child(2)"
    } ]
}
```

```json
[ { "firstName": "John", "secondName": "Doe" },
  { "firstName": "Mike", "secondName": "Albert" } ]
```

## Available filters

There are two types of filters you can use: pre and post.

Pre filters are applied to the queries, at the extraction level.

Post filters are applied to end result, at the delivery level.

You can see how this works in the *grabbing the story* example above.

### Pre filters

 * **trim** - trimp spaces from string borders
 * **reverse** - reverse string
 * **slice(start,end)** - slice string from offset start to end
 * **strip** - strips html from result
 * **spaceout** - unglues words from tags, adding a space between
 * **clean** - cleans up excessive spaces and carriage returns, turns them into a single space

### Post filters

 * **join** - joins an array of results into a single string

## Output formats

The output format is determined by the Cathode's feed url extension.

Currently, we support the following output transformations:

 * **JSON** - JSON format ([example here][6]).
 * **RSS** - RSS feed format ([example here][7]).

## Other notes

Cathode's engine is slightly based on [X-ray][1] with changes. X-ray is an excellent and modern web scraper known for its flexible schema, composable API and modular architecture. Its [query syntax][2] is based on enhanced jQuery-like strings.

## cathode-cli

[cathode-cli][8] is a command line tool that uses Cathode's API and helps developers to build and test their scrap formulas.


[1]: https://github.com/lapwinglabs/x-ray
[2]: https://github.com/lapwinglabs/x-ray/#selector-api
[3]: http://api.jquery.com/category/selectors/
[4]: https://github.com/brpx/cathode-scraps/blob/master/twitter.com.json
[5]: https://github.com/brpx/cathode-scraps
[6]: https://cathode.io/VTUrK7Q7zKbYtXlrEimbIbIwQMBH9Z.json
[7]: https://cathode.io/VTUrK7Q7zKbYtXlrEimbIbIwQMBH9Z.rss
[8]: https://github.com/brpx/cathode-cli
