# Yac - Yet Another Cache
[![Build Status](https://secure.travis-ci.org/laruence/yac.png)](http://travis-ci.org/laruence/yac) [![Build status](https://ci.appveyor.com/api/projects/status/6bu09pw8ukyx61m2/branch/master?svg=true)](https://ci.appveyor.com/project/laruence/yac/branch/master)

Yac is a shared and lockless memory user data cache for PHP.

it can be used to replace APC or local memcached.

## Requirement

- PHP 7 +

### Install

```
$/path/to/phpize
$./configure --with-php-config=/path/to/php-config
$make && make install
```

## Note

1.  Yac is a lockless cache, you should try to avoid or reduce the probability of multiple processes set one same key
2.  Yac use partial crc, you'd better re-arrange your cache content, place the most important(mutable) bytes at the head or tail

## Restrictions

1.  Cache key cannot be longer than 48 (YAC_MAX_KEY_LEN) bytes
2.  Cache Value cannot be longer than 64M (YAC_MAX_VALUE_RAW_LEN) bytes
3.  Cache Value after compressed cannot be longer than 1M (YAC_MAX_VALUE_COMPRESSED_LEN) bytes

## InIs

yac.enable = 1

yac.keys_memory_size = 4M ; 4M can get 30K key slots, 32M can get 100K key slots

yac.values_memory_size = 64M

yac.compress_threshold = -1

yac.enable_cli = 0 ; whether enable yac with cli, default 0

## Constants

YAC_VERSION

YAC_MAX_KEY_LEN = 48 ; if your key is longer than this, maybe you can use md5 result as the key

YAC_MAX_VALUE_RAW_LEN = 64M

YAC_MAX_VALUE_COMPRESSED_LEN = 1M

## Methods

### Yac::\_\_construct

```
   Yac::__construct([string $prefix = ""])
```

Constructor of Yac, you can specify a prefix which will used to prepend to any keys when doing set/get/delete

```php
<?php
   $yac = new Yac("myproduct_");
?>
```

### Yac::set

```
   Yac::set($key, $value[, $ttl])
   Yac::set(array $kvs[, $ttl])
```

Store a value into Yac cache, keys are cache-unique, so storing a second value with the same key will overwrite the original value.

```php
<?php
$yac = new Yac();
$yac->set("foo", "bar");
$yac->set(
    array(
        "dummy" => "foo",
        "dummy2" => "foo",
        )
    );
?>
```

### Yac::get

```
   Yac::get(array|string $key)
```

Fetches a stored variable from the cache. If an array is passed then each element is fetched and returned.

```php
<?php
$yac = new Yac();
$yac->set("foo", "bar");
$yac->set(
    array(
        "dummy" => "foo",
        "dummy2" => "foo",
        )
    );
$yac->get("dummy");
$yac->get(array("dummy", "dummy2"));
?>
```

### Yac::delete

```
   Yac::delete(array|string $keys[, $delay=0])
```

Removes a stored variable from the cache. If delay is specified, then the value will be deleted after \$delay seconds.

### Yac::flush

```
   Yac::flush()
```

Immediately invalidates all existing items. it doesn't actually free any resources, it only marks all the items as invalid.

### Yac::info

```
   Yac::info(void)
```

Get cache info

```php
<?php
  ....
  var_dump($yac->info());
  /* will return an array like:
  array(11) {
      ["memory_size"]=> int(541065216)
      ["slots_memory_size"]=> int(4194304)
      ["values_memory_size"]=> int(536870912)
      ["segment_size"]=> int(4194304)
      ["segment_num"]=> int(128)
      ["miss"]=> int(0)
      ["hits"]=> int(955)
      ["fails"]=> int(0)
      ["kicks"]=> int(0)
      ["slots_size"]=> int(32768)
      ["slots_used"]=> int(955)
  }
  */
```
