---
layout: page
title: Authentifier les utilisateurs d'une application
---

Un utilisateur d'une application y est enregistré par un document _User_ portant un identifiant _userId_ permanent et immuable.

L'objectif du processus d'authentification est de s'assurer que la personne _physique_ qui tente d'accéder à son _user_ est bien habilité à le faire.

### Les authentifications _physiques_
Elles utilisent une caractéristique physique, typiquement une empreinte digitale, 
- côté _serveur_ elle est enregistré dans le document _user_, 
- côté application terminale elle est donnée par un lecteur d'empreinte.

Il est concevable d'enregistrer plusieurs empreintes pour un même _user_ autorisant ainsi plusieurs personnes physiques à être authentifiables.

**Limitations:**
- tous les appareils n'ont pas de lecteur d'empreinte.
- il faut avoir enregistré **physiquement** d'avance toutes les empreintes ayant droit d'accès: on ne peut pas ajouter à distance un droit d'accès à une personne dûment authentifiée dans la vraie vie.

### _userId / password_ versus _passphrase_
**Dans le mode _login / password_**, l'utilisateur connaît au moins un identifiant externe unique (_login_) du document _user_.
- la donnée du _login_ aboutit à **un** _user_ (ou aucun);
- la donnée par l'utilisateur d'un _password_ lui permet de se faire authentifier par comparaison avec celui ou ceux enregistrés dans le _user_.

**Limitations:**
- le **login** est souvent une donnée de la vie réelle: e-mail ou numéro de téléphone. Ceci compromet la confidentialité personnelle des personnes physiques. Mais ce peut être aussi un _pseudo_ libre (mais unique).
- par négligence le **password** est trop court et peut être cassé soit par force brute, soit par usage de dictionnaires des passwords usuels.

**Dans le mode _passphrase_** une phrase longue (unique sur le serveur) effectue à la fois l'identification du user et son authentification.
- la passphrase peut être changée (sous condition).
- sa longueur (32 signes au minimum par exemple) empêche les hackers de les retrouver par force brute et les _dictionnaires_ sont inopérants.

**Limitations:**
- **si la passphrase a été générée aléatoirement**, sa sécurité est maximale mais de facto reportée sur l'utilisateur qui sauf pour les hypermnésiques sont incapables de mémoriser une telle clé. L'utilisateur va écrire celle-ci sur un support quel qu'il soit comportant un risque de diffusion / vol involontaire.
- **si la passphrase a été fixée par l'utilisateur** elle peut être longue MAIS correspondre à quelque chose de **signifiant / mnémotechnique** pour lui: 

      les$feuilles$mortes$l_ours_et_le_singe_animaux_sages

Les références poétiques ne sont pas exceptionnlles et simples à se souvenir mais un algorithme ne peut les essayer toutes en variant les séparateurs.

Un doublon de _passphrase_ n'est pas à exclure quand elle est déclarée par un humain. Or tomber par hasard sue phrase déjà enregistrée ouvre automatiquement l'accès à un _user_ qui n'est pas le sien: il faut donc interdire les doublons sur _une partie de la phrase_, par exemple les 16 premiers catactères (ou une primère ligne d'une phrase à 2 lignes), ce qui incite à choisir une seconde moitié sans rapport avec la première.

> La limitation évidente est que saisir une telle phrase à chaque connexion est pénible.

### Les authentifications à double facteur (2FA)
On connaît son _user_ par un _login_: le concept 2FA vise à authentifier l'utilisateur qui a identifié son compte.

#### Distributeur de mots de passe à durée de vie limitée
L'utilisateur installe Une application 2FA et lui fait  enregistrer les couples _application / userId_ auxquels il est susceptible de se connecter.

Juste avant de se conecter, l'application 2FA est ouverte et lui affiche les _passwords_ utilisables pour chaque couple _application / userId_: ça laise moins d'une minute à l'utilisateur pour le saisir.

**Limitations**
- si on se fait voler son matériel, ou pire **UN des matériels** sur lesquels l'application 2FA à été installée (et configurée) et que le hacker peut accéder à tous les logins enregistrés.
- certaines applications 2FA limitent le risque en demandant de saisir un code PIN: ceci rallonge la procédure. Toutefois ce code PIN est stocké sur l'appareil, donc pas à l'abri d'un casse technique.

#### Envoi d'un code sur un autre appareil
Soit un mot de passe temporaire généré par le serveur, soit une URL à cliquer pour valider l'accès (par SMS ou e-mail).

La sécurité repose sur le fait que l'utilisateur est le **seul** à pouvoir lire ses SMS ou e-mails.

Dans ce cas le hacker ne pourra pas recevoir le code ou l'URL de validation.

**Limitations:**
- numéro de téléphone ou e-mail sont nécessaires: ceci compromet la confidentialité personnelle des personnes physiques, un _user_ étant toujours porteur d'une réfrence dans le vrai monde (ce qui exclut les applications volontairement anonymes).
- si le mobile de l'utilisateur est volé, le voleur voit passer les SMS ou a accès aux e-mails. Certes il doit se connecter (quoi que pour un SMS ?) mais il semble que ce n'est pas si complexe que cela (du moins pour les policiers).
- si le mobile est _perdu_, la procédure peut être complexe et a minima longue.

# Recherche d'une _moins mauvaise_ solution
### Solution de base
Le choix de la _passphrase_ longue à fixer par l'utilisateur coche les cases de la sécurité et de l'universalité: si les autres options de connexion _simplifiée_ échouent, il reste toujours à l'utilisateur la possibilité de citer sa _passphrase longue_.

### Options de login _rapides_
Elles font appel au principe d'une application 2FA qui s'exécute et délivre à l'application cherchant à se connecter la _passphrase_ complète. Cette application pouvant s'exécuter,
- sur le même appareil `dev1` que l'application demandant le login,
- sur un AUTRE appareil `dev2`.

Une application 2FA sur un appareil doit être configuré d'avance pour tous les logins pouvant lui être présentés afin de transmettre à l'application demanderesse la _passphrase_ souhaitée.

Pour que ce soit rapide l'application 2FA ne devrait pas demander elle-même un code PIN long:
- Cas 1: 2FA ne demande pas de code PIN.
  - si dev1 est volé / perdu, toute personne qui pouura s'y connecter accèdera à tout ce qu'elle veut.
- Cas 2: 2FA demande un code PIN court.
  - il faut que la seconde tentative infructueuse de saisie du code PIN réinitialise 2FA en lui faisant perdre tous les logins qu'elle a enregistrés.

### Cas à envisager
Soit devX l'appareil _externe_ d'un hacker.
- dev1 et dev2 sont les mêmes.
- dev2 est perdu / volé: bref l'utilisateur N'A PAS dev2 sous la main pour agir: une demande issue de devX peut-elle être acceptée par dev2 ?
- dev1 et dev2 ont été volés par un hacker: peut-il se connecter ?
- le vol de dev1 ou dev2 donne-t-il la possibilité au voleur d'avoir accès aux _passphrases_ des users, le cas échéant par des moyens logiciels évolués ?
- la possession d'un numéro de téléphone est-elle requise ou une facilité éventuelle ?

## Solution A



