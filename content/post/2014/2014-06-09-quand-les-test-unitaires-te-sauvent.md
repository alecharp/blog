---
author:
  email: adrien.lecharpentier@gmail.com
  name: Adrien Lecharpentier
date: 2014-06-09T08:00:00Z
tags: dev integration-test
title: Quand les tests te sauvent les fesses
url: /2014/06/09/quand-les-test-unitaires-te-sauvent/
---

Depuis quelques temps, je me force à faire toute une panoplie de tests pour tous les développements que je fais. Qu'ils soient unitaires ou d'intégrations.

C'est une bonne pratique que je recommande depuis quelques temps déjà, il était donc grand temps que je le fasse...

>Faites ce que je dis, pas ce que je fais.

## Contexte

Depuis quelques semaines, quand j'ai du temps, je me consacre au développement encore secret mais bientôt relâché dans la nature de l'open-source.

Pour ce projet, j'utilise XStream pour écrire une liste de serveurs dans un fichier sous la forme :

{{< highlight xml >}}
<servers>
  <server name="toto">
  </server>
</servers>
{{< / highlight >}}

Pourquoi ce format : simple à comprendre et donc à éditer à la main.

## Réalisation

Rien de bien compliqué, on regarde les 2-3 libs connues sur la manipulation de XML dans le monde Java, on fait son choix, on lit la documentation et hop, on code.

### Première étape: Java --› XML

On crée une instance de XStream, on lui dit que les classes `Server.java` seront en fait appelées `server` dans l'xml. La liste, elle, sera appelée `servers`.

On veut que le nom du serveur soit un attribut du tag, soit on fait compliqué avec une implémentation de `Converter` soit on fait `xStream.useAttributeFor(Server.class, "name");`. Donc on fait le premier car un `Converter` c'est cool.

La transformation se fera alors avec un `xStream.toXml()`

{{< highlight java >}}
public class ServerListToXML {
    @Import private ServerService serverService;
    private final XStream xStream;
    public ServerListToXML() {
        xStream = new XStream();
        xStream.registerConverter(new ServerConverter());
        xStream.alias("server", Server.class);
        xStream.alias("servers", List.class);
    }
    public void save(OutputStream output) {
        List serverListing = serverService.getAll();
        xStream.toXML(serverListing, output);
    }
}
{{< / highlight >}}

Évidemment, on fait nos tests unitaires. Ça passe dans le vert, on `git add` / `git commit`.

### Deuxième étape: XML --› Java

Basiquement, le chemin aller a fonctionné, donc on fait `xStream.fromXml()` et tout se passe bien. Devrait en fait...

Oui car en fait, l'alias sur la classe `List`, c'est pas une super idée. Il vaut mieux utiliser `addImplicitCollection()`. Mais pour ça, il faut une classe concrète. Je décide donc de créer une classe pour contenir mes serveurs.

Je rajoute donc dans mon constructeur :

{{< highlight java >}}
...
    public ServerListToXML() {
        xStream = new XStream();
        xStream.registerConverter(new ServerConverter());
        xStream.alias("server", Server.class);
        xStream.alias("servers", ServerListing.class);

        xStream.addImplicitCollection(ServerListing.class, "list");
    }
    ...
    public void load(InputStream input) {
        ServerListing listing = (ServerListing) xStream.fromXML(input);
        listing.getList().stream().forEach(serverService::addServer);
    }
}
{{< / highlight >}}

Du coup, là, tout fonctionne parfaitement. Théoriquement. Donc on met en place des tests unitaires. Ça fonctionne, pour 1 serveur. Mais pas pour 2.

Donc on passse en Debug, on se prend la tête. Et puis, on se dit : "Qu'est-ce que j'ai codé qui pourrait faire que cela ne fonctionne pas ?". On retourne la question dans tous les sens puisque le code écrit ne casse pas le fonctionnement de la sauvegarde (test à l'appui) et là on se souvient que dans notre `ServerConverter` il y a aussi une méthode `unmarshal`. On revient dessus pour finalement voir un `moveUp()` en trop après le `while(reader.hasMoreChildren()) {..}`.

## Conclusion

Faire des tests m'a montré que ce que j'ai écrit fonctionne, que je peux ajouter de nouvelles fonctionnalités sans avoir peur et surtout que je peux faire du refactoring dessus. 
