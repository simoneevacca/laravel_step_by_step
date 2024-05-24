## STEPS CRUD

- Creazione del model con php artisan make:model collegandogli il controller, le migrazioni, il seeder se serve e le form request.

```bash

php artisan make:model NomeModello -mscrR

```
m: migration
s: seeder
cr: controller con crud e resource
R: form request

- creare la tabella del db nel file migration, indicando le tipologie di dato:

```php

public function up(): void
    {
        Schema::create('projects', function (Blueprint $table) {
            $table->id();
            $table->string('project_name', 50)->require()->unique();
            $table->string('slug', 50)->nullable();
            $table->text('description')->nullable();
            $table->string('preview_image', 255)->nullable();
            $table->string('link_view', 255)->nullable();
            $table->string('link_code', 255)->nullable();

            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('projects');
    }


⚠️⚠️⚠️ATTENZIONE, CAMBIARE I DATI CON QUELLI DEL MODELLO CORRENTE


```

- Se necessario implementiamo la logica nel file del seeder per seedare la tabella con dati reali o inventati con Faker

```php

public function run(): void
    {
        $projects = config('projectsdb.project');

        foreach ($projects as $project) {
            $newproject = new Project();
            $newproject->project_name = $project['project_name'];
            $newproject->slug = $project['slug'];
            $newproject->description = $project['description'];
            $newproject->preview_image = $project['preview_image'];
            $newproject->link_view = $project['link_view'];
            $newproject->link_code = $project['link_code'];
            $newproject->save();


        }
    }

```

⚠️⚠️⚠️ATTENZIONE, CAMBIARE I DATI CON QUELLI DEL MODELLO CORRENTE


- Per seedare più tabelle contemporaneamente nel DatabaseSeeder.php all'interno del metodo run:

```php

    $this->call([

        nomeSeeder1::class,
        nomeSeeder2::class,

    ])

```

- Inviare le migration e i seed:

```bash 

    php artisan migrate --seed

```
    

- creo il gruppo di rotte:

```php

/*route group*/
Route::middleware(['auth', 'verified'])
    ->name('admin.')
    ->prefix('admin')
    ->group(function (){

        Route::get('/', [DashboardController::class, 'index'])->name('dashboard'); // esiste se usiamo la repo di template
        Route::resource('/projects', ProjectController::class)->parameters(['projects' => 'project:slug']); 
        // ->parameters ...... slug solo se implementato
       
    });

```

⚠️⚠️⚠️ATTENZIONE, CAMBIARE I DATI CON QUELLI DEL MODELLO CORRENTE
⚠️⚠️⚠️ASSICURARSI DI AVER IMPORTATO I CONTROLLER


- Iniziare a impostare il layout per l'admin

```bash

cd ./resource/view
mkdir admin
cd ./admin
mkdir nomeProgettoAlPlurale
cd ./nomeProgettoAlPlurale
touch index.blade.php create.blade.php edit.blade.php show.blade.php

```

- Andiamo nel controller e iniziamo a lavorare sul metodo index, dove ci saranno tutti gli elementi della tabella principale e le eventuali azioni 

```php

public function index()
    {
        $projects = Project::orderByDesc('id')->paginate('15'); //paginate è opzionale, serve per visualizzare n risultati per paina
        return view('admin.projects.index', compact('projects'));
    }

```

⚠️⚠️⚠️USARE dd() PER VERIFICARE COSA STIAMO FACENDO
⚠️⚠️⚠️ATTENZIONE, CAMBIARE I DATI CON QUELLI DEL MODELLO CORRENTE

- Compilare il file index inserendo una tabella per la visualizzazione degli elementi del database riservando una colonna alle azione che riportano alle azioni crud (show, edit, delete) tramite la rotta e aggiungere un tasto fuori dalla tabella che riporta all'azione create

php artisan route:list  ci consente di visualizzare tutte le rotte e sapere se richiedono una variabile extra

```php


@forelse ($projects as $project)
 <tr class="">
    <td scope="row">{{ $project->id }}</td>
    <td>{{ $project->project_name }}</td>
    <td><a href="{{ $project->link_view }}">click</a></td>
    <td><a href="{{ $project->link_code }}">click</a></td>
</tr>
@empty
    <tr class="">
        <td scope="row">no record</td>
    </tr>
@endforelse

```
⚠️⚠️⚠️ATTENZIONE, CAMBIARE I DATI CON QUELLI DEL MODELLO CORRENTE



- Andiamo nel controller e iniziamo a lavorare sul metodo create per aggiungere nuovi elementi alla tabella del db

```php

 public function create()
    {
        return view('admin.projects.create');
    }


```
⚠️⚠️⚠️ATTENZIONE, CAMBIARE I DATI CON QUELLI DEL MODELLO CORRENTE


- andiamo nel create.blade.php e creiamo il form per l'inserimento dei dati della tabella

```php


@if($errors->any())

<div class="alert alert-danger" role="alert">
    <ul>
        @foreach ($errors->all() as $error )
        <li>{{$error}}</li>
        @endforeach
    </ul>
</div>

@endif
// snippet per mostrare gli errori di validazione


<form action="{{ route('admin.projects.store') }}" method="POST" enctype="multipart/form-data"> //enctype serve per le immagini 
@csrf

<div class="mb-3">
    <label for="project_name" class="form-label text-white ">Project Name</label>
    <input type="text" name="project_name" id="project_name" class="form-control" placeholder=""
        aria-describedby="project_nameId" value="{{ old('project_name') }}" />
        //old() per mantenere i dati in caso di errori di validazione
</div>
@error('project_name')
    <div class="text-danger">{{ $message }}</div>
@enderror
// vai a vedere la documentazione per stampare tutti gli errori di validazione all'inizio del form

......
......


```

⚠️⚠️⚠️ATTENZIONE, CAMBIARE I DATI CON QUELLI DEL MODELLO CORRENTE

- Andiamo nel controller e iniziamo a lavorare sul metodo store per salvare i dati nel db

 ```php
 public function store(StoreProjectRequest $request)
 
    {
        //validate data
        $val_data = $request->validated();
        dd($val_data);

        //create

        //redirect


    }

```

- prima di salvare lo store compilare i dati di validazione, impostando su true l'authorize() e inserendo i dati per la validazione nelle formRequest

```php

public function authorize(): bool
    {
        return true;
    }

public function rules(): array
    {
        return [
            'type_id' => 'nullable|exists:types,id',
            'project_name' => 'required|min:5|max:50|unique:projects',
            'slug' => 'nullable',
            'description' => 'nullable|max:200',
            'preview_image' => 'nullable|max:255',
            'link_view' => 'nullable|max:255',
            'link_code' => 'nullable|max:255',
        ];
    }


```

⚠️⚠️⚠️ATTENZIONE, CAMBIARE I DATI CON QUELLI DEL MODELLO CORRENTE




- Completare il metodo store

```php
public function store(StoreProjectRequest $request)
    {
        //validate data
        $val_data = $request->validated();
    
        // creazione dello slug 
        $slug = Str::slug($request->project_name, '-'); //Str va importato
        $val_data['slug'] = $slug;
        
        // se nel form richiediamo delle immagini
        if ($request->has('preview_image')) {
            $image_path = Storage::put('uploads', $val_data['preview_image']);
            $val_data['preview_image'] = $image_path;
        }
       
    
        Project::create($val_data);
        
        //redirect
        return to_route('admin.projects.index');
    }

```

⚠️⚠️⚠️ATTENZIONE, CAMBIARE I DATI CON QUELLI DEL MODELLO CORRENTE


- Aggiungere le fillable nel model per consentire l'assegnazione di massa:

```php 

    protected $fillable = ['project_name', 'slug', 'description', 'preview_image', 'link_view', 'link_code', 'type_id'];

```

⚠️⚠️⚠️ATTENZIONE, CAMBIARE I DATI CON QUELLI DEL MODELLO CORRENTE


- Andiamo nel controller e iniziamo a lavorare sul metodo show per visualizzare la singola pagina per ogni elemento
  In show.blade.php estendendere il layou e creare la pagina con i dati dell'elemento
```php

 public function show(Project $project)
    {
        return view('admin.projects.show', compact('project'));
    } 

```

- Andiamo nel controler e iniziamo a lavorare sul metodo edit per modificare i dati di un singolo elemento

```php

public function edit(Project $project)
    {

        return view('admin.projects.edit', compact('project'));
    }

```

- Andiamo in edit.blade.php e copiamo il form del create aggiungendo il @method('PUT'), cambiando la route e passandogli la singola variabile 

```php

<form action="{{ route('admin.projects.update', $project) }}" method="POST" enctype="multipart/form-data">
    @csrf
    @method('put')

    <div class="mb-3">
        <label for="project_name" class="form-label text-white">Project Name</label>
        <input type="text" name="project_name" id="project_name" class="form-control" placeholder=""
            aria-describedby="project_nameId" value="{{ old('project_name', $project->project_name) }}" />
    </div>

```

- Andiamo nel controler e iniziamo a lavorare sul metodo update per salvare i dati modificati

```php

public function update(UpdateProjectRequest $request, Project $project)
    {
        $val_data = $request->validated();

        // compilare i dati della validazione anche per l'UpdateProjectRequest

        if($request->has('preview_image')) {
            
            if($project->preview_image){
                Storage::delete($project->preview_image); //eliminare l'immagine prima di salvare la nuova
            }
            $image_path = Storage::put('uploads', $val_data['preview_image']);
            $val_data['preview_image'] = $image_path;

        }   

        $project->update($val_data);
        return to_route('admin.projects.show', $project);
    }

```

- Andiamo nel controler e iniziamo a lavorare sul metodo destroy per eliminare un elemento

```php

public function destroy(Project $project)
    {

        if($project->preview_image){
            Storage::delete($project->preview_image); //eliminare l'immagine prima di salvare la nuova
        }
        $project->delete();
        return to_route('admin.projects.index');
    }

```
⚠️⚠️⚠️ATTENZIONE, RICHIDERE UNA CONFERMA NELL'INDEX PRIMA DELL'ELIMINAZIONE, TRAMITE UNA MODALE
⚠️⚠️⚠️ATTENZIONE, CAMBIARE I DATI CON QUELLI DEL MODELLO CORRENTE



