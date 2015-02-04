Open Street Map XML Data:  

The XML usually consists of Node, Way, Relation – 

Node – The node are points on earth that are represented by Reference ID, longitude , latitude parameters which are essential for the XML parsing . The other data is not essential.  Nodes are used to represent shops, bus stops, benches, and post boxes. A node without any tags will always be a sub-element of another element.

<node id="2256232444" lat="38.7325109" lon="-9.2087326" version="2" timestamp="2013-11-08T00:55:31Z" changeset="18774798" uid="380552" user="topolusitania"/>
Way – An ordered list of nodes is called a way. A way has a maximum of 2000 nodes to ensure that tools and users are not overwhelmed with very large structures that are difficult to manipulate. They are used for representing linear features like footpaths, roads, rail lines, and power lines.Areas do not have a specific data type, and are simply a kind of closed way where the first node is the same as the last node. They are used to represent building outlines, parks, and landuse. The way ranges from primary roads, Secondary roads , interstate highways , trunks , streets and residential roads . 

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

Tags – These tags describe the feature they are attached to, and can be any pair of strings up to a maximum of 255 characters in length (including spaces), with the only restriction that keys be unique inside one element. If there are no tags associated with a feature, most renderings of the data won't display that feature. The tags are key – value <k,v>  pair that defines the way with a lot of parameters  . Some of the most commonly used keys are name , transportation , mode , oneway , bicycle , footway , highway , housename , address house number ,address street , address post code. 

<tag k="access" v="delivery"/>
<tag k="highway" v="residential"/>
<tag k="lit" v="yes"/>
<tag k="name" v="Rua João das Regras"/>
<tag k="oneway" v="yes"/>


Relation – A relation is one of the core data elements that consists of one or more tags and also an ordered list of one or more nodes, ways and/or reference ids.  

<relation id="3323894" version="1" timestamp="2013-11-13T17:44:34Z" changeset="18878616" uid="380552" user="topolusitania">
<member type="way" ref="246015129" role="outer"/>
<member type="way" ref="246015128" role="inner"/>
<tag k="type" v="multipolygon"/>
</relation>

Design the DOM XML Parser : 

Document Object Model parser is provides APIs that let you create, modify, delete, and rearrange nodes. It is great to manipulate the pulled XML parser. The DOM constructs an in -memory tree of the XML nodes. Document Builder , Node, child Elements, NamedNodeMap are some of the commonly imported libraries.

Code snippet: 

DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
Document document = builder.parse(new File("C:/Users/KRITHIVASAN CHANDRAN/Desktop/eclipse/LisbonNavigationOSMParser/Lisboa.xml"));
		
Getting the Child Elements by Named Node Name – Way and Node – 

NodeList nodeList = document.getDocumentElement().getElementsByTagName("node");
		
NodeList way_Node_List = document.getDocumentElement()
				.getElementsByTagName("way");
 
JSON Parser Implementation: 

JSON is a lightweight data – interchange format. It is easy for humans to read and write. It is easy for machines to parse and generate. JSON is also a collection of name-value pairs – an ordered list of values. 


public class JSONParser {
public static void main(String[] args) throws IOException {
// Getting the tag data
Map<String,String> popularkeys = new HashMap<String,String>();
for(int i=0;i<=2;i++)
{
URL url = new URL("http://taginfo.openstreetmap.org/api/4/tags/popular?sortname=count_all&sortorder=desc&page="+i+"&rp=15&qtype=tag&format=json_pretty");
InputStream is = url.openStream();
JsonParserFactory factory=JsonParserFactory.getInstance();
com.json.parsers.JSONParser parser=factory.newJsonParser();
Map jsonData=parser.parseJson(is, "UTF-8");
List<Map> value = (List) jsonData.get("data");
for(Map dataset : value){
String keyset = (String) dataset.get("key");
if(!keyset.equalsIgnoreCase("source") && keyset != null){
String datavalue = (String) dataset.get("value");
popularkeys.put(keyset, datavalue);
}}}
System.out.println(popularkeys.entrySet());
}}

The popular tag key elements are to be parsed from the URL given above to perform hash table lookup with the generated tags from the XML document structure.
PopularKeys prints the key value pair of the entire tags database table.



Writing to Neo4J Database: 

synchronized (coordinatesWay) {
				neo.DBWrite(coordinatesWay,tagAttributesContainer);
			}

Synchronized block helps in mutual exclusion of atmost one thread 
Writing the contents to the Database . 

coordinatesWay – HashMap – Contains the Reference ID , Longitude and Latitude points 

tagAttributesContainer – Contains Key-Value pair of all Tags .

Neo 4J JDBC – ODBC Connection - 

	Connection con = DriverManager.getConnection("jdbc:neo4j://localhost:7474/");

PreparedStatement statement = con.prepareStatement(synchronizercontainer.toString());
statement.executeQuery();

Prepared Statement to execute the generated Cypher Query. If the query returns one or more values it is stored in Result Set and later produced. 

con.close();

To close the connection. 

Writing Cypher Query to write nodes and relationships to Neo4J graph database: 

Creating a Node using Create Clause: 

Create a single node : Create (n:way) 

Create a node with reference Id , longitude , latitude and tag contents. 

query = "CREATE ( n:way {refid :'"+coordinateWay.get(jk).getReferenceId() +"',"+ bufferTag.toString() +" Lat:'"+coordinateWay.get(jk).getLatitude() +"', Long:'"+coordinateWay.get(jk).getLongitude()+"'})";


bufferTag.toString() is a StringBuffer which contains the tag <key,value> elements . 
Relationships between nodes are named by: LOCAL_CONNECT. For a Way node the set of interconnected nodes are connected using MATCH and CREATE UNIQUE clause.

CREATE UNIQUE (lane)-[:LOCAL_CONNECT]->(laner);

Where lane - MATCH (lane:way) WHERE lane.refid = '"+ coordinateWay.get(jk).getReferenceId() +"' ; // "33360496"

Where laner - MATCH (laner:way) WHERE laner.refid = '"+coordinateWay.get(jk+1).getReferenceId()+"'; // 33360529

CREATE UNIQUE (33360496)-[:localway]->(33360529)
CREATE UNIQUE (33360529)-[:localway]->(33360530)
CREATE UNIQUE (33360530)-[:localway]->(33360504)
CREATE UNIQUE (33360504)-[:localway]->(33360505)
CREATE UNIQUE (33360505)-[:localway]->(1135629360)

Cypher Query Language: 

1.	MATCH N RETURN COUNT(*) – Return the total number of nodes.
2.	START n=node(*) MATCH (n)-[r] (m) RETURN n,r,m; - Return all the nodes with following outgoing directed relationship 
3.	MATCH (lane : way {name : ‘Rua Augusta’ })  (way) RETURN way.name; - Returns the name of nodes connected to ‘Rua Augusta’. 
4.	MATCH (lane : way {name : ‘Rua Augusta’ })  (way) RETURN way.name; - Returns the name of nodes incoming relationship to ‘Rua Augusta’. 
5.	MATCH (lane:Way { name:"Rua da Alfândega" }),(laner:Way { name:"Oliver Stone" }),  p = shortestPath((lane)-[*..15](laner)) RETURN p;  - Returns the shortest path from Source to Destination in 15 hops . 
6.	MATCH (lane:Way { name:"Rua da Alfândega" }),(laner:Way { name:"Oliver Stone" }),  p = allShortestPaths((lane)-[*..15](laner)) RETURN p; -  Returns all the shortest path from Source to Destination

Visualizing the Neo4J Graph with respect to Lisbon :

START n=node(*) MATCH (n)-[r]->(m) RETURN n,r,m LIMIT 250;

This cypher query returns all the outgoing relationship associated with the node and is limited to 250 nodes. The screenshot of the graph is attached below . 

 

Node Relationships: 


LOCAL_CONNECT : 

Way Named Node contains child elements identified by referenceID. The relationship that exists between a single lane is [:LOCAL_CONNECT]. The relationship is as shown below: 

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

 
EXTERNAL_CONNECT: 

Totally distinct different ways with one similar reference Ids are connected by [:EXTERNAL_CONNECT] relationship. Cypher.java identifies the similar referenceID between different lanes and connects them relationally. 

GraphDatabaseService db = new GraphDatabaseFactory().newEmbeddedDatabase("C:/Users/KRITHIVASAN CHANDRAN/Documents/Neo4j/default.graphdb");
		ExecutionEngine engine = new ExecutionEngine( db );

result = engine.execute(" MATCH (n:way { refid :'" + str + "'}) RETURN DISTINCT  n.name AS name;");
				Iterator<String> column = result.columnAs( "name" );

Returns the Distinct referenceId ad stores in the ArrayList. 	

buffer.append("MATCH (n:way),(lane:way) WHERE n.name = '"+source+"' AND lane.name = '"+destination+"' AND n.refid ='"+str+"' AND lane.refid= '"+str+"'");
								buffer.append("CREATE UNIQUE (n)-[:EXTERNAL_CONNECT]-(lane);");

The StringBuffer contains the query where the external relationship happens. Interconnecting different lanes helps in reaching shortest path from source to destination. 

Calculation of Distance Metrics from Source to Destination :

Distance metrics are yet to be implemented but only one step away . The code snippet below finds the distance between two nodes using longitude and latitude values. 

public Double getDistanceBetweenTwoPoints(Double latitude1, Double longitude1, Double latitude2, Double longitude2) {
		    final int RADIUS_EARTH = 6371;

		    double dLat = getRad(latitude2 - latitude1);
		    double dLong = getRad(longitude2 - longitude1);

		    double a = Math.sin(dLat / 2) * Math.sin(dLat / 2) + Math.cos(getRad(latitude1)) * Math.cos(getRad(latitude2)) * Math.sin(dLong / 2) * Math.sin(dLong / 2);
		    double c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
		    return (RADIUS_EARTH * c) * 1000;
		    }

		    private Double getRad(Double x) {
		    return x * Math.PI / 180;
		    }

The radius of the earth is assumed to be arbitrary value ; 6371.

Finding Shortest Path between Source to Destination - A* algorithm ( Internal Neo4J Graph Implementation)

Shortest Path: 

Source : Calçada do Carmo , Lisbon , Portugal

Destination : Rua de São Julião , Lisbon , Portugal

Query :  MATCH (martin:way { name:"Calçada do Carmo" }),(oliver:way { name:"Rua -de São Julião" }),  p = shortestPath((martin)-[*..13]-(oliver))
RETURN p;

ScreenShot:

 


Output : Analysis_Json1 – Included in project Folder   *****

All the Shortest Path: 

Determines all the shortest path from source to destination . 

Source : Calçada do Carmo , Lisbon , Portugal

Destination : Rua de São Julião , Lisbon , Portugal

Source Node : Calçada do Carmo 
Transit Node : Rua de São Nicolau, Rua de Santa Justa , Rua dos Fanqueiros , Rua de São Julião
Destination Node : Rua da Madalena.


Query : 

MATCH (martin:way { name:"Rua dos Correeiros" }),(oliver:way { name:"Rua da Madalena" }), p = shortestPath((martin)-[*..10]->(oliver))
  RETURN p;

Returns 14 nodes, and 13 relationships. 

OUTPUT: Analysis_Json2 – Included in project Folder *****

TileMill  is used for custom map design – by uploading map data from various sources . Though TileMill takes a lot of time to render the output. The CSS is used to style the map. 

#lisbonportugal [highway='motorway'],
#lisbonportugal [highway='motorway_link']{
  line-width : 5;
  line-color : black;
  
  ::motorway-filler{
    line-width : 3;
    line-color : blue;
  }


The motorway link : cars , trunks  , bikes  used black as outer color and blue as inner line color. 	

#lisbonportugal [highway='primary'],
#lisbonportugal [highway='primary_link']{
   line-width : 3;
  line-color : black;
  
  ::trunk-filler{
    line-width : 2;
    line-color : red;
  }
  }

The primary highways – I-20 , I-10 are shown in red color in the map .

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

Buildings are shown in purple color. Zoom parameter helps in reloading the map with a different CSS which shows a detailed view of the streets and buildings. Screenshot shows the zoomed level map :

2.2. Requirements and Assumptions

2.2.1 Interfaces:

The application is a standalone – most of computation happens in the backend data sources. The user can provide the source and destination in the cypher query which returns the output result in the JSON format. 

2.2.2 Functional Capabilities:

Functions of the application involves importing the data sources (XML) , JSON – parsed using DOM parser , JSON parser . The data is analyzed based on input schema , sorted , stratified , made calculations , deriving patterns and relationships between nodes using Cypher query and visualized using graphical libraries inbuilt in Neo4J. 

2.2.3 Performance Levels:

Performance on the end system is satisfactory. The Performance of TileMill is slow and hence need to enhanced by reducing the layers.

2.2.4 Data Structures:

On the fly XML data capture is used – Java 8 – List.Stream.Parallel() is used for parallel computation of list elements. HashMap is used for storing values of NamedNode elements. StringBuffer is mostly used for String computations has it is synchronized and prevents lazy writes. Neo4J uses highly optimized shortest path algorithms with O(nlogn) time.

2.2.5 Safety:

The application uses Open Source modules.

2.2.6 Reliability: 

The application is not thoroughly tested and hence cannot be used for real life navigation.

2.2.7 Constraints and Limitation: 

Hardware Limitations leads to software processing capabilities. Processing 6000 lines of XML code using DOM parser takes 20 minutes when executed in local. Executing in Shark Cluster is still more slower . For highly mapped data sets (Washington , Seattle) more processing power is required for parsing millions or billions of nodes . 

Improvements to be made to Shortest path problem by taking external parameters into consideration (Open XC ) during navigation . 

The performance tuning and addressing performance issues in whole of application is not part of the release.





3.0. Results and Analysis

The Results are compared between Neo4J database output and Google Maps . 


(i)	Result and Comparison : 


Shortest Path: 

Source :  Calçada do Carmo , Lisbon , Portugal

Destination :  Rua de São Julião , Lisbon , Portugal

Query :  MATCH (martin:way { name:"Calçada do Carmo" }),(oliver:way { name:"Rua -de São Julião" }),  p = shortestPath((martin)-[*..13]-(oliver))
RETURN p;

The Output of Neo4J is in JSON format is included in project Jar file with Name Analysis_Json1 for analysis. 

(ii)	Result and Comparison: 

Source : Calçada do Carmo , Lisbon , Portugal

Destination : Rua de São Julião , Lisbon , Portugal

Source Node : Calçada do Carmo 
Transit Node : Rua de São Nicolau, Rua de Santa Justa , Rua dos Fanqueiros , Rua de São Julião
Destination Node : Rua da Madalena.

Query : 

MATCH (martin:way { name:"Rua dos Correeiros" }),(oliver:way { name:"Rua da Madalena" }), p = shortestPath((martin)-[*..10]->(oliver))
  RETURN p;

Returns 14 nodes, and 13 relationships. 


(iii)	Result and Comparison:

Source : Rua da Padaria, Lisboa, Portugal

Destination : Rua do Instituto Virgílio Machado, Lisboa, Portugal

Source Node : Rua da Padaria
Transit Node : Rua dos Bacalhoeiros, Rua dos Arameiros, Rua da Alfândega, Rua Instituto Virgilio Machado 

Destination Node : Rua Instituto Virgilio Machado.

Query : 

MATCH (martin:way { name:"Rua Instituto Virgilio Machado"}), (oliver:way { name:"Rua da Padaria"}), p = shortestPath((martin)-[*]-(oliver)) RETURN p;

Output : Analysis_Json3.json


4.0. Conclusion

My code delivers the solution for the requirement and is tested in Lisbon , Portugal with a radius of 100 miles . 

Future developments of the maps navigation would be turn – by – turn navigation for people by walk, by car, by bus or tram or other public transportation and OpenXC integration. 

Manual tests were done and meets expected results. 




5.0. References

StackOverflow.com 
Neo4J Manual – Cypher Query Language
Open Street Maps Wiki
Java 8 Complete reference.

7.0. Appendices

Refer Introduction.

7.1. Services and Software Used

•	Eclipse – Juno 
•	Neo4J Database
•	XML , JSON
•	TileMill – Custom Map creater
•	QGIS – Vector Mapping
•	OpenXC (Fords Open Source Platform)
•	Open Data Portugal - http://www.dados.gov.pt/pt/inicio/inicio.aspx
•	Sublime Text Editor 
•	Open Street Maps – Wiki 

