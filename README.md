MarkdownContentBundle
========
Basic idea behind the project was to be able to easily provide, provision and version content without a database backend. Also quite tempting: The markdown syntax.

[![Build Status](https://travis-ci.org/maschmann/MarkdownContentBundle.png?branch=master)](https://travis-ci.org/maschmann/MarkdownContentBundle) [![Latest Stable Version](https://poser.pugx.org/asm/markdown-content-bundle/v/stable.png)](https://packagist.org/packages/asm/markdown-content-bundle) [![Total Downloads](https://poser.pugx.org/asm/markdown-content-bundle/downloads.png)](https://packagist.org/packages/asm/markdown-content-bundle)

## What does it do?
You can configure following things:
 * Content directory
 * Markdown parser ([php-markdown](http://michelf.ca/projects/php-markdown/ "php-markdown") or [parsedown](http://parsedown.org "parsedown"))
 * Type of loader for the files (at the moment only file-loader)
 * And the prefix for your content routes

All URLs are dynamically generated and support subdirectories without limitations.

## Basics
Some things you need to know!

### Configuration
You can set following things in your app/config.yml:

    asm_markdown_content:
        content_directory: 'app/Resources/markdown' # app/Resources/markdown
        content_loader:    'file-loader'            # file-loader
        route_prefix:      'content'                # content
        markdown_provider: 'php-markdown'           # php-markdown

### Expansion
If you want, you can easily add new markdown parsers or content loaders, even hooks.
For each type there's an interface you can build a tagged service upon:

#### Loaders
Asm\MarkdownContentBundle\Content\ContentLoaderInterface

    asm_markdown_content.content_file_loader:
    class: 'Asm\MarkdownContentBundle\Content\ContentFileLoader'
    arguments:
        - '%kernel.root_dir%'
        - '%asm_markdown_content.content_directory%'
        - '%asm_markdown_content.content_path_depth%'
    tags:
        - { name: asm_markdown_content.content_loader, alias: file-loader }

#### Parsers
Asm\MarkdownContentBundle\Parser\ParserInterface

    asm_markdown_content.parsedown_parser:
        class: 'Asm\MarkdownContentBundle\Parser\ParsedownParser'
        tags:
            - { name: asm_markdown_content.parser, alias: parsedown }

#### Hooks
Asm\MarkdownContentBundle\Hook\HookDataInterface
Asm\MarkdownContentBundle\Hook\HookContentInterface

    asm_markdown_content.metadata_content_hook:
        class: 'Asm\MarkdownContentBundle\Hook\MetadataContentHook'
        tags:
            - { name: asm_markdown_content.hook, alias: metadata }

### Hook system
The system is based on two events:
 * pre-content conversion
 * post-content conversion

For each registered hook service you have to set the type of hook in the _getType()_ method. It's either 'post' or 'pre'. The hook just has one more interface method _workContent()_ which receives the content array:

    $content = array(
        'content' => string,
        'data'    => array(),
    );

That's also the data structure you're needed to return from _workContent()_. What you're doing with the data is completely up to you. The main difference between 'pre' and 'post' is the state of either data nodes. In 'pre', all content is still raw markdown, data is maybe already prefilled by other hooks (metadata hook e.g.). 'post' gets the already converted html string from your configured markdown parser. There you have the possibility to change some tags, add classes, whatever you want.

### Templating
The base layout uses bootstrap3 via CDN and provides some base for your own designs. Just override

    vendor/asm/markdown-content-bundle/Asm/MarkdownContentBundle/Resources/views/layout.html.twig
in your own Bundle(s). Have a look at the variable names!
Content template:

    vendor/asm/markdown-content-bundle/Asm/MarkdownContentBundle/Resources/views/Content/index.html.twig

### Meta data
You can add metadata for the page with a simple one-liner comment within the markdown file. The comment will be stripped by the metadata content hook.
Syntax:

    /* Title: pagetitle Description: some cool description for your page content */

Available metadata fields:

    $headers = array(
        'title'       => 'Title',
        'description' => 'Description',
        'author'      => 'Author',
        'date'        => 'Date',
        'robots'      => 'Robots',
        'template'    => 'Template',
    );

## Upcoming/planned
 * precaching for markdown file search on compiler pass
 * caching of markdown content
 * maybe more loader types
 * online editor
