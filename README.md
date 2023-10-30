### Galactic_Warfare_DB

1. Lea el siguiente enunciado y obtenga las entidades y relaciones que formarán el modelo de datos y su posterior implementación.

   "¡Bienvenidos a 'Galactic Warfare', un emocionante videojuego que te llevará a las profundidades del espacio exterior en una guerra intergaláctica llena de desafíos y aventuras, donde las naves espaciales son las protagonistas!

   En este universo, existen múltiples facciones espaciales en conflicto, cada una con sus propias naves y tecnologías únicas. Cada nave espacial tiene un identificador único, un nombre, un tipo y una clasificación.

   Los jugadores pueden unirse a diversas misiones. Cada jugador tiene un código, un nombre de usuario y un rango. Las misiones tienen un identificador, una descripción y un nivel de dificultad. En una misión pueden participar varios jugadores. Se necesita almacenar el estado de cada misión por parte de un jugador, que puede ser 'en curso', 'completa' o 'fracasada'.

   Cada nave espacial puede llevar a cabo múltiples misiones, y en una misma misión pueden participar varias naves.

   Para poder ganar el juego, los jugadores realizan alianzas. Así, un jugador pertenece a un clan y un clan puede tener muchos jugadores. Un clan tiene un identificador, un nombre y los territorios que conquistó."

2. Transformar la narrativa en un diseño conceptual con el diagrama Entidad-Relación.

3. Luego, pasar el DER al modelo lógico, quedando definidas las tablas con las que trabajará (recuerde realizar el proceso de normalización).

4. Con la definición anterior, se debe implementar un esquema MySQL. Allí debe cargar los datos necesarios para realizar, posteriormente, algunas consultas.

5.- Resuelva las siguientes consultas:

Obtener todas las misiones donde se encuentra el jugador X.

Muestre la cantidad de jugadores que tiene cada clan..

Muestre los tipos de naves que participaron en misiones cuyas dificultades son mayores a X.

Muestre el nombre de los jugadores que están unidos a la misión X y se encuentran en la nave de tipo Y.

Muestre los rangos de los jugadores que pertenezcan a un clan que haya conquistado el territorio X.


```
CREATE DATABASE Galactic_Warfare;
use Galactic_Warfare;

DELETE FROM Misiones;

CREATE TABLE `Clanes` (
  `id` int PRIMARY KEY NOT NULL AUTO_INCREMENT,
  `nombre` varchar(20) NOT NULL,
  `territorio` varchar(30) NOT NULL
);

CREATE TABLE `Naves` (
  `id` int PRIMARY KEY NOT NULL AUTO_INCREMENT,
  `nombre` varchar(20) NOT NULL,
  `tipo` varchar(30) NOT NULL,
  `clasificacion` int NOT NULL
);

CREATE TABLE `Misiones` (
  `id` int PRIMARY KEY NOT NULL AUTO_INCREMENT,
  `dificultad` varchar(20) NOT NULL,
  `descripcion` varchar(20) NOT NULL
);

CREATE TABLE `MisionesNave` (
  `idMisiones` int NOT NULL,
  `idNave` int NOT NULL,
  PRIMARY KEY (`idMisiones`, `idNave`),
  FOREIGN KEY (`idMisiones`) REFERENCES `Misiones` (`id`),
  FOREIGN KEY (`idNave`) REFERENCES `Naves` (`id`)
);

CREATE TABLE `Jugadores` (
  `codigo` int PRIMARY KEY NOT NULL AUTO_INCREMENT,
  `rango` varchar(20) NOT NULL,
  `nombre` varchar(20) NOT NULL,
  `idClan` int NOT NULL,
  FOREIGN KEY (`idClan`) REFERENCES `Clanes` (`id`)
);

CREATE TABLE `JugadorMisiones` (
  `codigo` int NOT NULL,
  `idMision` int NOT NULL,
  `estadoMision` varchar(30),
  PRIMARY KEY (`codigo`, `idMision`),
  FOREIGN KEY (`codigo`) REFERENCES `Jugadores` (`codigo`),
  FOREIGN KEY (`idMision`) REFERENCES `Misiones` (`id`)
);

-- Datos de ejemplo para Clanes
INSERT INTO Clanes (id, nombre, territorio) VALUES
(1, 'Clan A', 'Territorio 1'),
(2, 'Clan B', 'Territorio 2'),
(3, 'Clan C', 'Territorio 3');

-- Datos de ejemplo para Naves
INSERT INTO Naves (id, nombre, tipo, clasificacion) VALUES
(1, 'Nave X', 'Caza Estelar', 1),
(2, 'Nave Y', 'Nave de Asalto', 2),
(3, 'Nave Z', 'Nave de Exploración', 3);


-- Datos de ejemplo para Misiones
INSERT INTO Misiones (id, dificultad, descripcion) VALUES
(1, '10', 'Misión 1'),
(2, '30', 'Misión 2'),
(3, '40', 'Misión 3');



-- Datos de ejemplo para Jugadores
INSERT INTO Jugadores (rango, nombre, idClan) VALUES
('Novato', 'Jugador1', 1),
('Capitan', 'Jugador2', 2),
('Experto', 'Jugador3', 1),
('Sargento', 'Jugador4', 3),
('Teniente', 'Jugador5', 2);

-- Datos de ejemplo para JugadorMisiones
INSERT INTO JugadorMisiones (codigo, idMision, estadoMision) VALUES
(1, 1, 'En Curso'),
(2, 2, 'Completa'),
(3, 3, 'Fracasada');

-- Datos de ejemplo para MisionesNave
INSERT INTO MisionesNave (idMisiones, idNave) VALUES
(1, 1),
(2, 2),
(3, 3);




-- Tabla Clanes
SELECT * FROM Clanes;

-- Tabla Naves
SELECT * FROM Naves;

-- Tabla Misiones
SELECT * FROM Misiones;

-- Tabla MisionesNave
SELECT * FROM MisionesNave;

-- Tabla Jugadores
SELECT * FROM Jugadores;

-- Tabla JugadorMisiones
SELECT * FROM JugadorMisiones;



-- consulta1
SELECT Misiones.*
FROM Misiones
JOIN JugadorMisiones ON Misiones.id = JugadorMisiones.idMision
JOIN Jugadores ON JugadorMisiones.codigo = Jugadores.codigo
WHERE Jugadores.nombre = 'Jugador1';


-- consulta2
SELECT Clanes.nombre AS NombreDelClan, COUNT(Jugadores.codigo) AS CantidadDeJugadores
FROM Clanes
LEFT JOIN Jugadores ON Clanes.id = Jugadores.idClan
GROUP BY Clanes.nombre;

-- consulta 3
SELECT DISTINCT Naves.tipo
FROM Naves
INNER JOIN MisionesNave ON Naves.id = MisionesNave.idNave
INNER JOIN Misiones ON MisionesNave.idMisiones = Misiones.id
WHERE CAST(Misiones.dificultad AS SIGNED) > 10;

-- consulta 4 
SELECT Jugadores.nombre
FROM Jugadores
INNER JOIN JugadorMisiones ON Jugadores.codigo = JugadorMisiones.codigo
INNER JOIN Misiones ON JugadorMisiones.idMision = Misiones.id
INNER JOIN MisionesNave ON Misiones.id = MisionesNave.idMisiones
INNER JOIN Naves ON MisionesNave.idNave = Naves.id
WHERE Misiones.id = 1 AND Naves.tipo = 'Caza Estelar';

-- consulta 5
SELECT DISTINCT Jugadores.rango
FROM Jugadores
INNER JOIN Clanes ON Jugadores.idClan = Clanes.id
WHERE Clanes.territorio = 'Territorio 3';


SELECT Jugadores.*, JugadorMisiones.*, Misiones.*
FROM Jugadores
LEFT JOIN JugadorMisiones ON Jugadores.codigo = JugadorMisiones.codigo
LEFT JOIN Misiones ON JugadorMisiones.idMision = Misiones.id;

```
