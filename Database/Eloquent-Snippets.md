# Eloquent Snippets

## Ordering models by distant many-to-many related items

Let’s say you’re building a system that tracks and manages production chains. When assigning products to machines, you want to get a list of available machines ordered by what products they support, with the most compatible at the top of the list.

You might have a Machine entity that belongs to a Machine Type. The Machine Type entity has a Many-to-Many relationship with Products.

So you need to do a count and order based on a many-to-many relationship with the assigned Machine Type. You can use withCount with constraints to order the machine results based on whether or not the machine’s type relationship has the assigned products you’re looking for.

```php
$machines = $machines->withCount(['type' => function($query) use($product) {
	$query->whereHas('products', function($query) use($product) {
		$query->where('products.id', $product->id);
	});
}])->orderBy('type_count', 'desc');
```

Whether or not the assigned type has the assigned products will set the type_count to a 0 or a 1, which you can then order the results by.
