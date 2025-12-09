# ckanext-similar-datasets

*ckanext-similar-datasets* ist eine CKAN-Erweiterung und fügt eine Liste ähnlicher Datensätze in der Datensatz-Detailansicht hinzu.

*ckanext-similar-datasets* war ursprünglich Teil des größeren Plugins `ckanext-discovery` und kann jetzt als eigenständiges Plugin genutzt werden.

![Screenshot des similar_datasets Plugins](doc/similar_datasets.png)

Dieses Repo war ursprünglich nur in deutscher Sprache verfügbar. Übersetzungen sind willkommen.

## System-Voraussetzungen

Getestet wurde das Plugin mit CKAN 2.9.

Andere Versionen wurden nicht getestet. Feedback zur Funktionalität mit anderen Versionen ist herzlich willkommen.

## Installation

CKAN-Umgebung aktivieren:

```bash
. /usr/lib/ckan/default/bin/activate
```

Plugin installieren:

```bash
pip install -e git+https://github.com/ondics/ckanext-similar-datasets#egg=ckanext-similar-datasets
```

Eine bestimmte Version installieren:

```bash
pip install -e git+https://github.com/ondics/ckanext-similar-datasets@v0.1.1#egg=ckanext-similar-datasets
```

## Funktionsweise

Das Plugin nutzt das SOLR [More Like This](https://cwiki.apache.org/confluence/display/solr/MoreLikeThis) Feature. Solr muss dafür etwas erweitert werden.

Der [MoreLikeThisHandler](https://cwiki.apache.org/confluence/display/solr/MoreLikeThis#MoreLikeThis-ParametersfortheMoreLikeThisHandler) ist in `/etc/solr/conf/solrconfig.xml` einzurichten.

Den folgenden Codeblock direkt vor dem `</config>`-Tag am Ende der Datei einbauen:

```xml
<requestHandler name="/mlt" class="solr.MoreLikeThisHandler">
    <lst name="defaults">
        <int name="mlt.mintf">3</int>
        <int name="mlt.mindf">1</int>
        <int name="mlt.minwl">3</int>
    </lst>
</requestHandler>
```

Weitere Infos zum MoreLikeThisHandler für Konfigurationsdetails finden sich in der Solr-Dokumentation.

Zusätzlich muss [term vector storage](https://cwiki.apache.org/confluence/display/solr/Field+Type+Definitions+and+Properties#FieldTypeDefinitionsandProperties-FieldDefaultProperties) für das `text`-Feld in `/etc/solr/conf/schema.xml` aktiviert werden.

In dieser Zeile:

```xml
<field name="text" type="text" indexed="true" stored="false" multiValued="true" />
```

muss `termVectors="true"` ergänzt werden:

```xml
<field name="text" type="text" indexed="true" stored="false" multiValued="true" termVectors="true" />
```

Hinweis: term vectors vergrößern die Größe des Solr-Index erheblich.

Danach Solr neu starten:

```bash
sudo service jetty restart
```

Anschließend die Datensätze einmalig neu indexieren, damit die Term-Vektoren der bereits existierenden Datensätze aufgenommen werden (zukünftige Datensätze werden automatisch hinzugefügt):

```bash
. /usr/lib/ckan/default/bin/activate
ckan -c /etc/ckan/default/ckan.ini search-index rebuild
```

Zum Schluss `similar_datasets` zur Liste aktiver Plugins in der CKAN-Konfigurations-INI hinzufügen:

```ini
plugins = ... similar_datasets
```

und CKAN neu starten:

```bash
sudo service apache2 restart
```

oder:

```bash
sudo supervisorctl restart ckan-uwsgi
```

## Konfiguration

Das Plugin bietet eine zusätzliche Einstellung, die in der CKAN-Konfigurationsdatei hinzugefügt und angepasst werden kann:

```ini
# Die maximale Anzahl an ähnlichen Datensätzen, die gelistet werden können. Default ist 5.
ckanext.similar_datasets.max_num = 5
```

## Credits

Dank geht an die Repo-Ersteller:

- [CKAN](https://ckan.org)
- [SOLR](https://solr.apache.org/)
- [ckanext-discovery](https://github.com/stadt-karlsruhe/ckanext-discovery)

## Lizenz

Distributed unter der GNU Affero General Public License.

Details stehen in der Datei `LICENSE`.

## Autor

Copyright (C) 2021 Ondics GmbH  
https://ondics.de
