
# Flask pagination macro

In this post I will cover how to create a pagination with Jinja macro feature.

![](https://cdn-images-1.medium.com/max/2000/0*W7aEiaT1bRqsxm-i)

Requirements:

* show preconfigured limited number of pages at once

* collapse invisible pages under ...

* provide previous/next navigation buttons

## Jinja templates for Bootstrap4

I’ve created 3 tier structure of Jinja templates to use Bootstrap4. **First** — bootstrap4_base.html - loads css and js files from CDN and defines major blocks:

* head - holds content of the <head> tag and defines title, metas, styles

* body - holds content of the <body> tag and defines navbar, content, scripts

* navbar - for navigation bar

* content - for boostrap container (tag with class="container")

* scripts - goes in the end of the body, [here is why](https://stackoverflow.com/questions/383045/is-put-scripts-at-the-bottom-correct)

Blocks may be extended or/and overwritten in the later templates This template follows Bootstrap4 [intro guide](https://getbootstrap.com/docs/4.0/getting-started/introduction/)

**Second** — page_base.html - creates navbar and extends content block. It shows all the flash messages (the ones invoked with flask flash function) and adds page_content. That one will be extended in the last template and holds actual content.

**Third** — index.html - will overwrite title block and extend page_content

Hierarchy of templates is achieved with using of {% extends "<parent_template.html" %} block to refer to parent template.

To extend a block — scripts for example - inside child template:

    {% block scripts %}
        {{ super() }}
        <script>
            alert(1);
        </script>
    (% endblock %)

Drop super to just overwrite it.

## Jinja pagination macro

Macros are comparable with functions in regular programming languages. They are useful to put often used idioms into reusable functions to not repeat yourself (“DRY”). Macro is a bare block that starts with {% macro function_name(formal_params) %}. It holds the HTML, or rather templated Jinja code that will be reused. I’m putting it’s code into separate file - _macros.html.

Usage:

1. import it with {% import "_macros.html" as macros %}

1. call it with {{ macros.function_name(actual_params) }}

Bootstrap pagination follows that [doc](https://getbootstrap.com/docs/4.0/components/pagination/#working-with-icons)

I’ve tried to put as little logic in the macro itself and do all the calculations in the flask view. My version needs 2 params:

1. endpoint - the name of flask endpoint provided to url_for which builds the actual link to select page

1. pages list if dictionaries, each one has

* class key to define if link is active, normal or disabled

* page_label show the page number or navigation icons

* href - additional param for url_for which will hold page

Macro code:

    {**%** macro pagination_widget(pages, endpoint) **%**}
    **<**nav aria**-**label**=**"Page navigation example"**>**
        **<**ul class**=**"pagination"**>**
            {**%** **for** p **in** pages **%**}
            **<**li class**=**"page-item {{p['class']}}"**>**
                **<**a href**=**"{{ url_for(endpoint, page = p['href'], **kwargs) }}"
                    class**=**"page-link"
                    aria**-**label**=**{{p['page']}}**>**
                    **<**span aria**-**hidden**=**"true"**>**{{p['page_label'] **|** safe}}**</**span**>**
                    **<**span class**=**"sr-only"**>**{{p['page_label'] **|** safe}}**</**span**>**

                **</**a**>**
            **</**li**>**
            {**%** endfor **%**}
        **</**ul**>**
    **</**nav**>**
    {**%** endmacro **%**}

## Flask endpoint

Endpoint code relies on a Pager class to prepare pages. It first needs to get the page number from URL parameters. The actual data I’m using is just a list of numbers up to a count. In real world it’s going to be a query to the database like SELECT column from table LIMIT Y OFFSET X. Where X - is a page size * current page (zero based), Y - is a page_size.
> OFFSET may be slow with big numbers — it’s better to use [Sleek mode](https://blog.jooq.org/2013/10/26/faster-sql-paging-with-jooq-using-the-seek-method/)

Flask endpoint:

    @app.route("/")
    **def** **index**():
        page **=** int(request**.**args**.**get('page', 1))

        count **=** 300
        data **=** range(count)

        pager **=** Pager(page, count)
        pages **=** pager**.**get_pages()

        offset **=** (page **-** 1) ***** current_app**.**config['PAGE_SIZE']
        limit **=** current_app**.**config['PAGE_SIZE']
        data_to_show **=** data[offset: offset **+** limit]

        **return** render_template('index.html', pages**=**pages, data_to_show**=**data_to_show)

## Pager class

This class prepares a pages list for macro. To do the calculations it needs the number of all items and the current page. Page size and a number of visible pages are read from app config.

Difficult part was to show exactly predefined number if links to another pages. Invisible pages are collapsed under ....

## App configuration

App config contains two parameters:

1. PAGE_SIZE - how many elements show in a page

1. VISIBLE_PAGE_COUNT - how may links to pages how in pages inculding ...s

    app **=** Flask(__name__)
    app**.**secret_key **=** os**.**urandom(42)
    app**.**config['PAGE_SIZE'] **=** 20
    app**.**config['VISIBLE_PAGE_COUNT'] **=** 10

The whole code is available at [GitHub](https://github.com/smirnov-am/flask-pager)

Original [post](https://smirnov-am.github.io/2018/09/20/flask-pagination-macro.html)

Do you have another useful macro? Drop me a message on [LinkedIn](https://www.linkedin.com/in/smirnovam/)
