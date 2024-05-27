

// 1- avere gia tutte le migration di tutte le tabelle con i relativi modelli, controller ecc... 



// 2- Creare la migration per connettere le 2 tabelle creando la foreing key con il seguente comando da terminale:
```bash

 php artisan make:migration add_user_id_foreing_key_to_post_table

 ```

//     all'interno bisogna creare la colonna della foreing key come segue: 
//     è importante nel metodo down eliminare prima il vincolo e poi droppare la colonna    
        
```php
        return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::table('projects', function (Blueprint $table) {
            $table->unsignedBigInteger('type_id')->nullable()->after('id');
            $table->foreign('type_id')->references('id')->on('types')->onDelete('set null');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::table('projects', function (Blueprint $table) {
            $table->dropForeign('projects_type_id_foreign');
            $table->dropColumn('type_id');
        });
    }
};

```

// 3- mettere in relazione i 2 modelli, a seguire l'esempio per la relazione one to many

    // Nel model principale:

    ```php

        public function projects(){
            return $this->hasMany(Project::class);
        }
    ```

// Nel model secondario:
    
```php

    public function type(){
        return $this->belongsTo(Type::class);
    }

```

// 4- aggiornare le fillable nei model e le validazione in NameUpdateRequest e NamePostRequest

// 5- importare i modelli nelle crud

// 6- per reperire un dato dalla tabella es l'id

```php

    $project->type->id

```
---------------- RELAZIONE MANY TO MANY ------------------------------------

1- in ciascuno dei 2 modelli creare la funzione per collegare l'altro:

```php

public function projects() {    
        return $this->belongsToMany('App\Models\Projects');
    }

    public function technoligies() {    
        return $this->belongsToMany('App\Models\Technology');
    }

```

2- creare la migrazione per la tabella pivot con il seguente comando da terminale: 
mettere i nomi in ordine alfabetico (in questo caso project e technology)

```bash

php artisan make:migration create_project_technology_table

```

3- per inserire i dati nella tabella pivot usare attach:
4- per rimuoverli usare detach

```php

    $project->technoligies()->attach($val_data['technologies']);

```

5- per aggiungere e rimuovere contemporaneamente dei dati usare sync:
sync() accetta un array con all'interno gli id della tabella pivot e rimuove tutti quelli non inseriti

```php

$project->technoligies()->sync($val_data['technologies']);

```

