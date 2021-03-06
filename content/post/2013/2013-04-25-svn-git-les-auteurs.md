---
author:
  email: adrien.lecharpentier@gmail.com
  name: Adrien Lecharpentier
date: 2013-04-25T09:17:47Z
tags: git svn
title: SVN-Git -- les auteurs
url: /2013/04/25/svn-git-les-auteurs/
---

Juste pour mon souvenir : Git c'est mieux que Subversion. Rapide, discret, efficace. Bref je ne veux plus faire de Subversion.

Malheureusement, la vie c'est dur et du Subversion il y en a partout. Si si je vous jure ! La solution ? __git-svn__.

{{< highlight PowerShell >}}
git svn clone https://svn-repo-url
{{< / highlight >}}

Pas aussi simple : il vous faut les auteurs (au moins).

{{< highlight PowerShell >}}
svn log --quiet https://svn-repo-url | awk '/^r/ {print $3}' | sort -u
{{< / highlight >}}

\* merci à [stackoverflow et Mike DeSimone](http://stackoverflow.com/a/2495010)

Vous prenez la sortie, vous la mettez dans un fichier, disons `svn-authors.list`. Vous devez ensuite l'éditer pour le mettre sous la forme suivante :

{{< highlight PowerShell >}}
svn-login = Firstname Name <email>
{{< / highlight >}}

puis vous faites votre `git-svn`

{{< highlight PowerShell >}}
git svn clone https://svn-repo-url --authors-file=svn-authors.list
{{< / highlight >}}

Si vous souhaitez prendre les tags et autres branches ?

{{< highlight PowerShell >}}
git svn clone https://svn-repo-url --authors-file=svn-authors.list --stdlayout
{{< / highlight >}}

ou

{{< highlight PowerShell >}}
git svn clone https://svn-repo-url --authors-file=svn-authors.list -T trunk -b branches -t tags
{{< / highlight >}}

bon par contre, les tags Subversion sont vus comme des branches sur Git. Pour infos, c'est pas super grave mais ça se corrige __très__ facilement.

{{< highlight PowerShell >}}
for i in `git branch -r | grep "svn/tags"`; do
	tagname=`echo $i | perl -p -e 's/svn\\/tags\\///sig;'`;
	git tag -a -m "`git log --pretty=format:%s $i^\!`" $tagname $i^ 1> /dev/null;
	git branch -r -d $i 1> /dev/null;
done
{{< / highlight >}}

## Astuce

Le fichier des auteurs Subversion sera connu dans la configuration de Git via sont chemin absolu. Celui-ci se trouve pour le moment à l'extérieur du repository Git. Histoire de le garder avec, déplacez-le dans le dossier `.git` du repository et modifiez la configuration du repository :

{{< highlight PowerShell >}}
cp svn-authors.list <repo>/.git/
vim <repo>/.git/config
{{< / highlight >}}

configuration:
{{< highlight text >}}
[svn]
  authors file = .git/svn-authors.list
{{< / highlight >}}

## Nota bene

Si vous allez faire vos premiers commits Subversion sur le repository en passant par Git, il vous faut rajouter une ligne dans le fichier `svn-authors.list` avec votre identifiant Subversion et Git que vous devez utiliser.

## Références

 - [stackoverflow et Mike DeSimone](http://stackoverflow.com/a/2495010)
 - [Olivier Bazoud -- Conversion en masse de projets SVN vers Git](http://blog.bazoud.com/articles/2012-07-11-conversion-en-masse-de-projets-svn-vers-git/index.html)
