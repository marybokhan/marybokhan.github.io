# marybokhan.github.io

![No JavaScript](https://img.shields.io/badge/JavaScript-none-brightgreen.svg)

## Shortcodes

The theme adds two custom shortcodes related to image handling.

### `image`

Used to show images.

Required arguments:

- **`src`**

Optional arguments:

- **`alt`**
- **`position`** (center \[default\] | left | right)
- **`style`**

Example:

```
{{ image(src="/img/hello.png", alt="Hello Friend",
         position="left", style="border-radius: 8px;") }}
```
  
### `figure`

Same as `image`, but with a few extra optional arguments:

- **`caption`**
- **`caption_position`** (center \[default\] | left | right)
- **`caption_style`**

Example:

```
{{ figure(src="http://rustacean.net/assets/rustacean-flat-gesture.png",
          style="width: 25%;",
          position="right"
          caption_position="left",
          caption="Ferris, the (unofficial) Rust mascot",
          caption_style="font-weight: bold; font-style: italic;") }}
```

## Configuration

### Colors

Both the accent colors and background colors are
configurable.

By default, both accent and background are set
to `blue`.

To configure menu, add this in `[extra]` section
of your `config.toml`:

```toml
[extra]

# One of: blue, green, orange, pink, red.
# Defaults to blue. 
accent_color = "green"

# One of: blue, dark, green, orange, pink, red.
# Enabling dark background will also modify primary font color
# to be darker.
# Defaults to accent color (or, if not accent color specified, to blue). 
background_color = "dark"
```

### Logo text and link

You can set the "logo" text and what it links to,
by modifying `config.toml` like so:

```toml
[extra]

# The logo text - defaults to "Terminimal theme"
logo_text = "My blog"

# The logo link - defaults to base_url.
logo_home_link = "/take/me/away!"
```

### Author and copyright

You can set the footer's copyright author name like this:

```toml
[extra]

# Author name: when specified, modifies the default
# copyright text. Apart from author, it will
# contain current year and a link to the theme.
author = "My Name"
``` 

If you don't like the default copyright text,
you can set it to completely custom HTML:

```toml
[extra]

# Copyright text in HTML format. If specified,
# entirely replaces default copyright and author.
copyright_html = "My custom&nbsp;<b>copyright</b>"
```

### Menu

The menu is optional, static (all items are always shown,
no matter what the screen size) and fully user-configurable.

To configure menu, add this in `[extra]` section
of your `config.toml`:

```toml
[extra]

# menu is enabled by adding menu_items (optional)
menu_items = [
    # each of these is optional, name and url are required
    # $BASE_URL is going to be substituted by base_url from configuration
    {name = "blog", url = "$BASE_URL"},
    
    # tags should only be enabled if you have "tags" taxonomy
    # see documentation below for more details
    {name = "tags", url = "$BASE_URL/tags"},
    {name = "archive", url = "$BASE_URL/archive"},
    {name = "about me", url = "$BASE_URL/about"},
    
    # set newtab to true to make the link open in new tab
    {name = "github", url = "url-to-your-github", newtab = true},
]
```

### Tags

The theme optionally supports tags. To enable them, create
a "tags" taxonomy in your `config.toml`:

```toml
taxonomies = [
    {name = "tags"},
]
```

Enabling tags will create a new `/tags` page, and 
cause them to show up in `archive` section. Note
that you still need to create a menu link to the tags
page manually.

### Pagination

Pagination is fully supported for post list (main site)
and intra-post (you can navigate to earlier and later posts).

To make sure pagination works properly, you must first configure
it in `content/_index.md`:

```
+++
# number of pages to paginate by
paginate_by = 2

# sorting order for pagination
sort_by = "date"
+++
```

Then, tweak the theme's pagination config in `config.toml`:

```toml
[extra]

# Whether to show links to earlier and later posts
# on each post page (defaults to true).
enable_post_view_navigation = true

# The text shown at the bottom of a post,
# before earlier/later post links.
# Defaults to "Thanks for reading! Read other posts?"
post_view_navigation_prompt = "Read more"
```

### Language code

Internationalization / translation is not supported
but you can set the HTML language code for your
site:

```toml
default_language = "en"
```

### Hack font subset

By default, the theme uses a mixed subset of the Hack font.
Normal weight font uses full character set
(for Unicode icons and special symbols), but all others
(bold, italic etc) use a limited subset.

This results in much smaller transfer sizes, but the subset
might not contain all the Unicode characters you need.

You can enable full unicode support in `config.toml`:

```toml
[extra]

# Use full Hack character set, not just a subset.
# Switch this to true if you need full unicode support.
# Defaults to false.
use_full_hack_font = true
```

Also see [Hack's docs](https://github.com/source-foundry/Hack/blob/master/docs/WEBFONT_USAGE.md).

### Favicon

The theme supports adding a global favicon (applies to
all pages) to the site:

```toml
# Optional: Global favicon URL and mimetype.
#           Mimetype defaults to "image/x-icon".
#           The URL should point at a file located
#           in your site's "static" directory.
favicon = "/favicon.png"
favicon_mimetype = "image/png"
```

All the configuration options are also described in
[`config.toml`](../master/config.toml).

## Extending

Each of the templates defines named blocks, so
it should be quite easy to customize the most common things.

For example, if you want to add extra `<meta>` tags to the
base template, `index.html`, create file like this in `templates/index.html`:

```html
{% extends "terminimal/templates/index.html" %}

{% block extra_head %}
    <meta name="description" content="My awesome website"/>
    <meta name="keywords" content="Hacking,Programming,Ranting"/>
{% endblock %}
```