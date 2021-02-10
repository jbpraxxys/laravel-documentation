# How to create a selected page and data-object in laravel (CRUD)

### Creating **data-object** is just like creating a ** selected-page** , the only difference is that ** selected-page**  has a dedicated page to display the data-object

1. create a selected page on your *views folder*
2. create a route for your selected page

	Route::get('samples/selected-sample', 'EventController@showSelected')->name('events-show');

3. in your **App/Http/Controllers/Guest**
- Create a folder named after the plural form of your selected page. 
- Inside your folder is your dedicated controller.
*App/Http/Controllers/Samples/Guest/sample.php*
```

<?php

namespace App\Http\Controllers\Guest\Events;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use App\Traits\Controllers\PageTrait;

// use App\Models\Products\Product;
// use App\Models\Products\ProductCategory;
// use App\Models\Carousels\Carousel;
// use App\Models\Collections\Collection;

class EventController extends Controller
{
	use PageTrait;

	/* Show Home */
	public function showIndex() {

        return $this->view('guest.events.index', 'events', [
        	//
        ]);

	}

	public function showSelected() {

        return view('guest.events.show');

	}

}

```

4. Create a model for your selected-page
``php artisan make:model Sample -m``

- This will generate a migration table and a file called **Sample.php** in your App folder.
- Create **Sample** folder under *App/Models*, inside that folder transfer your **Sample.php**

```

<?php

namespace App\Models\Events;

use App\Extenders\Models\BaseModel as Model;
// use App\Traits\Pages\MetaTrait;
use App\Traits\FileTrait;
use App\Traits\ManyFilesTrait;

class Event extends Model
{
    use FileTrait, ManyFilesTrait;

    /**
     * The attributes that should be mutated to dates.
     *
     * @var array
     */

    // protected static $sortableField = 'order_column';

    /**
     * Get the options for generating the slug.
     */
    // public function getSlugOptions() : SlugOptions
    // {
    //     return SlugOptions::create()
    //         ->generateSlugsFrom('name')
    //         ->saveSlugsTo('slug')
    //         ->allowDuplicateSlugs();
    // }

    /**
     * Get the indexable data array for the model.
     *
     * @return array
     */
    public function toSearchableArray() {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'description' => $this->description,
        ];
    }

    /**
     * @Setters
     */
    public static function store($request, $item = null)
    {
        $vars = $request->only(['name', 'description']);

        if (!$item) {
            $item = static::create($vars);
        } else {
            $item->update($vars);
        }

        // $item->storeMeta($request);

        if ($request->hasFile('file_path')) {
            $item->storeFile($request->file('file_path'), 'file_path', 'event-images');
        }

        if ($request->hasFile('images')) {
            $item->addFiles($request->file('images'));
        }

        return $item;
    }

    /**
     * @Render
     */

    public function renderName() {
        return $this->name;
    }

    public function renderShowUrl($prefix = 'admin') {
        $route = $this->id;

        if ($prefix == 'guest') {
            $route = [$this->id, $this->slug];
        }

        return route($prefix . '.events.show', $route);
    }

    public function renderArchiveUrl($prefix = 'admin') {
        return route($prefix . '.events.archive', $this->id);
    }

    public function renderRestoreUrl($prefix = 'admin') {
        return route($prefix . '.events.restore', $this->id);
    }
}

```

5. Go to your migration table, located at *App/database/migrations*.
look for your **create_sample_table.php**

this file contains your database migration.

```

<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateEventsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('events', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name');
            $table->longText('description')->nullable();
            $table->text('file_path')->nullable();

            $table->softDeletes();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('events');
    }
}


```

after you alter all the needed data to store on your database, proceed to next step.

6. Run **php artisan migrate**
- This will generate your tables on your database

7. Create a store request *App/Http/Requests/Admin* Create a plural form folder of your selected page, inside that folder is your **SampleStoreRequest.php**
- **App/Http/Requests/Admin/Samples/SampleStoreRequest.php**

```

<?php

namespace App\Http\Requests\Admin\events;

use Illuminate\Foundation\Http\FormRequest;

use App\Rules\Varchar;
use App\Rules\HTMLText;
use App\Rules\DateTime;
use App\Rules\Image;

class eventstoreRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        $id = $this->route('id');

        $rules = [
            'name' => ['required', new Varchar],
            'description' => ['required', new HTMLText],
        ];

        if (!$id) {
            $rules = array_merge($rules, [
                'images' => 'required',
                'images.*' => [new Image],
                'file_path' => ['required', new Image],
            ]);
        } else {
            $rules = array_merge($rules, [
                'images' => 'nullable',
                'images.*' => [new Image],
                'file_path' => ['nullable', new Image],
            ]);
        }

        return $rules;
    }

    /**
     * Get custom attributes for validator errors.
     *
     * @return array
     */
    public function attributes()
    {
        return [
            'file_path' => 'image',
        ];
    }
}

```

- This will act as your validation before updating the content of your database.

8. On your *App/database/seeds/PermissionsTableSeeder.php*

create a dataobject manager by copying block of code and change it to your selected page manager.

```

[
    'name' => 'Events Management',
    'description' => 'Manage Events',
    'icon' => 'far fa-newspaper',
    'items' => [
        [
            'name' => 'admin.events.crud',
            'description' => 'Manage Events',
        ],
    ],
],

```

9. On your *Resources/views/admin/partials/sidebar.blade.php*
- insert your crud

```

@if ($self->hasAnyPermission(['admin.events.crud']))
    <li class="nav-item has-treeview {{ $checker->route->areOnRoutes([
            'admin.events.index','admin.events.create','admin.events.show',
        ]) }}">
        <a href="#" class="nav-link">
            <i class="nav-icon fas fa-vial"></i>
            <p>
                Events Management
                <i class="right fa fa-angle-left"></i>
            </p>
        </a>
        <ul class="nav nav-treeview">
            <li class="nav-item">
                <a href="{{ route('admin.events.index') }}" class="nav-link {{ $checker->route->areOnRoutes([
                    'admin.events.index','admin.events.create','admin.events.show',
                ]) }}">
                    <i class="nav-icon far fa-circle"></i>
                    <p>
                        Events Items
                    </p>
                </a>
            </li>
        </ul>
    </li>
@endif

```

10. On your terminal ***php artisan permission:update***

11. Create your dedicated admin controller
*App/Http/Controllers/admin* Create a plural form folder of your selected page, inside that folder is your **SampleController.php, SampleFetchController.php**

#### SampleController.php
```

<?php

namespace App\Http\Controllers\Admin\Events;

use App\Http\Controllers\Controller;

use Illuminate\Http\Request;

use App\Http\Requests\Admin\Events\EventStoreRequest;

use App\Models\Events\Event;

class EventController extends Controller
{
    protected $indexView = 'admin.events.index';
    protected $createView = 'admin.events.create';
    protected $showView = 'admin.events.show';
    protected $guard = 'admin';
    
    /**
     * Display a listing of the resource.
     */
    public function index()
    {
        return view($this->indexView, [
            //
        ]);
    }

    /**
     * Show the form for creating a new resource.
     */
    public function create()
    {
        return view($this->createView, [
            //
        ]);
    }

    /**
     * Store a newly created resource in storage.
     */
    public function store(EventStoreRequest $request)
    {
        $item = Event::store($request);

        $message = "You have successfully created {$item->renderName()}";
        $redirect = $item->renderShowUrl();

        return response()->json([
            'message' => $message,
            'redirect' => $redirect,
        ]);
    }

    /**
     * Display the specified resource.
     */
    public function show(Request $request, $id, $slug = null)
    {
        $item = Event::withTrashed()->findOrFail($id);

        if ($this->guard == 'guest' && !$slug) {
            return redirect()->to($item->renderShowUrl('guest'));
        }

        return view($this->showView, [
            'item' => $item,
        ]);
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(EventStoreRequest $request, $id)
    {
        $item = Event::withTrashed()->findOrFail($id);
        $message = "You have successfully updated {$item->renderName()}";

        $item = Event::store($request, $item);

        return response()->json([
            'message' => $message,
        ]);
    }

    /**
     * Remove the specified resource from storage.
     */
    public function archive($id)
    {
        $item = Event::withTrashed()->findOrFail($id);
        $item->archive();

        return response()->json([
            'message' => "You have successfully archived {$item->renderName()}",
        ]);
    }

    /**
     * Restore the specified resource from storage.
     */
    public function restore($id)
    {
        $item = Event::withTrashed()->findOrFail($id);
        $item->unarchive();

        return response()->json([
            'message' => "You have successfully restored {$item->renderName()}",
        ]);
    }
}


```

#### SampleFetchController.php
```

<?php

namespace App\Http\Controllers\Admin\Events;

use App\Extenders\Controllers\FetchController as Controller;

use App\Models\Events\Event;

class EventFetchController extends Controller
{
   /**
    * Set object class of fetched data
    *
    * @return void
    */
   public function setObjectClass()
   {
       $this->class = new Event;
   }

   /**
    * Custom filtering of query
    *
    * @param Illuminate\Support\Facades\DB $query
    * @return Illuminate\Support\Facades\DB $query
    */
   public function filterQuery($query)
   {
       /**
        * Queries
        *
        */
        if($this->request->filled('archive')) {
           $query = $query->onlyTrashed();
        }

       return $query;
   }

   /**
    * Custom formatting of data
    *
    * @param Illuminate\Support\Collection $items
    * @return array $result
    */
   public function formatData($items)
   {
       $result = [];

       foreach($items as $item) {
           $data = $this->formatItem($item);
           array_push($result, $data);
       }

       return $result;
   }

   /**
    * Build array data
    *
    * @param  App\Contracts\AvailablePosition
    * @return array
    */
   protected function formatItem($item)
   {
       return [
           'id' => $item->id,
           'name' => $item->name,
           'description' => $item->description,
           'created_at' => $item->renderDate(),
           'showUrl' => $item->renderShowUrl(),
           'archiveUrl' => $item->renderArchiveUrl(),
           'restoreUrl' => $item->renderRestoreUrl(),
           'deleted_at' => $item->deleted_at,
       ];
   }

   public function fetchView($id = null) {
       $item = null;
       $images = [];

       if ($id) {
           $item = Event::withTrashed()->findOrFail($id);
           $item->archiveUrl = $item->renderArchiveUrl();
           $item->restoreUrl = $item->renderRestoreUrl();
           $item->resume_path = $item->renderFilePath('file_path');
           // $item->careerUrl = $item->career->renderShowUrl();
           // $item->educationLabel = $item->renderEducation();
       }

       return response()->json([
           'item' => $item,
           'images' => $images
       ]);
   }
}

```

12. On your *resources/js/views/admin* create a plural folder of your selected page. Inside that folder create:
	- SampleTable.vue
	- SampleView.vue

	``Pattern all your files from **samples** folder``


13. On your *resources/views/admin* create a plural folder of your selected page. Inside that folder create:
	- create.blade.php
	- index.blade.php
	- show.blade.php

	``Pattern all your files from **samples** folder``

14. On your *resources/views/admin/index.js* insert your Vue.component

```

Vue.component('sample-table', require('./samples/SampleTable.vue').default);
Vue.component('sample-view', require('./samples/SampleView.vue').default);

```


15. On your *Routes/admin.php*
- create a route for your admin **SampleController.php**

```

Route::namespace('Events')->group(function() {

	Route::get('events', 'EventController@index')->name('events.index');
	Route::get('events/create', 'EventController@create')->name('events.create');
	Route::post('events/store', 'EventController@store')->name('events.store');
	Route::get('events/show/{id}', 'EventController@show')->name('events.show');
	Route::post('events/update/{id}', 'EventController@update')->name('events.update');
	Route::post('events/{id}/archive', 'EventController@archive')->name('events.archive');
	Route::post('events/{id}/restore', 'EventController@restore')->name('events.restore');

	Route::post('events/fetch', 'EventFetchController@fetch')->name('events.fetch');
	Route::post('events/fetch?archived=1', 'EventFetchController@fetch')->name('events.fetch-archive');
	Route::post('events/fetch-item/{id?}', 'EventFetchController@fetchView')->name('events.fetch-item');
	Route::post('events/fetch-pagination/{id}', 'EventFetchController@fetchPagePagination')->name('events.fetch-pagination');

});

```