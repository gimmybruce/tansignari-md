---
title: "Scaricare kmz in loop da un server arcgis"
linkTitle: "Scaricare kmz in loop da un server arcgis"
date: 2019-07-28
description: >
 Scaricare la viabilità storica della Regione Siciliana dal server ArcGIS.
tags:
  - CURL
  - GDAL/OGR
  - 7z
  - KML/KMZ
  - bash
  - script
issue: [84]
autori: ["Totò Fiandaca"]
chefs: ["Andrea Borruso","Totò Fiandaca"]
---

---

**Caso d'uso:** Scaricare la viabilità storica della Regione Siciliana dal server arcgis

## Script Bash

### scarico tutti i file KMZ
lo script seguente scarica in loop `n` file KMZ (`/toto0.kmz ./toto100.kmz ./toto200.kmz ./toto300.kmz ./toto400.kmz ./toto500.kmz ./toto600.kmz ./toto700.kmz ./toto800.kmz`)

```bash
#!/bin/bash
set -x
#inizializzo due variabili
DA=0
A=101
#stampo a video il valore delle due variabili
echo "Il valore di DA è $DA"
echo "Il valore di A è $A"
#ciclo while fino a 900
while [ $DA -lt 900 ]; do
  curl "http://map.sitr.regione.sicilia.it/ArcGIS/rest/services/BCC_PIANI_PAESAGGISTICI/CT_Componenti_Paesaggio_2018/MapServer/9/query?text=&geometry=&geometryType=esriGeometryEnvelope&inSR=&spatialRel=esriSpatialRelIntersects&where=objectid+%3E%3D$DA+AND+objectid+%3C%3D$A&returnGeometry=true&outSR=&outFields=*&f=KMZ" >toto$DA.kmz
  let DA=DA+100
  let A=A+100
  echo "Il valore di DA è $DA"
  echo "Il valore di A è $A"
done;
```

### creo unico file KML unendo tutti i file KMZ

```bash
#!/bin/bash
set -x
#ciclo for
for i in *.kmz;
do
  #crei una variabile che usi per estrarre nome e estensione
  filename=$(basename "$i")
  #estrai estensione
  extension="${filename##*.}"
  #estrai nome file
  filename="${filename%.*}"
  #un-zippo i KMZ
	7z e ./$i
  #rinomino i file doc.kml
	mv ./doc.kml ./"$filename".kml
  #unisco (merge) i file kml
	ogr2ogr -f KML -update -append merged.kml ./"$filename".kml -nln merged
done;
#cancello tutti i file kml inutili
rm ./toto*.kml
```

#### osservazioni

Il file **KMZ** è il formato zippato del file **KML**; un metodo per ottenere **KML** da **KMZ** è un-zipparlo (`7z e ./$i`) ma questa procedura crea un file `doc.kml` (è una convenzione), quindi occorre rinominarlo (`mv ./doc.kml ./"$filename".kml `).

### scarico e unisco in un unico loop

```bash
#!/bin/bash
set -x

DA=0
A=101

echo "Il valore di DA è $DA"
echo "Il valore di A è $A"

while [ $DA -lt 900 ]; do
  curl "http://map.sitr.regione.sicilia.it/ArcGIS/rest/services/BCC_PIANI_PAESAGGISTICI/CT_Componenti_Paesaggio_2018/MapServer/9/query?text=&geometry=&geometryType=esriGeometryEnvelope&inSR=&spatialRel=esriSpatialRelIntersects&where=objectid+%3E%3D$DA+AND+objectid+%3C%3D$A&returnGeometry=true&outSR=&outFields=*&f=KMZ" >toto$DA.kmz
    echo "Il valore di DA è $DA"
  echo "Il valore di A è $A"
  7z e ./toto$DA.kmz
  mv ./doc.kml ./toto$DA.kml
  ogr2ogr -f KML -update -append merged.kml ./toto$DA.kml -nln merged
  let DA=DA+100
  let A=A+100
done;
#cancello i file kml e kmz che non mi servono
rm ./toto*.km*
```

## Riferimenti utili

- [**curl**](https://curl.haxx.se/)
- [**GDAL/OGR**](https://gdal.org/programs/ogr2ogr.html)
- [**7z**](https://manpages.debian.org/jessie/p7zip-full/7za.1.en.html)