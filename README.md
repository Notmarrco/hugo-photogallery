# Photo gallery theme for Hugo

Add a gallery to your Hugo site.

It provides you with some predefined templates:

* photoswipe (from [sigal](https://github.com/saimn/sigal))
* fluid (fluid gallery like pinterest, also using photoswipe to display the carousel)
* tuftery (fluid gallery using E. Tufte style guidelines)

You can easily add your own template by defining `layouts/partials/your_template/`. See below. (TODO)

You also get a shortcode `{{ <photos> }}` to include a gallery inside another page of content.

## Installation

### Get the theme

Install the theme as a git submodule, or download it inside your hugo's project "theme" folder.

Activate it in your config file (probably `config.toml`):

```toml
#theme = ["name_of_photogallery_theme_folder", ...other_themes] # Example
theme = ["notmarrco.hugo-photogallery"]
```

### Install dependencies

First you need to have yarn or npm installed.

Then in the theme subfolder :

```js
yarn install
```

### Photoswipe (if necessary for the template)

We need >=v4.4 for photoswipe, but the only available version on npm is v4.1.
So we have to download it manually inside `node_modules` (to be consistent).

```sh
wget https://github.com/andi34/PhotoSwipe/archive/refs/tags/v4.4.0.zip .
unzip v4.4.0.zip
mv PhotoSwipe-4.4.0/ node_modules/photoswipe
```

### Include the js/css assets

Lastly you have to add the photogallery assets in your `layout/baseof.html` (or override the one from your theme ...).

To do this, just add `photogallerly_head` partial, and it will choose the right header
for the gallery template you will use, based on the frontmatter `gallery` option.

```html
<!-- eg. in layouts/_default/baseof.html -->
<head>
    {{- partial "photogallery_head.html" . -}}
</head>
```

If you use the shortcode with a named parameter : `{{ <photos gallery=fluid> }}`, then the photogallery_head
cannot access the `gallery` parameter, so you have to manually add the right `xx_head` partial in your head.

### Usage frontmatter

In your contents file, set the following frontmatter. (also listed in the `gallery` archetype)

```md
+++
data: datafile.yml
gallery: fluid # template chosen, default to photoswipe
shareOptions: ["download"] # options to show in photoswipe
defaultWidth: xxx
defaultHeight: yyy
hideCaption: true # default to false, hide the title/description of the image
+++
```

## Videos

Videos must be encoded to be played by the browser.

To do that, see [Vestride - encoding video gist](https://gist.github.com/Vestride/278e13915894821e1d6f).

## Usage

To invoke the display of the photogallery we can either :

* [x] use the shortcode `{{ <photos> }}` inside a markdown file, and it will use the frontmatter
  to determine the gallery theme and the photos to display
* [ ] set the tag "TO_BE_DETERMINED" in your frontmatter to `albums` or `album` or ...
* [ ] put your markdown files inside  `content/photos` directory

Multiple solutions to organize your files :

### Images path stored in a data.yaml file

```sh
.
├── content
│   └── photos
│       └── _index.md
├── data
│   └── photos_list.yml
└── assets
    └── images ??? TODO // URL -- api requests ...  // thumbs ...
```

```md
+++
datafile: photos_list.yaml
+++

{{<photos>}}
```

#### Photolist.yaml format

```yaml
zip: path_to_zip_file
sharedUntil: YYYY-mm-dd
medias:
  - title: Title
    type: image # or video
    mime: video/mp4 # used only for video
    thumb: path_to_thumb
    url: path_to_file
    width: 1024 # size in px for photoswipe # override the defaultWidth in config.toml
    height: 900 # size in px for photoswipe # override the defaultHeight in config.toml
```

### Multiple files (NOT YET IMPLEMENTED)

This allow the creation of albums with text and descriptions for images, and even
any markdown text to go with.

But it's more verbose, and we need to create a lot of files... (sigal can help probably)

```sh
.
├── content
│   └── photos
│       ├── _index.md
│       └── album1 // RECURSIVE ?
│           ├── _index.md
│           ├── photo1.md
│           ├── photo2.md
│           ├── photo3.md
│           └── ...
├── data
│   └── photos_list.yml
└── assets
    └── images ??? TODO
```

and each file may be like :

```md
+++
path: photo.jpg
exif:
  xxx:
albums: ["vacances"] => taxonomy ? useless if sub directories
type: "photo" | "album" ?
+++

Long markdown description ...

```

Le problème c'est de ne pas vouloir rendre dispo toutes les photos d'un rep, parce que avec
le serveur web ça pourrait exposer **toutes** les photos :-\

TODO : use Hugo to manage photo files (voir article loic ?)

```toml
[photogallery]
photosUrlPrefix = 'static/photos'
photosThumbUrlPrefix = 'static/thumbs'
```

### Hybrid

Datafile could contain a list of photos we wish to expose, allowing to mix photos in multiple albums.

And the path can be the path to the md file and not the JPG/Image file with thumbs et al.

## Custom template

To create a custom template, just create a subfolder with your template name (like `layouts/partials/my_template`),
containing the following files :

```sh
└── my_template
    ├── my_template_head.html # mandatory, contains the assets to be loaded in the html <head>
    ├── my_template.html # mandatory, load the template
    └── etc. # optional files : if necessary to reuse a part of the template, to be called in my_template.html
```

### my_template.html

```html
<!-- 
    MY_TEMPLATE gallery theme.

    Description.
-->

<!-- if the main_photogallery is enough, just load it, or use a custom my_template_main -->
{{ partial "photogallery_main" . }}

<!-- Include root element and js if necessary, for example photoswipe_root, or my_template_root -->
{{ partial "photoswipe/photoswipe_root" . }}
```

### my_template_head.html

This partial is called by `photogallery_head` where you put it, and will insert `my_template_head`.

An example of this `my_template_head` :

```html
{{ $echojs := resources.Get "js/vendor/echo-js/echo.min.js" }}
<script type="text/javascript" src="{{ $echojs.Permalink }}"></script>

{{ $pscss := resources.Get "js/vendor/photoswipe/photoswipe.css" }}
<link rel="stylesheet" href="{{ $pscss.Permalink }}">
```

> NB: photogallery auto mounts some node_modules inside `assets/js/vendor`, add your own
> in `config.toml`.
