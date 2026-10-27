# FishPig Magento 1 WordPress Integration (Fork)

Fork of [bentideswell/magento1-wordpress-integration](https://github.com/bentideswell/magento1-wordpress-integration) with PHP 8.2 compatibility fixes.

## What changed

### Sitemap observer fix

The original sitemap observer has two issues:

1. **Session conflict (PHP 8.2):** The observer calls `addIsViewableFilter()` which triggers `Mage::getSingleton('customer/session')` during admin store emulation. On PHP 8.2 with Redis sessions (Cm_RedisSession), this causes a session conflict and the observer silently fails, generating the sitemap without blog URLs.

2. **Save & Generate timing:** The observer hooks into `sitemap_sitemap_save_before`. When using "Save & Generate" in the admin, the save event fires before generation (modifying the old file), then `generateXml()` overwrites it. A singleton guard prevents the observer from running again after generation.

**Fix:** Added a second observer on `sitemap_urlset_generating_before` (OpenMage 20.x+) that writes blog URLs directly to the file stream during generation. The original `save_before` observer is kept as a fallback for older Magento 1.x and uses a flag to skip when the stream observer has already run. Both observers use `addStatusFilter()` instead of `addIsViewableFilter()` to avoid the session dependency.

**Files changed:**
- `app/code/community/Fishpig/Wordpress/Model/Observer.php`
- `app/code/community/Fishpig/Wordpress/etc/config.xml`

## Installation

```bash
composer config repositories.fishpig-wp vcs https://github.com/createhosting/magento1-wordpress-integration
composer require fishpig/magento1-wordpress-integration
```

### OpenMage 20.19.0 compatibility fixes

OpenMage 20.19.0 removed several long-deprecated APIs this extension relied on:

- **`htmlEscape()` removed:** templates calling `$this->htmlEscape()` now use `$this->escapeHtml()`
  (identical behaviour, `htmlEscape` was the old alias). Affected: `wordpress/shortcode/product.phtml`,
  `wordpress/post/associated/products.phtml`, the `wordpress/sidebar/widget/*` templates, and
  `wordpress-addons/yarpp/sidebar/widget.phtml`.
- **`_isAllowed(): bool` return type added to core:** admin controllers overriding `_isAllowed()` must
  declare the `: bool` return type or PHP 8.2 fatals ("declaration must be compatible"). Fixed in
  `Adminhtml/WordpressController.php` and `Adminhtml/Wordpress/AssociationsController.php`.

### CRITICAL: Symfony polyfill preload requirement (host app)

This extension (specifically the PluginShortcodeWidget addon's `Helper/Core.php`) **unregisters all
autoloaders** during WordPress simulation via `spl_autoload_unregister()`, including Composer's. Modern
WordPress uses PHP 8.4/8.5 native array functions (`array_all()`, `array_last()`) that trigger Symfony
`polyfill-php84` / `polyfill-php85`. With autoloaders stripped, the polyfills' backing classes
(`Symfony\Polyfill\Php84\Php84`, `Symfony\Polyfill\Php85\Php85`) cannot be resolved and the blog fatally
errors.

**The host application MUST preload these polyfill classes in `app/Mage.php`** immediately after the
Composer autoloader `require`, e.g.:

```php
require_once $autoloaderPath . DS . 'autoload.php';
if (PHP_VERSION_ID < 80400 && file_exists($autoloaderPath . DS . 'symfony' . DS . 'polyfill-php84' . DS . 'Php84.php')) {
    require_once $autoloaderPath . DS . 'symfony' . DS . 'polyfill-php84' . DS . 'Php84.php';
}
if (PHP_VERSION_ID < 80500 && file_exists($autoloaderPath . DS . 'symfony' . DS . 'polyfill-php85' . DS . 'Php85.php')) {
    require_once $autoloaderPath . DS . 'symfony' . DS . 'polyfill-php85' . DS . 'Php85.php';
}
```

**This preload is overwritten whenever OpenMage core updates `app/Mage.php` and MUST be re-applied after
any core upgrade.** Without it, `/blog` fatals once WordPress enqueues global styles.

## Licence

See the [original repository](https://github.com/bentideswell/magento1-wordpress-integration) for licence details.

## Credits

Original author: [Ben Tideswell / FishPig](https://github.com/bentideswell)  
Fork maintainer: [Create Hosting Ltd](https://createhosting.co.nz)
