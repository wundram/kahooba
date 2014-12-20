kahooba
=======

#What is it?

The idea is to create a service consiting of a web app, and plugins for various SDLC tools, that can be used to track software versions in a holistic way.  

#What does it do?

The core concept is that kahooba will store and query version objects.  

Version objects track some basic information like version, title, notes, status, and relationships.  (we probably also want to track history...)  The objects will probably live in some kind of document based nosql database, and we might also use lucene to provide a many to many style index.  The objects in the database should also support labels, so we can alway apply an arbitary tag to a set of objects and retreive it.

The relationships are a list of relationships with other version objects and other objects.  The relationships might be things like prceeded by, followed by, depends on, is depended upon by, contains, is contained by,contributed to, etc..  The objects that versions have relationships with are other versions, commits, issues, documentation, and so on.  Many of these objects are links the external systems.   

In addition to these simple relationships, we need something I call complex relationships.  These can have expressions as the source and target.  For example an issue might be related to `10.*` rather than a specific `10.n.n.n` version.   This will also require some kind of expression syntax for a version.  Wildcards are good for all versions in a branch or level.  We also we to indicate things like latest version in a branch, chronologicly last version, versions with a label, versions chronalogically before or after, versions before or after in the tree (direct decendant, or ansestor), and booleans of these.  

The status of a version can be used to indicate where this version is.  It is probably created when it is built by the build server.  It can have status like built, in qa, in staging, failed, rejected, retired, etc..  One status at a time.  Inn addition we have composite status, this is status of a branch of a version tree.  For example, an exec might want to know the status of `10.2.*` branch, he/she doesn't care about minutea, they want to know the status on a large scale, relating to when the branch is going to ship.  (we need to figure out how statuses combine to form composite status, this is probably something users will want to customize)

We also need to have some way to have relationships to versions sets without instances.  For example before we have done any build on the `8.7.*` we might be taking notes, linking web pages, referening to it in JIRAs, an so on.  To support this, we would have version branch objects.  Like `1.*`, `1.1.*`, and so on.  More specific branch objects and version objects inherit the contents of the less specific ones.  So `1.1.1.1` and `1.1.1.*` logically inherit all the contents of `1.*`.

We need some way to scope these objects, as you might be working on more than one product.  Versions belong to a product, and products belong to a project (or should we call that category maybe..).  Products and projects are objects, and they can have labels and relationships.  Only some relationship types will be allowed for trans product relationships.

Queries are one of two ways we can look at the information in our database.   Queries would be in some kind of expression format.  Complex relationships will probably use the same query engine and expression syntax.  The other way we look at infiomation in our database is through a drill down, where we follow relationships from object to object.

The UI for the application should be simple.  We figure most people will want to have their issue tracker or build server be the primary interface, and mainly see our UI by following a link, or in a div in another app via a plugin.  The main UI will have an interface for creating and managing projects and products, creating and editing version objects, applying labels, making queries, and some kind of dashboard.  Permissions will probably be on a product or project level, and will have read, write, and admin (can also delete, move, change settings).

Plugins for other apps will have a few purposes.  First to create the relationships between versions and the objects the app manages (builds, issues, deployments, commits, and so on).  The second is to make those relationships visible in the UI of the app.  For example seeing the related versions and their status in a JIRA issue summary.  Finally we will want to have the status of the version updated when important status changes happen in the app.  For example when a version is deployed into production by puppet or Octopus, we would want the status to be updated to in prod.

#How does it work

The core platform for the service will probably .net framework for the server, using OWIN and Web API to create the service API.  The client will be HTML5 and javascript.  Also we will be making plugins for various third party SDLC tools like Atlassian, github, team city, jenkins, puppet, chef, and Microsoft TFS.

As much as possible we want to use existing third party libraries for boilerplate functionality like authentication, data storage, search, and so on.   (We can track the other components we use here: https://github.com/wundram/kahooba/wiki/Technologies-used-in-Kahooba)

The database should be some kind of object database dynamic object types, and arbritatary relationships.  It does need to be immutable or distributed or anything.  If we have to shard it, it probably makes sense to do it by product or category.  Links to external apps are just objects that contain a bit of metadata (name, description), a URI to reference the resource in the other system, and a URL for a web page to navigate to the other app in the browser and land on that object.  The database should be able to manage queries referencing properties in my documents (objects).  Also it should be able to index over these properties, not just have a single lookup key index.

For handling search, we probably also want to build a lucene index of all the version objects.  The lucene index probably should crawl a bit beyond our own data, pulling things like git commit comments, and issue text into the index, to aid in search for a version.

We can use the built in identity provider for web api to handle authentiction, but we also wanto to consider having SSO capability with Atlassian Crowd, and support oauth, or SAML or whatever to allow easy linking between our app and other apps like TFS for JIRA.

#Use cases

-Providing the build server with a next version to tag the build with.  
-Answering what version questions, what version was the feature added, what version was the JIRA fixed, what version(s) is in prod.
-Recording info like version status ate various levels.  
-Allow manager to sign off a version for the next level (staging, prod).
-Produce good release notes with information for multiple products
-Show dashboard of products showing current versions of each, and status of next 'release'
-Display product roadmap with version numbers
-Show version graph, and visualize the various active versions, their status and hotfixes.


#Questions/notes

What does it mean for a commit to be in a version.  We get the chance to mark it when it first appears in a build, but it is contained in every build after that.  Questions we want to answer with this are when (what version) was a commit introduced? and does this version use this code (commit).  

Should changes to status on one version affect others.  For example if a put a version to "in prod" should any other versions marked as in prod in the branch or product be changed to supplanted?  This is going to need to be customizable if we do it.
