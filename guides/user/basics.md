---
layout: page
title: Basic Usage Scenarios
---

# Basic Usage Scenarios

## Instance Lists

A very common use case of OntoWiki is browsing instance data. That means display e.g. all instances of a RDFS-Class (when you click on a item in the Navigation module). Recently the code that generates these list has been refactored to enable developer- and user-configurable lists. Users can apply filters on a list (e.g. filter a list of foaf:Persons by their property values; e.g. all Persons that foaf:know a given Person). Users also can browse data very intuitive showing all values of a property as list (which is again filterable, browsable etc.). Developers can use the generic list to display various views on RDF-data, without having to think about boring sparql-result-to-html transformations. A list is generated by a managed query, which is configurable via a simple API. There are default templates to display the data and integration with existing modules makes reuse of code easier. This wikipage gives a overview on how to use that API and integrate it in your code. The user viewpoint is (or will be) described in the GettingStarted Guide.

### Architecture
This sections describes which parts of OntoWiki are involved in instance list creation. Its a overview; the actual API description follows in the next section.

The core component is the class [OntoWiki\_Model\_Instances](https://github.com/AKSW/OntoWiki/blob/develop/application/classes/OntoWiki/Model/Instances.php), whose instances represent instance lists. Instantiating it with a Store and a Graph-name results in a SPARQL-Query (to be specific: a Erfurt-Sparql-Query2 object) being created inside, that (after instantiation) selects all instances within that graph (so a "SELECT ?s WHERE {?s ?p ?o}"). Having such a object instantiated gives you the following functionality via its methods:
* getting the currently selected entities and their properties (execute the query and transform the result)
* configure which properties should be shown (e.g. for each entity its dc:title)
* configure filters (which entities should be shown)
* configure paging (limit and offset) 

Displaying such a instance list in a [Zend-View](http://framework.zend.com/manual/en/zend.view.introduction.html) is done via a [ActionHelper](http://framework.zend.com/manual/en/zend.controller.actionhelpers.html) called [OntoWiki\_Controller\_ActionHelper-List](https://github.com/AKSW/OntoWiki/blob/develop/application/classes/OntoWiki/Controller/ActionHelper/List.php). 
When you write a controller that wants to display a list, you instantiate a instances-object, get the action helper from the framework, and pass it the object. The ActionHelper then handles:
 * appending a customizable template that renders the instance list to a zend view
 * management of multiple lists in a user session

The User-Interface to these configurable lists is composed of two parts: first there are some Modules (the "show properties"-module, the "filter"-module, and the "tagging"-module). They can be added to the view and they host configuration forms. Second there is a [[Zend-Controller-Plugin|http://framework.zend.com/manual/de/zend.controller.plugins.html) called [[ListSetupHelper|https://github.com/AKSW/OntoWiki/blob/develop/application/classes/OntoWiki/Controller/Plugin/ListSetupHelper.php]]. It handles the configuration-requests by the modules and calls the corresponding methods on the instances-object. This means the configuration is done via HTTP-Request-Parameters containing JSON encoded configuration requests - send by the modules; received and processed by the ListSetupHelper. The nature of a Controller-Plugin is to be notified on each request, thus configurations on any list can implicitly be done within all controller action calls that want to use the list feature.

### APIs

The complete documentation for the classes is available  [[online|http://docs.ontowiki.net/fw/]]. Here I want to explain a minimal working examples of code, that can be used as a starting point for you.

    $listHelper = Zend_Controller_Action_HelperBroker::getStaticHelper('List');
    $listName = "mylist";
    if(!$listHelper->listExists($listName)){
      $list = new OntoWiki\_Model\_Instances($store, $this->_owApp->selectedModel, array());
      $list->addTypeFilter("http://xmlns.com/foaf/0.1/Person");
      $listHelper->addListPermanently($listName, $list, $this->view);
    } else {
      $list = $listHelper->getList($listName);
      $listHelper->addList($listName, $list, $this->view);
    }

This code can be pasted into a action of a controller. It uses the ActionHelper to manage the list in the user session and uses a instances-object in $list.

The Action Helper has four important methods:
 * addListPermanently(string $name, OntoWiki\_Model\_Instances $list, Zend\_View $view)
 it adds a list to a view and saves it for later reuse in the session
 * getList(string $name)
 it retrieves a previously permanently added list
 * addList(string $name, OntoWiki\_Model\_Instances $list, Zend_View $view)
 it adds a list to the view

the style of the list a default style that is pretty feature rich: it shows all classes the instances belongs to and all selected properties. furthermore the TitleHelper is used and each instance is a clickable link that refers to a detail view (which is also displayable inline via a expand button).

if you want to use a custom template, you can pass a additional parameter to the  "addList" and "addListPermanently" calls containing the name of a partial (without the ".phtml" ending). Within that partial you have access to the following view variables:
 * instances (containing the OntoWiki\_Model\_Instances object)
 * instanceData (cantaining all values: resource -> property -> value)
 * instanceInfo (containg info about all shown resources)
 * propertyInfo (containing info about all selected properties)
 * other (a variable that can be passed by adding another additional parameter to the "addList..." calls)
 * listName (the name of the list)
 * start (the offset)

within that custom template you may want to iterate over the instanceInfo array and get the property values from instanceData. the display of a instance could be outsourced to a additional template. the default templates have this separation and are called [[list_std_main|https://github.com/AKSW/OntoWiki/blob/develop/application/views/templates/partials/list_std_main.phtml?r=Feature-ExtensionUnification]] and [[list_std_element|https://github.com/AKSW/OntoWiki/blob/develop/application/views/templates/partials/list_std_element.phtml?r=Feature-ExtensionUnification]]. you can have a look there to get more insight.

to add the default modules ("show properties" and "filter") you can add the list-context: 

    $this->addModuleContext('main.window.list');

the instancs class is a pretty complex thing and slightly overloaded with methods. the most important are these that modify the list (retrieval is mostly done for your):
 * string addTypeFilter(string $classUri, string $id = null, array $options = array())
 $classUri contains obviously the URI of a class that all instances need to belong to, to be displayed. $options can contain a key "withChild" with a boolean value, indicating if instances of child-classes should be included. returns the id (can be used later to delete).
 * string addSearchFilter(string $str, string $id = null)
 adds a regex filter on the instance uris and titles (store dependant). returns the id (can be used later to delete).
 * string addTripleFilter(array $triples, string $id = null)
 add arbitrary triples to the query to filter (used by the navigation). returns the id (can be used later to delete).
 * removeFilter(string $id)
 remove a filter that was previously added
 * setLimit(int $limit)
 how many instances should be displayed per page
 * setOffset(int $offset)
 which page should be displayed
 * addShownProperty($propertyUri, $propertyName = null, $inverse = false, $datatype = null, $hidden = false)
 add a property that is shown for all instances.
  * propertyUri - the URI of the property
  * propertyName - the varName to be used for this property
  * inverse - set this true if the property doesnt have the instance as subject but as object
  * datatype - deprecated
  * hidden - set true if you dont want to the property to show up in the user interface
 * removeShownProperty(string key) 
 the key is composed of $propertyUri.'-'.$inverse 

### Usage
the list is currently used in 
 * the ResourceController (instancesAction - the default use case, list instance of a class)
 * the QueriesController (listqueriesAction - show saved queries)
 * the CommunityController (listAction - show recent comments)
 * the PatternManagerController (browseAction - show saved patterns)
 * the ExconfController (exploreRepoAction - show extensions in the repository - nice example of a remote query)

you can have a look there to sneak good code

### HTTP API
So how is Instance List GUI working; how is a click on a button like "next page" or "filter by" handeled technically. Of course by HTTP GET and POST, but not by a simple controller.
There is a Zend-Controller-Plugin named **ListSetupHelper** - that is registered as a plugin to _every_ other controller and listens to specific GET parameters. Thus the manipulation of the selected instances is possible in every controller e.g. the map controller filters the instances by location...
the **ListSetupHelper** reacts to these GET paramters by managing a **OntoWiki\_Model\_Instances** object, that is stored in the **Session**. It calls the corresponding methods (like addShownProperty(...)).
the *instancesAction* has only reading access and displays the result of the query that is handled inside the *OntoWiki\_Model\_Instances* object. Other Controllers e.g. the MapController or the SourceController can now get the currently selected resources (with all applied filters etc.) and do stuff with them (e.g. display them in other perspectives).

Before this reengineering, the query only was capable of selecting the instances of _one_ rdfs:class. Now the class-filter is just a specific filter. If there is no filter applied the query selects **all** resources in the model.

#### instance list config parameters
| **type** | **key** | **datatype** | **description** |
| GET  | class | URI (String) | loads instances from specific owl:class with subclass and transitive closure  (often used in ow system see classbrowser module) - _deprecated_ (should be done in the *instancesconfig*-param)|
| GET | search | String | searches subjects with the search string inside one of its literals (on various properties) _deprecated_ (should be done in the *instancesconfig*-param)|
| POST (GET?) | instancesconfig | json array with filter and show properties directives (see below)||
| GET/POST | savelist | String | maybe a parameter to overrride standard way of handling to configure individual persitency of list in ow system (eg. new, session, forget (which means do not store in session ) ) |
| GET/POST | limit | positive int | set limit for paging (0 means none) |
| GET/POST | offset | positive int | set offset for paging |
| GET/POST | init | * | initializes a new list - if there are other config options passed these are applied on the new list. if this parameter is omitted, the passed options are applied to the existing list as additional filters etc.|

the *instancesconfig* parameter is explained in in my thesis (Jonas Brekle: "Spezifikation und Implementierung einer SPARQL Query API für PHP sowie Integration in OntoWiki", 2010) and i will refer to an extract ([[here|http://filebin.ca/ygocsp/doc.pdf]])

The instancesconfig-Parameter contains a JSON-encoded datastructure, that can contain multiple commands, regarding filters and shown Properties.

It is an associative array with the keys „shownProperties“ and „filter“. Each of them is a array of config-objects.

#### shownProperties
The „shownProperties“-config-objects consist of a „uri“-field, an optional „label“-field (containing both strings), a „action“-field (containing either the keyword „add“ or „remove“) and a boolean „inverse“-field (indicating wether we want to add a inverse property or not).

Example:
    
{
        "shownProperties": [
            {
                "uri": "http://xmlns.com/foaf/0.1/name",
                "label": "name",
                "action": "add",
                "inverse": false 
            } 
        ]
    }


Short Table:
(top) object
  * shownProperties (array of objects)
    * every object has these keys:
      * uri (property uri)
      * label
      * action ("add", "remove")
      * inverse (true; false)

#### filter

The „filter“-field is more variable. There are various formats for different types of filter, that are declared with
the mode-Parameter.

##### mode = box
Parameter:
  * id
  * property
  * isInverse
  * propertyLabel
  * filter
  * value1
  * value2
  * valuetype
  * literaltype
  * hidden
  * negate

Depending of the filter parameter, the following query-parts are added:
  * filter = ’contains’: `?resourceUri <property> ?value. FILTER(REGEX("value1",?value)).`
  * filter = ’equals’: `?resourceUri <property> <value1>`
  * filter = ’bound’: `?resourceUri <property> ?value. FILTER(BOUND(?value))`
  * filter = ’larger’: ` ?resourceUri <property> ?value. FILTER(?value > value1)`

##### mode = search
Parameter:
  * searchText

##### mode = rdfsclass
Parameter:
  * rdfsclass

##### mode = query
Parameter:
  * query

The changes to the query from the „box“-mode are not a simple template, but depend of the combination of the isInverse-Parameter, the filter-Parameter, the Type of the Value and the negate-Parameter. The „filter“-Parameter indicates the type of the filter (as shown in the last column) and can for example filter by equality of a property-value to a given value (there is also „between“). The negate-Parameter negates the condition:
a „larger“ becomes „smaller“; a `REGEX(x,y)` becomes `!REGEX(x,y)` etc. The value type contains string-keywords „uri“, „literal“, „typed-literal“ (then literaltype is evaluated as a URI) or „language-tagged-literal“ (not implemented yet, but literaltyp will be evaluated as a language tag).