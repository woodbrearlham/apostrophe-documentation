---
title: Accessing images inside related pages
layout: tutorial
---

Out of the box, Apostrophe makes it very easy to access the children
of the home page, in order to create "tab" navigation:

```markup
{# In your layout template #}
<ul class="tabs">
  {% for tab in data.home._children %}
    <li class="tab">
      <a href="{{ tab._url }}">{{ tab.title }}</a>
    </li>
  {% endfor %}
</ul>
```

So let's try to expand this to display an image that comes
from a singleton or area inside each subpage of the home page:

```markup
<ul class="tabs">
  {% for tab in data.home._children %}
    <li class="tab">
      {% set image = apos.images.first(tab.thumbnail) %}
      {% if image %}
        <img src="{{ apos.attachments.url(image, { size: 'one-sixth' }) }}" />
      {% else %}
        <span class="missing-thumbnail"></span>
      {% endif %}
      <a href="{{ tab._url }}">{{ tab.title }}</a>
    </li>
  {% endfor %}
</ul>
```

Maybe you also added that thumbnail singleton to every page's
"page settings" dialog box, like this:

```javascript
// in your own, project-level
// lib/modules/apostrophe-custom-pages/index.js file
module.exports = {
  beforeConstruct: function(self, options) {
    options.addFields = [
      {
        name: 'thumbnail',
        type: 'singleton',
        widgetType: 'apostrophe-images',
        options: {
          aspectRatio: [ 1, 1 ],
          limit: 1
        }
      }
    ].concat(options.addFields || []);
  }
}; 
```

We're very close! But it doesn't quite work yet. For performance, Apostrophe
intentionally does not load the images of related pages. 

Fortunately we can switch this on for the children of the home page:

```javascript
// in lib/modules/apostrophe-pages/index.js
module.exports = {
  // other configuration, such as the types option, then...
  filters: {
    ancestors: {
      children: {
        areas: [ 'thumbnail' ]
      }
    }
  }
};
```

> *"What's happening in this code?"* We're changing the
> way `apostrophe-pages` loads the ancestors of the
> current page. By specifying `areas: [ 'thumbnail' ]` for
the children of each ancestor, we tell Apostrophe to
> fully load the widgets in that area, fetching the
> images we need.
>
> *"What does `filters` mean here?"* When `apostrophe-pages`
> fetches the current page, it makes an Apostrophe cursor
> object. It then loops through all the properties of the
> `filters` option and calls those methods on the cursor,
> passing on any arguments given. For more information
> see [working with cursors](../intermediate/cursors.html).

## Drop-down menus with pictures: images from second-level pages

Yes, you can do that too. But, keep in mind that we're doing extra work
here that could slow Apostrophe down a little. The life of a web
developer is full of tradeoffs!

```javascript
  // in lib/modules/apostrophe-pages/index.js
module.exports = {
  // other configuration, such as the types option, then...
  filters: {
    ancestors: {
      children: {
        depth: 2,
        areas: [ 'thumbnail' ]
      }
    }
  }
};
```

> Adding `depth: 2` tells `apostrophe-pages` to go two levels
> deep fetching the subpages of each ancestor of the current page.
> Now you can loop over `_children` two levels deep in your
> template and produce dropdown menus with images... at the cost
> of a little speed.

## Fetching images from children of the current page

You can do a similar trick for children of the current page:

```javascript
// in lib/modules/apostrophe-pages/index.js
module.exports = {
  // other configuration, such as the types option, then...
  filters: {
    children: {
      areas: [ 'thumbnail' ]
    }
  }
};
```

> See the earlier example on this page for how to add `thumbnail`
to the schema of every page. You can then access the thumbnails
> in your templates by looping over `data.page._children` with
> code similar to the template shown above.


