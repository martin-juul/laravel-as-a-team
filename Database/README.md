# Database

When interacting with the database, you should **always** use Eloquent.

This is important because model event will not fire, when using the `DB` facade.
Usually some kind of caching layer is also implemented. But that also depends on eloquent.

## When not to use eloquent?

As a rule of thumb, you **should** bypass eloquent if you're doing bulk data **insertion** or **updating***.

NEVER delete with the DB facade, as the model will still exist in the cache.
