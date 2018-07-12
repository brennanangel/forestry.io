---
title: Add Functionality to Your Hugo Site With Theme Components
description: ''
date: 2018-07-13 01:05:06 -1100
authors:
- DJ Walker
publishdate: 2018-07-12 17:00:00 -1100
expirydate: 2030-01-01 04:00:00 +0000
headline: ''
textline: ''
images:
- "/uploads/2018/07/leaf-group.jpg"
categories:
- Frontend-Friday
tags: []
cta:
  headline: ''
  textline: ''
  calls_to_action: []
private: false
weight: ''
aliases: []
menu: []
draft: true

---
Theme Components are a relatively new addition to Hugo, first appearing in the `0.42` release. Previously, you could specify a single theme in your `config.toml` file. When Hugo looks for certain files (such as data and layout files,) it will first look in the top level of your project, and then look in the subdirectory of `themes/` that matches the theme you have set in `config.toml`. The way theme components work is pretty simple: they allow you to specify an array of themes in `config.toml`. The same lookup occurs as before, but now it will check all of the themes in the array, moving from left to right. 

When I was trying to come up with a project to try out theme components, I ultimately decided that re-implementing [the JSON API from our previous post](https://forestry.io/blog/build-a-json-api-with-hugo/) would be the perfect candidate. The resulting component will be something that can be installed on any existing Hugo site to provide JSON output for all content types, without the user having to write their own layouts. Thanks to theme components, they can install this alongside an existing theme and use both!

In this article, I will show you how to create [this theme component](https://github.com/dwalkr/hugo-json-api-component) yourself, and discuss some considerations to be made when building a theme component.

## Planning The API Component

Our goal is to create a self-contained component that will enable anyone to add a JSON API to the content on their site. 

Since we want this to work for a variety of use cases without modification, it would be great if there were a way for the user to specify their own schema: that is, we should let the user define the shape of their data that will ultimately be rendered as JSON. This may seem like more work at first, but the end result will enable users to define their own schema without having to override and re-implement most of our layout code.

One thing that will also make our layout code simpler and less error-prone will be to use `.Scratch` to build the data object that we want to output, and then outputting it all at once using Hugo’s `jsonify` function. We briefly touched on this approach [the last time we talked about rendering JSON with Hugo](https://forestry.io/blog/hugo-json-api-part-2#addendum-creating-json-output-with-jsonify).


## Implementing the Component

Since we’re making this a theme component, we will keep all of our code at `themes/json-api` inside of our project. Our component will only need a few files:


    data/
      json_schema.yml
    layouts/
      _default/
        list.json.json
        single.json.json
      partials/
        schema_item.tmpl
    theme.toml

`json_schema.yml` is how we’ll let users define their own schema. More on that later. The files in `layouts` will do the heavy lifting of rendering our JSON.

### The JSON Output Format
JSON is one of Hugo’s built-in output formats. This theme component will follow Hugo’s default JSON behavior, so we don’t need to do any additional configuration. JSON files will be available at `{{slug}}/index.json`

### Configurable Schema
Our theme component will provide a schema file defining the default schema, and users can override this data file in their project to define their own schema. Our component will read the schema from this data file and generate the appropriate JSON.

Our default schema looks like this:


    default:
      list:
        fields:
        - key: title
          field: title
        - key: date
          field: date
      single:
        fields:
        - key: title
          field: title
        - key: date
          field: date
        - key: content
          field: $PAGECONTENT

The top-level key represents which content section this schema applies to, and the second-level denotes whether the configuration is for the list view or single view. There is one special field I’ve defined here called `$PAGECONTENT`, which we will deal with later.

{{% tip %}}
Hugo will actually **merge** data files with the same name when resolving theme components. This means that when someone using our theme component overrides `json_schema.yml` inside their project, they don't need to redefine the `default` schema if they don't want to: it will still be available from the theme's `json_schema.yml` file!
{{% /tip %}}

### Rendering JSON with *jsonify*

If you read our initial article about creating a JSON API with Hugo, you’ll know that there are two layouts we need to create: `list.json.json` and `single.json.json`. These layouts will both output JSON, with `list.json.json` handling lists of content, and `single.json.json` taking care of single pages.

As mentioned, we’re going to use Hugo’s `.Scratch` feature to build a data structure instead of us writing out the JSON syntax. Unlike Jekyll, Hugo doesn’t let you create new variables to use within your templates. Instead, it offers `.Scratch`, a key-value store that we will use to build our data object.

To get a feel for how to use `.Scratch`, let’s start building `list.json.json`:


    {{- .Scratch.Set "items" slice -}}
    {{- range .Pages -}}
        <!-- Load the page's data into "item" -->
        {{- $.Scratch.Add "items" ($.Scratch.Get "item") -}}
        {{- $.Scratch.Delete "item" -}}
    {{- end -}}
    {{- .Scratch.Get "items" | jsonify -}}

Using `.Scratch.Set` and `.Scratch.Get`, we can set and retrieve values on the `Scratch` object. `.Scratch.Add` adds a value to a slice. In this case, we’re iterating over all of the pages in our list, adding the page’s data into a `.Scratch` value called `item`, and then adding that value to the `items` slice. This slice is then output as JSON.

Alright, here comes the tricky part: parsing the schema from the `json_schema.yml` datafile and building the appropriate content into our data structure. Since this is something we will need to do for both `list` and `single` layouts, it is a good idea to encapsulate this behavior into a partial.


    {{- partial "schema_item.tmpl" (dict "currentPage" . "Root" $ "SchemaType" "list") -}}

We need to pass a lot of information to this partial, so we’re going to wrap it all together in a single object using the `dict` function. We’re passing the current page in our loop as `currentPage`, the root context as `Root`, and another one called `SchemaType`. Since we’re planning to use this same partial for the single layout as well, we need some way for the `schema_item.tmpl` template to know whether we’re in a single or list context, so we create a variable called `SchemaType` and pass it the value of `list`.

Inside of our `schema_item.tmpl` file, all we have to do is set up our data object using a `.Scratch` value with the key of `item` following the schema defined in the `json_schema.yml` file.

First, let’s find a compatible schema configuration. We will first look for a schema defined for the current content section, falling back to `default` if it isn’t found. For example, if this were rendering the list view for posts, we would check for configuration at `posts.list`, falling back to `default.list` if that doesn’t exist.


    {{- if and (isset $.Root.Site.Data.json_schema .currentPage.Section) (isset (index $.Root.Site.Data.json_schema .currentPage.Section) .SchemaType) -}}
        {{- $.Root.Scratch.Set "schema" (index (index $.Root.Site.Data.json_schema .currentPage.Section) .SchemaType) -}}
    {{- else -}}
        {{- $.Root.Scratch.Set "schema" (index $.Root.Site.Data.json_schema.default .SchemaType) -}}
    {{- end -}}

As you can see, we’re using `.Scratch` again to temporarily store the schema configuration we’re going to use. This will make it easier to reference it in the subsequent code.

The next thing we’re going to do is set the `uri` value of our data item. I decided to set this regardless of what is defined in the item schema, because it is useful not only to locate the single item URL for an item in a list, but also to serve as a unique identifier for the item in the absence of anything else:


    {{- $.Root.Scratch.SetInMap "item" "uri" ($.currentPage.OutputFormats.Get "json").Permalink -}}

`(.OutputFormats.Get` `"``json``"``).Permalink` returns the URL of the item for the JSON output format.

Now that that’s done, all that’s left is to loop over the fields defined in our schema and add each one to the `.Scratch` object. At this point we need to check for any special fields in our schema, like `$PAGECONTENT`. The `$PAGECONTENT` field is just a placeholder for the HTML content of the page. The rest can be accessed via `.Params`.


    {{- range ($.Root.Scratch.Get "schema").fields -}}
        {{- if eq .field "$PAGECONTENT" -}}
        {{- $.Root.Scratch.SetInMap "item" (default "content" .key) (index $.currentPage.Content) -}}
        {{- else -}}
            {{- $.Root.Scratch.SetInMap "item" (default .field .key) (index $.currentPage.Params .field) -}}
        {{- end -}}
    {{- end -}}

{{% tip %}}
Using the `default` function, we make the `key` parameter optional when defining a schema. It will default to the name of the front matter field.
{{% /tip %}}

That’s all we have to do for our `schema_item.tmpl` partial! Deciding to make schema user-definable may have seemed like more work at first, but the resulting code is pretty light and it makes our theme component very flexible.

Here’s the cool part: since we abstracted the work of processing the schema into our `schema_item.tmpl` partial, our `single.json.json` layout code is only two lines!


    {{- partial "schema_item.tmpl" (dict "currentPage" . "Root" $ "SchemaType" "single") -}}
    {{- .Scratch.Get "item" | jsonify -}}

We just need to load the `schema_item.tmpl` partial and tell it we’re using the `single` schema, and then output the results (stored in `.Scratch`  as `item` again) with the `jsonify` function. Isn’t it great when it’s easy?



## Installing the JSON Theme Component On An Existing Site

To use this theme component on an existing site, we just have to take care of a few quick steps:

1. Add our theme component to `themes/` as a submodules
2. Update the `theme` setting in `config.toml` to include our theme component
3. Set up JSON output for pages and sections

To demonstrate this, I've created a demo site that uses the [Paper theme](https://github.com/nanxiaobei/hugo-paper/). This site has a couple of blog posts, and three pages about different cars. [Here's a demo running on Netlify](https://hardcore-knuth-fc0978.netlify.com/).

### Setting up the JSON component

We can install the theme component with the following command:

```
git submodule add https://github.com/dwalkr/hugo-json-api-component themes/json-api
```

Then, we just have to open up `config.toml` and change the following line:

```
theme = "paper"
```

to this:

```
theme = ["paper","json-api"]
```

Finally, to enable the JSON output format for our list and single views, we need to specify it in the `outputs` section of our `config.toml` file:

```
[outputs]
    page = ["html","json"]
    section = ["html","json"]
```

Once this is done, restart your Hugo server and you should be able to access the JSON data by adding `/index.json` to the end of section and page URLs.


### Customizing the Schema

The default schema works pretty well for our content in `posts`, but the cars that we added to the `garage` section have additional front matter that I want to expose in the JSON. Since the JSON API component lets us customize the JSON schema by section, this is really easy to do! Just add a file at `data/json_schema.yml` and configure it like this:

```
garage:
  list:
    fields:
      - field: year
      - field: make
      - field: model
  single:
    fields:
      - field: title
        key: name
      - field: year
      - field: make
      - field: model
      - field: engine
      - field: $PAGECONTENT
        key: description
```

Remember that Hugo merges data files, so even though we are overriding this file from our theme component, the `default` configuration will still be available to Hugo.

[Check out the list view](https://hardcore-knuth-fc0978.netlify.com/garage/index.json) in our example to see the new schema in action.

## The Tip of the Iceberg

While theme components aren't exactly "plugins", they are a novel way to provide self-contained functionality to a Hugo site. I'm looking forward to seeing some themes that take advantage of this new feature, and seeing what else might come out of this more modular way to develop Hugo sites. If you've discovered a great use for theme components, {{% slack_invite_link "Share it with us in our community Slack!" %}}