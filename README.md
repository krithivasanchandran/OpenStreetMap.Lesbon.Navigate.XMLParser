###Open Street Map XML Data:  

The XML usually consists of Node, Way, Relation – 

###Node:
The node are points on earth that are represented by Reference ID, longitude , latitude parameters which are essential for the XML parsing . The other data is not essential.  Nodes are used to represent shops, bus stops, benches, and post boxes. A node without any tags will always be a sub-element of another element.

```
<node id="2256232444" lat="38.7325109" lon="-9.2087326" version="2" timestamp="2013-11-08T00:55:31Z" changeset="18774798" uid="380552" user="topolusitania"/>
<way id="23658520" version="10" timestamp="2014-11-27T01:14:11Z" changeset="27059043" uid="47097" user="Josef K">
<nd ref="218034780"/>
<nd ref="1135629318"/>
<nd ref="1128851607"/>
<nd ref="1128851677"/>
<tag k="access" v="delivery"/>
<tag k="highway" v="residential"/>
<tag k="lit" v="yes"/>
<tag k="name" v="Rua João das Regras"/>
<tag k="oneway" v="yes"/>
</way>
```
###Tags:
These tags describe the feature they are attached to, and can be any pair of strings up to a maximum of 255 characters in length (including spaces), with the only restriction that keys be unique inside one element. If there are no tags associated with a feature, most renderings of the data won't display that feature. The tags are key – value <k,v>  pair that defines the way with a lot of parameters  . Some of the most commonly used keys are name , transportation , mode , oneway , bicycle , footway , highway , housename , address house number ,address street , address post code. 
```
<tag k="name" v="Rua JoÃ£o das Regras"/>
<tag k="oneway" v="yes"/>
</way>

<tag k="access" v="delivery"/>
<tag k="highway" v="residential"/>
<tag k="lit" v="yes"/>
<tag k="name" v="Rua João das Regras"/>
<tag k="oneway" v="yes"/>
```

###Relation:
A relation is one of the core data elements that consists of one or more tags and also an ordered list of one or more nodes, ways and/or reference ids.  

```
<relation id="3323894" version="1" timestamp="2013-11-13T17:44:34Z" changeset="18878616" uid="380552" user="topolusitania">
<member type="way" ref="246015129" role="outer"/>
<member type="way" ref="246015128" role="inner"/>
<tag k="type" v="multipolygon"/>
</relation>
```
###LOCAL_CONNECT : 

Way Named Node contains child elements identified by referenceID. The relationship that exists between a single lane is [:LOCAL_CONNECT]. The relationship is as shown below: 
```
<way id="23658520" version="10" timestamp="2014-11-27T01:14:11Z" changeset="27059043" uid="47097" user="Josef K">
<nd ref="218034780"/>
<nd ref="1135629318"/>
<nd ref="1128851607"/>
<nd ref="1128851677"/>
<tag k="access" v="delivery"/>
<tag k="highway" v="residential"/>
<tag k="lit" v="yes"/>

<tag k="name" v="Rua João das Regras"/>

<tag k="name" v="Rua JoÃ£o das Regras"/>

<tag k="oneway" v="yes"/>
</way>
```
 
###EXTERNAL_CONNECT: 

Totally distinct different ways with one similar reference Ids are connected by [:EXTERNAL_CONNECT] relationship. Cypher.java identifies the similar referenceID between different lanes and connects them relationally. 
```
GraphDatabaseService db = new GraphDatabaseFactory().newEmbeddedDatabase("C:/Users/User/Documents/Neo4j/default.graphdb");
		ExecutionEngine engine = new ExecutionEngine( db );

result = engine.execute(" MATCH (n:way { refid :'" + str + "'}) RETURN DISTINCT  n.name AS name;");
				Iterator<String> column = result.columnAs( "name" );

//Returns the Distinct referenceId ad stores in the ArrayList. 	

buffer.append("MATCH (n:way),(lane:way) WHERE n.name = '"+source+"' AND lane.name = '"+destination+"' AND n.refid ='"+str+"' AND lane.refid= '"+str+"'");
buffer.append("CREATE UNIQUE (n)-[:EXTERNAL_CONNECT]-(lane);");
```
The StringBuffer contains the query where the external relationship happens. Interconnecting different lanes helps in reaching shortest path from source to destination. 

###Maps Design Using TileMill:

TileMill  is used for custom map design â€“ by uploading map data from various sources . Though TileMill takes a lot of time to render the output. The CSS is used to style the map. 

```
#lisbonportugal [highway='motorway'],
#lisbonportugal [highway='motorway_link']{
  line-width : 5;
  line-color : black;
  
  ::motorway-filler{
    line-width : 3;
    line-color : blue;
  }
```

The motorway link : 
cars , trunks  , bikes  used black as outer color and blue as inner line color. 	
```
#lisbonportugal [highway='primary'],
#lisbonportugal [highway='primary_link']{
   line-width : 3;
  line-color : black;
  
  ::trunk-filler{
    line-width : 2;
    line-color : red;
  }
  }
```

```
  #lisbonbuilding [building != '' ]{
   
    building-fill:green;
    building-fill-opacity:1;
    
   }
  
  #lisbonbuilding [highway = 'footway' ]{
   
      line-width:  1;
    line-color : purple;
   }
  
 
}

[zoom > 9]{
 #lisbonportugal [waterway = 'riverbank']{
  	  ::shadow { polygon-fill: #07f; }
  ::fill {
    // a white fill and overlay comp-op lighten the polygon-fill from ::shadow.
    polygon-fill: #fff;
    comp-op: soft-light;
    // blurring reveals the polygon fill from ::shadow around the edges of the water
    image-filters: agg-stack-blur(2,2);
    
  }
}
```
Buildings are shown in purple color. Zoom parameter helps in reloading the map with a different CSS which shows a detailed view of the streets and buildings. 

###2.2. Requirements and Assumptions

###2.2.1 Interfaces:

The application is a standalone – most of computation happens in the backend data sources. The user can provide the source and destination in the cypher query which returns the output result in the JSON format. 

###2.2.2 Functional Capabilities:

Functions of the application involves importing the data sources (XML) , JSON – parsed using DOM parser , JSON parser . The data is analyzed based on input schema , sorted , stratified , made calculations , deriving patterns and relationships between nodes using Cypher query and visualized using graphical libraries inbuilt in Neo4J. 

###2.2.3 Performance Levels:

Performance on the end system is satisfactory. The Performance of TileMill is slow and hence need to enhanced by reducing the layers.

###2.2.4 Data Structures:


On the fly XML data capture is used – Java 8 – List.Stream.Parallel() is used for parallel computation of list elements. HashMap is used for storing values of NamedNode elements. StringBuffer is mostly used for String computations has it is synchronized and prevents lazy writes. Neo4J uses highly optimized shortest path algorithms with O(nlogn) time.

On the fly XML data capture is used  Java 8  List.Stream.Parallel() is used for parallel computation of list elements. HashMap is used for storing values of NamedNode elements. StringBuffer is mostly used for String computations has it is synchronized and prevents lazy writes. Neo4J uses highly optimized shortest path algorithms with O(nlogn) time.

### Results and Analysis

The Results are compared between Neo4J database output and Google Maps . 


###Result and Comparison : 


Shortest Path: 
```
Source :  Calçada do Carmo , Lisbon , Portugal

Destination :  Rua de São Julião , Lisbon , Portugal

Query :  MATCH (martin:way { name:"Calçada do Carmo" }),(oliver:way { name:"Rua -de São Julião" }),  p = shortestPath((martin)-[*..13]-(oliver))
RETURN p;
```
```
Source :  CalÃ§ada do Carmo , Lisbon , Portugal

Destination :  Rua de SÃ£o JuliÃ£o , Lisbon , Portugal

Query :  MATCH (martin:way { name:"CalÃ§ada do Carmo" }),(oliver:way { name:"Rua -de SÃ£o JuliÃ£o" }),  p = shortestPath((martin)-[*..13]-(oliver))
RETURN p;
```
The Output of Neo4J is in JSON format is included in project Jar file with Name Analysis_Json1 for analysis. 

(ii)	Result and Comparison: 
```
Source : Calçada do Carmo , Lisbon , Portugal

Destination : Rua de São Julião , Lisbon , Portugal

Source Node : Calçada do Carmo 
Transit Node : Rua de São Nicolau, Rua de Santa Justa , Rua dos Fanqueiros , Rua de São Julião

Source : CalÃ§ada do Carmo , Lisbon , Portugal

Destination : Rua de SÃ£o JuliÃ£o , Lisbon , Portugal

Source Node : CalÃ§ada do Carmo 
Transit Node : Rua de SÃ£o Nicolau, Rua de Santa Justa , Rua dos Fanqueiros , Rua de SÃ£o JuliÃ£o

Destination Node : Rua da Madalena.

Query : 

MATCH (martin:way { name:"Rua dos Correeiros" }),(oliver:way { name:"Rua da Madalena" }), p = shortestPath((martin)-[*..10]->(oliver))
  RETURN p;

Returns 14 nodes, and 13 relationships. 
```

(iii)	Result and Comparison:
```
Source : Rua da Padaria, Lisboa, Portugal


Destination : Rua do Instituto Virgílio Machado, Lisboa, Portugal

Source Node : Rua da Padaria
Transit Node : Rua dos Bacalhoeiros, Rua dos Arameiros, Rua da Alfândega, Rua Instituto Virgilio Machado 

Destination : Rua do Instituto VirgÃ­lio Machado, Lisboa, Portugal

Source Node : Rua da Padaria
Transit Node : Rua dos Bacalhoeiros, Rua dos Arameiros, Rua da AlfÃ¢ndega, Rua Instituto Virgilio Machado 


Destination Node : Rua Instituto Virgilio Machado.

Query : 

MATCH (martin:way { name:"Rua Instituto Virgilio Machado"}), (oliver:way { name:"Rua da Padaria"}), p = shortestPath((martin)-[*]-(oliver)) RETURN p;

Output : Analysis_Json3.json
Returns 23 nodes, and 24 relationships. 
```

###4.0. Conclusion

My code delivers the solution for the requirement and is tested in Lisbon , Portugal with a radius of 100 miles . 

Future developments of the maps navigation would be turn – by – turn navigation for people by walk, by car, by bus or tram or other public transportation and OpenXC integration. 

Future developments of the maps navigation would be turn â€“ by â€“ turn navigation for people by walk, by car, by bus or tram or other public transportation and OpenXC integration. 

Manual tests were done and meets expected results. 


###References

StackOverflow.com 
Neo4J Manual – Cypher Query Language
Neo4J Manual â€“ Cypher Query Language
Open Street Maps Wiki
Java 8 Complete reference.

### Appendices

Refer Introduction.

###Services and Software Used


•	Eclipse – Juno 
•	Neo4J Database
•	XML , JSON
•	TileMill – Custom Map creater
•	QGIS – Vector Mapping
•	OpenXC (Fords Open Source Platform)
•	Open Data Portugal - http://www.dados.gov.pt/pt/inicio/inicio.aspx
•	Sublime Text Editor 
•	Open Street Maps – Wiki 

###For more details please visit : http://kchander.github.io/OpenStreetMap.Lesbon.Navigate.XMLParser/


