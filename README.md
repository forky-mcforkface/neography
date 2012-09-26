[![Build Status](https://secure.travis-ci.org/maxdemarzi/neography.png?branch=master)](http://travis-ci.org/maxdemarzi/neography)
[![Code Climate](https://codeclimate.com/badge.png)](https://codeclimate.com/github/maxdemarzi/neography)

## Welcome to Neography 

Neography is a thin Ruby wrapper to the Neo4j Rest API, for more information:
* [Getting Started with Neo4j Server](http://neo4j.org/community/)
* [Neo4j Rest API Reference](http://docs.neo4j.org/chunked/milestone/rest-api.html)

If you want to the full power of Neo4j, you will want to use JRuby and the excellent Neo4j.rb gem at https://github.com/andreasronge/neo4j by Andreas Ronge

Complement to Neography are the:

* [Neo4j Active Record Adapter](https://github.com/yournextleap/activerecord-neo4j-adapter) by Nikhil Lanjewar
* [Neology](https://github.com/lordkada/neology) by Carlo Alberto Degli Atti
* [Neoid](https://github.com/elado/neoid) by Elad Ossadon

An alternative is the Architect4r Gem at https://github.com/namxam/architect4r by Maximilian Schulz

### Neography in the Wild

* [Vouched](http://getvouched.com)
* [Neovigator](http://neovigator.herokuapp.com) fork it at https://github.com/maxdemarzi/neovigator
* [Neoflix](http://neoflix.herokuapp.com) fork it at  https://github.com/maxdemarzi/neoflix

### Getting started with Neography

* [Getting Started with Ruby and Neo4j](http://maxdemarzi.com/2012/01/04/getting-started-with-ruby-and-neo4j/)
* [Graph visualization with Neo4j](http://maxdemarzi.com/2012/01/11/graph-visualization-and-neo4j/)
* [Neo4j on Heroku](http://maxdemarzi.com/2012/01/13/neo4j-on-heroku-part-one/)


### Installation

```sh
  gem install 'neography'
```

After that, in your ruby script:

```ruby
require 'rubygems'
require 'neography'
```

in order to access the functionality.


### Dependencies

For use:

```
  os
  rake
  json
  httparty
```

For development:

```
  rspec
  net-http-spy
```


#### Rails

Just add gem 'neography' to your Gemfile and run bundle install.

The following tasks will be available to you:

```sh
rake neo4j:install                           # Install Neo4j to the neo4j directory under your project
rake neo4j:install[community,1.6.M03]        # Install Neo4j Community edition, version 1.6.M03
rake neo4j:install[advanced,1.5]             # Install Neo4j Advanced edition, version 1.5
rake neo4j:install[enterprise,1.5]           # Install Neo4j Enterprise edition, version 1.5
rake neo4j:start                             # Start Neo4j
rake neo4j:stop                              # Stop Neo4j
rake neo4j:restart                           # Restart Neo4j
rake neo4j:reset_yes_i_am_sure               # Wipe your Neo4j Database
```

Windows users will need to run in a command prompt with Administrative Privileges
in order to install Neo4j as a Service.

If you are not using Rails, then add:

```ruby
require 'neography/tasks'
```

to your Rakefile to have access to these tasks.

`rake neo4j:install` requires `wget` to be installed.  It will download and install Neo4j into a neo4j directory in your project regardless of what version you choose.


### Documentation

Configure Neography as follows:

```ruby
# these are the default values:
Neography.configure do |config|
  config.protocol       = "http://"
  config.server         = "localhost"
  config.port           = 7474
  config.directory      = ""  # prefix this path with '/' 
  config.cypher_path    = "/cypher"
  config.gremlin_path   = "/ext/GremlinPlugin/graphdb/execute_script"
  config.log_file       = "neography.log"
  config.log_enabled    = false
  config.max_threads    = 20
  config.authentication = nil  # 'basic' or 'digest'
  config.username       = nil
  config.password       = nil
  config.parser         = {:parser => MultiJsonParser}
end
```

Then initialize as follows:

```ruby
@neo = Neography::Rest.new
```

Or you can override the configured values as follows:

```ruby
@neo = Neography::Rest.new({ :protocol       => 'http://',
                             :server         => 'localhost',
                             :port           => 7474,
                             :directory      => '',
                             :authentication => 'basic',
                             :username       => 'your username',
                             :password       => 'your password',
                             :log_file       => 'neography.log',
                             :log_enabled    => false,
                             :max_threads    => 20,
                             :cypher_path    => '/cypher',
                             :gremlin_path   => '/ext/GremlinPlugin/graphdb/execute_script' })
```

Quick initializer (assumes basic authorization if username is given):

```ruby
@neo = Neography::Rest.new("http://username:password@myserver.com:7474/mydirectory")
```

Usage:

```ruby
@neo = Neography::Rest.new                                 # Inialize using all default parameters

@neo.get_root                                              # Get the root node
@neo.create_node                                           # Create an empty node
@neo.create_node("age" => 31, "name" => "Max")             # Create a node with some properties
@neo.create_unique_node(index_name, key, unique_value,     # Create a unique node
                        {"age" => 31, "name" => "Max"})    # this needs an existing index

@neo.get_node(node2)                                       # Get a node and its properties
@neo.delete_node(node2)                                    # Delete an unrelated node
@neo.delete_node!(node2)                                   # Delete a node and all its relationships

@neo.reset_node_properties(node1, {"age" => 31})           # Reset a node's properties
@neo.set_node_properties(node1, {"weight" => 200})         # Set a node's properties
@neo.get_node_properties(node1)                            # Get just the node properties
@neo.get_node_properties(node1, ["weight","age"])          # Get some of the node properties
@neo.remove_node_properties(node1)                         # Remove all properties of a node
@neo.remove_node_properties(node1, "weight")               # Remove one property of a node
@neo.remove_node_properties(node1, ["weight","age"])       # Remove multiple properties of a node

@neo.create_relationship("friends", node1, node2)          # Create a relationship between node1 and node2
@neo.create_unique_relationship(index_name, key, value,    # Create a unique relationship between nodes
                        "friends", new_node1, new_node2)   # this needs an existing index

@neo.get_relationship(rel1)                                # Get a relationship
@neo.get_node_relationships(node1)                         # Get all relationships
@neo.get_node_relationships(node1, "in")                   # Get only incoming relationships
@neo.get_node_relationships(node1, "all", "enemies")       # Get all relationships of type enemies
@neo.get_node_relationships(node1, "in", "enemies")        # Get only incoming relationships of type enemies
@neo.delete_relationship(rel1)                             # Delete a relationship

@neo.reset_relationship_properties(rel1, {"age" => 31})    # Reset a relationship's properties
@neo.set_relationship_properties(rel1, {"weight" => 200})  # Set a relationship's properties
@neo.get_relationship_properties(rel1)                     # Get just the relationship properties
@neo.get_relationship_properties(rel1, ["since","met"])    # Get some of the relationship properties
@neo.remove_relationship_properties(rel1)                  # Remove all properties of a relationship
@neo.remove_relationship_properties(rel1, "since")         # Remove one property of a relationship
@neo.remove_relationship_properties(rel1, ["since","met"]) # Remove multiple properties of a relationship

@neo.list_node_indexes                                     # gives names and query templates for all defined indices
@neo.create_node_index(name, type, provider)               # creates an index, defaults are "exact" and "lucene"
@neo.create_node_auto_index(type, provider)                # creates an auto index, defaults are "exact" and "lucene"
@neo.add_node_to_index(index, key, value, node1)           # adds a node to the index with the given key/value pair
@neo.remove_node_from_index(index, key, value, node1)      # removes a node from the index with the given key/value pair
@neo.remove_node_from_index(index, key, node1)             # removes a node from the index with the given key
@neo.remove_node_from_index(index, node1)                  # removes a node from the index
@neo.get_node_index(index, key, value)                     # exact query of the node index with the given key/value pair
@neo.find_node_index(index, key, value)                    # advanced query of the node index with the given key/value pair
@neo.find_node_index(index, query)                         # advanced query of the node index with the given query
@neo.get_node_auto_index(key, value)                       # exact query of the node auto index with the given key/value pair
@neo.find_node_auto_index(query)                           # advanced query of the node auto index with the given query

@neo.list_relationship_indexes                             # gives names and query templates for relationship indices
@neo.create_relationship_index(name, "fulltext", provider) # creates a relationship index with "fulltext" option
@neo.create_relationship_auto_index("fulltext", provider)  # creates a relationship auto index with "fulltext" option
@neo.add_relationship_to_index(index, key, value, rel1)    # adds a relationship to the index with the given key/value pair
@neo.remove_relationship_from_index(index, key, value, rel1) # removes a relationship from the index with the given key/value pair
@neo.remove_relationship_from_index(index, key, rel1)      # removes a relationship from the index with the given key
@neo.remove_relationship_from_index(index, rel1)           # removes a relationship from the index
@neo.get_relationship_index(index, key, value)             # exact query of the relationship index with the given key/value pair
@neo.find_relationship_index(index, key, value)            # advanced query of the relationship index with the given key/value pair
@neo.find_relationship_index(index, query)                 # advanced query of the relationship index with the given query
@neo.get_relationship_auto_index(key, value)               # exact query of the relationship auto index with the given key/value pair
@neo.find_relationship_auto_index(query)                   # advanced query of the relationship auto index with the given query

@neo.get_node_auto_index_status                            # true or false depending on node auto index setting
@neo.get_relationship_auto_index_status                    # true or false depending on relationship auto index setting
@neo.set_node_auto_index_status(true)                      # set the node auto index setting to true
@neo.set_relationship_auto_index_status(false)             # set the relationship auto index setting to false
@neo.get_node_auto_index_properties                        # array of currently auto indexed node properties
@neo.get_relationship_auto_index_properties                # array of currently auto indexed relationship properties
@neo.add_node_auto_index_property(property)                # add to auto indexed node properties
@neo.remove_node_auto_index_property(property)             # remove from auto indexed node properties
@neo.add_relationship_auto_index_property(property)        # add to auto indexed relationship properties
@neo.remove_relationship_auto_index_property(property)     # remove from auto indexed relationship properties

@neo.execute_script("g.v(0)")                              # sends a Groovy script (through the Gremlin plugin)
@neo.execute_script("g.v(id)", {:id => 3})                 # sends a parameterized Groovy script (optimized for repeated calls)
@neo.execute_query("start n=node(0) return n")             # sends a Cypher query (through the Cypher plugin)
@neo.execute_query("start n=node({id}) return n", {:id => 3}) # sends a parameterized Cypher query (optimized for repeated calls)

@neo.get_path(node1, node2, relationships, depth=4, algorithm="shortestPath") # finds the shortest path between two nodes
@neo.get_paths(node1, node2, relationships, depth=3, algorithm="allPaths")    # finds all paths between two nodes
@neo.get_shortest_weighted_path(node1, node2, relationships,   # find the shortest path between two nodes
                                weight_attr='weight', depth=2, # accounting for weight in the relationships
                                algorithm='dijkstra')          # using 'weight' as the attribute

nodes = @neo.traverse(node1,                                              # the node where the traversal starts
                      "nodes",                                            # return_type "nodes", "relationships" or "paths"
                      {"order" => "breadth first",                        # "breadth first" or "depth first" traversal order
                       "uniqueness" => "node global",                     # See Uniqueness in API documentation for options.
                       "relationships" => [{"type"=> "roommates",         # A hash containg a description of the traversal
                                            "direction" => "all"},        # two relationships.
                                           {"type"=> "friends",           #
                                            "direction" => "out"}],       #
                       "prune evaluator" => {"language" => "javascript",  # A prune evaluator (when to stop traversing)
                                             "body" => "position.endNode().getProperty('age') < 21;"},
                       "return filter" => {"language" => "builtin",       # "all" or "all but start node"
                                           "name" => "all"},
                       "depth" => 4})

# "depth" is a short-hand way of specifying a prune evaluator which prunes after a certain depth.
# If not specified a depth of 1 is used and if a "prune evaluator" is specified instead of a depth, no depth limit is set.

@neo.batch [:get_node, node1], [:get_node, node2]                        # Gets two nodes in a batch
@neo.batch [:create_node, {"name" => "Max"}],
           [:create_node, {"name" => "Marc"}]                            # Creates two nodes in a batch
@neo.batch [:set_node_property, node1, {"name" => "Tom"}],
           [:set_node_property, node2, {"name" => "Jerry"}]              # Sets the property of two nodes
@neo.batch [:create_unique_node, index_name, key, value,
             {"age" => 33, "name" => "Max"}]                             # Creates a unique node
@neo.batch [:get_node_relationships, node1, "out",
           [:get_node_relationships, node2, "out"]                       # Get node relationships in a batch
@neo.batch [:get_relationship, rel1],
           [:get_relationship, rel2]                                     # Gets two relationships in a batch
@neo.batch [:create_relationship, "friends",
             node1, node2, {:since => "high school"}],
           [:create_relationship, "friends",
             node1, node3, {:since => "college"}]                        # Creates two relationships in a batch
@neo.batch [:create_unique_relationship, index_name,
             key, value, "friends", node1, node2]                        # Creates a unique relationship
@neo.batch [:get_node_index, index_name, key, value]                     # Get node index
@neo.batch [:get_relationship_index, index_name, key, value]             # Get relationship index

@neo.batch [:create_node, {"name" => "Max"}],
           [:create_node, {"name" => "Marc"}],                           # Creates two nodes and index them
           [:add_node_to_index, "test_node_index", key, value, "{0}"],
           [:add_node_to_index, "test_node_index", key, value, "{1}"],
           [:create_relationship, "friends",                             # and create a relationship for those
             "{0}", "{1}", {:since => "college"}],                       # newly created nodes
           [:add_relationship_to_index,
             "test_relationship_index", key, value, "{4}"]               # and index the new relationship

@neo.batch *[[:create_node, {"name" => "Max"}],
             [:create_node, {"name" => "Marc"}]]                         # Use the Splat (*) with Arrays of Arrays
```

See http://docs.neo4j.org/chunked/milestone/rest-api-batch-ops.html for Neo4j Batch operations documentation.


Please see the specs for more examples.


Experimental:

```ruby
nodes = @neo.create_nodes(5)                                              # Create 5 empty nodes
nodes = @neo.create_nodes_threaded(5)                                     # Create 5 empty nodes using threads
nodes = @neo.create_node_nodes([{"age" => 31, "name" => "Max"},
                                {"age" => 24, "name" => "Alex"})          # Create two nodes with properties
nodes = @neo.create_node_nodes_threaded([{"age" => 31, "name" => "Max"},
                                         {"age" => 24, "name" => "Alex"}) # Create two nodes with properties threaded
nodes = @neo.get_nodes([17,86,397,33])                                    # Get four nodes by their id

one_set_nodes = @neo.create_nodes(3)
another_node = @neo.create_node("age" => 31, "name" => "Max")
nodes = @neo.get_nodes([one_set_nodes, another_node])                     # Get four nodes
```

### Phase 2

Trying to mimic the Neo4j.rb API.

Now we are returning full objects.  The properties of the node or relationship can be accessed directly (node.name).
The Neo4j ID is available by using node.neo_id .

```ruby
@neo2 = Neography::Rest.new({:server => '192.168.10.1'})

Neography::Node.create                                               # Create an empty node
Neography::Node.create("age" => 31, "name" => "Max")                 # Create a node with some properties
Neography::Node.create({"age" => 31, "name" => "Max"}, @neo2)        # Create a node on the server defined in @neo2

Neography::Node.load(5)                                              # Get a node and its properties by id
Neography::Node.load(existing_node)                                  # Get a node and its properties by Node
Neography::Node.load("http://localhost:7474/db/data/node/2")         # Get a node and its properties by String

Neography::Node.load(5, @neo2)                                       # Get a node on the server defined in @neo2

n1 = Node.create
n1.del                                                               # Deletes the node
n1.exist?                                                            # returns true/false if node exists in Neo4j

n1 = Node.create("age" => 31, "name" => "Max")
n1[:age] #returns 31                                                 # Get a node property using [:key]
n1.name  #returns "Max"                                              # Get a node property as a method
n1[:age] = 24                                                        # Set a node property using [:key] =
n1.name = "Alex"                                                     # Set a node property as a method
n1[:hair] = "black"                                                  # Add a node property using [:key] =
n1.weight = 190                                                      # Add a node property as a method
n1[:name] = nil                                                      # Delete a node property using [:key] = nil
n1.name = nil                                                        # Delete a node property by setting it to nil

n2 = Neography::Node.create
new_rel = Neography::Relationship.create(:family, n1, n2)            # Create a relationship from my_node to node2
new_rel.start_node                                                   # Get the start/from node of a relationship
new_rel.end_node                                                     # Get the end/to node of a relationship
new_rel.other_node(n2)                                               # Get the other node of a relationship
new_rel.attributes                                                   # Get the attributes of the relationship as an array

existing_rel = Neography::Relationship.load(12)                      # Get an existing relationship by id
existing_rel.del                                                     # Delete a relationship

Neography::Relationship.create(:friends, n1, n2)
n1.outgoing(:friends) << n2                                          # Create outgoing relationship
n1.incoming(:friends) << n2                                          # Create incoming relationship
n1.both(:friends) << n2                                              # Create both relationships

n1.outgoing                                                          # Get nodes related by outgoing relationships
n1.incoming                                                          # Get nodes related by incoming relationships
n1.both                                                              # Get nodes related by any relationships

n1.outgoing(:friends)                                                # Get nodes related by outgoing friends relationship
n1.incoming(:friends)                                                # Get nodes related by incoming friends relationship
n1.both(:friends)                                                    # Get nodes related by friends relationship

n1.outgoing(:friends).incoming(:enemies)                             # Get nodes related by one of multiple relationships
n1.outgoing(:friends).depth(2)                                       # Get nodes related by friends and friends of friends
n1.outgoing(:friends).depth(:all)                                    # Get nodes related by friends until the end of the graph
n1.outgoing(:friends).depth(2).include_start_node                    # Get n1 and nodes related by friends and friends of friends

n1.outgoing(:friends).prune("position.endNode().getProperty('name') == 'Tom';")
n1.outgoing(:friends).filter("position.length() == 2;")

n1.rel?(:friends)                                                    # Has a friends relationship
n1.rel?(:outgoing, :friends)                                         # Has outgoing friends relationship
n1.rel?(:friends, :outgoing)                                         # same, just the other way
n1.rel?(:outgoing)                                                   # Has any outgoing relationships
n1.rel?(:both)                                                       # Has any relationships
n1.rel?(:all)                                                        # same as above
n1.rel?                                                              # same as above

n1.rels                                                              # Get node relationships
n1.rels(:friends)                                                    # Get friends relationships
n1.rels(:friends).outgoing                                           # Get outgoing friends relationships
n1.rels(:friends).incoming                                           # Get incoming friends relationships
n1.rels(:friends,:work)                                              # Get friends and work relationships
n1.rels(:friends,:work).outgoing                                     # Get outgoing friends and work relationships

n1.all_paths_to(n2).incoming(:friends).depth(4)                      # Gets all paths of a specified type
n1.all_simple_paths_to(n2).incoming(:friends).depth(4)               # for the relationships defined
n1.all_shortest_paths_to(n2).incoming(:friends).depth(4)             # at a maximum depth
n1.path_to(n2).incoming(:friends).depth(4)                           # Same as above, but just one path.
n1.simple_path_to(n2).incoming(:friends).depth(4)
n1.shortest_path_to(n2).incoming(:friends).depth(4)

n1.shortest_path_to(n2).incoming(:friends).depth(4).rels             # Gets just relationships in path
n1.shortest_path_to(n2).incoming(:friends).depth(4).nodes            # Gets just nodes in path
```

See Neo4j API for:
* [Order](http://components.neo4j.org/neo4j-examples/1.2.M04/apidocs/org/neo4j/graphdb/Traverser.Order.html)
* [Uniqueness](http://components.neo4j.org/neo4j-examples/1.2.M04/apidocs/org/neo4j/kernel/Uniqueness.html)
* [Prune Evaluator](http://components.neo4j.org/neo4j-examples/1.2.M04/apidocs/org/neo4j/graphdb/StopEvaluator.html)
* [Return Filter](http://components.neo4j.org/neo4j-examples/1.2.M04/apidocs/org/neo4j/graphdb/ReturnableEvaluator.html)

### Examples

A couple of examples borrowed from Matthew Deiters's Neo4jr-social:

*  [Facebook](https://github.com/maxdemarzi/neography/blob/master/examples/facebook.rb)
*  [Linked In](https://github.com/maxdemarzi/neography/blob/master/examples/linkedin.rb)

Phase 2 way of doing these:

*  [Facebook](https://github.com/maxdemarzi/neography/blob/master/examples/facebook_v2.rb)
*  [Linked In](https://github.com/maxdemarzi/neography/blob/master/examples/linkedin_v2.rb)

### Testing

To run testing locally you will need to have two instances of the server running. There is some
good advice on how to set up the a second instance on the
[neo4j site](http://docs.neo4j.org/chunked/stable/server-installation.html#_multiple_server_instances_on_one_machine).
Connect to the second instance in your testing environment, for example:

```ruby
if Rails.env.development?
  @neo  = Neography::Rest.new({:port => 7474})
elsif Rails.env.test?
  @neo  = Neography::Rest.new({:port => 7475})
end
```

Install the test-delete-db-extension plugin, as mentioned in the neo4j.org docs, if you want to use
the Rest clean_database method to empty your database between tests. In Rspec, for example,
put this in your spec_helper.rb:

```ruby
config.before(:each) do
  @neo.clean_database("yes_i_really_want_to_clean_the_database")
end
```

### To Do

* Batch functions
* Phase 2 Index functionality
* More Tests
* More Examples
* Mixins ?

### Contributing

[<img src="https://secure.travis-ci.org/maxdemarzi/neography.png" />](http://travis-ci.org/maxdemarzi/neography)

Please create a [new issue](https://github.com/maxdemarzi/neography/issues) if you run into any bugs.
Contribute patches via pull requests.

### Help

If you are just starting out, or need help send me an e-mail at maxdemarzi@gmail.com.
Check you my blog at http://maxdemarzi.com where I have more Neography examples.

### Licenses

* Neography - MIT, see the LICENSE file http://github.com/maxdemarzi/neography/tree/master/LICENSE.
* Lucene -  Apache, see http://lucene.apache.org/java/docs/features.html
* Neo4j - Dual free software/commercial license, see http://neo4j.org

