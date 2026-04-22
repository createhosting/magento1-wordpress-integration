# FishPig Magento 1 WordPress Integration (Fork)

Fork of [bentideswell/magento1-wordpress-integration](https://github.com/bentideswell/magento1-wordpress-integration) with a fix for PHP 8.2 compatibility.

## What changed

**Sitemap observer fix** (`app/code/community/Fishpig/Wordpress/Model/Observer.php`)

The sitemap observer calls `addIsViewableFilter()` to fetch blog posts when generating the XML sitemap. That method calls `Mage::getSingleton('customer/session')`, which fails during admin store emulation on PHP 8.2 with Redis sessions (Cm_RedisSession). The observer catches the exception silently, so the sitemap generates without any blog URLs.

Replaced `addIsViewableFilter()` with `addStatusFilter(array('publish', 'protected'))` in the sitemap observer. Same result, no session dependency.

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
