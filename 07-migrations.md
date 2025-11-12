# Les migrations

Nous allons maintenant créer plusieurs fichiers qui vont nous servir à ajouter les tables nécessaires à notre base de données.
 
 Dans le dossier `database/migrations`, des fichiers sont déjà présents. Examinons le fichier `create_users`, plus particulièrement la méthode `up()` :
```php
public function up()
{
        Schema::create('users', function (Blueprint $table) {
            $table->id(); // Création de la clé primaire 'id' auto-incrémentée
            $table->string('name'); // Champ 'name', type string, pour le nom de l'utilisateur
            $table->string('email')->unique(); // Champ 'email', type string, avec contrainte d'unicité
            $table->timestamp('email_verified_at')->nullable(); // Champ 'email_verified_at', type timestamp, pour la vérification de l'email, nullable
            $table->string('password'); // Champ 'password', type string, pour stocker le mot de passe
            $table->rememberToken(); // Champ 'remember_token', utilisé pour le système de "remember me" lors de la connexion
            $table->timestamps(); // Champs 'created_at' et 'updated_at', de type timestamp, qui sont gérés automatiquement par Eloquent
        });

        Schema::create('password_reset_tokens', function (Blueprint $table) {
            $table->string('email')->primary(); // 'email' comme clé primaire
            $table->string('token'); // Champ 'token' pour stocker le token de réinitialisation
            $table->timestamp('created_at')->nullable(); // Champ 'created_at' de type timestamp, nullable, pour indiquer la date de création du token
        });

        Schema::create('sessions', function (Blueprint $table) {
            $table->string('id')->primary(); // Champ 'id' comme clé primaire
            $table->foreignId('user_id')->nullable()->index(); // Champ 'user_id' qui est une clé étrangère vers la table 'users'
            $table->string('ip_address', 45)->nullable(); // Champ 'ip_address' pour stocker l'adresse IP de l'utilisateur
            $table->text('user_agent')->nullable(); // Champ 'user_agent' pour stocker les informations sur le navigateur de l'utilisateur
            $table->longText('payload'); // Champ 'payload' pour stocker des données supplémentaires relatives à la session
            $table->integer('last_activity')->index(); // Champ 'last_activity' pour stocker le dernier moment d'activité de la session
        });
}
```
Cette méthode créera les tois tables à suivantes : 
* users, 
* password_reset_tokens
* sessions
  
1. users 

- `$table->id()` : Crée un champ id auto-incrémenté qui servira de clé primaire pour cette table.
- `$table->string('name')` : Crée un champ `name` de type string pour stocker le nom de l'utilisateur.
- `$table->string('email')->unique()` : Crée un champ `email` de type string. La contrainte `unique()` signifie que chaque email doit être unique dans la base de données, ce qui empêche les utilisateurs d'avoir le même email.
- `$table->timestamp('email_verified_at')->nullable()` : Crée un champ `email_verified_at` de type timestamp, qui sera utilisé pour enregistrer la date et l'heure de la vérification de l'email. Le mot-clé `nullable()` permet à ce champ d'être vide (c'est-à-dire que la vérification peut ne pas avoir été effectuée).
- `$table->string('password')` : Crée un champ `password` de type string pour stocker le mot de passe de l'utilisateur.
- `$table->rememberToken()` : Crée un champ `remember_token` qui est utilisé par Laravel pour gérer le mécanisme de "se souvenir de moi" lors de l'authentification. Cela permet à un utilisateur de rester connecté après avoir fermé son navigateur.
- `$table->timestamps()` : Crée les champs `created_at` et `updated_at` de type timestamp. Ces champs sont automatiquement gérés par Laravel pour suivre la date de création et la date de la dernière mise à jour de chaque enregistrement.

2. password_reset_tokens

- `$table->string('email')->primary()` : Le champ `email` est utilisé comme clé primaire pour cette table, ce qui signifie qu'un seul token peut être généré pour chaque adresse email. Cela empêche de générer plusieurs tokens pour le même utilisateur.
- `$table->string('token')` : Ce champ contient le **token de réinitialisation**, qui est généré lorsque l'utilisateur demande à réinitialiser son mot de passe. Il est envoyé par email à l'utilisateur et est utilisé pour vérifier la demande.
- `$table->timestamp('created_at')->nullable()` : Ce champ contient la date et l'heure de la création du token. Il est marqué comme `nullable()` pour permettre qu'il soit vide si nécessaire.

3. sessions

* `$table->string('id')->primary()` : Le champ `id` est utilisé comme clé primaire pour la session. Il représente généralement un identifiant unique pour la session de l'utilisateur.
* `$table->foreignId('user_id')->nullable()->index()` : Ce champ `user_id` est une clé étrangère vers la table users. Il permet de lier une session à un utilisateur spécifique. Le mot-clé `nullable()` permet à une session d'exister sans être liée à un utilisateur spécifique (par exemple, pour les utilisateurs non authentifiés). Le mot-clé `index()` crée un index sur ce champ pour accélérer les requêtes basées sur cet identifiant.
* `$table->string('ip_address', 45)->nullable()` : Ce champ stocke l'adresse IP de l'utilisateur. Le type `string(45)` permet de stocker des adresses IPv6. `nullable()` indique que ce champ n'est pas obligatoire.
* `$table->text('user_agent')->nullable()` : Ce champ stocke l'agent utilisateur, qui contient des informations sur le navigateur ou le dispositif utilisé par l'utilisateur.
* `$table->longText('payload')` : Ce champ est utilisé pour stocker des informations supplémentaires sur la session (par exemple, des données spécifiques à la session). Il est de type `longText` pour pouvoir contenir une grande quantité de données.
* `$table->integer('last_activity')->index()` : Ce champ stocke le moment de la dernière activité de la session sous forme d'entier (généralement un timestamp Unix). L'index permet de rendre plus rapide la recherche des sessions actives ou expirées.


#### Création de migrations
Entrez la commande suivante :
```bash
php artisan make:migration create_articles_table
```
La table s'appellera `articles` mais la convention laravel est de nommer le fichier 'create_articles_table'.  
**Il est important de respecter cette convention car Laravel se sert des noms de fichiers pour automatiser les tâches**.  
Un fichier est créé dans le dossier `database/migrations`, et le squelette est déjà en place, il ne reste qu'à décrire les champs qu'on veut.  

```php
public function up()
{
    Schema::create('articles', function (Blueprint $table) {
        $table->id(); // Crée la colonne 'id', clé primaire auto-incrémentée
        $table->string('title', 255); // Crée la colonne 'title', un champ de type string, avec une longueur maximale de 255 caractères
        $table->text('body'); // Crée la colonne 'body', un champ de type 'text' pour stocker des contenus plus longs
        $table->string('image'); // Crée la colonne 'image', un champ de type string pour stocker le chemin de l'image
        $table->foreignId('user_id')->nullable(); // Crée la colonne 'user_id', qui est une clé étrangère vers la table 'users', et permet que cette colonne soit vide (nullable)
        $table->timestamps(); // Crée les colonnes 'created_at' et 'updated_at' pour suivre les dates de création et de mise à jour des articles
        $table->foreign('user_id')->references('id')->on('users')->onDelete('SET NULL'); // Définit la relation étrangère entre 'user_id' et 'id' de la table 'users', et définit la règle de suppression à 'SET NULL'
    });
}
```
**Explication des différentes lignes :**
1. `$table->id();` :

 * Crée un champ `id` qui sert de clé primaire pour la table.
 * Ce champ est de type auto-incrémenté, ce qui signifie qu'à chaque nouvel enregistrement, une valeur unique sera automatiquement attribuée à ce champ.

2. `$table->string('title', 255);` :

 * Crée un champ `title` de type **string**, utilisé pour stocker le titre de l'article.
 * La longueur maximale de ce champ est de **255 caractères**. Cela signifie que le titre ne peut pas dépasser cette longueur.

3. `$table->text('body');` :

 * Crée un champ `body` de type **text**, utilisé pour stocker le contenu principal de l'article.
 * Le type `text` est plus adapté que **string** lorsque le contenu est plus long, comme un texte ou un paragraphe.

4. `$table->string('image');` :

 * Crée un champ `image` de type **string**, utilisé pour stocker le chemin du fichier image associé à l'article.
 * Ici, il s'agit d'une chaîne de caractères qui contient probablement l'URL ou le chemin relatif vers l'image (par exemple, `uploads/articles/mon-image.jpg`).

5. `$table->foreignId('user_id')->nullable();` :

 * Crée un champ `user_id` de type **foreignId**, qui est utilisé pour établir une relation avec la table `users`.
 * Ce champ fait référence à la clé primaire `id` de la table `users`, ce qui signifie que chaque article peut être associé à un utilisateur spécifique.
 * La méthode `nullable()` permet de rendre ce champ **facultatif**. Cela signifie qu'un article peut être créé sans être associé à un utilisateur spécifique.

6. `$table->timestamps();` :

 * Crée deux colonnes **timestamp** : `created_at` et `updated_at`.
 * Laravel gère automatiquement ces colonnes pour enregistrer respectivement la date de création et la dernière mise à jour de l'article. Ces colonnes sont automatiquement remplies par Eloquent, le système ORM de Laravel, lors de la création et de la mise à jour des articles.

7. `$table->foreign('user_id')->references('id')->on('users')->onDelete('SET NULL');` :

 * Cette ligne définit une relation étrangère entre le champ `user_id` de la table articles et le champ `id` de la table `users`. Cela établit une contrainte d'intégrité référentielle.
 * `references('id')` indique que la colonne `user_id` de la table `articles` fait référence à la colonne `id` de la table `users`.
 * `on('users')` spécifie que la référence est faite sur la table `users`.
 * `onDelete('SET NULL')` définit ce qui se passe lorsqu'un utilisateur est supprimé. Si l'utilisateur référencé est supprimé, alors la valeur de `user_id` dans les articles sera mise à **NULL**. Cela évite la suppression en **cascade** des articles associés à cet utilisateur et permet de garder l'article avec `user_id` vide (**NULL**).

**Récapitulatif de la table articles :**

* **Champs :**

 * `id` : Clé primaire auto-incrémentée.
 * `title` : Titre de l'article, limité à 255 caractères.
 * `body` : Contenu de l'article (long texte).
 * `image` : Chemin de l'image associée à l'article.
 * `user_id` : Clé étrangère qui fait référence à la table `users`. Peut être vide (`nullable`).
 * `created_at` : Timestamp de création.
 * `updated_at` : Timestamp de mise à jour.
 
* **Relation étrangère :**

 * `user_id` est une clé étrangère qui fait référence à la table `users` et se met à `NULL` si l'utilisateur associé est supprimé.
  
Cela permet de gérer des articles qui peuvent être associés à des utilisateurs tout en assurant l'intégrité des données (en cas de suppression de l'utilisateur, les articles restent présents avec `user_id` à `NULL`).

Créons une table pour les commentaires :
```bash
php artisan make:migration create_comments_table
```
```php
public function up()
{
    Schema::create('comments', function (Blueprint $table) {
        $table->id();  // Crée un champ 'id' qui sert de clé primaire auto-incrémentée
        $table->string('comment', 255);  // Crée un champ 'comment' de type string avec une longueur maximale de 255 caractères
        $table->foreignId('user_id');  // Crée un champ 'user_id' qui est une clé étrangère pour lier le commentaire à un utilisateur
        $table->foreignId('article_id');  // Crée un champ 'article_id' qui est une clé étrangère pour lier le commentaire à un article
        $table->timestamps();  // Crée les colonnes 'created_at' et 'updated_at' pour suivre la création et la mise à jour du commentaire
        $table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');  // Définie une relation étrangère entre 'user_id' et 'id' de la table 'users'. Si un utilisateur est supprimé, tous ses commentaires seront supprimés aussi ('cascade')
        $table->foreign('article_id')->references('id')->on('articles')->onDelete('cascade');  // Définie une relation étrangère entre 'article_id' et 'id' de la table 'articles'. Si un article est supprimé, tous les commentaires associés seront supprimés aussi ('cascade')
    });
}
```
Rien de compliqué ! On effacera les commentaires si l'utilisateur efface son compte ou si l'article est supprimé.  
On modifie maintenant le fichier `create_users` pour y ajouter un booléen `admin` dont le défaut sera `false` :
 ```php
public function up()
{
    Schema::create('users', function (Blueprint $table) {
        // ajout
        $table->boolean('admin')->default(false);
    });
}
```
On peut maintenant taper la commande suivante :
```bash
php artisan migrate
```
En considérant la table `user`, on constate que la mise à jour de notre base de données n'a concerné que les tables `comments` et `articles`. 

Pour prendre en compte les modifications effecutées sur notre table déjà existante, on va taper la commande suivante : 

```bash
php artisan migrate:fresh
```

En cas d'erreurs, corrigez-les dans vos fichiers de migrations, puis retapez la commande ci-dessus.

Voici ce que vous devez obtenir :

![image data](../img/lara-tables.PNG) 


_Note sur les clés étrangères :_
Une autre convention laravel est de nommer les clés étrangères `table_id`, où `table` est le nom de la table au singulier.  
Quand on verra les modèles et les relations `eloquent`, on verra que laravel recherche automatiquement les noms sous cette forme. 

#### Factory
Maintenant qu'on a notre BDD, on a besoin de données. Pour en créer, laravel est fourni avec une librairie appelée `faker` qui permet de générer des fausses données et de les enregistrer par la suite en BDD.
Dans le dossier `database/factory`, un exemple nous est fourni pour générer des utilisateurs, modifions le pour y ajouter le champ `admin` que nous avons créé plus tôt.  
Attention, ici nous utilisons la syntaxe `laravel 8`, ou la génération de données se fait avec des classes, avant laravel 8, c'était des fonctions, vous n'aurez aucun mal à vous y retrouver quand vous devrez gérer ce type de code.
```php
public function definition()
{
    return [
        'name' => fake()->name(),
        'email' => fake()->unique()->safeEmail(),
        'email_verified_at' => now(),
        'admin' => false,
        'password' => static::$password ??= Hash::make('password'),
        'remember_token' => Str::random(10),
    ];
}
```
Créons les factory pour nos autres tables :
```bash
php artisan make:factory ArticleFactory; php artisan make:factory CommentFactory
```

Les articles :
```php
public function definition()
{
    return [
        'title' => fake()->sentence(15), // on veut 15 mots
        'body' => fake()->paragraph(50), // on veut 50 phrases
        'user_id' => 1,
        'image' => fake()->image('public/images'),
    ];
}
```
On garde les choses simples pour l'instant et on code `user_id` en 'dur'. 

Maintenant, les commentaires :
```php
public function definition()
{
    return [
        'comment' => fake()->text(50),
        'user_id' => 2,
        'article_id' => 2,
    ];
}
```
Même chose pour les commentaires, on assigne le même `id` pour l'instant pour les `user_id` et `article_id`.
Ce code ne fonctionnera pas en l'état, on a besoin de voir d'autres notions avant.


#### Tinker
Dans le dossier `app/Models`, on a un modèle `User` présent dans le fichier User.php, on va pouvoir tester si tout fonctionne comme attendu.
Ouvrez ce fichier et modifiez la ligne suivante pour ajouter le champ `admin` :
```php
 protected $fillable = [
    'name', 'email', 'password', 'admin' // on ajoute admin
];
```
Tinker est un outil en ligne de commande qui permet d'interagir avec l'application directement dans le terminal.
Notez que si vous modifiez un fichier, il vous faudra redémarrer tinker pour que la modification soit prise en compte.
```bash
php artisan tinker
```
Ensuite :
```php
User::factory()->create();
```
Voici ce que vous devez obtenir, un utilisateur a été créé et enregistré en BDD.
Ceci est possible grâce au `trait` `HasFactory` utilisé par la classe `User`

![image data](../img/lara-user.PNG) 

C'est tout pour l'instant avec les migrations. Nous créerons plus tard les autres données. Auparavant, il nous faut voir les modèles.
