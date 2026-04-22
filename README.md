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

## Licence

See the [original repository](https://github.com/bentideswell/magento1-wordpress-integration) for licence details.

## Credits

Original author: [Ben Tideswell / FishPig](https://github.com/bentideswell)  
Fork maintainer: [Create Hosting Ltd](https://createhosting.co.nz)
