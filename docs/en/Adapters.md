Summary of Adapters
===================

The adapter can be set with the following YML:
```
ShopSearch:
  adapter_class: ShopSearchSolr
```

ShopSearchSimple
----------------
This module is for quick, out of the box, development/testing usage.
It doesn't really do any full-text search, just uses the built-in
:PartialMatch filter of SilverStripe's DataObjects. It's likely to be
quite slow and not too smart, especially given a lot of products.

It is the default adapter and is does have the same features as the
others, though.

**Setup**
Use searchable_fields config/static on your model(s).

NOTE: Paging is not yet implemented


ShopSearchMysql
---------------
This module uses MySQL's fulltext searching.

NOTE: It's not required that you use SilverStripe's FullTextSearchable
extension along with this. It could work, but may also cause conflicts.
That extension is fine but you're limited to searching SiteTree and File.

**Setup**
If you're not using FullTextSearchable, set up a fulltext index like so:
```
class MyModel extends DataObject {
	// ...

	private static $indexes => array(
		'SearchFields' => array(
			'type' => 'fulltext',
			'name' => 'SearchFields',
			'value' => 'Title,Content',
		)
	);

	// ...
}
```
The adapter will look for the "SearchFields" index.

NOTE: Paging is not yet implemented


ShopSearchSolr
--------------
This adapter is the quickest and most feature-complete. Fulltext and filter
fields are set using yml configuration options on the ShopSearch class.
This is because there is additional config that may need to happen.

First, add something like the following to _config.php:

```
// Configure Solr
Solr::configure_server(array(
	'host' => 'localhost',
	'extraspath' => Director::baseFolder() . '/shop_search/solr/',
	'indexstore' => array(
		'mode' => 'file',
		'path' => BASE_PATH . '/.solr'
	)
));
```

An example `mysite/_config/search.yml` would be the following:

```
---
Name: appsearch
---
ShopSearch:
  adapter_class: ShopSearchSolr
  buyables_are_searchable: 0
  searchable:
    - Product
  facets:
    Category:
      Label: Category
      Type: checkbox
      Values: 'ShopSearch::get_category_hierarchy(0,"",2)'
    Price:
      Label: Price
      Type: range
      RangeMin: 0
      RangeMax: 2000
      LabelFormat: Currency
    PromoSavings:
      Label: Discount
      Type: range
      RangeMin: 0
      RangeMax: 50
      LabelFormat: Currency
  sort_options:
    'score desc': 'Relevance'
    'SiteTree_Title asc': 'Alphabetical'
  solr_fulltext_fields:
    - Title
    - Content
    - ShortContent
  solr_filter_fields:
    Price:
      field: VFI_Price
      type: Double
    PromoSavings:
      field: VFI_PromoSavings
      type: Double
    DirectCategory:
      field: AllCategoryIDs
      type: Int
      multiValued: 'true'
    Category:
      field: AllCategoryIDsRecursive
      type: Int
      multiValued: 'true'
```

Which would allow sorting and filtering by category and price among other things.
The above example depends on some other things being in place - additional methods
added to the Product model (in this case via extension) and some VirtualFieldIndexes
being set up. It's not a drop-in example. Once you've got fields set, following these
steps should get you up and running:

1. cd PROJECTROOT/fulltextsearch-localsolr
2. ./start.sh
3. cd PROJECTROOT
5. framework/sake dev/tasks/Solr_Configure
6. framework/sake dev/tasks/Solr_Reindex

Good luck.

