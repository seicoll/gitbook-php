# PDO 

## Introducció a PDO

> **PDO** (_**PHP Data Object**_) és un extensió de PHP que ens propociona una capa d'abstracció d'accés a dades.

Per tant, ens permet utilitzar les mateixes funcions per realitzar consultes independentment de la base de dades que estiguem utilitant.

* **PDO** ve amb **PHP 5.1**.
* Requereix característiques de OO (Orientació Objectes) del nucli de PHP 5.
  * Per tant, no funcionarà en versions anteriors a 5.1.

## Accés a la base de dades
  
El passos per **accedir a una base de dades** són:

1. Obrir la base de dades.
2. Trametre la comanda SQL a la base de dades.
3. Retornar el resultat de la consulta.
4. Tancar la connexió de la base de dades.

## Connexió a base de dades

```php
<?php
$servername= "localhost";
$dbname = "dbname";
$username = "username";
$password = "password";

//Fem la gestió d'errors amb les intruccions try...catch(...){}
try{
  //Creem una nova connexió a la BD
  //amb new es crea una instància de la classe PDO definint el tipus de basde de dades, nom de la base de dades, usuari i password.
  $conn = new PDO("mysql:host=$servername;dbname=$dbname",$username, $password);
  
  // establim el mode PDO error a exception per poder
  // recuperar les excepcions
  $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
  echo "Connected successfully";
}
catch(PDOException $error)
{
   //Si falla la connexió amb la BD es mostra l'error.
   echo "Connection failed: " . $error->getMessage();
}

//Tancar la connexió de la base de dades.
$conn=null;
?>
```

Per **habilitar les exempcions** i poder detectar si s'ha produeix algun error amb la base de dades, cal canviar el model PDO error a:

`$conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);`

Els possibles valors que es poden assignar a `ATTR_ERRMODE` són:
* `PDO::ERRMODE_SILENT`: valor per defecte, no llença cap tipus d'error ni excepció.
* `PDO::ERRMODE_WARNING`: genera un error E_WARNING de PHP si es produeix algun error.
* `PDO::ERRMODE_EXCEPTION`: genera i llença una excepció si es produeix un error.

## PDO Select

```php
<?php
// sql to select a record
$sql = 'SELECT firstname, lastname, email FROM Users ORDER BY firstname';

//obtenir les files de la BD
$rows = $conn->query($sql);	// use query() because results are returned

//recorrem cadauna de les files per mostrar les dades
foreach ($rows as $row) {
	 echo $row['firstname'] . <br/>;
	 echo $row['lastname'] . <br/>;
	 echo $row['email'] . <br/>;
}
?>
```

Si realitzem un INSERT en una taula amb un **camp AUTO_INCREMENT**, podem obtenir el ID de lúltim registre inserit.

```php
<?php
$sql= "INSERT INTO Users(firstname, lastname, email)
VALUES ('John', 'Doe', 'john@example.com')";

// use exec() because no results are returned
$conn->exec($sql);
$last_id = $conn->lastInsertId();

echo "New record created successfully. ";
echo "Last inserted ID is: " . $last_id;
?>
```

Si volem inserir les **dades enviades per POST** des d'un formulari:

```php
<?php
$firstname = $_POST['firstname'];
$lastname = $_POST['lastname'];
$email = $_POST['email'];

$sql= "INSERT INTO Users(firstname, lastname, email)
VALUES ($firstname, $lastname, $email)";

// use exec() because no results are returned
$conn->exec($sql);
echo "New record created successfully. ";
?>
```

## PDO Update
```php
<?php
	// sql to update a record
	$sql= "UPDATE Users SET lastname='Doe' WHERE id=2";
	
	// use exec() because no results are returned
	// execute the query and returns the number of affected rows
	$count = $conn->exec($sql);

	// echo a message to say the UPDATE succeeded
	echo $count . " records updated successfully";
?>
```

## PDO Delete

```php
<?php
	// sql to delete a record
	$sql= "DELETE FROM Users WHERE id=3";

	// use exec() because no results are returned
	// execute the query and returns the number of affected rows

	$count = $conn->exec($sql);
	echo $count . " records deleted successfully";
?>
```

## PDO Objectes

**PDO** també permet la realització de consultes i mapeig de resultats en objectes del model de l'aplicació.

### Sense indicar la classe de l'objecte

Si no s'indica la classe a la que pertany l'objecte, retornarà una instància de `stdClass` on les propietats de l'objecte correspondran a les columnes de la taula de la base de dades.

```php
$stmt = $conn->query("SELECT * FROM Clients");
$clients = $stmt->fetchAll(PDO::FETCH_OBJ);
foreach ($clients as $client){
    echo $client->nom . ' ' . $client->cognom . '<br>';
}

/*
stdClass::__set_state(array(
   'id' => '104',
   'nom' => 'John',
   'cognom' => 'Doe',
   'telf' => '666 66 66 66',
))*/
```

### Indicant la classe de l'objecte

En primer lloc cal crear una classe amb el model de dades:

```php
<?php
class Usuari
{
  private $nom;
  private $cognoms;

  public function nomComplet () {
    return $this->nom . ' ' . $this->cognoms;
  }
}
?>
```

El nom d'atributs de la classe ha de ser igual a les columnes de la taula de la base de dades:

```php
$stmt= $conn->prepare('SELECT nom, cognoms FROM personal LIMIT 1');
$stmt->execute();
$stmt->setFetchMode(PDO::FETCH_CLASS, 'Usuari');
$usuari = $stmt->fetch()

echo $usuari->nomComplet() . '<br>';
```

En l'script es pot observar la crida al mètode `setFetchMode()` passant com a primer argument la constant `PDO::FETCH_CLASS` que indica que es realitzi un mapeig en la classe que s’indica com a segon argument (la classe **_Usuari_** creada anteriorment).

Amb `fetch()` s'obté l'objecte del tipus indicat enlloc d'un array associatiu.

## Referències

* **PDO** PHP.net: [http://php.net/manual/es/book.pdo.php](http://php.net/manual/es/book.pdo.php)