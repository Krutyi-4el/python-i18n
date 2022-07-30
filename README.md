# python-i18n [![tests](https://github.com/Krutyi-4el/python-i18n/actions/workflows/ci.yml/badge.svg)](https://github.com/Krutyi-4el/python-i18n/actions/workflows/ci.yml)


This library provides i18n functionality for Python 3 out of the box. The usage is mostly based on Rails i18n library.

## Installation

Just run

    pip install python-i18n

If you want to use YAML to store your translations, use

    pip install python-i18n[YAML]

## Usage
### Basic usage

The simplest, though not very useful usage would be

    import i18n
    i18n.add_translation('foo', 'bar')
    i18n.t('foo') # bar

### Using translation files

YAML and JSON formats are supported to store translations. With the default configuration, if you have the following `foo.en.yml` file

    en:
      hi: Hello world !

in `/path/to/translations` folder, you simply need to add the folder to the translations path.

    import i18n
    i18n.load_path.append('/path/to/translations')
    i18n.t('foo.hi') # Hello world !

Please note that YAML format is used as default file format if you have `yaml` module installed.
If both `yaml` and `json` modules available and you want to use JSON to store translations, explicitly specify that: `i18n.set('file_format', 'json')`

### Memoization

Setting the configuration value `enable_memoization` in the settings dir will load the files from disk the first time they
are loaded and then store their content in memory. On the next use the file content will be provided from memory and not
loaded from disk, preventing disk access. While this can be useful in some contexts, keep in mind there is no current way of
issuing a command to the reloader to re-read the files from disk, so if you are updating your translation file without restarting
the interpreter do not use this option.
 
### Namespaces

#### File namespaces
In the above example, the translation key is `foo.hi` and not just `hi`. This is because the translation filename format is by default `{namespace}.{locale}.{format}`, so the {namespace} part of the file is used as translation.

To remove `{namespace}` from filename format please change the `filename_format` configuration.

    i18n.set('filename_format', '{locale}.{format}')
            
#### Directory namespaces
If your files are in subfolders, the foldernames are also used as namespaces, so for example if your translation root path is `/path/to/translations` and you have the file `/path/to/translations/my/app/name/foo.en.yml`, the translation namespace for the file will be `my.app.name` and the file keys will therefore be accessible from `my.app.name.foo.my_key`.

## Functionalities
### Placeholder

You can of course use placeholders in your translations. With the default configuration, the placeholders are used by inserting `%{placeholder_name}` in the ntranslation string. Here is a sample usage.

    i18n.add_translation('hi', 'Hello %{name} !')
    i18n.t('hi', name='Bob') # Hello Bob !

### Pluralization

Pluralization is based on Rail i18n module. By passing a `count` variable to your translation, it will be pluralized. The translation value should be a dictionnary with at least the keys `one` and `many`. You can add a `zero` or `few` key when needed, if it is not present `many` will be used instead. Here is a sample usage.

    i18n.add_translation('mail_number', {
        'zero': 'You do not have any mail.',
        'one': 'You have a new mail.',
        'few': 'You only have %{count} mails.',
        'many': 'You have %{count} new mails.'
    })
    i18n.t('mail_number', count=0) # You do not have any mail.
    i18n.t('mail_number', count=1) # You have a new mail.
    i18n.t('mail_number', count=3) # You only have 3 new mails.
    i18n.t('mail_number', count=12) # You have 12 new mails.

### Fallback

You can set a fallback which will be used when the key is not found in the default locale.

    i18n.set('locale', 'jp')
    i18n.set('fallback', 'en')
    i18n.add_translation('foo', 'bar', locale='en')
    i18n.t('foo') # bar
    
### Skip locale from root
Sometimes i18n structure file came from another project or not contains root element with locale eg. `en` name.

    {
        "foo": "FooBar"
    }

However we would like to use this i18n .json file in our Python sub-project or micro service as base file for translations.
`python-i18n` has special configuration tha is skipping locale eg. `en` root data element from the file.

    i18n.set('skip_locale_root_data', True)

### Custom functions

Add your custom functions and choose translation variants during runtime.
All arguments given to `t` would be passed to the function as keyword arguments.
This may be an alternative for pluralization, especially if a language has more than one plural form.

Example (correct plural form of days in Ukrainian):

    i18n.set("locale", "uk")
    i18n.add_translation("days", "%{count} %{p(день|дні|днів)}")

    def determine_plural_form(*, count):
        count = abs(count)
        if count % 10 >= 5 or count % 10 == 0 or (count % 100) in range(11, 20):
            return 2
        elif count % 10 == 1:
            return 0
        return 1

    i18n.add_function("p", determine_plural_form, "uk")
    i18n.t("days", count=1) # 1 день
    i18n.t("days", count=2) # 2 дні
    i18n.t("days", count=5) # 5 днів
