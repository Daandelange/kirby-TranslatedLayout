# Kirby TranslatedLayout field plugin

This plugin brings translation logics into the native `layouts` and `blocks` fields.  

### Experimental

While the kirby team is waiting for some heavy refactoring for recursively bringing translation logics into their "complex fields", this plugin aims to provide a temporary workaround for multi-language websites.

This is an experimental draft trying to bring some translation logic to blocks, columns and layouts.  
It turns out to be quite powerful already with just a minimal set of changes compared to the native field behaviour.

**Current state** :  
Tested on a single configuration, works well but not extensively tested. Therefore, please note that **there remains a risk of data loss**. (Do not use without backups!)

### Implementation

 - The **primary language** (default) inherits the default `LayoutField` behaviour and remains *(almost?)* identical to the native Kirby Layout field.
 - The **seconday languages** (translations) of this field are always syncronized on parse (aka `$field->fill($value)`).
    - **Identical structures** : The layouts and blocks structures are defined by the default language using their unique `id`. 
    - **Fallback** : If a block has no translation, it's replaced with the default language.
    - **Sanitation** : If a block translation is not available in the default language, it's removed. All blocks from the default language are guaranteed to be available for translation in the panel.
    - **Panel GUI** : Non-translateable fields and blocks are disabled, preventing panel users from changing the layout and adding blocks in translations.
    - **Data** : The syncronized translation is saved as a blocks collection and works fine with the native `$field->toLayout()` & `$field->toBlocks()`. Some disk space is saved by not writing the layout data multiple times.

![Screenshot of Kirby 3 plugins TranslatedLayout](TranslatedLayout.gif)

## Requirements

- Version `0.3.3-beta` : Kirby 3.8 or above.
- Version `1.0.0` : Kirby 5 or above. *(maybe Kirby4 compatible?)*

- **Note**: This plugin heavily relies on the use of the panel. If you'd like to manually edit a `translatedlayout` field via the text content file, it's not recommended to use this plugin, as it's probably not recommended to use blocks without the panel. (Meanwhile, it still is possible, and this plugin even simplifies the translation files).  

## Installation

_Choose one:_

- Download: Download and copy this repository to `/site/plugins/translatedlayout`.
- Git submodule: `git submodule add https://github.com/daandelange/kirby-translatedlayout.git site/plugins/translatedlayout`.
- Composer: `composer require daandelange/translatedlayout`.

## Setup

### Import existing data

**Warning!** If you already have a layout with translated content, switching to this field will erase all translations unless you manually give the same `id` to blocks/rows/columns in the translations data structure. There is no automatic script available.  


### Blueprints

In your page blueprints, you can simply replace a `type: layout` or `type: blocks` field by `type: translatedlayout` or `type: translatedblocks`. Read more about how to use the respective fields in the official Kirby docs.

The only difference is an extra `translate` property on fields, please refer to this example:

````yml
sections:
  content:
    type: fields
    fields:
      mylayout:
        label: Translated Layout Demo
        type: translatedlayout
        translate: true # <--- enables syncing of translations (layout field)
        layouts:
          - "1/1"
          - "1/2, 1/2"
          - "1/3, 1/3, 1/3"
        fieldsets:
          translateable:
            label: Fully Translateable Blocks
            type: group
            fieldsets:
              heading:
                extends: blocks/heading
                translate: true # same as default value
              - list
              - text
          partiallytranslateable:
            label: Blocks with some translateable fields
            type: group
            fieldsets:
              image: # over-rule the translated option of existing fields
                label: Image (non translateable src)
                type: image
                translate: false
                fields:
                  link:
                    translate: false
              url: # custom block example
                name: Url (non-translateable source)
                icon: cog
                fields:
                  link:
                    type: url
                    translate: false
                    required: true
                  text:
                    type: text
                    translate: true
                  
          nontranslateable:
            label: Non-translated blocks
            type: group
            fieldsets:
              line:
                extends: blocks/line
                translate: false # Completely disable whole block translations
        settings: # You can also translate layout settings
          fields:
            class:
              type: text
              width: 1/2
              translate: false  # Don't translate
            purpose:
              type: text
              translate: true  # Translate
      myblock:
        label: Translated Blocks Demo
        type: translatedblocks
        fieldsets:
            heading:
              extends: blocks/heading
              translate: true # same as default value
            - text
            line:
              extends: blocks/line
              translate: false # Completely disable whole block translations
````

To use predefined translation settings for the default kirby blocks, you may use :  

````yml
fields:
  content:
    type: translatedlayout
    extends: fields/translatedlayoutwithfieldsets
````
This can be useful for quickly setting up this plugin in a test environment.  
*Beware that this will add the fields to your fieldsets if they don't exist yet.*  

To setup your own fieldsets, prefer copy/pasting from [translatedlayoutwithfieldsets.yml](https://github.com/Daandelange/kirby-TranslatedLayout/blob/master/src/blueprints/fields/translatedlayoutwithfieldsets.yml) and adapt it to your needs.

### Templates

As for the native `LayoutField` and `BlocksField`, use : `$field->toLayout()`, `$field->toBlocks()`, etc. in your templates to fetch the field contents. There is absolutely no difference as the plugin acts during the data parse state.

## Options

There are no options available yet. Would you like to contribute some ?

## Development

- A small hack to fix KirbyUp's alias `@KirbyPanel` sub-includes : ([more info](https://github.com/johannschopplich/kirbyup/issues/7))  
  - osx: `cd /path/to/translatedLayout/ && ln -s "../../../../kirby/panel/src/mixins" ./src/mixins`
  - linux: *todo*
  - other: Create an alias/symlink pointing from `translatedlayout/src/mixins` to `/kirby/panel/src/mixins`.
- `npm install` : Install the required dependencies.
- `npm run dev` : Develop mode (listen/compile).
- `npm run build` : Compile for publishing.

## Feature ideas

- Plugin options : Set rather to fill with (untranslated) default language, or leave the translateable blocks empty ? (on translation creation only).

## Similar Plugins

- [Synced-Structure](https://gist.github.com/lukaskleinschmidt/1c0b94ffab51d650b7c7605a4d25c213) : Syncs structures across languages using UUIDs.

## License

MIT - Free to use, free to improve !

However, for usage in commercial projects, please seriously consider to improve the plugin a little and contribute back the changes with a PR, or hire someone to do so.  
For contribution suggestions, you can search for `todo` in the source code or refer to open issues.

## Credits

- [Daan de Lange](https://daandelange.com/)
