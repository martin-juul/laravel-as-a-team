# Models

## Return types

You might be tempted to use return types, as introduced in php 7.0 when defining relationships and scopes.  
As seen below:

```php
<?php

namespace App\Models;

class Article extends AbstractModel {
    // model definitions

    /**
    * Filter out non public articles
    * 
    * @param \Illuminate\Database\Eloquent\Builder $query
    * @return \Illuminate\Database\Eloquent\Builder
    */
    public function scopeWherePublic(Builder $query): \Illuminate\Database\Eloquent\Builder
    {
        return $query->where('public', '=', true);
    }
    
    /**
    * Article Author 
    * 
    * @return App\Models\User
    */
    public function author(): User
    {
        return $this->belongsTo(User::class);
    }
}
```

This will work fine in most cases. But when you implement a caching solution, that's when it'll start to break.

The common way to create a caching library for eloquent, is to extend `\Illuminate\Database\Eloquent\Builder` - and replace it at runtime in the service container

Instead you should define the type hints in docblocks:

```php
<?php

namespace App\Models;

class Article extends AbstractModel {
    // model definitions

    /**
    * Filter out non public articles
    * 
    * @param \Illuminate\Database\Eloquent\Builder $query
    * @return void
    */
    public function scopeWherePublic($query)
    {
        $query->where('public', '=', true);
    }
    
    /**
    * Article Author 
    * 
    * @return App\Models\User
    */
    public function author()
    {
        return $this->belongsTo(User::class);
    }
}
```
