StructuredData
==============
A Joomla ```3.2+``` system plugin (in the ```./lib``` folder is present the new proposed version of the [JMicrodata](https://github.com/joomla/joomla-cms/tree/master/libraries/joomla/microdata "JMicrodata") library).  
Created during the Google Summer of Code 2014.

If you want to keep your views separated from the logic, ```plg_system_structureddata``` is system plugin for parsing the HTML markup and converting the ```data-*``` HTML5 attributes into the correctly formatted Microdata or RDFa Lite 1.1 semantics.  

The ```data-*``` attributes are new in HTML5, they give us the ability to embed custom data attributes on all HTML elements. So if you disable the library output, the HTML will still be validated. The default suffix that the library will search for is ```data-sd```, where sd stands for structured data, but you can register more than one custom suffix.  

Installation
============
[Download the last version of the plugin](https://github.com/alexprut/plg_system_structureddata/archive/master.zip "Download plg_system_structureddata") and install it. __Don't forget to publish this plugin!__

Usage
=====
Since the system plugin runs immediately before the final content is sent to the client, you can use the plugin syntax everywhere, including native and third-party extensions that allow HTML editing.

### Markup Syntax
##### setType
![ParserPlugin Syntax](https://alexprut.github.io/PHPStructuredData/images/parser-plugin-syntax-v1.3.0-setType.png)
The _type_ defines which schema is being used for the following markup.  The Type must always have the first character Uppercase to be correctly interpreted. If the type is a valid schema, the global scope for the page from this point onwards is updated to this schema. The plugin will replace the data tag with ```itemscope itemtype='https://schema.org/Type'``` in case of Microdata semantics or ```vocab='https://schema.org' typeof='Type'``` in case of RDFa Lite 1.1 semantics.  
  
###### Example:
```html
<div data-sd="Article">
    <p>This is my article</p>
</div>
```

This will be output using ```Microdata``` semantics as:
```html
<div itemscope itemtype="http://schema.org/Article">
    <p>This is my article</p>
</div>
```
Or using ```RDFa``` semantics as:
```html
<div vocab="http://schema.org" typeof="Article">
    <p>This is my article</p>
</div>
```

##### Specifying generic item properties
![ParserPlugin Syntax](https://alexprut.github.io/PHPStructuredData/images/parser-plugin-syntax-v1.3.0-global.png)  
Once a schema has been declared, the next step is to declare individual properties – explaining the content and giving it semantic meaning.
  
The _property_ must always have the first character as lowercase to be correctly interpreted. If the property is found to be part of the current schema, the plugin will replace the data tag with ```itemprop='property'``` in case of Microdata semantics or ```property='property'``` in case of RDFa Lite 1.1 semantics.  If the property is not found to be a valid property of the active schema, it will be ignored and the next available property will be parsed.

###### Example:
```html
<div data-sd="Article">
    <p data-sd="articleBody">This is my article</p>
</div>
```

This will be output using ```Microdata``` semantics as:
```html
<div itemscope itemtype="http://schema.org/Article">
    <p itemprop="articleBody">This is my article</p>
</div>
```
Or using ```RDFa``` semantics as:
```html
<div vocab="http://schema.org" typeof="Article">
    <p property="articleBody">This is my article</p>
</div>
```

##### Specifying schema—dependant item properties
![ParserPlugin Syntax](https://alexprut.github.io/PHPStructuredData/images/parser-plugin-syntax-v1.3.0-specialized.png)  
Sometimes you may want to explicitly state a property which should only be used when a specific schema is active – for example, if the property has a specific property in one schema, which is called something different in another schema.

It is possible to achieve this by using a schema–dependant property.  This works by using a combination between both _Type_ and _property_, separated by a full stop. In short, if the current global scope is equal to Type and the property is part of that Type, the plugin will replace the data tag with ```itemprop='property'``` in case of Microdata semantics or ```property='property'``` in case of RDFa Lite 1.1.

###### Example:
```html
<div data-sd="Article">
    <p data-sd="articleBody">This is my article</p>
    <p data-sd="Article.wordcount">4</p>
</div>
```

This will be output using ```Microdata``` semantics as:
```html
<div itemscope itemtype="http://schema.org/Article">
    <p itemprop="articleBody">This is my article</p>
    <p itemprop="wordcount">4</p>
</div>
```
Or using ```RDFa``` semantics as:
```html
<div vocab="http://schema.org" typeof="Article">
    <p property="articleBody">This is my article</p>
    <p property="wordcount">4</p>
</div>
```

### Using multiple properties
![ParserPlugin Syntax](https://alexprut.github.io/PHPStructuredData/images/parser-plugin-syntax-v1.3.0.png)  
It is possible, using a combination of these, to specify multiple properties including some which are specific for a schema and others which are generic. The order of the building blocks isn't significant and a white space is used as a separator.

###### Example:
```html
<div data-sd="Article">
    <p data-sd="articleBody">This is my article</p>
    <p data-sd="Article.wordcount">4</p>
    <p data-sd="Recipe.recipeCategory Article.articleSection description">Amazing dessert recipes</p>
</div>
```

This will be output using ```Microdata``` semantics as:
```html
<div itemscope itemtype="http://schema.org/Article">
    <p itemprop="articleBody">This is my article</p>
    <p itemprop="wordcount">4</p>
    <p itemprop="articleSection">Amazing dessert recipes</p>
</div>
```
Or using ```RDFa``` semantics as:
```html
<div vocab="http://schema.org" typeof="Article">
    <p property="articleBody">This is my article</p>
    <p property="wordcount">4</p>
    <p property="articleSection">Amazing dessert recipes</p>
</div>
```

##### Nesting schemas
Sometimes it is necessary to nest schemas – for example if you want to describe a person when you have the Article schema open. This is possible using nested schemas. To use this, simply append the schema preceeded by a full stop, __after__ the property.  Once you have finished using the nested schema, close the containing tag, and re-set the original schema.

###### Example:
```html
<div data-sd="Article">
    <p data-sd="articleBody">This is my article</p>
    <p data-sd="Article.wordcount">4</p>
    <div data-sd="author.Person">
        <p data-sd="name">John Doe</p>
    </div>
    <p data-sd="Article keywords">Cake</p>
</div>
```

This will be output using ```Microdata``` semantics as:
```html
<div itemscope itemtype="http://schema.org/Article">
    <p itemprop="articleBody">This is my article</p>
    <p itemprop="wordcount">4</p>
    <div itemprop="author" itemscope itemtype="http://schema.org/Person">
        <p itemprop="name">John Doe</p>
    </div>
    <p itemprop="keywords">Cake</p>
</div>
```
Or using ```RDFa``` semantics as:
```html
<div vocab="http://schema.org" typeof="Article">
    <p property="articleBody">This is my article</p>
    <p property="wordcount">4</p>
    <div property="author" vocab="http://schema.org" typeof="Person">
        <p property="name">John Doe</p>
    </div>
    <p itemprop="keywords">Cake"</p>
</div>
```

##### The Algorithm:
1. First the parser checks for __setTypes__. If one or more matches are found then the current global scope will be updated with the first match. At this point if there are no specific or generic properties the algorithm will finish and replace the data tag with the specified scope. Otherwise continue to point 2.
2. The parser checks for __specific item properties__. If one or more valid matches are found, then the algorithm will finish and replace the data tag with the first match property. Otherwise go to point 3
3. The parser checks for __generic properties__. If one or more valid matches are found, then the algorithm will replace the data tag with the first property that is matched, and complete the algorithm.

Semantically marking up 'on the fly' within HTML editors
========================================================
To apply markup within articles simply use the data tag and the method explained above.  
See below for an example.  
![plg_system_structureddata content editor usage](https://alexprut.github.io/plg_system_structureddata/images/plg_system_structureddata-editor.png)

Incorporating into your extensions and templates
================================================
Let's suppose that somewhere in your code you need to add Microdata or RDFa semantics to the following HTML which is part of an article (_e.g._ ```$scope='Article';```).
```html
<div data-sd="<?php echo $scope;?>">
    <!-- Title -->
    <span data-sd="Review.itemReviewed name">
        How to Tie a Reef Knot
    </span>
    <!-- Author -->
    <span>
        Written by
        <span data-sd="author.Person">
            <span data-sd="name">John Doe</span>
        </span>
    </span>
    <!-- Date published -->
    <meta data-sd='<?php echo $scope;?> datePublished' content='2014-01-01T00:00:00+00:00'/>1 January 2014
    <!-- Content -->
    <span data-sd='reviewBody articleBody'>
        Lorem ipsum dolor sit amet...
    </span>
<div>
```
The ```Microdata``` output will be:
```html
<div itemscope itemtype='https://schema.org/Article'>
    <!-- Title -->
    <span itemprop='name'>
        How to Tie a Reef Knot
    </span>
    <!-- Author -->
    <span>
        Written by
        <span itemprop='author' itemscope itemtype='https://schema.org/Person'>
            <span itemprop='name'>John Doe</span>
        </span>
    </span>
    <!-- Date published -->
    <meta itemprop='datePublished' content='2014-01-01T00:00:00+00:00'/>1 January 2014
    <!-- Content -->
    <span itemprop='articleBody'>
        Lorem ipsum dolor sit amet...
    </span>
<div>
```
The ```RDFa``` output will be:
```html
<div vocab='https://schema.org' typeof='Article'>
    <!-- Title -->
    <span property='name'>
        How to Tie a Reef Knot
    </span>
    <!-- Author -->
    <span>
        Written by
        <span property='author' vocab='https://schema.org' typeof='Person'>
            <span property='name'>John Doe</span>
        </span>
    </span>
    <!-- Date published -->
    <meta property='datePublished' content='2014-01-01T00:00:00+00:00'/>1 January 2014
    <!-- Content -->
    <span property='articleBody'>
        Lorem ipsum dolor sit amet...
    </span>
<div>
```
Instead, if you decide to change the current Type (_e.g._ ```$scope="Review";```).  
The ```Microdata``` output will be:
```html
<div itemscope itemtype='https://schema.org/Review'>
    <!-- Title -->
    <span itemprop='itemReviewed'>
        How to Tie a Reef Knot
    </span>
    <!-- Author -->
    <span>
        Written by
        <span itemprop='author' itemscope itemtype='https://schema.org/Person'>
            <span itemprop='name'>John Doe</span>
        </span>
    </span>
    <!-- Date published -->
    <meta itemprop='datePublished' content='2014-01-01T00:00:00+00:00'/>1 January 2014
    <!-- Content -->
    <span itemprop='reviewBody'>
        Lorem ipsum dolor sit amet...
    </span>
<div>
```
The ```RDFa``` output will be:
```html
<div vocab='https://schema.org' typeof='Review'>
    <!-- Title -->
    <span property='itemReviewed'>
        How to Tie a Reef Knot
    </span>
    <!-- Author -->
    <span>
        Written by
        <span property='author' vocab='https://schema.org' typeof='Person'>
            <span property='name'>John Doe</span>
        </span>
    </span>
    <!-- Date published -->
    <meta property='datePublished' content='2014-01-01T00:00:00+00:00'/>1 January 2014
    <!-- Content -->
    <span property='reviewBody'>
        Lorem ipsum dolor sit amet...
    </span>
<div>
```

License
-------
plg_system_structureddata is licensed under the GNU GPL v2 License – see the LICENSE file for details.
