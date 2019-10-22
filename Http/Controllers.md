# Controllers

A controller is responsible for handling input data and formatting output data.

If you find yourself with a method body, spanning more than 20 lines, you should refactor to a service.

## Requests

If the request expects any data, you should **always** create a FormRequest. Despite the name, they work for json too ;)

Here you place your validation logic, and then define the type as the request parameter in your controller.

For each Create/Update action, you create separate FormRequest classes.

## Sample implementation

### Model

```shell script
php artisan make:model Models\\Article
```

```php
<?php

namespace App\Models;

use App\Traits\Sluggable;

/**
* Class Article
 * @package App\Models
 * 
 * @property string $title
 * @property string $slug
 * @property string $markdown
 * @property string $user_id
 */
class Article extends AbstractModel
{
    use Sluggable;
    
    protected $fillable = [
        'title',
        'slug',
        'markdown',
        'user_id',
    ];
    
    // Eager load author
    protected $with = ['author'];
    
    public function author()
    {
        return $this->belongsTo(User::class);
    }
}

```

### POST Request

```shell script
php artisan make:request Article\\CreateArticleRequest
```

```php
<?php

namespace App\Http\Requests\Article;

use Illuminate\Foundation\Http\FormRequest;

class CreateArticleRequest extends FormRequest
{
    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title'    => 'required|string|min:10|max:260',
            'markdown' => 'required|string|min:150|max:10000', // hint: always define a max.
        ];
    }
}
```

### PUT Request

```shell script
php artisan make:request Article\\CreateArticleRequest
```

```php
<?php

namespace App\Http\Requests\Article;

use Illuminate\Foundation\Http\FormRequest;

class CreateArticleRequest extends FormRequest
{
    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title'    => 'sometimes|string|min:10|max:260',
            'markdown' => 'sometimes|string|min:150|max:10000', // hint: always define a max.
        ];
    }
}
```

### Resource

Then we define the resource, representing the public view of the model.

> Hint: create a docblock for the class, and add **@mixing App\Models\<your-model>** - then phpstorm will typehint $this->{property} for you.

**NEVER EVER EXPOSE THE INTERNAL ID!**

The id column is reserved for the database ONLY! If you need a public unique identifier, either use a slug or a second id column.
Use a meaningful name like `public_id` - then you know what it's for, when you see it.

```php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

/**
* Class ArticleResource
 * @package App\Http\Resources
 * @mixin App\Models\Article
 */
class ArticleResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'title'      => (string)$this->title,
            'markdown'   => (string)$this->markdown,
            'slug'       => (string)$this->slug,
            'author'     => (string)$this->author->display_name,
            'created_at' => (string)$this->created_at,
            'updated_at' => (string)$this->updated_at,
        ];
    }
}

```

### Controller

First scaffold the controller

```shell script
php artisan make:controller ArticleController --api
```

Then we'll implement the logic

```php
<?php

namespace App\Http\Controllers;

use App\Services\MarkdownService;
use App\Models\Article;
use App\Http\Resources\ArticleResource;
use App\Http\Requests\Article\CreateArticleRequest;
use Illuminate\Http\Request;

class ArticleController extends Controller
{
    private $markdownService;
    
    public function __construct(MarkdownService $markdownService)
    {
        $this->markdownService = $markdownService;
    }
    
    public function index(Request $request)
    {
        $perPage = $request->query('perPage', 25);
        
        $articles = Article::paginate($perPage);
        
        return ArticleResource::collection($articles);
    }
    
    public function store(CreateArticleRequest $request)
    {
        $data = $request->validated();
        
        $data['markdown'] = $this->sanitize($data['markdown']);
        $data['user_id'] = auth()->getUser()->id;
        
        $article = Article::create($data);
        
        return new ArticleResource($article);
    }
    
    public function update(UpdateArticleRequest $request, string $slug)
    {
        $data = $request->validated();
        $data['markdown'] = $this->sanitize($data['markdown']);
        
        $article = Article::whereSlug($slug)->firstOrFail();
        
        if (!$article->update($data)) {
            return response()->json([
                'error' => 'update_failed',
                'message' => 'could not update resource',    
            ]);
        }
        
        return new ArticleResource($article);
    }
    
    public function destroy(Request $request, string $slug)
    {
        try {
            $article = Article::whereSlug($slug)->firstOrFail();
        } catch (ModelNotFoundException $e) {
            return response(null, 204); // 204 no content is recommended.
        }
        
        $article->delete();
    }
    
    private function sanitize($input)
    {
        return $this->markdownService->sanitize($input);
    }
}
```

### Mapping in routes/api.php

As this is a standard CRUD Api resource, it's really easy to configure its routing.

```php
<?php

use App\Http\Controllers\ArticleController;

Route::apiResource([
    'articles' => ArticleController::class,
]);
```

Now the methods will be available at:

| HTTP Verb |  Route  | Method                    |
|:----------|:---|:--------------------------|
| GET       |  /api/articles  | ArticleController@index   |
| POST      |  /api/articles  | ArticleController@store   |
| PUT       |  /api/articles  | ArticleController@update  |
| DELETE    |  /api/articles  | ArticleController@destroy |
