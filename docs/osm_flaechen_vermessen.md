### Parkplatz Flächen vermessen mit Openstreetmaps  

Openstreetmaps (OSM) hat viele offene Informationen zu geographischen Objekten. So zum Beispiel Flächen wie Parkplätzen. Das kann man schon auf der [Website](openstreetmap.org) sehen. Dazu muss man recht nah an ein Objekt (Parkplatz) ranzoomen. Dann rechtsklicken und auf "Objektanfrage" klicken. Dort sind nun mehrere benachbarte oder umschließende Objekte zu sehen.  
![image](./assets/img/objektabfrage.png)  

Es ist jeweils beschrieben um was für ein Objekt es sich handelt z.B. Parkplatz, Zufahrtsstraße oder Einzelhandelsgebäude. Dazu jeweils eine ID. Klickt man auf die ID, so erhält man zusätzliche Informationen. Unter [Tags](https://wiki.openstreetmap.org/wiki/Tags) sind einige Key-Value Paare gelistet. Die wichtigsten für diese Zwecke sind "amenity", was uns anzeigt, wenn es ="parking" ist, dass es sich um einen Parkplatz handelt, und "parking". Dies gibt die Parkplatzart an. Diese sind [hier](https://wiki.openstreetmap.org/wiki/DE:Key:parking?uselang=de) zu finden. Darüberhinaus sind noch die Knoten zu finden. Jede Fläche (bei OSM Way genannt) besteht aus mehreren Knoten. Diese geben die Eckpunkte der Fläche an.  
![image](./assets/img/osm_parkplatz.png)  
Die Eckpunkte sind wiederrum eigene OSM-Objekte. Das bringt uns den Vorteil, dass wir all das über die OSM-Api abfragen können. Wie das funktioniert wird im folgenden beschrieben.  

Zunächst benötigen wir das Modul [OSMPythonTools](https://pypi.org/project/OSMPythonTools/). Genauer gesagt die Klasse Api aus diesem Modul. 


```python
from OSMPythonTools.api import Api
api = Api()
```

Mit der Way ID, die wir aus der Weboberfläche von OSM auslesen können, kann man das entsprechende Objekt per Api abfragen. 


```python
way_id = 28366845
pplatz = api.query(f'way/{way_id}')
```

    [api] downloading data: way/28366845
    

Das Objekt pplatz enthält auch Informationen über alle Knoten. Diese erhält man mit


```python
pplatz.nodes()
```




    [<OSMPythonTools.api.ApiResult at 0x28a9ee2a110>,
     <OSMPythonTools.api.ApiResult at 0x28a9ee2a7d0>,
     <OSMPythonTools.api.ApiResult at 0x28a9ee2b0d0>,
     <OSMPythonTools.api.ApiResult at 0x28a9ee2ba50>,
     <OSMPythonTools.api.ApiResult at 0x28a9ee306d0>,
     <OSMPythonTools.api.ApiResult at 0x28a9ee31250>,
     <OSMPythonTools.api.ApiResult at 0x28a9ee31c10>,
     <OSMPythonTools.api.ApiResult at 0x28a9ee32750>,
     <OSMPythonTools.api.ApiResult at 0x28a9ee33250>,
     <OSMPythonTools.api.ApiResult at 0x28a9ee333d0>,
     <OSMPythonTools.api.ApiResult at 0x28a9ee54790>,
     <OSMPythonTools.api.ApiResult at 0x28a9ee55250>,
     <OSMPythonTools.api.ApiResult at 0x28a9ee55d10>,
     <OSMPythonTools.api.ApiResult at 0x28a9ed76f10>,
     <OSMPythonTools.api.ApiResult at 0x28a9ed747d0>,
     <OSMPythonTools.api.ApiResult at 0x28a9ed755d0>,
     <OSMPythonTools.api.ApiResult at 0x28a9ed764d0>,
     <OSMPythonTools.api.ApiResult at 0x28a9ed77390>,
     <OSMPythonTools.api.ApiResult at 0x28a9ee34990>,
     <OSMPythonTools.api.ApiResult at 0x28a9ee354d0>]



Jeder Node enthält die entsprechende Position als GPS-Koordinate:


```python
n1 = pplatz.nodes()[1]
print(f'Lat: ', n1.lat(), '\nLon: ', n1.lon())
```

    [api] downloading data: node/311555552
    

    Lat:  49.8668256 
    Lon:  8.6576262
    

Iteriert man darüber, so kann man alle GPS-Koordinaten in zwei Listen speichern:


```python
lats = []
lons = []
for n in pplatz.nodes():
    lats.append(n.lat())
    lons.append(n.lon())

print('Lats: ', lats)
print('Lons: ', lons)
```

    [api] downloading data: node/311555551
    [api] downloading data: node/311555554
    [api] downloading data: node/311555556
    [api] downloading data: node/2711480123
    [api] downloading data: node/2711480154
    [api] downloading data: node/311555558
    [api] downloading data: node/311555559
    [api] downloading data: node/311555560
    [api] downloading data: node/2711480130
    [api] downloading data: node/311555562
    [api] downloading data: node/2711480124
    [api] downloading data: node/311555563
    [api] downloading data: node/660518371
    [api] downloading data: node/2711480149
    [api] downloading data: node/1155488956
    [api] downloading data: node/1155488984
    [api] downloading data: node/1155489033
    [api] downloading data: node/1155488939
    

    Lats:  [49.866723, 49.8668256, 49.867099, 49.8671189, 49.8671522, 49.8674877, 49.8677068, 49.8676581, 49.8672221, 49.8672086, 49.8671979, 49.8671644, 49.8669892, 49.8669532, 49.8674594, 49.8674569, 49.8674032, 49.8669819, 49.8669967, 49.866723]
    Lons:  [8.6568879, 8.6576262, 8.657535, 8.6577965, 8.657788, 8.6577025, 8.6576467, 8.6571888, 8.6573276, 8.6572253, 8.6571448, 8.6571555, 8.6572112, 8.6569385, 8.6567774, 8.6567584, 8.6565453, 8.656692, 8.6568052, 8.6568879]
    

Um daraus nun eine Fläche berechnen zu können, habe ich [Stackoverflow bemüht](https://stackoverflow.com/questions/4681737/how-to-calculate-the-area-of-a-polygon-on-the-earths-surface-using-python). Zunächst müssen dei GPS-Koordinaten mit der [Sinusoidal-Projektion](https://de.wikipedia.org/wiki/Sinusoidal-Projektion) flächentreu umgewandelt werden. Danach wird die Fläche des Polygons berechnet. OSM liefert die Knoten in ordentlicher Reihenfolge. Wenn man diese verbindet, erhält man ein Polygon ohne Sprünge. Jedoch muss der letzte Knoten nochmal am Ende hinzugefügt werden, damit das Polygon geschlossen wird.


```python
from math import pi, cos, radians
def reproject(latitude, longitude):
    #https://stackoverflow.com/questions/4681737/how-to-calculate-the-area-of-a-polygon-on-the-earths-surface-using-python
    """Returns the x & y coordinates in meters using a sinusoidal projection"""

    earth_radius = 6371009 # in meters
    lat_dist = pi * earth_radius / 180.0

    y = [lat * lat_dist for lat in latitude]
    x = [long * lat_dist * cos(radians(lat)) 
                for lat, long in zip(latitude, longitude)]
    return x, y

def area_of_polygon(x, y):
    """Calculates the area of an arbitrary polygon given its verticies"""
    area = 0.0
    for i in range(-1, len(x)-1):
        area += x[i] * (y[i+1] - y[i-1])
    return abs(area) / 2.0

x, y = reproject(lats, lons)
x.append(x[0]) # Polygon schließen
y.append(y[0]) # Polygon schließen

flaeche = area_of_polygon(x,y)
print('Der Parkplatz ist ', flaeche, ' Quadratmeter groß.')
```

    Der Parkplatz ist  4705.30635510385  Quadratmeter groß.
    

Das ganze kann man natürlich in eine Funktion packen und damit alle in OSM hinterlegten Flächen bestimmen, so auch die vom zugehörigen Supermarkt:  


```python
def osm_flaeche_berechnen(way_id):
    pplatz = api.query(f'way/{way_id}')
    lats = []
    lons = []
    for n in pplatz.nodes():
        lats.append(n.lat())
        lons.append(n.lon())
    
    x, y = reproject(lats, lons)
    x.append(x[0]) 
    y.append(y[0]) 

    flaeche = area_of_polygon(x,y)
    return flaeche

print('Der Supermarkt ist', osm_flaeche_berechnen(51754350), 'Quadratmeter groß')
```

    [api] downloading data: way/51754350
    [api] downloading data: node/660518376
    [api] downloading data: node/1155488892
    [api] downloading data: node/1155488952
    [api] downloading data: node/660518377
    [api] downloading data: node/2344129057
    

    Der Supermarkt ist 2807.503959339112 Quadratmeter groß
    

Die Parkplatzfläche ist also etwa 1,68 mal so groß wie der Supermarkt an sich! Das eröffnet natürlich die Frage, wie das bei anderen Supermärkten in der Stadt aussieht. Leider habe ich noch keinen Weg gefunden, wie man die Verknüpfung von Parkplatz zu Supermarkt automatisiert abfragen kann. Deswegen habe ich mich zunächst auf alle Parkplätze einer Stadt konzentriert. Mehr dazu in Kürze


```python

```
