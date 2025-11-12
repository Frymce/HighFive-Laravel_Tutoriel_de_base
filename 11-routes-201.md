# Routes 201
### Avant de débuter
Nous avons modifié la structure du dossier `view` pour correspondre à ce qu'on est en train de faire.
Le nommage des fichiers était loin d'être idéal. voici la structure actuelle, modifiez également la vôtre :

![illustration](../img/lara-structure.PNG)

Les contrôleurs ont également été modifié pour gérer cette nouvelle structure. À vous de les modifier aussi.

---
Dans l'exercice précédent, vous avez créé un lien vers un article, voyons comment faire fonctionner ce lien avec le routeur.  
Le lien :
```blade
<a href="/article/{{ $article->id }}">Article</a>   
```

Pour la route, encore une fois, tout est très simple. On passe l'ID entre des accolades simple :
```php
Route::get('/article/{id}', [ArticlesController::class, 'show']);
```
Et on la récupère dans le contrôleur : 
```php
public function show($id)
{
    $article = Article::with('user')->where('id', $id)->firstOrFail();
    // dd($article);
    // ddd($article);
    return view('articles.show', compact('article'));
}
```
Notez la fonction `dd()` en commentaire, cette fonction est le `var_dump()` de laravel.  
La fonction `ddd()` pourra aussi être utilisée pour obtenir toutes sortes d'informations.  
On crée un fichier `show.blade.php` dans le dossier `articles` et on affiche l'article.

Ceci est une façon de faire, mais on peut faire encore plus simple.

#### Route Model Binding
Le framework contient une aide précieuse ce qu'on appelle `route model binding`. A partir de l'ID passée en paramètres, on peut obtenir automatiquement la resource souhaitée. 
Pour cela, il suffit de modifier la route et la méthode `show()` du contrôleur :
```php
Route::get('article/{article}', [ArticlesController::class, 'show']);
```
```php
public function show(Article $article)
{
    return view('articles.show', compact('article'));
}
```
Et c'est tout ! 😊  
Laravel va automatiquement chercher la resource correspondante en se servant des noms et de l'ID passée en paramètre dans le lien. Si l'article n'existe pas, une erreur '404' est retournée.
Ceci fonctionne entre autres grâce à la méthode `getRouteKeyName()` qui retourne 'id' par défaut.
Si on veut changer ce comportement on peut surcharger (modifie le comportement par défaut) cette méthode dans le modèle :
```php
public function getRouteKeyName()
{
    return 'title';
}
```
On devra donc modifier aussi les liens en conséquence :
```blade
<a href="/article/{{ $article->title }}">Article</a>
```
Ou plus simple encore, on spécifie directement à la route ce que l'on veut :
```php
Route::get('/article/{article:title}', [ArticlesController::class, 'show']);
```
Dans ce dernier cas, on peut se passer de la méthode `getRouteKeyName()`.
 
Cette caractéristique est très pratique, néanmoins il ne faut pas l'utiliser sans réfléchir.
Installons une librairie pour nous aider à analyser ce qu'il se passe dans notre code :
```bash
composer require barryvdh/laravel-debugbar --dev
```
Cette librairie dont voici le lien [github](https://github.com/barryvdh/laravel-debugbar) sera active quand la variable d'environnement `APP_DEBUG` est `true`.

Ajoutons des commentaires à nos articles à l'aide de `tinker`
```php
Comment::factory()->count(100)->create();
```
Dans notre vue `articles/show.blade.php`, on ajoute le code pour afficher les commentaires :
```blade
@foreach($article->comments as $comment)
    <p><strong>{{ $comment->user->name }}</strong> a réagi :</p>
    <p>{{ $comment->comment }}</p>
    <p><small>{{ $comment->created_at->diffForHumans() }}</small></p>
@endforeach
```
Naviguez sur la page, et ouvrez la `debugbar` sur l'onglet `queries`.  
Analysez les requêtes et constatez que l'on a autant de requêtes que de commentaires, ainsi qu'une requête pour l'auteur de l'article, les requêtes pour les auteurs des commentaires et une requête pour l'article en question.  
Imaginez un blog avec des milliers de commentaires, avoir autant de requêtes n'est pas raisonnable.  
Ce problème s'appelle le '`n+1 problem`', voir [stack overflow](https://stackoverflow.com/questions/97197/what-is-the-n1-selects-problem-in-orm-object-relational-mapping).  
Ces requêtes se produisent dans la vue quand on va chercher les commentaires ainsi : `$article->comments as $comment`.

On modifie donc liens, route et articles pour effectuer un minimum de requêtes :
`ArticlesController`
On crée une sous-requête avec `eloquent`, la méthode `with()` acceptant un tableau avec un callback :
```php
public function show($id)
{
    $article = Article::with('user')->with(['comments' => function ($query) {
        $query->with('user');
    }])->findOrFail($id);

    return view('articles.show', compact('article'));
}
```

### Et si on expliquait une peu ce code 😅😅😅 

1. **Article::with('user') :**

  * `with('user')` : Cela indique que nous chargeons la relation `user` associée à l'article.
  * `user` est une relation définie dans le modèle `Article` qui associe chaque article à un utilisateur (l'auteur de l'article). Par défaut, cela chargera la colonne `user_id` dans la table articles pour trouver l'utilisateur correspondant dans la table `users`. Ce type de relation est défini par une méthode belongsTo dans le modèle `Article`.
  * En utilisant `with('user')`, vous effectuez ce qu'on appelle **l'éager loading** (chargement anticipé). Cela signifie que l'utilisateur lié à l'article sera récupéré en même temps que l'article, ce qui permet d'éviter des requêtes supplémentaires lorsque vous accédez à la relation `user` par la suite.          

2. **with(['comments' => function ($query) { $query->with('user'); }]) :**

  * `with('comments')` : Cela charge la relation `comments` associée à l'article. Un article peut avoir plusieurs commentaires, donc cette relation est définie dans le modèle `Article` par une méthode `hasMany`.
  * `function ($query) { $query->with('user'); }` : Ceci est une **fonction de closure** qui permet de spécifier des options supplémentaires lors du chargement des commentaires. Dans ce cas, la closure permet de charger, pour chaque commentaire, la relation `user` associée.
    * `$query->with('user')` : Ici, `with('user')` est utilisé pour charger également l'utilisateur qui a écrit chaque commentaire (un commentaire est associé à un utilisateur via une relation `belongsTo` dans le modèle `Comment`).

3. **findOrFail($id) :**

  * `findOrFail($id)` : Cette méthode récupère un article spécifique en fonction de son identifiant (`$id`). Si aucun article avec cet identifiant n'est trouvé, Laravel générera une exception `ModelNotFoundException`, ce qui entraînera une erreur 404 automatiquement.
  * Cela signifie que l'article avec l'ID passé en paramètre doit exister, sinon l'exécution du programme s'arrêtera avec une erreur.



`routes/web.php`
```php
Route::get('/articles/{id}', [ArticlesController::class, 'show']);
```
`partials/article` 
```blade
<a href="/article/{{ $article->id }}">
    <p>{{ $article->title }}</p>
</a>
```
Nous avons maintenant quatre requêtes. C'est déjà plus raisonnable.

#### Une dernière technique pour récupérer les relations automatiquement :
**`protected $appends`**  
Ajoutons cette propriété à notre modèle `Article` afin de charger l'auteur automatiquement.
`Article.php`
```php
protected $appends = [
    'author'
];
// !! Le nom de cette méthode n'est pas optionnel !!
// get 'author' attribute
// méthode obligatoire pour faire fonctionner notre $appends
public function getAuthorAttribute()
{
    return $this->user->name;
}
```
`ArticlesController.php`
```php
// dans la méthode show()
$article = Article::with(['comments' => function ($query) {
    $query->with('user');
}])->findOrFail($id);
```
Maintenant votre deuxième exercice.
