---
title: "Creating a Hugo Theme From Scratch"
date: 2020-01-25T00:43:54Z
draft: false
tags: ["software", "tools"]
author: "Pedro Lopez"
---

![image](/images/creating-a-hugo-theme-from-scratch.jpg)

In this post I will show you how to build a minimalist theme for Hugo.

<!--more-->

#### Pre-requisites

- A basic understanding of Hugo templating and folder structure
- Having installed Hugo locally https://gohugo.io/getting-started/installing/

Hugo is written in Go, but you do not need to install it for now.

#### Getting started

- Create a new Hugo site
{{< highlight bash>}}hugo new site example{{< /highlight >}}
- Go into the newly created folder
{{< highlight bash>}}cd example{{< /highlight >}}
- Create a new theme
{{< highlight bash>}}hugo new theme exampleTheme{{< /highlight >}}
- This will have created a `themes` folder with a `exampleTheme` subfolder, the folder structure should look like this
{{< highlight bash>}}
.
├── archetypes
│   └── default.md
├── config.toml
├── content
├── data
├── layouts
├── resources
│   └── _gen
│       ├── assets
│       └── images
├── static
└── themes
    └── exampleTheme
        ├── LICENSE
        ├── archetypes
        │   └── default.md
        ├── layouts
        │   ├── 404.html
        │   ├── _default
        │   │   ├── baseof.html
        │   │   ├── list.html
        │   │   └── single.html
        │   ├── index.html
        │   └── partials
        │       ├── footer.html
        │       ├── head.html
        │       └── header.html
        ├── static
        │   ├── css
        │   └── js
        └── theme.toml
{{< /highlight >}}
- Check that everything is working fine by starting the server
{{< highlight bash>}}
hugo server
{{< /highlight >}}
- Navigate to http://localhost:1313, and check there are no errors in the developer console (Chrome) or equivalent in your browser of choice
- Check the `config.toml` file and make sure it looks like this
{{< highlight html>}}
baseURL = "http://example.org/"
languageCode = "en-us"
title = "My New Hugo Site"
{{< /highlight >}}
- I will be using this for the rest of the post, but feel free to customise it

#### Partials

Partials are small, context-aware components that can be used economically to keep your templating DRY.

##### Head

- Open `example/themes/exampleTheme/layouts/partials/head.html` in a text editor
- This will be the place for metadata like the document title, character set, styles, scripts, and other meta information
- Paste the following
{{< highlight html>}}
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" type="text/css" href="/css/bootstrap.min.css">
    <link rel="stylesheet" type="text/css" href="/css/style.css">
    {{ $title := print .Site.Title " | " .Title }}
    {{ if .IsHome }}{{ $title = .Site.Title }}{{ end }}
    <title>{{ $title }}</title>
</head>
{{< /highlight >}}
- Here we are specifying the charset that our document will use, `utf-8` in this case, the viewport information and couple of stylesheets
- Let's go ahead and add the missing CSS files
- Go to `example/themes/exampleTheme/static/css` and create a file called `style.css`, we will place our custom CSS here later on
- Go to https://getbootstrap.com/ and download the latest version of Bootstrap (v4.4.1 at the time of writing)
- Extract the ZIP file and copy `bootstrap.min.css` to `example/themes/exampleTheme/static/css`
- Finally, for the title, we will render the site title as it appears in `config.toml` when we are at the homepage and we will concatenate the section name when we are in a section page e.g. `My New Hugo Site | Posts`

##### Script

- In addition to Bootstrap, we are going to use Feather icons to display icons on our site
- Go to https://github.com/feathericons/feather#client-side-javascript and download `feather.min.js` into `example/themes/exampleTheme/static/js`
- Create a file `example/themes/exampleTheme/layouts/partials/script.html` and paste the following
{{< highlight html>}}
<script src="/js/feather.min.js"></script>
<script>
  feather.replace()
</script>
{{< /highlight >}}

##### Header and footer

- Let's add a nav to the header so that we can navigate between the different sections of our site
- Open `example/themes/exampleTheme/layouts/partials/header.html` and paste the following
{{< highlight html>}}
<div id="nav-border" class="container">
    <nav id="nav" class="nav justify-content-center">
        {{ range .Site.Menus.main }}
        <a class="nav-link" href="{{ .URL }}">
        {{ if .Pre }}
        {{ $icon := printf "<i data-feather=\"%s\"></i> " .Pre | safeHTML }}
        {{ $icon }}
        {{ end }}
        {{ $text := print .Name | safeHTML }}
        {{ $text }}
        </a>
        {{ end }}
    </nav>
</div>
{{< /highlight >}}
- Here we are creating a nav menu that will be centred at the top of each page
- Each menu item will have a link and also an icon if we configure it as a `pre` property in `config.toml`, more on that later
- For the footer let's just add a basic copyright disclaimer
{{< highlight html>}}
<p class="footer text-center">Copyright (c) {{ now.Format "2006"}} John Doe</p>
{{< /highlight >}}
- You can replace `John Doe` with your name
- The current year will be displayed as well
- If you are wondering why "2006", you can find out more about it [here](https://golang.org/src/time/format.go#L479)

##### Metadata

- Let's create one more partial to display metadata about each post e.g. date and tags
- Create a new file `example/themes/exampleTheme/layouts/partials/metadata.html`
{{< highlight html>}}
{{ $dateTime := .PublishDate.Format "2006-01-02" }}
{{ $dateFormat := .Site.Params.dateFormat | default "Jan 2, 2006" }}
<i data-feather="calendar"></i>
<time datetime="{{ $dateTime }}">{{ .PublishDate.Format $dateFormat }}</time>
{{ with .Params.tags }}
<i data-feather="tag"></i>
{{ range . }}
{{ $href := print (absURL "tags/") (urlize .) }}
<a class="btn btn-sm btn-outline-dark tag-btn" href="{{ $href }}">{{ . }}</a>
{{ end }}
{{ end }}
{{< /highlight >}}
- We are using Feather icons to spice things up a little, but in fact we are just rendering the publication date and the tags
- Tags will be passed in as parameters from the post itself in the front matter, more information [here](https://gohugo.io/content-management/front-matter/)
{{< highlight markdown>}}
---
author: "John Doe"
title: "My First Post"
date: "2006-02-01"
tags: ["foo", "bar"]
---
{{< /highlight >}}

#### Default layouts

- Now we have all the basic ingredients to work on the layouts
- These are placed under `example/themes/exampleTheme/layouts/_default` and, at least, consist of
  - baseof
  - list
  - single

##### Baseof

This will be the base template for all pages, and should reference the different partials we edited above.

- Open `example/themes/exampleTheme/layouts/_default/baseof.html`, it should look like this
{{< highlight html>}}
<!DOCTYPE html>
<html>
    {{- partial "head.html" . -}}
    <body>
        {{- partial "header.html" . -}}
        <div id="content">
        {{- block "main" . }}{{- end }}
        </div>
        {{- partial "footer.html" . -}}
    </body>
</html>
{{< /highlight >}}
- Let's also include the `script.html` partial with our Feather icons
{{< highlight html>}}
<!DOCTYPE html>
<html>
    {{- partial "head.html" . -}}
    <body>
        {{- partial "header.html" . -}}
        <div id="content">
        {{- block "main" . }}{{- end }}
        </div>
        {{- partial "footer.html" . -}}
        {{- partial "script.html" . -}}
    </body>
</html>
{{< /highlight >}}
- It's good practice to place our scripts at the end of the HTML code so that it doesn't block rendering the page

##### List

This layout will be used to display the list of posts on our site.

- Open `example/themes/exampleTheme/layouts/_default/list.html`
- We will define the `main` section of the `baseof` layout like this
{{< highlight html>}}
{{ define "main" }}
<h1>{{ .Title }}</h1>
{{ range .Pages.ByPublishDate.Reverse }}
<p>
    <h3><a class="title" href="{{ .RelPermalink }}">{{ .Title }}</a></h3>
    {{ partial "metadata.html" . }}
    <a class="summary" href="{{ .RelPermalink }}">
        <p>{{ .Summary }}</p>
    </a>
</p>
{{ end }}
{{ end }}
{{< /highlight >}}
- We are displaying the metadata i.e. publication date and tags, and the summary of the post
- We are also sorting posts by publication date in reverse order, most recent first

##### Single

This layout will be used to display a single post.

- Open `example/themes/exampleTheme/layouts/_default/single.html`
- Similarly to what we did above, let's define the `main` section
{{< highlight html>}}
{{ define "main" }}
<h1>{{ .Title }}</h1>
{{ partial "metadata.html" . }}
<br><br>
{{ .Content }}
{{ end }}
{{< /highlight >}}
- Like before, we will display the metadata, but in this case, we will display the full content instead of the summary

#### Homepage

This will be the landing page at the root of our site.

- Open `example/themes/exampleTheme/layouts/index.html` in a text editor
- With Bootstrap in place, we can now use the Jumbotron component to render a hero section
- Paste the following
{{< highlight html>}}
{{ define "main" }}
<div id="home-jumbotron" class="jumbotron text-center">
  <h1 class="title">{{ .Site.Title }}</h1>
</div>
{{ end }}
{{< /highlight >}}

#### 404

This is used to customise an error message for when a link is broken. However, Hugo comes with a default one, so let's delete it for now.

Feel free to play with the contents of the file if you want to customise your error messages.

#### Configuration

We have created the template, now we need to tell Hugo to use it for our site.

- Open `example/config.toml` and add a line to select the theme like this
{{< highlight toml>}}
theme = "exampleTheme"
{{< /highlight >}}
- We'll also need a menu to navigate to the different sections, let's add it
{{< highlight toml>}}
[menu]
  [[menu.main]]
    name = "Home"
    pre = "home"
    url = "/"
    weight = 1
  [[menu.main]]
    name = "Posts"
    pre = "pen-tool"
    url = "/posts/"
    weight = 2
  [[menu.main]]
    name = "Tags"
    pre = "tag"
    url = "/tags/"
    weight = 3
{{< /highlight >}}
- Here we are defining a menu, and referencing it with the name `main` to match our header (`.Site.Menus.main`)
- In Hugo there is another way to create a menu quickly, and that is by adding this line to your `config.toml` file instead, more information [here](https://gohugo.io/templates/menu-templates/#section-menu-for-lazy-bloggers)
{{< highlight toml>}}
sectionPagesMenu = "main"
{{< /highlight >}}
- However we want to be able to customise our nav menu and add icons, therefore we'll go with the manual approach

#### Write your first post

The real value of a website is its content. Let's create a post before we have a look at the final result.

- Open a terminal and use the following command from the root of your site to create a post
{{< highlight bash>}}
hugo new posts/my-first-post.md
{{< /highlight >}}
- Open the newly created `example/content/posts/my-first-post.md`
- Add some tags to the front matter
{{< highlight markdown>}}
tags: ["foo", "bar"]
{{< /highlight >}}
- Add some content under the front matter
- Hugo automatically takes the first 70 words of your content as its summary and stores it into the `.Summary` variable
- Instead, you can manually define where the summary ends with a `<!--more-->` divider
- Alternatively, you can add a `summary` to the front matter if you don't want your summary to be the beginning of your post
- The final result should look similar to this
{{< highlight markdown>}}
---
title: "My First Post"
date: 2020-01-26T23:11:13Z
draft: true
tags: ["foo", "bar"]
---
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor 
incididunt ut labore et dolore magna aliqua. Pellentesque eu tincidunt tortor 
aliquam nulla facilisi cras fermentum odio. A erat nam at lectus urna duis. 
Sed velit dignissim sodales ut eu sem. Lectus urna duis convallis convallis 
tellus. Diam sit amet nisl suscipit adipiscing bibendum est. Sed felis eget 
velit aliquet sagittis id consectetur. Vulputate dignissim suspendisse in est 
ante in nibh mauris cursus. Morbi quis commodo odio aenean. Mollis nunc sed id 
semper risus in hendrerit gravida rutrum.

<!--more-->

Ac ut consequat semper viverra nam. Hac habitasse platea dictumst vestibulum 
rhoncus. Amet porttitor eget dolor morbi non. Justo eget magna fermentum 
iaculis eu non. Id eu nisl nunc mi ipsum faucibus vitae aliquet nec. Aliquam 
id diam maecenas ultricies. Non sodales neque sodales ut etiam. Amet massa 
vitae tortor condimentum lacinia quis. Erat imperdiet sed euismod nisi porta. 
Nisl suscipit adipiscing bibendum est ultricies integer quis auctor. Viverra 
suspendisse potenti nullam ac. Tincidunt id aliquet risus feugiat in. Varius 
quam quisque id diam vel. Egestas erat imperdiet sed euismod nisi. Scelerisque 
felis imperdiet proin fermentum leo vel orci porta non. Ut faucibus pulvinar 
elementum integer. Fermentum odio eu feugiat pretium nibh ipsum consequat nisl.
{{< /highlight >}}

#### Trying it out

- Open a terminal and run the following from the root folder of your site
{{< highlight bash>}}
hugo server -D
{{< /highlight >}}
- The `-D` option means to include content marked as draft
- Alternatively, edit the front matter of your post and change this line `draft: true` to `draft: false`
- Navigate to http://localhost:1313
- You should see something like this
![image](/images/creating-a-hugo-theme-from-scratch-result.jpg)

#### Final thoughts

In this post I've showed how to create a simple template and apply it to a new Hugo site. From here, you can customise the look and feel with the `style.css` file and modify the template to add more features.

In future posts I will show how to add share buttons and analytics to our site.

If you want to jump ahead and see the final template that I use for my site, you can find it in GitHub [here](https://github.com/plopcas/papaya).

Thanks for reading!