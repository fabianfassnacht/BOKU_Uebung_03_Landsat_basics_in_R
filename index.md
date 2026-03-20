
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

### Step 2:  Visualizing Landsat data ###

After loading the Landsat scene, we would like to have a first glance how the image looks like. There are two basic options for plotting the satellite scene in R. The first option uses the code:

	plot(ls_wien)

With this code, all bands of the satellite scene will be individually plotted next to each other in a matrix of plots as shown below. 

![](Tut_1_Fig_3.png)

In some cases, executing the above code may result in an error message that the margins are too small to plot the data. The solution to this problem is to first open a pop-up plotting window in R and then execute the plot commant. This works by using the following code.

	x11()
	plot(ls_wien)

While this plot gives us some information about how the individual bands of the satellite scene look like, they are still somehow disappointing, as we normally would prefer to see a true-color-visualization of the satellite scene. That is, a visualization that imitates the impression that we create with our visual perception.

For such a plot we need a different plotting command which is provided by the *raster* package:

	plotRGB(ls_wien, r=3, g=2, b=1, stretch="hist")

The plotRGB command requires several settings as seen in the code. The first variable is the image to be plotted. Then we have to define which bands of the image should be plotted as the three available colors r = red, g = green, b = blue. In our case, we assign the correct Landsat bands to the the corresponding visualization colors. That is, we assign the Landsat band collecting information about light in the red portion of the spectrum to the red visualization, the green Landsat channel to the green visualization and the blue Landsat channel to the blue visualization. Finally, we have to define a method for stretching the image (pixel) values to the available visualization range. In our case we tell the plotting command to use a histogram ("hist") created by a representative sample of pixels of the image to automatically find a suitable stretch-setting. More detailed information about the plotRGB command can be obtained using the help-function of R which can be accessed for the plotRGB command using.

	?plotRGB

Running the command with these settings will result in an image as shown below. This image more or less matches our visual perception (that is, what we would see when we would fly across this area with an airplane or a space rocket and the atmosphere would be very clear). 

![](Tut_1_Fig_4.png)

Such images can directly be interpreted - green areas typically refer to vegetation, blue areas to water and the white areas to snow or in this case clouds.

While these settings are great, as we can directly interpret the information, there is one additional combination of bands that is frequently used in studies related to vegetation. As we have learned in the course, vegetation has a very strong reflection in the near-infrared portion of the light. However, so far this information is not used in the visualization as we currently only use the red, the green and the blue channel of the Landsat image. We will now change this by replacing the red-channel with the near-infrared channel, the green with the red channel and the blue with the green channel. The corresponding command looks like this:

	plotRGB(ls_wien, r=4, g=3, b=2, stretch="hist")

The resulting image is displayed below. 

![](Tut_1_Fig_5.png)

In this image, vegetation now appear in red colors while greenish areas indicate areas without vegetation. Water bodies will appear very dark as most of the electromagnet radiation is absorbed by water in the near-infrared portion of the light while the highest reflection of water occurs in the blue channel which is not  considered in this visualization option.

#### Exercise: Explore visualization settings #####

To practise a bit the so far learned commands, try to load the second image provided in the downloaded data (stored in the folder **Landsat/D310**). Store the second image in a variable named **ls_d310**. Try to play around a bit more with the **plotRGB** command and try out differing visualization settings (for example modify the channels used for plotting and see how the image colors change).

### Step 3: Clipping Landsat data ###

#### Approach 1: Clipping to the maximum extent ####

Quite often our area of interest is smaller than a complete Landsat scene which covers several thousand square kilometers. In this part of the Tutorial, we will hence learn how to clip-out a part of the Landsat scene using a polygon-shapefile.

As first step, we will load the shapefile by executing the following command:
	
	# set wd to shapefile
	setwd("D:/remote_sensing/Landsat/Shape")
	# load Shapefile 
	vec<-vect(".","area2")

Executing this commant will result in the following console output:

![](Tut_1_Fig_6.png)

A basic summary of the loaded shapefile can be obtained by simply running its variable name. In our case:

	vec

This will result in the following console output which gives us information about the extent of the loaded Shapefile, the number of features (polygons) and the coordinate reference system:

![](Tut_1_Fig_7.png)

Next, we will plot the shapefile over the Landsat image to see whether the two datasets match and which part of the satellite scene we will clip. This required the following commands:

	plotRGB(ls_wien, r=4, g=3, b=2, stretch="hist")
	plot(vec, add=T, col="red")

and should results in the image below.

![](Tut_1_Fig_8.png)

We can now clearly see that the shapefile overlaps with the image and we should hence be able to use it to clip the Landsat dataset.

In this first step, we will use the maximum extent of the shapefile polygon to clip the satellite image. There is also another option to clip the image using the exact shape of the polygon, but we will have a look at this option later.
 
For the first approach, we will first derive the extent of the Shapefile using the command:

	e <- extent(vec)

If we run the variable using

	e

We can see in the console output the maximum extent that is covered by the shapefile.

![](Tut_1_Fig_9.png)

In the next step, we will use this extent-variable to clip the satellite scene:

	setwd("D:/remote_sensing/Landsat/Output")
	ls_wien_clip <- crop(ls_wien, e, filename="ls_wien_clipped.tif", overwrite=TRUE)

This process will now clip the Landsat scene using the extent stored in the variable e and save the clipped image into the variable ls_d239 _clip. Additionally, a new tif-file will be created on the hard-disk and stored into the last defined path (in the example, we switch the folder before running the clip command to control where the clippped files are stored).

After the clipping, we can have a look at the clipped satellite scene using the plotRGB command:

	plotRGB(ls_wien_clip, r=3, g=2, b=1, stretch="hist")

We can now see, that area covered by the clipped satellite scene is a lot smaller than our original scene. On the other hand, more details become visible in the plotted image.

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


