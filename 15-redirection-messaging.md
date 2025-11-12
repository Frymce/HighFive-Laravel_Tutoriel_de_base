# Redirection et messages de session

Laravel offre des méthodes pour effectuer des redirections et envoyer des messages d'information à l'utilisateur.  

Commençons par les redirections. Nous allons modifier un contrôleur pour effectuer une redirection après la création d'un article et envoyer un message de confirmation.  

Nous apporterons également les modifications nécessaires au formulaire pour afficher des erreurs s'il y en a.  
Ce sera à vous de faire les redirections et les messages pour les méthodes restantes.

En premier, le formulaire.  Pour afficher les erreurs éventuelles, on ajoute simplement des directives `@error` en dessous des champs du formulaire avec en argument, le `name` de l'input concerné :
```blade
<input type="text" name="title"  value="{{ old('title',  isset($article->title) ? $article->title : null) }}">
@error('title')
    <div>{{ $message }}</div>
@enderror
<textarea name="body" id="" cols="30" rows="10">{{ old('body',  isset($article->body) ? $article->body : null) }}</textarea>
@error('body')
    <div>{{ $message }}</div>
@enderror
<input type="file" name="image">
@error('image')
    <div>{{ $message }}</div>
@enderror
<button type="submit">Enregistrer</button>
```
Essayez maintenant d'envoyer le formulaire en omettant un champ ou envoyez un fichier autre qu'une image, vous verrez les erreurs s'afficher automatiquement.
Il est possible de customiser ces messages, **ce sera à vous de parcourir la documentation**.

Voyons maintenant la redirection avec message après la création d'un article :
Il suffit d'ajouter une ligne au contrôleur  
`ArticlesController@create`
```php
Article::create($validatedData);
return redirect('/articles')->with(['success_message' => 'L\'article a été créé avec succès !']);
```
Cette méthode se lit tout simplement et ne devrait pas nécessiter d'explications, si ce n'est que la méthode `with` va envoyer un message de session `success_message`. Vous pouvez choisir le nom que vous voulez pour ce message.

Pour afficher ce message, on crée un dossier `messages` dans le dossier `view` et on met une vue `allMessages.blade.php`.  
Voici le contenu de ce fichier, il nous permet de gérer les différents types de messages que l'on pourrait devoir envoyer :  
```blade
@if (session()->has('success_message'))
    <div class="alert alert-success">
        <strong>{{ session()->get('success_message') }}</strong>
    </div>
@endif
@if (session()->has('warning_message'))
    <div class="alert alert-warning">
        <strong>{{ session()->get('warning_message') }}</strong>
    </div>
@endif
@if (session()->has('error_message'))
    <div class="alert alert-danger">
        <strong>{{ session()->get('error_message') }}</strong>
    </div>
@endif
```
 Ensuite, on inclue ce fichier dans la vue principale `layouts/master.blade.php` :
 ```blade
@include('messages.allMessages')
```
Le message s'affichera automatiquement le cas échéant.

Plusieurs options sont disponibles pour les redirections. Pour en savoir plus,  **la documentation ou les recherches sur le web seront vos alliés.**
