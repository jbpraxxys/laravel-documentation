# How To Display Your Dataobject on Laravel?

1. In your **SampleController.php**

You should have this block of code.
```
	public function showIndex() {

       	$events = Event::orderBy('created_at')->get();
        return $this->view('guest.events.index', 'events', [
        	'events' => $events,
        ]);


	}
```

2. You need to loop your object on your page.

```

@foreach ($events as $event)<a href="{{ $event->renderShowURL('guest') }}" class="news-card">
	<div class="news-img-cntnr">
		<div  class="news-img" style="background-image: url('{{ $event->renderFilePath('file_path') }}')"></div>
	</div>
	<div class="news-excerpt">
		<p>{!! str_limit($event->description, 200) !!}</p>
	</div>
</a
>@endforeach

```
``{{ $event->renderShowURL('guest') }}`` for href.
``{{ $event->renderFilePath('file_path') }}`` for images
``{!! str_limit($event->description, 200) !!}`` for htmleditor with limit
``{{ $event->name }}`` for text