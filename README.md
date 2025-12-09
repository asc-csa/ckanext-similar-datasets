# ckanext-similar-datasets

`ckanext-similar-datasets` is a CKAN extension that adds a list of similar datasets to the dataset detail page.

It was originally part of the larger `ckanext-discovery` plugin and can now be used as a standalone plugin.

![Screenshot of the similar_datasets plugin](doc/similar_datasets.png)

German version: see `README.DE.md`.

## System requirements

Tested with CKAN 2.9.

Other versions have not been tested. Feedback about compatibility with additional versions is welcome.

## Installation

Activate the CKAN virtual environment:

```bash
. /usr/lib/ckan/default/bin/activate
```

Install the plugin:

```bash
pip install -e git+https://github.com/ondics/ckanext-similar-datasets#egg=ckanext-similar-datasets
```

Install a specific version:

```bash
pip install -e git+https://github.com/ondics/ckanext-similar-datasets@v0.1.1#egg=ckanext-similar-datasets
```

## How it works

The plugin uses Solr's [More Like This](https://cwiki.apache.org/confluence/display/solr/MoreLikeThis) feature. Solr needs a small configuration change for this to work.

Add the [MoreLikeThisHandler](https://cwiki.apache.org/confluence/display/solr/MoreLikeThis#MoreLikeThis-ParametersfortheMoreLikeThisHandler) to `/etc/solr/conf/solrconfig.xml`.

Insert the block below right before the closing `</config>` tag near the end of the file:

```xml
<requestHandler name="/mlt" class="solr.MoreLikeThisHandler">
    <lst name="defaults">
        <int name="mlt.mintf">3</int>
        <int name="mlt.mindf">1</int>
        <int name="mlt.minwl">3</int>
    </lst>
</requestHandler>
```

See the Solr documentation for more configuration details about the MoreLikeThis handler.

Next, enable [term vector storage](https://cwiki.apache.org/confluence/display/solr/Field+Type+Definitions+and+Properties#FieldTypeDefinitionsandProperties-FieldDefaultProperties) for the `text` field in `/etc/solr/conf/schema.xml`.

Find this line:

```xml
<field name="text" type="text" indexed="true" stored="false" multiValued="true" />
```

Add `termVectors="true"` to it:

```xml
<field name="text" type="text" indexed="true" stored="false" multiValued="true" termVectors="true" />
```

Note: enabling term vectors significantly increases the size of the Solr index.

Then restart Solr:

```bash
sudo service jetty restart
```

Rebuild the search index so that term vectors for existing datasets are generated (new datasets will be added automatically):

```bash
. /usr/lib/ckan/default/bin/activate
ckan -c /etc/ckan/default/ckan.ini search-index rebuild
```

Finally, add `similar_datasets` to the list of active CKAN plugins in your configuration INI:

```ini
plugins = ... similar_datasets
```

and restart CKAN:

```bash
sudo service apache2 restart
```

or:

```bash
sudo supervisorctl restart ckan-uwsgi
```

## Configuration

The plugin exposes one configuration option that can be added to your CKAN configuration file:

```ini
# The maximum number of similar datasets to show. Default is 5.
ckanext.similar_datasets.max_num = 5
```

## Credits

Thanks to the original projects and authors:

- [CKAN](https://ckan.org)
- [SOLR](https://solr.apache.org/)
- [ckanext-discovery](https://github.com/stadt-karlsruhe/ckanext-discovery)

## License

Distributed under the GNU Affero General Public License.

See `LICENSE` for details.

## Author

Copyright (C) 2021 Ondics GmbH  
https://ondics.de
