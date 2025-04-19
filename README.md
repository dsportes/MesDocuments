Documentation personnelle de Daniel Sportès.

Le site est formaté par Jekyll.

[Site public](https://dsportes.github.io/MesDocuments)

## Installation et problèmes rencontrés

Quand on lance la commande `bundle install` on récupère l'erreur:

    Installing wdm 0.1.1 with native extensions
    Gem::Ext::BuildError: ERROR: Failed to build gem native extension.

Il faut installer wdm:

    gem install wdm
    // Relever la version installée : ici 0.2.0

Puis il faut corriger le Gemfile:

    # Performance-booster for watching directories on Windows
    gem "wdm", "~> 0.2.0", :platforms => [:mingw, :x64_mingw, :mswin]

On peut relancer:

    bundle install
    bundle exec jekyll serve

### Paths

    gem environment

## Changement de skin
Dans Gemfile:

    # Pour utiliser le skin dark
    gem "jekyll-remote-theme", "~> 0.4.3"

Dans `_config.yml`:

    remote_theme: jekyll/minima
    
    minima:
      skin: dark

Puis:

    bundle install
    bundle exec jekyll serve

## Build et test de la build en local (Windows)

    bundle exec jekyll build
    xcopy /E _site ..\public\fr\
    cd ..
    npx http-server -p 8080

    URL: http://localhost:8080/fr

