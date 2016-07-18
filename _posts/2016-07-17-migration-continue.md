---
layout: post
title: La migration continue
categories:
- blog
---

Lorsque j'ai rejoins Y-Proximité en tant que lead développeur, j'ai ressenti après avoir collecté de nombreux signaux décrit dans cette suite de billets, le besoin de lancer un processus de migration de notre application "CMS" orienté réseau, multi-tenant (1BDD = X clients) & marque blanche. Il héberge aujourd'hui environ 2,000 sites/landing page.

On peut lire de à certains endroits qu'[une refonte c'est le mal](http://www.joelonsoftware.com/articles/fog0000000069.html). J'ai décidé de raconter comment s'est passé notre migration (moins abbrasive sur le plan politique qu'une refonte) qui s'est déroulée sur 2 ans. Voici mon cahier de route de la migration vers "Yprox v2" qui couvre plusieurs sujets.

* I. [Migration: Les raisons]({% post_url 2016-07-16-les-raisons-dune-migration %})
* II. Migration: Le processus de migration continue
* III. [Migration: Le bilan]({% post_url 2016-07-18-migration-couts-et-gains %})

# La migration continue, pourquoi et comment

Le but de la migration continue est de mettre à disposition une nouvelle version de son application tout en gardant l'existante disponible, s'assurant une transition continue d'une version X vers une version Y.
Cela permet de réduire les risques de la migration, au lieu du gros bouton de "bascule" de la version Y vers la version X, on active la migration pour des fonctionnalité précises, laissant plus de place à l'amélioration basé sur le feedback des utilisateurs.
De plus, cela envoie des signaux rassurant à votre hierarchie: l'avancement de la migration est visible en "temps réel" (opposé au fait que votre hierarchie doive attendre 2 ans dans mon cas avant de voir "tourner" le produit en prod).

Sur le sujet je vous conseille l'excellente présentation de François Zaninotto sur la [Migration continue d'une application Symfony2](https://www.youtube.com/watch?v=CvPD9iG0w-E)

Dans notre cas, nous avons mis en place une migration continue **partielle**. Seuls nos utilisateurs internes ont bénéficié de cette migration continue. Elle n'a touché que le backoffice de notre application. La partie frontoffice est le fameux "bouton rouge" de mise en prod finale.
Le but était de migrer continuellement tout ce qui était possible (comprennez par "possible" le ratio avantage/temps).

## L'organisation des branches & de la production

La première étape fût la création d'une brache _develop-2_ qui allait contenir tous les changements.
Nous avons donc mis en place 2 URL différentes pour acceder en temps réel aux 2 backoffice. **La base de données est partagée entre ces 2 versions**.

## Statégies de migrations

1/ Statégie de **cohabitation**

Le module X est accessible sur v1 et sur v2.

2/ Statégie de **remplacement**

Le module X est migré sur v2, il contient des BC Breaks au niveau du modèle de données.
La nouvelle entité est alors "backportée" sur v1 (car la BDD est partagées entre les 2 versions). La migration SQL est jouée en prod et le module X n'est plus accessible sur v1

## Comment appliquer les patchs de votre ancienne version sur la nouvelle ?

Au début c'est assez facile a coup de rebase/merge, et puis en fonction de la fracture, il sera de plus en plus difficile de le faire, voir impossible.

**Scénario**: Vous patchez un bug présent sur l'ancienne version et qui forcément impacte la nouvelle (puisque la branche a été crée de v1).

**Résultat**: Très vite, cette opération de rebase (pour garder un historique propre devient un châtiment mental):

<blockquote class="twitter-tweet" data-lang="fr"><p lang="en" dir="ltr">Please kill me.... <a href="http://t.co/EvjbdAAeQe">pic.twitter.com/EvjbdAAeQe</a></p>&mdash; Tristan Bessoussa (@sf_tristanb) <a href="https://twitter.com/sf_tristanb/status/565523044404580352">11 février 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

(Pour vous, développeurs à tendance SM qui souhaite quand même s'engager dans cette voie, vous pouvez aussi essayer : [Git rerere](http://www.git-attitude.fr/2014/11/04/git-rerere/)

Et puis, ce gouffre de changement structurel (changement de namespace, réorganisation du code), ne permettait plus de faire une opération de rebase/merge depuis la branche de la v1.
Quelques mois plus tard, cette branche à définitivement rompu avec master. </3

Avec du recul, le point _"pain in the ass"_, c'est que dès que vous corrigez un bug sur v1, vous devez savoir si ça impacte v2, et patcher les méthodes qui n'ont parfois en commun entre les versions, que leur objectif métier.

## Déploiement continu

La branche `develop-2` à bénéficié d'un processus de déploiement continu (Travis qui va lancer un playbook Ansible au travers d'un serveur Tower). Nous sommes montés jusq'à **32 déploiements par jour** (Amazon n'à qu'à bien se tenir...)

Lire la suite : III. [Migration: Le bilan]({% post_url 2016-07-18-migration-couts-et-gains %})