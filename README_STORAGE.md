config/filestystems.php

```php
'default' => env('FILESYSTEM_DISK', 'public'),
```
.env
```php
FILESYSTEM_DISK=public
```

Creazione Symlink
```bash
php artisan storage:link

```

Nel form create e update: enctype="multipart/form-data"
```php
<form action="{{ route('admin.projects.store') }}" method="POST" enctype="multipart/form-data">

```
Nel controller:
```php
use Illuminate\Support\Facades\Storage;

public function store(StoreProjectRequest $request)

    {
        if ($request->has('preview_image')) {
            $image_path = Storage::put('uploads', $val_data['preview_image']);
            $val_data['preview_image'] = $image_path;
        }

        .......
    }


    public function update(UpdateProjectRequest $request, Project $project)
    {
        if($request->has('preview_image')) {
            

            if($project->preview_image){
                Storage::delete($project->preview_image);
            }
            $image_path = Storage::put('uploads', $val_data['preview_image']);
            $val_data['preview_image'] = $image_path;

        }   

        ......
    }


    public function destroy(Project $project)
    {

        if($project->preview_image){
            Storage::delete($project->preview_image);
        }
        
        ......
    }

```

Visualizzazione dei file:
```php
    <img src="{{ asset('storage/' . $project->preview_image) }}" alt="">

```