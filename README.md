# Plateforme de streaming

Contexte du projet
En tant que développeur passionné par le cinéma, vous avez toujours été fasciné par la magie du grand écran. Cette passion ne se limite pas seulement à regarder des films. Vous avez toujours été curieux de connaître les coulisses, d'étudier qui a joué dans tel film, qui l'a réalisé, et comment ces chefs-d'œuvre ont été créés. Vous trouvez aussi que les plateformes de streaming sont un formidable accès à un catalogue d'oeuvres de toute sorte à découvrir. Vous avez donc envie de créer, vous aussi, votre propre plateforme de streaming sur votre temps libre. Mais comme Rome ne s'est pas construite en un jour, vous voulez commencer par la mise en place d'un site web permettant de procéder à différentes opérations de recherches à propos de films, d'acteurs/actrices ou de réalisateurs. Sauf que. La partie site web n'est pas pour tout de suite 😃 Avant celà, vous avez besoin d'une base de données pour le stockage. Et donc de la concevoir et la mettre en place! A vous de jouer 🙂​

# Contraintes

Le noSQL (MongoDB...) n'est pas autorisé

Vous devez créer votre propre environnement Docker

Un trigger doit être mis en place, également appelé déclencheur

Seul l'administrateur de la BDD pourra ajouter, modifier ou supprimer des données.

Pour chaque entrée dans la base de données, il y aura la date de création et de modification.

# Deadline

4 jours.

# Modalités d'évaluation

Correction entre pairs.
Vos requêtes seront testées en local après l'importation de votre environnemnt docker.

# Mise en place

J'ai crée un container docker auquel j'ai relié une image ( postgres ) obtenu sur docker hub qui contient ma conception de ma base de données streaming-postgredb . Pour l'analyse et la rélexion de ma BDD je me suis aider de draw.io pour créer mon MCD , MLD. Pour remplir ma database j'ai utiliser mockaroo qui me fournissait des datas en lui passant mes informations.

# Requêtes SQL

Voici un jeu de requêtes minimal à fournir pour tester votre bdd :

- les titres et dates de sortie des films du plus récent au plus ancien

  SELECT title, releaseDate FROM movies ORDER BY releaseDate DESC;

- les noms, prénoms et âges des acteurs/actrices de plus de 30 ans dans l'ordre alphabétique

  SELECT lastName, firstName, DATE_PART('year', AGE(CURRENT_DATE, birthDate)) AS age FROM actors WHERE DATE_PART('year', AGE(CURRENT_DATE, birthDate)) > 30 ORDER BY lastName, firstName;

- la liste des acteurs/actrices principaux pour un film donné

````SQL
    SELECT actors.lastName, actors.firstName
    FROM actors
    INNER JOIN plays_in ON actors.Id_actors = plays_in.Id_actors
    INNER JOIN movies ON plays_in.Id_movies = movies.Id_movies
    WHERE movies.title = 'You Are God (Jestes Bogiem)' AND plays_in.role = 'main_actor';```
- la liste des films pour un acteur/actrice donné
  `SELECT movies.title FROM movies INNER JOIN plays_in ON movies.Id_movies = plays_in.Id_movies INNER JOIN actors ON plays_in.Id_actors = actors.Id_actors WHERE actors.lastName = 'Brenna';`
- ajouter un film
  `INSERT INTO movies (title, length, releaseDate) VALUES ('Pattaya', '01:37:00', '2016-02-24');`
- ajouter un acteur/actrice
  `INSERT INTO actors (lastName, firstName, birthDate) VALUES ('Cena', 'John', '1977-04-23');`
- modifier un film
  `UPDATE movies SET title = 'Nouveau titre', releaseDate = '2023-12-31' WHERE title = 'Boy Interrupted';`
- supprimer un acteur/actrice
  `DELETE FROM actors WHERE lastName = 'Nom de l'acteur';`
- afficher les 3 derniers acteurs/actrices ajouté(e)s
  `SELECT lastName, firstName, birthDate FROM actors ORDER BY Id_actors DESC LIMIT 3;`

Nous avons aussi besoin de manipulations avancées:

- Lister grâce à une procédure stockée les films d'un réalisateur donné en paramètre
  `BEGIN RETURN QUERY SELECT movies.title, movies.releaseDate FROM movies INNER JOIN directs ON movies.Id_movies = directs.Id_movies INNER JOIN directors ON directs.Id_directors = directors.Id_directors WHERE directors.lastName = directorName; END;`
  `SELECT * FROM ListMoviesByDirector('Tann');`

- Garder grâce à un trigger une trace de toutes les modifications apportées à la table des utilisateurs. Ainsi, une table d'archive conservera la date de la mise à jour, l'identifiant de l'utilisateur concerné, l'ancienne valeur ainsi que la nouvelle.

`CREATE OR REPLACE FUNCTION UserUpdateFunction() RETURNS TRIGGER AS $$ BEGIN INSERT INTO archives (updateDate, id_users, oldValue, newValue) VALUES (NOW(), NEW.Id_users,  JSON_BUILD_OBJECT('firstName', OLD.firstName, 'lastName', OLD.lastName, 'email', OLD.email), JSON_BUILD_OBJECT('firstName', NEW.firstName, 'lastName', NEW.lastName, 'email', NEW.email)); RETURN NEW; END; $$ LANGUAGE plpgsql; -- CREATE TRIGGER UserUpdateTrigger AFTER UPDATE ON users FOR EACH ROW EXECUTE FUNCTION UserUpdateFunction();`
````
