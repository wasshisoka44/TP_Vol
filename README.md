# TP_Vol

## Compiler

Se placer à la racine puis faire : `./gradlew build` puis `./gradlew run` 

## Sortie 
```text
Wassim SAIDANE - Aurélien Authier
------------------------------------------------------------
Réservation n° 1
Client : Wassim
Passager : Wassim
Vol : Air France 1
Date : 20:00, 21 oct. 2020
-----------------------------------------------------------------
Réservation n° 2
Client : Wassim
Passager : Aurélien
Vol : Air France 1
Date : 20:00, 21 oct. 2020
-----------------------------------------------------------------
Flight id : Air France 1
Nombre de passager : 2
Departure : 20:00, 21 oct. 2020 Europe/Paris a Charles de Gaules(Paris)
Arrival : 23:00, 21 oct. 2020 Europe/Paris a Barcelone-El Prat(Barcelone)
Duration : 3H
Stopover : Francfort-Hahn(Francfort) from 21:00, 21 oct. 2020 to 22:00, 21 oct. 2020(1H)
-----------------------------------------------------------------
Flight id : Air France 2
Nombre de passager : 0
Departure : 23:00, 21 oct. 2020 Europe/Paris a Barcelone-El Prat(Barcelone)
Arrival : 02:00, 22 oct. 2020 Europe/Paris a Orly(Paris)
Duration : 3H
-----------------------------------------------------------------
Flight id : Luftensa 1
Nombre de passager : 0
Departure : 20:00, 21 oct. 2020 Europe/Paris a Francfort-Hahn(Francfort)
Arrival : 23:00, 21 oct. 2020 Europe/Paris a Barcelone-El Prat(Barcelone)
Duration : 3H
-----------------------------------------------------------------
```
## UML 
![UML](UML.png)

## A faire

- [x] Implémentation les class et leurs constructeurs

- [x] Gérer les exceptions de dates 

- [x] Implémentation de la durée 

- [x] ID de vol unique 

- [x] Relier la compagnie au vol 

- [X] Relier l'aéroport au vol

- [x] Relier la ville aux aéroports 

- [x] Gérer les escales

- [x] Lier le client à la reservation 

- [x] Lier la réservation au passager 

- [x] Relier Vol à Réservation

- [x] Faire l'UML  

## Problèmes rencontrés

### Manipuler des dates

- Utilisation du type `ZonedDateTime`



### Gestion des exceptions

Les exceptions sont gérer dans la méthode verif().

#### Vol
- [X] Date d'arrivée antérieure à celle de départ
- [X] Date d'arrivée égal à celle d'arrivée
- [x] Aeroport de départ est le même que celui d'arrivée
- [x] L'aeroport ne desert pas la ville entrée

```java
public void verif(){
	if (this.date_arrivee.isBefore(this.date_depart)){
		throw new IllegalArgumentException("Arrival cannot be prior to departure");
	}
	if (this.date_depart.equals(this.date_arrivee)){
		throw new IllegalArgumentException("Arrival cannot be at the same time as departure");
	}
	if (this.depart.equals(this.arrivee)){
		throw new IllegalArgumentException("The place of arrival cannot be the same as the place of departure");
	}
	if(this.depart.dessert.contains(this.arrivee.get_ville())){
		throw new IllegalArgumentException("This airport does not serve the city entrance");
		}
	}
```
#### Escale 
- [x] Le vol arrive à l'escale après le départ de l'escale
- [x] Le vol arrive à l'escale avant le départ
- [x] Le vol part de l'escale après l'arrivée  
```java
public void verif(){
		if (this.date_arrivee.isAfter(this.date_depart)){
			throw new IllegalArgumentException("The flight cannot arrive after its departure");
		}
		if (this.date_arrivee.isBefore(this.vol.date_depart)){
			throw new IllegalArgumentException("The flight cannot arrive at the stopover before taking off");
		}
		if (this.date_depart.isAfter(this.vol.date_arrivee)){
			throw new IllegalArgumentException("The flight cannot leave the stopover after landing");
		}
	}
```
#### Réservation
- [x] La réservation se fait après le décollage 
```java 
public void verif(){
		if (this.date.isAfter(this.vol.date_depart)){
			throw new IllegalArgumentException("The flight is already gone!");
		}
	}
``` 
### Implémentation de la durée

Utilisation du type `duration` qui prend en argument les deux ZonedDateTime. 
Duration va renvoyer des caractères inutiles qui dont supprimer grâce à `replaceAll`.

```java
public String get_Duree(){ 
	String duree=Duration.between(this.depart, this.arrivee).toString();
	duree=(duree == null) ? null : duree.replaceAll("PT", "");
	duree=(duree == null) ? null : duree.replaceAll("H", ":");
	duree = (duree == null) ? null : duree.replaceAll("M", "");
	return duree;
}
```

### Vol unique 

Utilisation d'un seter qui incrémente un numéro de vol dans Compagnie. 

```java
public int get_id(){
	return this.idvol++;
}
```

### Agrégation compagnie - vol 

Nous avons décider d'implémenter une `ArrayList`.
La compagnie aura donc une liste de vol à sa disposition 
```java 
public class Compagnie{
	public String nom;
	private List<Vol> vols = new ArrayList<Vol>();
```

La compagnie va ensuite créer le vol.

Compagnie va utiliser des seters fonctionnant par couples implémenter dans Vol. `verif()` va vérifier si les données entrées sont bonnes.
```java
public void creer_vol(Aeroport aeroport1, ZonedDateTime date1, Aeroport aeroport2, ZonedDateTime date2){
		Vol v = new Vol(this.nom,this.get_id());
		v.de(aeroport1, date1);
		v.vers(aeroport2, date2);
		this.vols.add(v);
		v.verif();
	}
``` 

### Association Aéroport - Ville 
L'aéroport va s'ajouter tous seule dans la listes des aéroports d'une ville pour garentir la doublle navigabilité : 
```java
public class Aeroport {
    private String nom;
    private Ville ville;
    public List<Ville> dessert = new ArrayList<Ville>();

    public Aeroport(String nom, Ville ville){
        this.nom = nom;
        this.ville=ville;
        this.ville.add_aeroport(this);
	}
``` 

### Gestions des escales
L'escale va être ajouter par la compagnie : 
```java
	public void add_escale(Escale e, int num){
		for(Vol v : vols){
			if (v.get_id()==num){
				v.escales.add(e);
			}
		}
	}
``` 
Ensuite Vol va pouvoir afficher toute(s) se(s) escale(s). 

### Client - Réservation 
Client agit comme Compagnie dans l'autre package, il va avoir une liste de réservation et créer des réservations grâce `creer_reservation()`
```java 
public void creer_reservation(ZonedDateTime date, Client client, int vol, Passager passager){
		Reservation r = new Reservation(this.get_id(), date, this, vol, passager);
		this.reference.add(r);
		r.print_all();
	}
``` 
