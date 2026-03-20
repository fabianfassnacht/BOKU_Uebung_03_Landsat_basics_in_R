
Fernerkundung in der Landschaftsplanung - Tag 3 - Prozessierung von Landsatdaten in R

**Autoren:** Dieses Tutorial wurde von Fabian Fassnacht entwickelt.

## Grundlegende Prozessierung von Landsat-Daten with R ##


### Overview ###

In diesem Tutorial lernen Sie den grundlegenden Umgang mit Landsat-Daten in der Programmierumgebung R (als ein Beispiel für multispektrale Satellitendaten). Zu den behandelten Verarbeitungsschritten gehören:

- Laden von Landsat-Daten
- Visualisieren von Landsat-Daten
- Zuschneiden von Landsat-Daten mit einer Vektordatei
- Maskieren von Landsat-Daten

Die in diesem Tutorial verwendeten Datensätze sind hier verfügbar:

https://drive.google.com/drive/folders/1cKngQQMJCMTNfnh1OXvygAX5ntseIJ8X?usp=sharing


### Verwendete Datensätze ###

In diesem Tutorial verwenden wir ein Landsat-8 und ein Landsat-9-Bild. Genauer gesagt nutzen wir Produkte, die Informationen zur Oberflächenreflexion enthalten. DIe Bilder wurden von der USGS Earth Explorer-Webseite [https://earthexplorer.usgs.gov/](https://earthexplorer.usgs.gov/) heruntergeladen. Wie man die entsprechenden Bilder herunterlädt, wird in den Hausaufgaben von dieser Woche behandelt (siehe Ende des Tutorials).

Detaillierte Informationen (Level-2 Scene-based Science Products-Handbücher) zur Struktur der zwei Datensätze finden Sie hier: https://www.usgs.gov/landsat-missions/landsat-science-products

Es ist sehr lohnenswert, sich diese Informationen anzusehen, da sie der Schlüssel dafür sind die heruntergeladenen Produkte vollkommen zu verstehen. Sie werden sehen, dass die heruntergeladenen Datensätze aus einer Vielzahl an Dateien bestehen und die Information was diese Dateien genau enthalten finden sich alle in den oben genannten **Level-2 Scene-based Science Products**-Handbüchern.

### Schritt 1: Vorbereitung der Satellitenbilddaten

Bitte laden Sie die oben verlinkten Dateien herunter und speichern Sie sie in einem Ordner, den sie wiederfinden können. Danach navigieren Sie in den Ordner und entpacken Sie die zwei gepackten Dateien. Dies sollte zu zwei neuen Ordnern führen, die jeweils eine grössere Zahl an Dateien enthält (Abbildung 1 zeigt ein Beispiel für die Landsat 8 Szene).

![](Fig_01.png)

**Abbildung 1: Die entpackte Landsat-Dateien**

Innerhalb dieses Ordners erstellen wir nun einen neuen Ordner, den wir "bands" nennen (rechtsklick => Neu => Ordner). Dann kopieren wir die 6 Hauptbänder (markiert mit 1 in Abbildung 1) von Landsat 8/9 (der Schritt ist derselbe, unabhängig davon ob man das Landsat 8 oder Landsat 9 Bild verwendet) in den soeben erstellten Ordner. Wenn wir diesen Ordner nun öffnen sollte das zu einer Situation führen wie sie in Abbildung 2 dargestellt ist.

![](Fig_02.png)

**Abbildung 2: Die sechs Landsat-Bänder in einem separaten Ordner**

Nun haben wir alles vorbereitet, um das Satellitenbild in R zu laden.

### Schritt 3: Starten von R-Studio und erste Schritte ###

Wir starten nun das Programm R-Studio indem wir im Startmenü von windows "Rstudio" eintippen und das entsprechende Symbol klicken (Abbildung 3).

![](Fig_03.png)

**Abbildung 3: Starten von R-Studio**

An diesem Punkt gehe ich davon aus, dass Sie die Hausaufgabe von letzter Woche durchgearbeitetin Form des Tutorials auf dieser Webseite: 

https://rspatial.org/intr/2-basic-data-types.html

durchgearbeitet haben. Sollten Sie dies nicht getan haben, würde ich raten dies nun zuerst zu tun, um den nachfolgenden Schritten gut folgen zu können. Ich gehe an dieser Stelle davon aus, dass Sie wissen, wie man Code in R-Studio ausführt und sie ein Grundverständnis dafür besitzen was Variablen sind und wie man mit diesen in R umgehen kann.

**Wichtige allgemeine Tipps zum Arbeiten mit R**

Vermutlich mindestens 90% aller Fehlermeldungen in R hängen mit einem der im folgenden gelisteten Punkte zusammen. Sollten Sie eine Fehlermeldung erhalten, so überprüfen sie bitte zuerst ob eine dieser Punkte das Problem sein könnte:

1. Der Pfad zu Dateien ist falsch (darauf achten entweder den kompletten Pfad zu einer Datei anzugeben, oder dass der aktuelle "Arbeitsraum" von R dem Ordner entspricht wo die entsprechende Datei liegt)
2. Variablennamen sind falsch geschrieben (**ACHTUNG:** R unterscheidet zwischen groß- und kleinschreibung; d.h., die variable "insekten" ist nicht dasselbe wie die Variable "Insekten" oder "inSekten")
3. Ein Paket ist nicht geladen (wenn sie versuchen einen Befehl anzuwenden, der in R nur über ein Paket zur Verfügung gestellt werden kann, können sie irreführende Fehlermeldungen erhalten)
4. Ein- oder Ausgangsdaten sind in einem falschen Datenformat (z.B. eine Tabelle kann nicht als Bild abgepeichert werden)

Das Ziel im Folgenden ist nicht, dass Sie sich den Code komplett selbst erarbeiten, sondern, dass sie ihn ausführen können und so anpassen, dass sie mit eigenen Daten dieselben Prozessierungen durchführen können.

### Schritt 4: Laden der Landsat-Daten ###

Als ersten Schritt laden wir das R-Paket "terra" welche die wichtigsten Funktionen für die Verarbeitung von geocodierten Bildern/Rasterdaten in R beinhaltet:

	require(terra)

R gibt eine Warnmeldung aus, falls ein Paket noch nicht installiert ist. Ist dies der Fall, installieren Sie die Pakete bitte entweder über das Hauptmenü von RStudio, indem Sie **„Tools“ =>** **„Install packages“** auswählen und den Anweisungen im angezeigten Dialog folgen, oder indem Sie den entsprechenden R-Code zur Installation der Pakete in die Konsole eingeben. Um beispielsweise das Paket „terra“ zu installieren, verwenden Sie den folgenden Code:

	install.packages(„terra“)    

Nachdem alle Pakete erfolgreich installiert wurden, laden wir das erste Satellitenbild in zwei Schritten. Dafür speichern wir zunächst die vollständigen Dateipfade aller Landsat-Bänder, die wir soeben in den "bands"-Ordner kopiert haben in einer Textvariable. Wir führen folgenden Befehl aus:

    bandnames <- list.files(„D:/remote_sensing/Landsat/Bands“, pattern="\\.tif$", full.names = T)
	bandnames

Der im obigen Code angegebene Dateipfad sollte so geändert werden, dass er mit dem Pfad übereinstimmt, unter dem Sie die entsprechenden Dateien auf Ihrem Computer gespeichert haben. Wenden Sie anschließend den Befehl „rast“ des terra-Pakets an, um das Bild in ein R-Rasterobjekt zu laden:

    ls_wien <- rast(bandnames)
	
Der Befehl „stack“ lädt noch nicht den gesamten Datensatz in den Speicher, sondern liest lediglich die Metadaten der Datei und stellt Verknüpfungen zu den Daten auf der Festplatte her. Führen Sie abschließend den Variablennamen aus, um eine Zusammenfassung der Rasterdatei zu erhalten:

    ls_wien

Dies sollte eine Konsolenausgabe wie die folgende ergeben:

![](Tut_1_Fig_

Übersetzt mit DeepL.com (kostenlose Version)

### Schritt 2:  Visualisierung von Landsat-Daten ###

Nachdem wir die Landsat-Aufnahme geladen haben, möchten wir uns einen ersten Eindruck davon verschaffen, wie das Bild aussieht. Es gibt zwei grundlegende Möglichkeiten, die Satellitenaufnahme in R darzustellen. Die erste Möglichkeit nutzt den folgenden Code:

    plot(ls_wien)

Mit diesem Code werden alle Bänder der Satellitenaufnahme einzeln nebeneinander in einer Matrix aus Diagrammen dargestellt, wie unten gezeigt. 

![](Tut_1_Fig_3.png)

In einigen Fällen kann die Ausführung des obigen Codes zu einer Fehlermeldung führen, dass die Ränder zu schmal sind, um die Daten darzustellen. Die Lösung für dieses Problem besteht darin, zunächst ein Popup-Fenster in R zu öffnen und dann den Plot-Befehl auszuführen. Dies funktioniert mit dem folgenden Code.

    x11()
    plot(ls_wien)

Diese Darstellung liefert uns zwar einige Informationen darüber, wie die einzelnen Bänder der Satellitenaufnahme aussehen, ist aber dennoch etwas enttäuschend, da wir normalerweise eine Echtfarben-Visualisierung der Satellitenaufnahme bevorzugen würden. Das heißt, eine Visualisierung, die den Eindruck nachahmt, den wir mit unserer visuellen Wahrnehmung erzeugen.

Für eine solche Darstellung benötigen wir einen anderen Plot-Befehl, der vom *raster*-Paket bereitgestellt wird:

	plotRGB(ls_wien, r=3, g=2, b=1, stretch="hist")

Der Befehl `plotRGB` erfordert mehrere Einstellungen, wie im Code zu sehen ist. Die erste Variable ist das darzustellende Bild. Anschließend müssen wir festlegen, welche Bänder des Bildes als die drei verfügbaren Farben r = rot, g = grün, b = blau dargestellt werden sollen. In unserem Fall ordnen wir die richtigen Landsat-Bänder den entsprechenden Darstellungsfarben zu. Das heißt, wir ordnen das Landsat-Band, das Informationen über Licht im roten Bereich des Spektrums sammelt, der roten Darstellung zu, den grünen Landsat-Kanal der grünen Darstellung und den blauen Landsat-Kanal der blauen Darstellung. Schließlich müssen wir eine Methode definieren, um die Bildwerte (Pixelwerte) auf den verfügbaren Visualisierungsbereich zu strecken. In unserem Fall weisen wir den Plot-Befehl an, ein Histogramm („hist“) zu verwenden, das aus einer repräsentativen Stichprobe von Bildpixeln erstellt wurde, um automatisch eine geeignete Streckungseinstellung zu finden. Detailliertere Informationen zum Befehl plotRGB erhalten Sie über die Hilfefunktion von R, die SIE für den Befehl plotRGB über

	?plotRGB

aufrufen können.

Wenn Sie den Befehl mit diesen Einstellungen ausführen, erhalten Sie ein Bild wie unten dargestellt. Dieses Bild entspricht mehr oder weniger unserer visuellen Wahrnehmung (das heißt, dem, was wir sehen würden, wenn wir mit einem Flugzeug oder einer Weltraumrakete über dieses Gebiet fliegen würden und die Atmosphäre sehr klar wäre). 

![](Tut_1_Fig_4.png)

Solche Bilder lassen sich direkt interpretieren – grüne Bereiche stehen typischerweise für Vegetation, blaue Bereiche für Wasser und die weißen Bereiche für Schnee oder in diesem Fall für Wolken.

Diese Einstellungen sind zwar komfortabel, da wir die Informationen direkt interpretieren können, doch gibt es eine weitere Kombination von Bändern, die in Studien zur Vegetation häufig verwendet wird. Wie wir im Kurs gelernt haben, reflektiert Vegetation sehr stark im nahen Infrarotbereich des Lichts. Bislang wird diese Information jedoch nicht in der Visualisierung genutzt, da wir derzeit nur den roten, den grünen und den blauen Kanal des Landsat-Bildes verwenden. Wir werden dies nun ändern, indem wir den roten Kanal durch den Nahinfrarotkanal, den grünen durch den roten Kanal und den blauen durch den grünen Kanal ersetzen. Der entsprechende Befehl sieht wie folgt aus:

	plotRGB(ls_wien, r=4, g=3, b=2, stretch="hist")

Das resultierende Bild ist unten dargestellt. 

![](Tut_1_Fig_5.png)

In diesem Bild erscheint die Vegetation nun in Rottönen, während grünliche Bereiche auf vegetationsfreie Gebiete hinweisen. Gewässer erscheinen sehr dunkel, da der größte Teil der elektromagnetischen Strahlung im nahen Infrarotbereich vom Wasser absorbiert wird, während die stärkste Reflexion des Wassers im Blaukanal auftritt, der bei dieser Visualisierungsoption nicht berücksichtigt wird.

#### Übung: Visualisierungseinstellungen erkunden #####

Um die bisher erlernten Befehle ein wenig zu üben, versuche, das zweite Bild aus den heruntergeladenen Daten zu laden. Speichern Sie das zweite Bild in einer Variablen namens **ls9_wien**. Experimentieren Sie noch ein wenig mit dem Befehl **plotRGB** und probieren Sie verschiedene Visualisierungseinstellungen aus (ändern Sie beispielsweise die für die Darstellung verwendeten Kanäle und beobachten Sie, wie sich die Farben des Bildes verändern).

### Schritt 3: Zuschneiden von Landsat-Daten ###

#### Ansatz 1: Zuschneiden auf die maximale Ausdehnung ####

Häufig ist unser Untersuchungsgebiet kleiner als eine vollständige Landsat-Szene, die mehrere tausend Quadratkilometer umfasst. In diesem Teil des Tutorials lernen wir daher, wie man mithilfe einer Polygon-Shapefile einen Teil der Landsat-Szene ausschneidet.

Als ersten Schritt laden wir die Shapefile, indem wir den folgenden Befehl ausführen:
    
    # wd auf Shapefile setzen
    setwd(„D:/remote_sensing/Landsat/Shape“)
    # Shapefile laden 
    vec<-vect(„.“,„area2“)

Die Ausführung dieses Befehls führt zu folgender Konsolenausgabe:

![](Tut_1_Fig_6.png)

Eine grundlegende Zusammenfassung des geladenen Shapefiles erhält man, indem man einfach dessen Variablennamen aufruft. In unserem Fall:

    vec

Dies führt zu folgender Konsolenausgabe, die uns Informationen über die Ausdehnung des geladenen Shapefiles, die Anzahl der Objekte (Polygone) und das Koordinatenreferenzsystem liefert:

![](Tut_1_Fig_7.png)

Als Nächstes werden wir das Shapefile über das Landsat-Bild legen, um zu sehen, ob die beiden Datensätze übereinstimmen und welchen Teil der Satellitenaufnahme wir ausschneiden werden. Dazu waren die folgenden Befehle erforderlich:

    plotRGB(ls_wien, r=4, g=3, b=2, stretch="hist")
    plot(vec, add=T, col="red")

Dies sollte zu dem unten gezeigten Bild führen.

![](Tut_1_Fig_8.png)

Wir können nun deutlich erkennen, dass sich das Shapefile mit dem Bild überschneidet, und sollten es daher nutzen können, um den Landsat-Datensatz zu beschneiden.

In diesem ersten Schritt verwenden wir die maximale Ausdehnung des Shapefile-Polygons, um das Satellitenbild zu beschneiden. Es gibt auch eine weitere Möglichkeit, das Bild anhand der exakten Form des Polygons zu beschneiden, aber darauf werden wir später noch eingehen.
 
Für den ersten Ansatz leiten wir zunächst die Ausdehnung der Shapefile mithilfe des Befehls ab:

    e <- extent(vec)

Wenn wir die Variable mit

    e

ausführen, sehen wir in der Konsolenausgabe die maximale Ausdehnung, die von der Shapefile abgedeckt wird.

![](Tut_1_Fig_9.png)

Im nächsten Schritt verwenden wir diese Ausdehnungsvariable, um die Satellitenaufnahme zu beschneiden:

    setwd(„D:/remote_sensing/Landsat/Output“)
    ls_wien_clip <- crop(ls_wien, e, filename="ls_wien_clipped.tif", overwrite=TRUE)

Dieser Vorgang schneidet nun die Landsat-Aufnahme anhand des in der Variablen e gespeicherten Ausmaßes zu und speichert das zugeschnittene Bild in der Variablen ls_d239_clip. Zusätzlich wird eine neue TIF-Datei auf der Festplatte erstellt und im zuletzt definierten Pfad gespeichert (im Beispiel wechseln wir den Ordner, bevor wir den Befehl zum Zuschneiden ausführen, um zu steuern, wo die zugeschnittenen Dateien gespeichert werden).

Nach dem Zuschneiden können wir uns die zugeschnittene Satellitenaufnahme mit dem Befehl `plotRGB` ansehen:

	plotRGB(ls_wien_clip, r=3, g=2, b=1, stretch="hist")

Wir sehen nun, dass der von der beschnittenen Satellitenaufnahme abgedeckte Bereich deutlich kleiner ist als unsere ursprüngliche Aufnahme. Andererseits werden im ausgegebenen Bild mehr Details sichtbar.

![](Tut_1_Fig_10.png)

#### Approach 2: Clipping to the exact outline of the Shapefile ####

To clip the raster file to the exact shape of the polygon, only one additional step is required. Basically, the clipping procedure remains the same but after the image has been clipped to the rectangular extent of the shapefile, the remainding pixels that are not located within the polygon are masked out using the **mask** command of the *raster* package. The results in the following code:

	setwd("D:/remote_sensing/Landsat/Output")
	ls_wien_clip2 <- mask(ls_wien_clip, vec)

The masking procedure may take quite a long time depending on the computer's performance. In the end the image should look like this:

![](Tut_1_Fig_11.png)

#### Exercise: Clipping #####

To practise the clipping procedure, try to implement the code to also clip the second Landsat image (from the folder **D310**) using the same R commands. We will use the clipped version of this second image (which should look like the image below) in the next Step.

![](Tut_1_Fig_12.png)


### STEP 4: Applying a cloud mask ###

In this next step, we will make use of the cloud-mask product that is routinely delivered with the Landsat 8 surface reflectance product. The cloud-mask product is currently stored in the folder **Landsat/D310/cl_mask**. As only the second Landsat image (**D310**) is affected by some clouds, we will use this scene as example.

To load the cloud-mask we will use the already familiar code to load a raster image:

	setwd("D:/remote_sensing/Landsat/D310/cl_mask")
	d310_mask <- stack("LC81910282015310LGN00_cfmask.tif")

We can have a look at the just loaded data by plotting the raster using:

	plot(d310_mask)

This leads to the following image:

![](Tut_1_Fig_13.png)

Be aware that it makes no sense to use the **plotRGB** command here, as the cloud-mask raster contains only a single raster layer/channel.

We can see that the cloud-mask layer contains five different values in our case. To understand what each of the 5 classes exactly mean, it is necessary to refer to the Landsat 8 surface reflectance guide (Link provided above).

To make the masking-process more effective, we will now first clip the cloud-mask to the same extent as the already clipped Landsat images. Therefore, we will use the same commands as just learned:

	d310_mask_clip <- crop(d310_mask, e)
	d310_mask_clip2 <- mask(d310_mask_clip, vec)
	plot(d310_mask_clip2)

This should result in the following image:

![](Tut_1_Fig_14.png)

If we compare this to the original image:

![](Tut_1_Fig_12.png)

we can infer, that the value 3 seems to indicate cloud-cover, value 2 cloud-shadows and value 1 water bodies. Clear pixels are indicated by a value of 0. Please be aware that these values may change depending on the version of the surface reflectance product applied. It is highly recommended to double-check this information in the corresponding product guide.

As next step, we will have to mask our the clouds, or alternatively, all pixels that are neither clear nor water. In this case, we will follow the latter approach. To achieve this, we will first create a binary mask from the cloud-cover layer using the command:

	d310_mask_clip_bin <- d310_mask_clip2 > 1
	plot(d310_mask_clip_bin)

In this new layer, all areas affected by clouds or cloud-cover are now marked with a value of 1 while all pixels being either clear of water bodies have a value of 0.

![](Tut_1_Fig_15.png)

Now, we can apply this mask to the original clipped Landsat image using the following command:

	d310_masked <- mask(ls_d310_clip2, d310_mask_clip_bin, maskvalue=1,  updatevalue=NA)
	plotRGB(d310_masked, r=3, g=2, b=1, stretch="hist")

!! Be aware that you have to create the ls_d310_clip2 image first by adapting the code above provided for the d239 image as mentioned in the exercise - this code is not included here!! This will lead to a new version of the Landsat image where all cloud-affected pixels were masked out by replacing all values in the Landsat image with NA (=not available).

![](Tut_1_Fig_16.png)

To save this image, we can either define a filename in the **mask()**-function as done before in the **crop()**-function or we can use a separate command:

	writeRaster(d310_masked, filename="Landsat8_D310_cloud_masked.tif", format="GTiff")

We may want to change the current path to an output folder before saving the raster file. You can use the **setwd()**-function to achieve this. 


