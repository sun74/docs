---
title: New translation system
menuTitle: New system
weight: 1
---

# New module translation system 

{{% minver v="1.7.6" title="true" %}}

The new module translation system is based on the new system introduced in PrestaShop 1.7 for the Core and Native modules. It aims to harmonize translation systems throughout PrestaShop while providing backwards compatibility.

{{% notice info %}}
This feature is only available for PrestaShop 1.7.6 and later. If you need to support older versions, see the [Classic translation system]({{< ref "classic-system.md" >}}).
{{% /notice %}}

## Vocabulary

- **Wording** – A string that you may want to translate.
- **Translation Domain** – A contextual group of wordings, that allows translated a given wording differently according to the context in which they appear.

## How it works

Similarly to the classic system, the new translation system works in two steps. 

##### 1. Translation functions are used to display your module's wordings in another language

PrestaShop provides functions that allow PHP files and Smarty/Twig templates to display translated wordings. By leveraging a list of translation sources (including dictionary files from _classic translation_), a module can use this feature display a wording in another language during runtime.

{{< figure src="../img/new-translation-system.jpg" title="How wordings are translated at runtime" >}}


##### 2. Creating module dictionaries

The Back Office's Translation page (_International > Translations > Modify Translations_) is used to generate module dictionaries. It extracts the module's wordings by analyzing its source code, then inspects a list of sources to compile a list of translations, and finally displays a form that allows you to customize or complete those translations in a given language. Once saved, this information is stored in the Database.

{{< figure src="../img/new-translation-workflow.jpg" title="Translation workflow" >}}


{{% notice note %}}
If your module has already been translated using the [classic translation system]({{< ref "classic-system.md" >}}), the new translation interface can also source translations from your existing classic translation dictionary files. This means that you can keep your old controllers for compatibility with older PrestaShop versions, and add Symfony controllers for 1.7.6+, without losing your previous translations.
{{% /notice %}}


## Making your module translatable

To make your module translatable, you need to adapt your module's source code. Find any wording you want to make translatable, then wrap it using the appropriate method as explained below. Once wrapped, your wordings will be ready to be translated through the translation interface.

{{% notice tip %}}
**By default, wordings are displayed as originally written.**

Don't worry if you don't translate everything to all languages right away. Any wording left untranslated will be shown in its original language. Because of this, we suggest writing all your wordings in English, and then translating them to other languages.
{{% /notice %}} 

### Translation domain

An important part of the new translation system is **Translation Domains**, which replaces the classic system's [contextualization][contextualization]. In the new translation system, all wordings must be linked to at least one translation domain.

While the Core and Native modules have clearly defined [translation domain naming scheme][core-translation-domains], non-native modules must respect a specific naming convention:

```
Modules.Nameofthemodule.Specificpart
```

Translation Domain names are always made of three parts, separated by dots:

1. The first part must always be **"Modules"**
2. **"Nameofthemodule"** is the name of your module, with some rules:

  - The first letter must be in upper case, and the rest in lower case (eg. "MyModule" becomes "Mymodule")
  - Only word characters (A-z, 0-9) are supported.  
     Make sure that the technical name of your module does not contain contains underscores (`_`) or any other unsupported symbol, or else translation may not work.
  - If your module name starts with `ps_`, that part must be removed. This is an exception to the previous rule.

3. **"Specificpart"** allows for contextualization and can be set to whatever you like, following this rules:
  
  - The first letter must be in upper case, and the rest in lower case.
  - Symbols like dots, dashes and underscores are allowed (but discouraged).
  - If you want to use previously-generated classic translation dictionary files, you need to follow a [specific naming convention for backwards compatibility](#backwards-compatibility).  
     Otherwise, consider using the [Native module conventions][native-module-conventions] (_"Admin"_ or _"Shop"_).

#### Backwards compatibility

If you want your module to be compatible with previously-generated [classic translation dictionary files]({{< ref "classic-system.md" >}}), then the third component of the Translation domain must be set to the name of the file where the wording is used, respecting the following rules:
  
  - The first letter must be in upper case, and the rest in lower case
  - If the file extension is `.tpl`, the extension must be removed
  - If the file name ends in "controller", that part must be removed as well.       

##### Examples
 
Assuming your module is called "my_module":

File where the wording is used | Expected translation domain
--- | ---
my_module.php | Modules.Mymodule.My_module.php
SomeFile.php | Modules.Mymodule.Somefile.php
a_certain_template.tpl | Modules.Mymodule.A_certain_template
ps_somefile.tpl | Modules.Mymodule.Ps_somefile
another-template.html.twig | Modules.Mymodule.Another-template.html.twig

You can find more examples in the [test fixtures for DomainHelper][domainhelper].

### PHP files

In PHP files, translation is performed using the module's `trans()` method.

This method takes four parameters:
     
1. `$id` – The wording you want to translate.
2. `$parameters` – An array of replacements, if any. ([Learn more about translation placeholders](https://symfony.com/doc/3.4/components/translation/usage.html#component-translation-placeholders)).
3. `$domain` – The translation domain for that wording, as explained above.
4. `$locale` – (optional) The locale identifier (eg. "en-US") if you want to translate in a different language than the current one.

Now let's see some examples on how to use it.

#### Module's main class

When translating wordings in the module's main class, since it extends the `Module` class, you can simply call `$this->trans()`.

```php
// file: mymodule.php

class mymodule extends Module
{
    public function __construct()
    {
        $this->displayName = $this->trans('My module', [], 'Modules.Mymodule.Mymodule');
        $this->description = $this->trans('Description of my module.', [], 'Modules.Mymodule.Mymodule');
    }
}
```

Since the module is called MyModule, the translation domain should be `Modules.Mymodule.Mymodule`. The third part matches the file name, which is also "mymodule".

#### Module controllers

`ModuleAdminController` and `ModuleFrontController` can access the module instance via the `$this->module` property.

```php
// file: controllers/front/something.php

class MymoduleSomethingModuleFrontController extends ModuleFrontController
{
    public function initContent()
    {
        $this->title = $this->trans('My module title', [], 'Modules.Mymodule.Something);
    }
}
```

Symfony controllers work exactly the same as the Core's. Just use `$this->trans` method.

{{% notice warning %}}
Be aware that in symfony controllers, the second and third arguments have been swapped to make `$replacements` optional.
{{% /notice %}}

```php
namespace PrestaShop\Module\MyModule;

class SomeAdminController extends FrameworkBundleAdminController
{
    public function someAction()
    {
        $this->trans('Some text being translated, [], 'Modules.Mymodule.Admin');
    }
}
```

#### Other classes

Other classes will need to retrieve the module's instance somehow. We recommend passing it as as a parameter in the constructor and storing it for later use.

```php
class CustomModuleClass 
{
    private $module;
    
    public function __construct(Module $module)
    {
        $this->module = $module
    }
    
    public function foo()
    {
        $this->text = $this->module->trans('My text to translate', [], 'Modules.Mymodule.Custommoduleclass');
    }
}
```

If you really need to, you can also retrieve a new instance of your module using `Module::getInstanceByName('mymodulename')`. This should be avoided though, as it's not a good practice.

### Templates

#### Smarty files

Wordings in Smarty .tpl files can be translated using the `{l}` function call, which Smarty will replace by the translation in the current language.

This function accepts three parameters:

- `s` – The wording to be translated.
- `d` – The translation domain.
- `sprintf` – Optional, it can be used to interpolate variables in your wording.

For instance, translating the string "Welcome to this page!" can be done like this:

```smarty
{* file: somefile.tpl *}

{l s='Welcome to this page!' d='Modules.Mymodule.Somefile'}
```

#### Twig files

Wordings in Twig .twig files can be translated using the `trans` filter. It works similarly as the `trans()` method described for PHP files above:

```twig
{# file: something.twig #}

{{ 'Welcome to this page!'|trans({}, 'Modules.Mymodule.Admin') }}
```

The first parameter can be used to replace tokens in your wording after it's translated:

```twig
{{ 'Hello %username%!'|trans({'%username%': 'John'}, 'Modules.Mymodule.Admin') }}
```

## Translating your module

To be translatable through the new translation interface, modules must opt-in. This can be done by declaring this function on your module's main class:

```php
public function isUsingNewTranslationSystem()
{
    return true;
}
```

After this:

- Go to the "Translations" page under the "International" menu,
- In the "Modify translations" section, find the "Type of translation" drop-down and select "Installed modules translations",
- Choose the module you want to translate.
- Choose the language you want to translate the module into. The destination language must already be installed to enable translation in it.
- Click the "Modify" button.

You will be presented with a page that displays all the wordings for the selected module, grouped by translation domain.

Once saved, translations are stored in the database in the table `ps_translation`.

## Creating translation dictionary files

This feature has not been implemented as of 1.7.6. 

If you need to distribute translated wordings with your module, you can either [write classic dictionary files manually]({{< ref "classic-system#editing-a-dictionary-file-manually" >}}), or export your module's wordings from the database into a file, then import it during the module's install process.

{{% notice tip %}}
If you choose to export wordings from the database, you can easily extract only your module's wordings from the `ps_translation` table by filtering domains that start with `ModulesYourmodulename*`. You can disregard `id_translation`, but you will have to match the original `lang_id` to the shop's one (see `ps_lang`) when you re-import them.
{{% /notice %}}

Automatic export of classic dictionary files and XLIFF catalogues for the new translation system is [due to be implemented in 1.7.8](https://github.com/PrestaShop/PrestaShop/issues/14968).

## Limitations and caveats

### Making your wordings appear in the translation interface

The translation interface relies on code analysis to "discover" wordings for translation. Therefore, when declaring wordings in your code, some care is needed in order to make sure they can be discovered.

1. The translation interface only detects wordings used through the `trans()` function, the `{l}` Smarty tag, and the `trans` Twig filter. Therefore, they must be declared in a PHP, TPL, or TWIG file. They will be detected regardless of whether that code is actually used in runtime or not.

2. **Always use literal values, not variables, with the `trans()` function, the `{l}` Smarty tag, and `trans` Twig filter.** Although variables are interpreted at runtime, they won't be understood by the code analyzer, which only supports literals. Passing variables to these methods will prevent those wordings from appearing in the translation interface.

Example:

```php
// literal values will work
$this->trans('Some wording', [], 'Modules.Mymodule.Something');

// dynamic content can be injected using placeholders & replacements
$this->trans('Some wording with %foo%', ['%foo%' => $dynamicContent], 'Modules.Mymodule.Bar');

// this won't work, the interpreter will ignore variables
$wording = 'Some wording';
$domain = 'Modules.Mymodule.Foo';
$this->trans($wording, [], $domain);

// this will yield unexpected results
$this->trans('Some '. $var . ' wording', [], 'Modules.Mymodule.Foo');

// dynamic behavior, like aliasing the trans() function, won't work well either
function translate($wording) {
   $this->trans($wording, [], 'Modules.Mymodule.Foo');
}
```

In Twig files, you can use `trans_default_domain` to set up your default domain. Keep in mind this works on a per-file basis:

```twig
{% trans_default_domain 'Modules.Mymodule.Foo' %}
{{ 'Hello world'|trans }}
{{ 'Something else'|trans }}
``` 

[contextualization]: {{< ref "new-system.md#contextualization" >}}
[native-module-conventions]: {{< ref "/1.7/development/internationalization/translation/translation-domains.md#modules" >}}
[core-translation-domains]: {{< ref "/1.7/development/internationalization/translation/translation-domains.md#understanding-the-domains-structure" >}}
[domainhelper]: https://github.com/PrestaShop/TranslationToolsBundle/blob/master/Tests/Translation/Helper/DomainHelperTest.php#L61
