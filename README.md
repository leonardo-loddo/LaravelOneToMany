vado nelle due request di author e su authorize metto true

creo le rotte e il controller migrazioni request e moddello con i metodi visti la scorsa volta

aggiungo la cartella author e ci metto dentro le view richieste dal crud

vado nella migrazione e aggiungo i dati che voglio nella tabella authors
    public function up(): void
    {
        Schema::create('authors', function (Blueprint $table) {
            $table->id();
            $table->string('firstname');
            $table->string('lastname');
            $table->dateTime('birthday')->nullable();
            $table->timestamps();
        });
    }

vado nel modello author e mappo questi dati all'interno del fillable
    {
        use HasFactory;
        protected $fillable = [
            'firstname', 'lastname', 'birthday'
        ];
    }

php artisan migrate

completo le funzioni presenti in AuthorController

modifico le viste in modo da corrispondere alle varie rotte e funzioni

ora vado sulle request create e aggiungo le regole di validazione
    public function rules(): array
    {
        return [
            'firstname' => 'required',
            'lastname' => 'required',
        ];
    }

per creare una relazione tra gli articoli e gli autori che li hanno creti dovremo andare ad aggiungere la la colonna con l'id dell'autore alla tabella degli articoli

per prima cosa dobbiamo creare una migrazione
    php artisan make:migration add_to_articles_table

modifico la funzione up
    public function up(): void
    {
        Schema::table('articles', function (Blueprint $table) {
            $table->unsignedBigInteger('author_id');//aggiungo la colonna per l'id
            $table->foreign('author_id')->references('id')->on('authors');//dico che author_id é una foreign key che fa riferimento alla chiave id nella tabella authors
        });
    }

modifico la funzione down
    public function down(): void
    {
        Schema::table('articles', function (Blueprint $table) {
            $table->dropForeign(['author_id']); //elimino la relazione
            $table->dropColumn(['author_id']); //elimino la colonna
        });
    }

php artisan migrate

vado nel modello Article e aggiungo la funzione author
    public function author()
    {
        return $this->belongsTo(Author::class);
    }
attraverso questa funzione potremo accedere ai dati dell'autore dell'articolo

aggiungo anche il nuovo attributo nel fillable del modello
    protected $fillable = [
        'title', 'body', 'image', 'author_id'
    ];

vado nel modello Author e aggiungo la funzione articles
    public function articles()
    {
        return $this->hasMany(Article::class);
    }

passo alla view create tramite un compact nella sua funzione tutti gli autori in modo da poterli mostrare come opzioni in una select
    public function create(){
        $authors = Author::all();
        return view('article.create', compact('authors'));
    }
ripeto anche nella funzione edit

vado nella view create di article perché devo aggiunger una select per inserire l'autore
    <div class="mb-3">
        <label for="author_id" class="form-label">Autore</label>
        <select name="author_id" id="">
        @foreach ($authors as $author)
            <option value="{{$author->id}}">{{$author->firstname . ' ' . $author->lastname}}</option>  
        @endforeach
        </select>
    </div>
ripeto anche nella edit

ora aggiungo author_id anche nella funzione store
    Article::create([
        'title' => $request->title,
        'body' => $request->body,
        'image' => $path_image,
        'author_id' => $request->author_id,
    ]);
ripeto nella update

voglio mostrare l'autore nel dettaglio di un articolo quindi vado nella show e lo richiamo attraverso la funzione author definita nel model Article
    <h2>{{$article->author->firstname . ' ' . $article->author->lastname}}</h2>

nella vista article.edit faccio una modifica alle option che verranno generate nella select in modo tale che, grazie ad un if di impostare come opzione selezionata di default l'autore attuale dell'articolo
    @foreach ($authors as $author)
        <option value="{{$author->id}}" @if($article->author_id == $author->id) selected @endif>
            {{$author->firstname . ' ' . $author->lastname}}
        </option>  
    @endforeach

