# Resource

A resource is an identity of information on a PersonaSpace server. Resources are not always a single file, though they can be.

Examples of resources are below:

## Data, just data

Data resources are one of the few that are considered single file resources. They are simply json files that have representative data in them. Because there is no PersonaSpace resource presenter (`psr:presenter`) specified, the json is the `@data` property will be served as `application/json`. You can limit the properties returned by sending a custom header `ps-query`. This will return only requested properties and their dependencies

**Folder structure**
```
ebntly/
  data.json
  settings.json
  data/
    profile.json      
```

**`profile.json`**
```json
{
    "@context": {
        "i": "https://schema.personaspace.com/2019/draft/identity#",
        "c": "https://schema.personaspace.com/2019/draft/contact#internet"
    },
    "@acl": {...},
    "@data": {
        "i:fullname": "Eric L. Bentley",
        "i:emails": ["#/~6254982364", "#/~7234698772"],
        "~6254982364": {
            "type": "c:internet",
            "value": "ebntly.personaspace.com@psmail.net"
        },
        "~7234698772": {
            "type": "c:internet",
            "value": "ebntly@example.com"
        }
    }
}
```

**Served https://ebntly.personspace.com/profile**
```json
{
    "https://schema.personaspace.com/2019/draft/identity#fullname": "Eric L. Bentley",
    "https://schema.personaspace.com/2019/draft/identity#emails": [
        "https://ebntly.personspace.com/profile#/~6254982364",
        "https://ebntly.personspace.com/profile#/~7234698772"
    ],
    "https://ebntly.personspace.com/profile#/~6254982364": {
        "type": "https://schema.personaspace.com/2019/draft/contact#internet",
        "value": "ebntly.personaspace.com@psmail.net"
    },
    "https://ebntly.personspace.com/profile#/~7234698772": {
        "type": "https://schema.personaspace.com/2019/draft/contact:internet",
        "value": "ebntly@example.com"
    }
}
```


## Notes
Notes are a built-in app on PersonaSpace servers. When a visitor accesses https://ebntly.personaspace.com/private/notes/testnote, the file at `ebntly/data/private/notes/testnote/~1782093756.txt` will be presented. Alternatively, a visitor accessing https://ebntly.personaspace.com/private/notes/testnote/~1782093756.txt will be redirected to https://ebntly.personaspace.com/private/notes/testnote as the server will always search for the corresponding file containing the reference to it by searching up the tree.

When the PersonaSpace server router finds a `$refdata` property it will use the data there to contruct a response, unless "application/json" is the explicit `contentType` in the request.

**Folder structure**
```
ebntly/
  data.json
  settings.json
  data/
    private/
      notes/
        testnote.json
        testnote/
          ~1782093746.txt
```

**`testnote.json`**
```json
{
    "@context": {...},
    "@acl": {...},
    "@data": {
        "r:name": "testnote.txt",
        "r:editor": "notes",
        "r:assets": [
            "./testnote/~1782093746.txt"
        ],
        "r:present": {
            "rm:middleware": "text",
            "rm:data": {
                "http:content-type": "text"
            }
        }
    }
}
```

## Web pages
Simple web pages can be created with the built-in PersonaSpace server app, similar to most online front-end code editors. When saved it stores the CSS, JS, and HTML into separate files, with CSS and JS being imported by the HTML file. It supports cache breaking as well, so that changes to the CSS and JS will be updated automatically without visitors having to clear cache. 

**Folder structure**
```
ebntly/
  data.json
  settings.json *includes a reference to serve ?? when accessing their profile over the web, unless explicitly asking for json
  data/
    public/
      homepage.json
      homepage/
        ~7829461083.html
        ~2937504928.css
        ~8365043829.js
```

**`homepage.json`**
```json
{
    "@context": {
        "@": "./homepage/" 
    },
    "@acl": {...},
    "@data": {
        "r:name": "homepage.html",
        "r:app": "notes",
        "r:assets": [
            "@:~7829461083.html",
            "@:~2937504928.css",
            "@:~8365043829.js"
        ],
        "r:present": {
            "r:middleware": "webpage",
            "rm:data": {
                "http:content-type": "text/html",
                "psmWebpage:bodyClasses": ["dark"],
                "psmWebpage:entryHTML": "@:~7829461083.html",
                "psmWebpage:headContent": {
                    "psmWebpage:meta": [],
                    "psmWebpage:headCss": [],
                    "pwmWebpage:headJs": []
                },
                "psmWebpage:bodyClasses": [],
                "psmWebpage:bodyJs": []
            }
        }
        "$refdata": {
            "middleware": "webpage",
            "data": {
            "title": "ebntly - homepage",
            "content-type": "text/html",
            "head-content": [],
            "head-classes": [],
            "body-classes": ["dark"],
            "file": "./homepage/~7829461083.html",
            "css": ["./~2937504928.css"],
            "js": [
                {
                    "src": "https://code.jquery.com/jquery-3.3.1.slim.min.js",
                    "integrity": "sha256-3edrmyuQ0w65f8gfBsqowzjJe2iM6n0nKciPUp8y+7E=",
                    "crossorigin": "anonymous"
                }, "./~8365043829.js"]
            }
        }
    }
}
```
**`~7829461083.html`**
```html
<p>Hello <span id="name">world</span></p>
```
**`~2937504928.css`**
```css
.dark {
    background: #888888;
    color: #ffffff;
}
```
**`~8365043829.js`**
```js
$(document).ready(() => {
    const name = () => { /* do something to get a name*/ }
    name && $('#id').text(name)
})
```
**https://ebntly.personaspace.com/profile**
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>ebntly - homepage</title>
    <link rel="stylesheet" type="text/css" href="./~2937504928.css?h=~7829461083" />
  </head>
  <body>
    <p>Hello <span id="name">world</span></p>
    <script type="text/javascript"
        src="https://code.jquery.com/jquery-3.3.1.slim.min.js"
        integrity="sha256-3edrmyuQ0w65f8gfBsqowzjJe2iM6n0nKciPUp8y+7E="
        crossorigin="anonymous">
    </script>
    <script type="text/javascript" src="./~8365043829.js?h=~7829461083"></script>
  </body>
</html>
```
