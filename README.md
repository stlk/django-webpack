# Django + Webpack example project

There's plenty of options for you to choose when deciding how to integrate a  javascript framework. But most of the time you don't a luxury of not choosing a Webpack-based environment. That's why this guide will focus on using Webpack-based project in Django templates.

Webpack is established solution to build Javascript applications used by both Vue CLI and Create React App. In this example we'll use Vue application but you would integrate any other Webpack-based app in similar fashion.

Most of the time all you need is just a sprinkle of interactivity to the template. That why we recommend starting with small components and switching to Single Page Apps only when there's real benefit. This will make our components much more manageable.

### webpack-bundle-tracker + django-webpack-loader

`django-webpack-loader` and `webpack-bundle-tracker` work together in the following way. webpack-bundle-tracker plugin emits information about webpack’s compilation process and results so django-webpack-loader can consume it. To add `webpack-bundle-tracker` to CRA we will need to hook up a plugin to webpack.

## Project Setup

I used following commands to bootstrap this project.

```sh
mkdir django-webpack
cd django-webpack

pipenv install django django-webpack-loader
pipenv run django-admin startproject django_webpack .
pipenv run python manage.py startapp things

npm install -g vue-cli

vue init webpack vue-test
cd vue-test

npm install webpack-bundle-tracker
```

#### Configure Vue app

##### `vue-test/build/webpack.base.conf.js`

```diff
@@ -4,7 +4,9 @@ const utils = require('./utils')
 const config = require('../config')
 const vueLoaderConfig = require('./vue-loader.conf')

+var BundleTracker = require('webpack-bundle-tracker');
+
 function resolve(dir) {
   return path.join(__dirname, '..', dir)
 }

@@ -88,5 +90,11 @@ module.exports = {
     net: 'empty',
     tls: 'empty',
     child_process: 'empty'
-  }
+  },
+  plugins: [
+    new BundleTracker({
+      path: __dirname,
+      filename: '../dist/webpack-stats.json',
+    }),
+  ]
 }
```

After running `npm start` we should see additional file `webpack-stats.json` in `dist` folder containing all the data we need.

#### Configure Django app

Install `django-webpack-loader` using [pipenv](https://packaging.python.org/tutorials/managing-dependencies/) or pip. Then go to your `settings.py`. Add `things` and `webpack_loader` to `INSTALLED_APPS`. This will enable us to use template tags.

```py
INSTALLED_APPS = [
    ...
    'things',
    'webpack_loader',
]
```

Update static files configurations.

```py
STATIC_ROOT = os.path.join(BASE_DIR, "staticfiles")
STATIC_URL = "/static/"

STATICFILES_DIRS = [os.path.join(BASE_DIR, "vue-test", "dist", "static")]

WEBPACK_LOADER = {
    "DEFAULT": {
        "CACHE": not DEBUG,
        "BUNDLE_DIR_NAME": "vue-test/dist/static/",  # must end with slash
        "STATS_FILE": os.path.join(BASE_DIR, "vue-test", "dist", "webpack-stats.json"),
    }
}
```

##### `things/templates/things/home.html`

```html
<div id="app"></div>

{% load render_bundle from webpack_loader %}
{% render_bundle 'manifest' %}
{% render_bundle 'vendor' %}
{% render_bundle 'app' %}
```

##### `django_webpack/urls.py`

```diff
 from django.contrib import admin
 from django.urls import path
+from django.views.generic import TemplateView

 urlpatterns = [
+    path("", views.TemplateView(template_name = "things/home.html").as_view()),
     path("admin/", admin.site.urls)
 ]
```

### Build

In one terminal:

```sh
cd vue-test
npm run build
```

In the other terminal:

```sh
pipenv run python manage.py runserver
```

#### Projects

- [django-webpack-loader](https://github.com/owais/django-webpack-loader)
- [webpack-bundle-tracker](https://github.com/owais/webpack-bundle-tracker)
