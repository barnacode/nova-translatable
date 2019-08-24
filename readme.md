# Nova Translatable Field
<!-- [![Latest Version on Packagist](https://img.shields.io/packagist/v/yeswedev/nova-translatable.svg?style=flat-square)](https://packagist.org/packages/yeswedev/nova-translatable) -->

This package is based on [emilianotisato/nova-tinymce](https://github.com/emilianotisato/nova-tinymce).

Adds the ability to show and edit translated fields created with [dimsav/laravel-translatable](https://github.com/dimsav/laravel-translatable) package.

It will show up in the detail view like this:

<img width="400" src="http://barnacode.com/github/nova-translatable-images/nova-spatie-translatable-detail.png">

And in the edit view like this:

<img width="400" src="http://barnacode.com/github/nova-translatable-images/nova-spatie-translatable-editor.png">

## Installation and usage
You can require this package using composer:

```
composer require barnacode/nova-translatable
```

You can add the field follows:

``` php
use Barnacode\Nova\Translatable\Translatable;

Translatable::make('Description'),
```

Make sure, that you have your Eloquent model setup correct:

- First, you need to add the `Barnacode\Translatable\Translatable`-trait.
- Next, you should create a public property `$translatedAttributes` which holds an array with all the names of attributes you wish to make translatable.
- Finally, you should make sure that all translatable attributes are set to the `text`-datatype in your database. If your database supports `json`-columns, use that.

Here's an example of a prepared model:

``` php
use Illuminate\Database\Eloquent\Model;
use Barnacode\Translatable\Translatable;

class NewsItem extends Model
{
    use Translatable;
    
    public $translatedAttributes = ['name'];
}
```


### Defining Locales
Locales can be defined via config file ```config/translatable.php``` by adding a ```locales``` array:

``` php
// config/translatable.php
return [
    ...
    'locales' => [
        'en' => 'English',
        'de' => 'German',
        'fr' => 'French',
    ],
];
```

Alternatively you can "override" the config locales with the ```locales(...)``` method:

``` php
Translatable::make('Description')->locales([
    'en' => 'English',
    'de' => 'German',
]),
```

### Single Line Option
By default the input field on the edit view is a textarea. If you want to change it to a single line input field you can add the ```singleLine()``` option:

``` php
Translatable::make('Description')->locales([...])->singleLine(),
```

### Trix Editor
You can use the trix editor for your translated fields by using the ```trix()``` option:

<img width="400" src="http://barnacode.com/github/nova-translatable-images/nova-spatie-translatable-trix.png">

``` php
Translatable::make('Description')->trix(),
```

### TinyMce Editor
You can use the tinymce-vue editor for your translated fields by using the ```trix()``` option:

<img width="400" src="http://barnacode.com/github/nova-translatable-images/nova-spatie-translatable-tinymce.png">

``` php
Translatable::make('Description')->tinymce(),
```
In order to work with this editor you need to [create a free account](https://apps.tiny.cloud/signup/) to get the editor API_KEY. Once you have don this, create this key inside your `config/nova.php` file

```
    'tinymce_api_key' => env('TINYMCE_API_KEY'),
```

By default, the editor comes with default options and the image without the filemanager.
You can use custome options like this:

```php
Translatable::make('body')->tinymce()
            ->options([
                'plugins' => [
                    'advlist autolink lists link image charmap print preview hr anchor pagebreak'
                ],
                'toolbar' => 'insertfile undo redo | styleselect | bold italic'
            ]),
```

#### Using the upload images feature

Now if you need to upload images from the text editor, we need to install [UniSharp Laravel Filemanager](https://unisharp.github.io/laravel-filemanager/installation), and pass the `use_lfm` key to your options array:

```php
Translatable::make('body')->tinymce()
                ->options([
                'plugins' => [
                    'advlist autolink lists link image charmap print preview hr anchor pagebreak',
                    'searchreplace wordcount visualblocks visualchars code fullscreen',
                    'insertdatetime media nonbreaking save table contextmenu directionality',
                    'emoticons template paste textcolor colorpicker textpattern'
                ],
                'toolbar' => 'insertfile undo redo | styleselect | bold italic | alignleft aligncenter alignright alignjustify | bullist numlist outdent indent | link image media',
                'use_lfm' => true
                ]),
```

In case you change the `laravel-filemanager` URL in the package config file, you need to pass that info to this nova field with the `lfm_url` key in the options array.

```php
// ...
    'use_lfm' => true,
    'lfm_url' => 'laravel-filemanager'
// ...
```

#### Extra configuration and plugin customization

You can virtually pass any configuration option for the javascript SDK to the array in the `options()` method.

For example, you like to have increased the height of the text area:

```php
            Translatable::make('body')->tinymce()
                ->options([
                'height' => '980'
                ]),
```

You can see the full list of parameters in the docs:
[https://www.tiny.cloud/docs-3x/reference/Configuration3x/](https://www.tiny.cloud/docs-3x/reference/Configuration3x/)

### Index View
By default the locale used when displaying the field on the index view is determined by ```app()->getLocale()```. To override this you can use the ```indexLocale($locale)``` option:

``` php
Translatable::make('Description')->indexLocale('de'),
```

