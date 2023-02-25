# FCGP_Core_9.000

## Principe
Fcgp est un ensemble de 3 programmes destin�s aux applications de cartographie, aux GPS Garmin et � l'application ORUX Map sur smartphone Android, 
en produisant des cartes raster g�or�f�renc�es et des fichiers tuiles afin d'�tre autonome (off ligne). Voir le [site FCGP](http://fcgp.e-monsite.com/ "aller sur le site") pour plus de d�tail
- ***Format des cartes*** 
  - Georef : format texte propri�taire. Contient toutes les informations de g�or�f�rencement
  de la carte.
  - Imp : format pour le logiciel CompeLand (Gps TwoNav).
  - Map : format pour le logiciel OziExploreur.
  - Tab : format pour le logiciel MapInfo.
  - Bpw, Pgw, JgW : format pour le logiciel QGIS.
- ***Format des fichiers tuiles***
  - KMZ : fichier kml compress� pour Google Earth, BaseCamp et les GPS Garmin supportant les CustomMap.
  - JNX : fichier propri�taire Garmin pour BaseCamp et les GPS Garmin supportant les BirdsEye.
  - ORUX : R�pertoire contenant 2 fichiers pour l'application Orux Map sur smartphone Android.

## Particularit� du code VB
- ***Impl�mentation des programmes*** : Le programme d�marre � partir d'une proc�dure Main qui lance le formulaire principal. Voir dans les propri�t�s du projet  la partie Application. Cela permet d'�tablir une similitude avec un programme C# et ainsi de pouvoir comparer les 2 langages.
- ***Impl�mentation des formulaires WindowsForms*** : Les formulaires WindowsForms n'utilise pas le modificateur de variable `WithEvents` ni la clause `Handles` sp�cifique � VB mais ajoute explicitement le constructeur `New`.  Les �v�nements du formulaire et de ces contr�les sont ajout�s dans la `Sub InitialiserEvenements`.
Cela n'emp�che pas l'utilisation du concepteur de formulaire qui intervient sur la `Sub InitializeComponent()` et facilite aussi la traduction VB --> C# des formulaires en enlevant beaucoup de code inutile. L'utilisation du concepteur de formulaire d�truira tout ou partie de la mise en forme actuelle du code du designer. 
```VB
	'Dans le designer
	'Suppression par rapport au d�signer VB de la d�claration d'une variable avec le modificateur WithEvents
	'Friend WithEvents Button1 As Button
	'Ajout par rapport au designer VB de la d�claration normale d'une variable au lieu du modificateur WithEvents
	Friend Button1 As Button

	'Ajout du constructeur du formulaire
	Friend Sub New()
	   AjouterEvenements()
	   InitializeComponent()
	End Sub

	'Ajout d'une proc�dure qui centralise l'ajout des �v�nements li�s au formulaire
	Private Sub AjouterEvenements()
	   AddHandler Me.Button1.Click, New EventHandler(AddressOf Button1_Click)
	End Sub

	'Suppression de la clause Handles. Si on ne supprime pas la clause il y aura 2 passages sur la Sub
	Private Sub Button1_Click(sender As Object, e As EventArgs) 'Handles Button1.Click
	End Sub
```
- ***Autres particularit�s***
  - D'une mani�re g�n�rale il n'est pas fait appel aux proc�dures, fonctions et constantes sp�cifiques de VB disponibles avec l'assembly `Microsoft.VisualBasic`. On retrouve presque tout dans le framework. Seules les fonctions de conversion telque Cint, Ctype, et DirectCast sont utilis�es. 
  - Les classes ou modules principaux contiennent toutes des structures ou des classes priv�es qui les concernent y compris des formulaires. Ces formulaires peuvent �tre modifi�s avec le concepteur si besoin. 

## Conversion du code VB en C#
- ***Utilitaires de conversion*** : Le plus gros de la traduction a �t� faite avec l'extension Visual Studio de Code Converter. Quelques passages ont �t� compar�s avec la traduction de Instant C# de Tangible Software (version gratuite) ou avec dnSpy principalement pour voir si il y avait une meilleure traduction. La finalisation du code C# a �t� faite par la comparaison des 2 codes pour enlever les noms de domaine superflus inclus au niveau du projet ou les noms de classe report�s dans les directives `using static` et par des tests servant � comparer la similitude des fichiers de sortie. 
  - Les fichiers Sqlite (.db) diff�rent car le texte de cr�ation des tables est sauvegard� dans les m�tadonn�es de la base et que le texte de cr�ation diff�re entre les 2 codes. Cel est d� � l'emploi d'une `String multilignes`. Il y a 5 espaces par ligne en d�faveur de C#, 4 pour une indentation et un pour le caract�re de chaine textuelle `@`.
  - Les fichiers JNX diff�rent car le Guid est al�atoire.
  - Les fichiers KMZ diff�rent car les tags des entr�es d'un fichier ZIP sont al�atoires.
- ***Particularit�s de conversion et diff�rences*** : 
  - Nombres Entiers : Les fonctions de conversion de VB vers des nombres entiers, `Byte, Short, Integer, Long, UShort, UInteger, ULong`, ne tronquent pas la valeur d'origine mais appliquent un arrondi dit 'Financier'. Si l'on veut obtenir exactement les m�mes r�sultats entre VB et C#, il faut utiliser par exemple pour `Cint(Valeur)` `(int)Math.Round(Valeur)` sauf pour les valeurs qui sont connues et consid�r�es comme des entiers tel que les valeurs retourn�es par les fonctions `Math.Floor`, `Math.Ceiling` et `Math.Truncate` ou la valeur d�cimale de certains contr�les de formulaires en fonction de leur configuration.
  - Propri�t�s : 
    - VB d�clare automatiquement une variable priv�e `_NomPropri�t�` associ�e � chaque Propri�t� impl�ment�e automatiquement ou readOnly. Cela permet d'utiliser la variable plut�t que la propri�t� � l'int�rieur du bloc de la d�claration de la propri�t� et d'�viter ainsi des appels inutiles aux fonctions sous-jacentes `get` et `set`. C# ne propose pas ce genre de facilit� mais on peut le faire manuellement si besoin.
	- Les propri�t�s avec param�tres ne sont pas accept�es en C#, elles sont traduites en fonction `get_NomPropri�t�` et `set_NomPropri�t�`. On peut renommer �ventuellement une des deux en `NomPropri�t�`.
  - Op�rateur \ : VB a un op�rateur sp�cifique pour la division enti�re. Les op�randes sont d'abord arrondies avant de proc�der � la division. C# choisit le type du retour de la division en fonction du type des op�randes.
  - Variable static : VB permet de d�clarer des variables `static` �ventuellement initialis�es � l'int�rieur d'une Sub ou Function. C# ne le permet pas. Cela oblige � d�clarer un champ priv� au niveau de la classe. 
  - Fonction : VB d�clare automatiquement une variable ayant le m�me nom que celui de la fonction. Cela permet d'affecter une valeur de retour sans obligation d'utiliser le `Return`. Lors de la traduction en C# une variable sera cr��e et le `return` sera utilis� pour retourner la valeur.
  - Exit Try : VB n'est pas avare de mots cl�s pour sortir d'une boucle ou d'une structure. `Exit try` n'existe pas en C# mais peut �tre remplac� par `return`. Dans le cas d'une fonction `return` sera suivi d'une valeur correspondante au type de retour de la fonction. D'autres traductions sont possibles notamment avec un `label` et le mot cl� `goto`. Attention si il y a du code apr�s le bloc `Try Catch Finally`
  - Autres : 
    - Les variables doivent �tre d�clar�es avant de servir de param�tre pour une proc�dure en VB. C# permet de faire la d�claration au niveau de l'appel de la proc�dure dans le cas d'une valeur `out`
	- VB oblige l'affectation d'une variable lors de la d�claration d'un objet `Dim Info = New TuileAffichage(Col, Row, Me)`, C# permet d'ignorer l'affectation de la cr�ation de l'object `_ = new TuileAffichage(Col, Row, this);`
    - L'�change des valeurs de 2 variables n�cessite une variable interm�diaire en VB `Dim Tempo As Integer = Futur : Futur = Encours : Encours = Tempo`. En C# l'utilisation des tuples permet de le faire plus naturellement `(Encours, Futur) = (Futur, Encours);`
    - VB n'a pas d'op�rateur de plage alors que celui-ci est syst�matiquement employ� dans le code C# lorsque cela est possible.
	- La forme simplifi�e du `using` C# n'est pas utilis�e car elle n'existe pas en VB. Son emploi ne facile pas la comparaison des 2 codes �tant donn� qu'on ne sait pas o� fini le bloc `using`.
  - Remarques : 
    - Les 2 codes sont tr�s similaires � la lectutre. Cela est du au fait que le code VB par du postulat qu'il faut utiliser le moins possible les particularit�s sp�cifiques de VB. 
	- Le framework .Net int�gre des nouveaux types qui sont inacessibles � VB, cela sous entend qu'un code sp�cifique � C# pourrait am�liorer la r�activit� des programmes FCGP. 

## Type de projet 
- Il faut une version de Visual Studio qui supporte le `.Net 6` et la derni�re version du langage VB et C#. Les solutions sont des versions de Visual Studio 2017 mais vous pouvez utiliser Visual Studio 2022 Community. Attention les versions 2022 17.1 � 2022 17.4 ne peuvent plus afficher les propri�t�s de certains projets VB. La version 2022 17.5 n'a plus ce probl�me.
- Le projet PARTAGER est commun aux 3 programmes FCGP. Vous pouvez soit l'int�grer comme r�f�rence dans les autres projets ou laisser les liens sur les diff�rents fichiers qui le composent. Cela facilite les modifications du code. 
- Il est fait appel � la DLL de Sqlite sous forme de package Nugets pour la base de donn�es des Settings ou les fichiers Orux. Voir le [site SQLite.org](https://www.sqlite.org/index.html)
- Il est fait appel � la DLL de ScottPlot sous forme de package Nugets et au code pour le cont�le formulaire qui permet l'affichage des graphiques li�s aux traces. Voir le [site ScottPlot.Net ](https://scottplot.net/)
- Le module FormatJNX est bas� sur la description du format JNX et le code FreePascal de Whiter Brinkster. Voir le [blog JNX Raster Maps](http://whiter.brinkster.net/en/JNX.shtml). T�l�charger le code de la [Library Pascal](http://whiter.brinkster.net/JNXLib.rar)
- Les encodeurs d'image Jpeg et Png du .Net sont limit�s � des dimensions de 65500 pixels environ. On peut trouver des codes pour encodeur et les modifier pour qu'ils acceptent des dimensions plus importantes. Il n'est pas certain que les programmes de lecture d'images Open Source soient capables de lire les grandes images de ce type et pour les programmes du commerce je ne sais pas. Cependant cela permettrai d'extraire directement les tuiles Jpeg sans passer par le d�coupage de l'image au format Raw.
- Code sous license GNU GENERAL PUBLIC LICENSE Version 3.