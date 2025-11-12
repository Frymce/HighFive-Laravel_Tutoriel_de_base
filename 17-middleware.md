# Middleware guest et auth

Un middleware est une classe qui va se situer entre une route et un contrôleur, qui va intercepter la requête et effectuer des vérifications.


Commençons par deux middlewares très utilisés et qui nous sont fournis.
Nous écrirons un middleware plus tard.
 
 Commençons par protéger les routes qu'on ne veut pas rendre accessible à tous le monde.
 Pour l'instant on cache les liens avec les directives blade, mais les routes sont toujours accessibles.
 
 On veut donc interdire l'accès aux routes suivantes :
 `profile`, `create/article`, `/edit/article`, `/edit/article`, `/register`, `/login` et `/logout` 
```php
Route::get('/articles/create', [ArticlesController::class, 'create'])->middleware('auth');
Route::post('/articles/create', [ArticlesController::class, 'store'])->middleware('auth');

Route::get('/article/{article}/edit', [ArticlesController::class, 'edit'])->middleware('auth');
Route::patch('/article/{article}/edit', [ArticlesController::class, 'update'])->middleware('auth');
Route::delete('article/{article}/delete', [ArticlesController::class, 'delete'])->middleware('auth');

// Auth
Route::get('/register', [RegisterController::class, 'index'])->name('register')->middleware('guest');
Route::post('/register', [RegisterController::class, 'create'])->middleware('guest');
Route::get('/login', [SessionsController::class, 'index'])->name('login')->middleware('guest');
Route::post('/login', [SessionsController::class, 'authenticate'])->middleware('guest');
Route::post('/logout', [SessionsController::class, 'logout'])->name('logout')->middleware('auth');
// profile
Route::get('/profile', [UserController::class, 'index'])->name('profile')->middleware('auth');
```

**Redirections Automatiques**
Avec la configuration ci-dessus :
  * Les utilisateurs non authentifiés (`guest`) seront redirigés vers la page de connexion lorsqu'ils essaieront d'accéder aux routes protégées.
  * Les utilisateurs authentifiés qui essaient d'accéder aux routes `/register` ou `/login` seront redirigés vers la page d'accueil.

> Vous pouvez appliquer des middlewares directement dans un contrôleur. Pour cela, utilisez l'interface `HasMiddleware`. 
Voici un exemple avec `ArticlesController` :

```php
use Illuminate\Routing\Controllers\HasMiddleware; 
use Illuminate\Routing\Controllers\Middleware;

class ArticlesController extends Controller implements HasMiddleware
{

   public static function middleware(): array
    {
        // Déclarer ici les middlewares pour ce contrôleur
        return [
            new Middleware('auth', except: ['index', 'show']),
        ];
    }
}
```
En procédant ainsi, on bloque l'accès à toutes les méthodes sauf celles qui nous permettent de lire les articles. On peut se passer de la définition des middleware sur nos routes dans le `web.php`.
