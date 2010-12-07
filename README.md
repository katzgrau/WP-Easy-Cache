# WPEasyCache

A Wordpress PHP class for easily caching data programmatically.

## But Why?

Wordpress has its own built-in caching functions, sure. But the functions available are actually very primitive. You would think that the function `wp_cache_get( $key, $group )` for instance, could take a parameter named `$default` as a value to return if the cache doesn't return anything.

Another example inflexibility is `wp_cache_set( $key, $data, $group, $expire )`. This function will try to set a cache item by key. If the item doesn't exist, the function returns false. That's strange.. most developers would hope the cache item would be created in that instance.

No big deal, but it essentially forces developers to write a wrapper around these functions.

## So what does WPEasyCache do differently?

1. WPEasyCache uses the database as a caching back-end. No new tables re required, it rolls of the wp_option table
2. WPEasyCache gives you all the expected functionality of a reasonably robust caching class. The set function will add the item if it doesn't exist, and the the get function takes a default parameter.
3. It serializes cached objects in JSON, so you can cache pretty much anything that doesn't depend on references. Don't try and cache circular arrays — but if you really want to, change the serialization method to PHP's native serialize; I used JSON for readbility and portability.

## How about an example?

Sure thing. Here's and example getting a tweet for a given user.

    <?php

    require_once dirname(__FILE__) . '/path/to/class/WPEasyCache.php';

    function get_last_tweet($username, $default = 'Nothing to say')
    {
        if(!($last = WPEasyCache::get('last_tweet')))
        {
            $url     = "http://api.twitter.com/1/statuses/user_timeline.json?screen_name=$username&count=1";
            $content = @file_get_contents($url);

            if($content)
                $content = @json_decode($content);
            else
                $content = WPEasyCache::get('last_tweet', FALSE, TRUE);

            if(!is_array($content) || count($content) == 0) return $default;

            $last = $content[0]->text;
            WPEasyCache::set('last_tweet', $last, 60*60); # Cache for an hour
        }

        return $last;
    }

    /* ... Somewhere in the code */
    get_last_tweet('_kennyk_');

## Anything else?

Yes. The class only has four methods:

`WPEasyCache::get($key, $default = FALSE, $force = FALSE)` — Gets a cache item with the given `$key`. If the item does not exist, `$default` will be returned. If you want the item to come back regardless of whether it's expired, set `$force` to `TRUE`.

`WPEasyCache::public static function set($key, $value, $expire = FALSE)` — Set an item with a given `$key` in the cache with a `$value`. If an item does not already exist with the given key, it will be created. `$expire` is the number of seconds you want to cache the item for. Set `$expire` to `FALSE` if you do not want it to expire.

`WPEasyCache::delete($key)` — Delete the item with the given `$key` from the cache.

`WPEasyCache::flush()` — Remove all items that WPEasyCache has ever cached from the cache.

## Bugs

This class was written an is maintained by [Kenny Katzgrau](http://codefury.net). Email him if you find bugs, or have a feature request.