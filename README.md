Configuration
====================
You are required to set ``base_dir`` and ``base_url``, the rest are optional,
but we currently support:

 * ``base_dir``: The directory to output and search for assets
 * ``base_url``: The url static assets will be located
 * ``debug``: If webassets should be in debug mode (i.e no compression)
 * ``updater``: Different update configurations (i.e always, timestamp)
 * ``cache``: If we should use webassets cache
 * ``jst_compiler``: A custom jst compiler, by default it uses underscore
 * ``url_expire``: If a cache-busting query string should be added to URLs
 * ``static_view``: If assets should be registered as a static view using Pyramid config.add_static_view()
 * ``cache_max_age``: If static_view is true, this is passed as the static view's cache_max_age argument (allowing control of expires and cache-control headers)

``` python
webassets.base_dir=%(here)s/app/static
webassets.base_url=/static
webassets.debug=True
webassets.updater=timestamp
webassets.cache=False
webassets.jst_compiler=Handlebars.compile
webassets.url_expire=False
webassets.static_view=True
webassets.cache_max_age=3600
```

Then you can just use config.add_webasset to add bundles to your environment

``` python
from webassets import Bundle

jst = Bundle('templates/*.html',
        filters='jst',
        output='js/jst.js', debug=False)

config.add_webasset('jst', jst)
```

Using asset specs instead of files and urls
----------------------------------------------
It's possible to use an asset specifications (package:file) instead of simple file names.

In *Bundles* this can have two effects
- If the asset specifications declares a path outside the base_dir, the file will be copied.
- Otherwise, it will work like a normal bundle file.

If files are bundled from other packages and those packages act like pyramid
plugins adding their own ``add_static_view``. Webassets will use those static
view urls to show the individual files if needed (for example, in development mode).

If you have defined your own static route and you want to use it with webassets.
for example you have:

``` python
config.add_static_view('/static-stuff', 'my.super.app:static')
```

Setting the base url configuration option to an asset specification:

```
base_url = my.super.app:static
```

Will make webassets use the ``/static-stuff`` route for your assets. Note:
the absolute or relative path depends on where is your application is deployed.

 Mako
====================
You can use the global webassets tag:
``` python
% for url in webassets(request, 'css/bootstrap.css', 'css/bootstrap-responsive.css', output='css/generated.css', filters='cssmin'):
    <link href="${url}" rel="stylesheet">
% endfor
```

or you can grab the environment from the request.

From The Request
====================
If you are not using Jinja2, you can still access the environment from the request.

```python
jst_urls = request.webassets_env['jst'].urls()
```


Jinja2
====================
If you are using Jinja2, you can just do the following configuration (this assumes use of pyramid_jinja2):

``` python
config.add_jinja2_extension('webassets.ext.jinja2.AssetsExtension')
assets_env = config.get_webassets_env()
jinja2_env.assets_environment = assets_env
```
and then:

``` python
{% assets "jst" %}
<script type="text/javascript" src="{{ ASSET_URL }}"></script>
{% endassets %}
```

Extras
====================
There are a few utility methods you can use:

``get_webassets_env_from_settings(settings, prefix='static_assets')``: Pass it a dictionary of your settings and an
optional keyword argument of the prefix in your configuration and it will return you a webassets environment.

``get_webassets_env(request or config)``: This will pull the environment out of the registry, you can use either
a configurator object or a request.
