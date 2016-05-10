#AsseticDemo

How the Symfony Assetic Bundle works for your assets

3 targets:

- Compile (scss -> css, coffeescript -> javascript)
- Minify (multiple lines -> one line) 
- Combine (1.css 2.css 3.css ... -> main.css, 1.js 2.js 3.js ... -> main.js)

## Create a new empty symfony project

	symfony new AsseticDemo 2.3

(**2.3** is my symfony version)

## Install NodeJS

> https://nodejs.org/en/


## Install other toolkits via NodeJS

> http://symfony.cn/docs/cookbook/assetic/uglifyjs.html


### Install UglifyJS (local installation)
```
cd /path/to/your/symfony/project
npm install uglify-js --prefix app/Resources
```
Option `--prefix` indicates the UglifyJs to be installed to `app/Resources/node_modules/uglify-js` folder

### Install UglifyCSS (local installation)

```
cd /path/to/your/symfony/project
npm install uglifycss --prefix app/Resources
```
Option `--prefix` indicates the UglifyCSS to be installed to `app/Resources/node_modules/uglifycss` folder

### Install [CoffeeScript](http://coffeescript.org/) (local installation)

```
cd /path/to/your/symfony/project
npm install coffee-script --prefix app/Resources
```
Option `--prefix` indicates the CoffeeScript to be installed to `app/Resources/node_modules/coffee-script` folder


### Install [scssphp](http://leafo.net/scssphp/)

	composer require leafo/scssphp

scssphp is a compiler for [SCSS](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#css_extensions) written in PHP. 

### Configure Assetic

In the `app/config/config.yml`

```
assetic:
    debug:          "%kernel.debug%"
    use_controller: false
    bundles:        [ AppBundle ]
    #java: /usr/bin/java
    #node: /usr/local/bin/node
    filters:

        uglifyjs2:
            bin: "%kernel.root_dir%/Resources/node_modules/uglify-js/bin/uglifyjs"

        uglifycss:
            bin: "%kernel.root_dir%/Resources/node_modules/uglifycss/uglifycss"

        coffee:
            bin: "%kernel.root_dir%/Resources/node_modules/coffee-script/bin/coffee"
            apply_to: "\.coffee$"

        scssphp: ~

        cssrewrite: ~
        #closure:
        #    jar: "%kernel.root_dir%/Resources/java/compiler.jar"
        #yui_css:
        #    jar: "%kernel.root_dir%/Resources/java/yuicompressor-2.4.7.jar"               
```
- `bundles: [ AppBundle ]` Add name of your bundles to the `[]` to enable Assetic serves to your bundles
- `%kernel.root_dir%` is your project's root directory
- `node_modules` is an auto-generated directory while installing with NodeJS
- `apply_to` Compiles coffeescripts(any files suffix with `.coffee`) into javascripts

### Create assets directories in the `web` directory

- web/bundles/app/css/
- web/bundles/app/fonts/
- web/bundles/app/images/
- web/bundles/app/js/

### Process CSS stylesheets, JavaScript files


```
   <!-- css -->
    {% stylesheets filter="cssrewrite, scssphp, ?uglifycss" output="assets/css/frontend/application.css"
        "bundles/app/css/frontend/*"
    %}
        <link href="{{ asset_url }}" rel="stylesheet" />
    {% endstylesheets %}

    <!-- js -->
    {% javascripts filter="?uglifyjs2" output="assets/js/frontend/application.js"
        "bundles/app/js/frontend/*"
    %}
        <script src="{{ asset_url }}"></script>
    {% endjavascripts %}
```

- Files wrapped inside `stylesheets` or `javascripts` code block will be packed output after running `php app/console assetic:dump --env=prod` command

- `filter="cssrewrite, scssphp, ?uglifycss"` is a filter chain just like `pipeline` in Linux that the output from previous filter will be as input passed to the next filter

- `cssrewrite` [Fixing CSS Paths with the cssrewrite Filter](http://symfony.com/doc/current/cookbook/assetic/asset_management.html#cookbook-assetic-cssrewrite)

- `?uglifycss` and `?uglifyjs2` Only minify css and js in the prod environment
  
-  `output` Defaults are `web/css/XXX.css` and `web/js/XXX.js`. To override default settings by explicitly specifying your directory and filename


### Process Image files


	<img src="{{ asset('bundles/app/images/frontend/bg.gif') }}" />

or

```
{% image "bundles/app/images/frontend/bg.gif" %}
    <img src="{{ asset_url }}" alt="example" />
{% endimage %}
```
It's better to process images on 3rd party CDN provider

### Dump Asset files in the prod environment

	
	php app/console assetic:dump --env=prod
	

### Check Effects

    php app/console server:run

###### <http://localhost:8000/app_dev.php/>
View page source

```
    <!-- css -->
            <link href="/app_dev.php/assets/app/css/frontend/application_part_1_bootstrap-theme_1.css" rel="stylesheet" />
            <link href="/app_dev.php/assets/app/css/frontend/application_part_1_bootstrap_2.css" rel="stylesheet" />
            <link href="/app_dev.php/assets/app/css/frontend/application_part_1_test_3.css" rel="stylesheet" />
    
    <!-- js -->
            <script src="/app_dev.php/assets/app/js/frontend/application_part_1_bootstrap_1.js"></script>
            <script src="/app_dev.php/assets/app/js/frontend/application_part_1_npm_2.js"></script>
            <script src="/app_dev.php/assets/app/js/frontend/application_part_1_test_3.js"></script>
```


###### <http://localhost:8000/app.php/>
View page source

```
    <!-- css -->
            <link href="/assets/app/css/frontend/application.css" rel="stylesheet" />
    
    <!-- js -->
            <script src="/assets/app/js/frontend/application.js"></script>
```


# 最佳实践（个人观点）

### 1. 不使用@XXX这种路径写法，可以避免css中引用图片不显示问题

```
	<!-- css --> 
    {% stylesheets filter="cssrewrite, scssphp, ?uglifycss" output="assets/app/css/frontend/application.css"
        "@AppBundle/Resources/public/css/frontend/*"
    %}
        <link href="{{ asset_url }}" rel="stylesheet" />
    {% endstylesheets %}

    <!-- js -->
    {% javascripts filter="?uglifyjs2" output="assets/app/js/frontend/application.js"
        "@AppBundle/Resources/public/js/frontend/*"
    %}
        <script src="{{ asset_url }}"></script>
    {% endjavascripts %}
```    
改写成

```
    <!-- css -->
    {% stylesheets filter="cssrewrite, scssphp, ?uglifycss" output="assets/app/css/frontend/application.css"
        "bundles/app/css/frontend/*"
    %}
        <link href="{{ asset_url }}" rel="stylesheet" />
    {% endstylesheets %}
    
    <!-- js -->
    {% javascripts filter="?uglifyjs2" output="assets/app/js/frontend/application.js"
        "bundles/app/js/frontend/*"
    %}
        <script src="{{ asset_url }}"></script>
    {% endjavascripts %}
 ```
    
### 2. 直接在web目录下创建资源文件目录，省去通过`php app/console assets:install`命令把src目录下的资源文件同步到web目录的操作

推荐目录结构如下

```
   web
    |--- bundles
           |--- app
                 |--- css
                       |--- backend
                       |--- frontend
                       |--- ...
                       
                 |--- fonts
                       |--- backend
                       |--- frontend
                       |--- ...
                       
                 |--- images
                       |--- backend
                       |--- frontend
                       |--- ...
                       
                 |--- js
                       |--- backend
                       |--- frontend
                       |--- ...
           
```

### 3. 把web目录下的资源文件目录纳入Git一并提交