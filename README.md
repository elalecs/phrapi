# Controles
Son clases donde se contiene la programación de las distintas acciones del sistema

## Ubicación de los controles
Se almacenan en `phrapi/controllers` los nombres de los archivos deben corresponder al nombre de la clase y se deben nombrar con notación de camello, por ejemplo un control de notas la clase se llamaría `Notas` y el archivo se guardaría en `phrapi/controllers/Notas.php`.

## Ejecución de los controles
Su ejecución se puede hacer en 2 formas, por ruta o directamente en las páginas donde se requiera a través de `$factory`

### Por Ruta (routing)
Las rutas sirve para solicitar la ejecución de un método directamente por la URL, para esto se necesita modificar el arreglo de configuración en `phrapi/config.php`, agregar un nuevo indice en el arreglo `routing` poniendo:

    'ejemplo-ruta' => ['controller' => 'ClaseControl', 'action' => 'metodoEnControl']

Para ejecutar dicha ruta solo se solicita la URL, por ejemplo `http://proyecto.com/phrapi/ejemplo-ruta`

### Por $factory
Para ejecutar un control desde una página, es requerido que el archivo PHP incluya al framework phrapi

    <? include_once 'phrapi/index.php' ?>

Una vez hecho esto, el framework creará una instancia `Factory` en la variable `$factory` y con esto desde la página podemos solicitar ejecutar un control.

    <? $retorno_de_control = $factory->ClaseControl->metodoEnControl() ?>

La primera vez que se solicita un control a través de `$factory` se crea una instancia, las veces subsecuentes se utiliza la instancia ya creada.

Para solo obtener la instancia de ese control es necesario no especificar nada después del nombre del control.

    <? $instancia_control = $factory->ClaseControl ?>

Se pueden crear nuevas instancias de `Factory` y cada una de ellas contendría sus propias instancias de los controles que se le soliciten.

# Console o D

Phrapi cuenta con una función global para ver el contenido de variables, es algo similar al `console.log` de *JS*, dependiendo de las constantes definidas en *phrapi/framework/globals.php* es donde se mostrarán los mensajes, pudiendo ser a un archivo log, en el cuerpo de la página, en la consola del navegador usando `console.log` o en la cabecera de la respuesta del servidor, por defecto esto se hace en `console.log` y en el cuerpo de la página.

Para hacer uso de la consola solo se requiere mandar llamar a la función `D` pasando por argumentos las variables o datos que se requiera pintar.

	D(['un','arreglo]);
	D('una cadena');
	D($una_variable_con_cualquier_tipo_de_datos);
	D('una',['mezcla'],$de,$valores);

La forma de alterar el comportamiento de `D` se hace con las constantes:


|           Constante          |     Significado    |
|------------------------------|--------------------|
| PHRAPI_DEBUG_FLAG_BODY       | Al cuerpo de la página |
| PHRAPI_DEBUG_FLAG_HEADER     | A los headers de la respuesta |
| PHRAPI_DEBUG_FLAG_LOG        | A un archivo log |
| PHRAPI_DEBUG_FLAG_WEBCONSOLE | Al `console.log` |
| PHRAPI_DEBUG_FLAG_ALL        | Todas las banderas |
| PHRAPI_DEBUG_FLAG_NONE       | Ninguna bandera |
| PHRAPI_DEBUG                 | Bandera sobre la que valida D su funcionamiento, pudiende ser una o varias de las banderas anteriores |

Por lo tanto para indicarle al framework que queremos pintar los datos en el cuerpo de la página y en la consola del navegador tendremos que definir en el archivo *phrapi/framework/globals.php* a la consonante `PHRAPI_DEBUG` así:


	define("PHRAPI_DEBUG", PHRAPI_DEBUG_FLAG_BODY | PHRAPI_DEBUG_FLAG_WEBCONSOLE);


# getValueFrom, $_GET, $_POST
Para facilitar la obtención de las variables que se obtienene por los métodos $_GET, $_POST o de cualquier arreglo u objeto existe la función `getValueFrom` y sus formas especiales.

Por ejemplo para obtener una variable de $_GET, validando que si no viene especificada la variable solicitada se obtenga un valor por defecto y haciendo que el valor resultante sea del tipo entero, en PHP puro se tendría que hacer lo siguiente:

    $numero = (int) isset($_GET['variable']) ? $_GET['variable'] : 10;

Haciendo uso de getValueFrom lo mismo se haría así:

    $numero = getValueFrom($_GET, 'variable', 10, FILTER_SANITIZE_PHRAPI_INT);

Y se puede hacer aún más sencillo con la variante de `getValueFrom`, `getInt`:

    $numero = getInt('variable', 10);

## getValueFrom y sus argumentos

| getValueFrom( | $arreglo_u_objeto | $indice_o_path | $valor_default | $sanitizacion | $nombre_sesion | $callback) |
|---------------|-------------------|----------------|----------------|---------------|----------------|-----------|
|               | requerido, puede ser $_GET, $_POST a cualquier arreglo u objeto | requerido, indice del arreglo, nombre de la propiedad del objeto o path separado por . para recorrer un arreglo de multiples dimesiones | opcional, valor por defecto a devolver en caso de que no se encuentre el path, indice o propiedad, por defecto es `null` | opcional, constante para especificar el tipo de dato que se está esperando obtener | opcional, nombre de la sesión en la que se almacenará el valor en caso de que se encuentre o nombre de la sesión de donde se obtendrá el valor en caso de que no se encuentre | opcional, función a ejecutar antes de devolver el valor |

## getValueFrom, Sanitize

Constantes que se pueden pasar en el argumento `$sanitize`

| Constante                      | Para que es |
| -------------------------------|-------------|
| FILTER_SANITIZE_PHRAPI_MYSQL   | Prepara una cadena para meter a un SQL |
| FILTER_SANITIZE_PHRAPI_INT     | Devuelve enteros |
| FILTER_SANITIZE_PHRAPI_FLOAT   | Devuelve flotantes |
| FILTER_SANITIZE_PHRAPI_ARRAY   | Devuelve arreglo |
| FILTER_SANITIZE_PHRAPI_BOOLEAN | Devuelve booleano |
| FILTER_SANITIZE_*              | Cualquier otra constante de PHP |

## getValueFrom y sus formas cortas

### Arreglos
Forma corta:

	getArray($name, $default = [], $session_name = null, $callback = null)

Equivale a:

	getValueFrom($_GET, $name, $default, FILTER_SANITIZE_PHRAPI_ARRAY, $session_name, $callback);

Forma corta:

	postArray($name, $default = [], $session_name = null, $callback = null)

Equivale a:

	getValueFrom($_POST, $name, $default, FILTER_SANITIZE_PHRAPI_ARRAY, $session_name, $callback);

### Enteros
Forma corta:

	getInt($name, $default = 0, $session_name = null, $callback = null)

Equivale a:

	getValueFrom($_GET, $name, $default, FILTER_SANITIZE_PHRAPI_INT, $session_name, $callback);

Forma corta:

	postInt($name, $default = 0, $session_name = null, $callback = null)

Equivale a:

	getValueFrom($_POST, $name, $default, FILTER_SANITIZE_PHRAPI_INT, $session_name, $callback);

### Flotantes
Forma corta:

	getFloat($name, $default = 0, $session_name = null, $callback = null)

Equivale a:

	getValueFrom($_GET, $name, $default, FILTER_SANITIZE_PHRAPI_FLOAT, $session_name, $callback);

Forma corta:

	postFloat($name, $default = 0, $session_name = null, $callback = null)

Equivale a:

	getValueFrom($_POST, $name, $default, FILTER_SANITIZE_PHRAPI_FLOAT, $session_name, $callback);

### Booleanos
Forma corta:

	getBoolean($name, $default = 0, $session_name = null, $callback = null)

Equivale a:

	getValueFrom($_GET, $name, $default, FILTER_SANITIZE_PHRAPI_BOOLEAN, $session_name, $callback);

Forma corta:

	postBoolean($name, $default = 0, $session_name = null, $callback = null)

Equivale a:

	getValueFrom($_POST, $name, $default, FILTER_SANITIZE_PHRAPI_BOOLEAN, $session_name, $callback);

### Cadenas
Forma corta:

	getString($name, $default = "", $session_name = null, $callback = null)

Equivale a:

	getValueFrom($_GET, $name, $default, FILTER_SANITIZE_STRING, $session_name, $callback);

Forma corta:

	postString($name, $default = "", $session_name = null, $callback = null)

Equivale a:

	getValueFrom($_POST, $name, $default, FILTER_SANITIZE_STRING, $session_name, $callback);

# Configuración, phrapi/config.php

La configuración es un arreglo en que se le establecen las distintas banderas con las que funciona el framework, dentro de este arreglo nos encontramos con el índice *servers* el cual se destruye al iniciar la ejecución, esto se hace primero buscando un índice en *servers* con el nombre del dominio desde donde se está ejecutando, por ejemplo teniendo la siguiente configuración:

	$config = [
		'gmt' => '-05:00',
		'locale' => 'es_MX',
		'php_locale' => 'es_MX',
		'timezone' => 'America/Mexico_City',
		'offline' => false,
		'website' => 1,
		'token_api' => '',
		'servers' => [
			'localhost' => [
				'url' => 'http://localhost/',
				'db' => [
					[
						'host' => 'localhost',
						'name' => 'base_de_datos',
						'user' => 'usuario',
						'pass' => 'contraseña'
					]
				]
			],
		],
		'smtp' => [
			'host' => '',
			'pass' => '',
			'from' => [
				'Contacto' => ''
			]
		],
		'routing' => []
	];

Si el sistema estuviera corriendo desde *localhost*, el arreglo de configuración quedará así:

	$config = [
		'gmt' => '-05:00',
		'locale' => 'es_MX',
		'php_locale' => 'es_MX',
		'timezone' => 'America/Mexico_City',
		'offline' => false,
		'website' => 1,
		'token_api' => '',
		'url' => 'http://localhost/',
		'db' => [
			[
				'host' => 'localhost',
				'name' => 'base_de_datos',
				'user' => 'usuario',
				'pass' => 'contraseña'
			],
			'externo' => [
				'host' => 'servidor_externo',
				'name' => 'base_de_datos',
				'user' => 'usuario',
				'pass' => 'contraseña'
			]
		],
		'smtp' => [
			'host' => '',
			'pass' => '',
			'from' => [
				'Contacto' => ''
			]
		],
		'routing' => []
	];

De esta forma podemos correr el mismo sistema con distintas configuraciones dependiende el servidor sobre el que se esté ejecutando.

# Base de Datos, phrapi/framework/libs/DB.php

La conexión y comunicación con la base de datos lo gestiona la clase DB usando el driver de PDO de PHP, lo recomendable es no generar nuevas instancias, para obtener una instancia se debe mandar llamar su singletón:

	$db = DB::getInstance();

Le método `DB::getInstance()` es el que se encarga de crear la conexión a la base de datos en base a la configuración del arreglo `$config['db']`, se puede especificar el índice pasandolo a `getInstance` por argumento, cuando no se especifica se obtiene el primer indice.

Si dentro de un control se va a estar consultando en varias ocaciones la base de datos lo recomendable es asignar la instancia a una propiedad del control desde el constructor, por ejemplo:

	class MiControl {
		private $db;

		public function __construct(){
			$this->db = DB::getInstance();
		}
	}

## Método en DB para interactuar con la base de datos

Para explicar los métodos se tomará como base la siguiente tabla.

	CREATE TABLE `the_users` (
	  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
	  `name` char(250) DEFAULT NULL,
	  `status` enum('activo','inactivo') DEFAULT NULL,
	  `created_at` date DEFAULT NULL,
	  PRIMARY KEY (`id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8;

Con los siguientes registros.

|  id | name           | status   | created_at |
|----:|----------------|----------|------------|
| 100 | Carmen Montoro | activo   | 2015-06-06 |
| 101 | Avril Galindo  | inactivo | 2015-06-06 |


### queryAll($sql, [$params[, $fetch_style]])

Regresa un arreglo de objetos con todos los registros que devuelve la query.

	D($this->db->queryAll("SELECT * FROM the_users"));
	Array
	(
		[0] => stdClass Object
			(
				[id] => 100
				[name] => Carmen Montoro
				[status] => activo
				[created_at] => 2015-06-06
			)

		[1] => stdClass Object
			(
				[id] => 101
				[name] => Avril Galindo
				[status] => inactivo
				[created_at] => 2015-06-06
			)

	)

### queryAllOne($sql[, $params])

Devuelve un arreglo con solo la primer columna de todos los registros que devuelva la query.

	D($this->db->queryAllOne("SELECT name FROM the_users"));
	Array
	(
		[0] => Carmen Montoro
		[1] => Avril Galindo
	)

### queryAllSpecial($sql[, $params])

Devuelve un arreglo buscando que la query devuelva 2 columnas una por nombre *id* y otra por nombre *label* para regresarlo como índice:valor.

	D($this->db->queryAllSpecial("SELECT id, name as label FROM the_users"));
	Array
	(
		[100] => Carmen Montoro
		[101] => Avril Galindo
	)

### queryRow($sql[, $params[, $fetch_style]])

Retorna un solo registro completo en base a la query ingresada.

	D($this->db->queryRow("SELECT * FROM the_users WHERE id = 101"));
	stdClass Object
	(
		[id] => 101
		[name] => Avril Galindo
		[status] => inactivo
		[created_at] => 2015-06-06
	)

### queryOne($sql[, $params])

Regresa solo el primer valor en base a lo devuelto por la query.

	D($this->db->queryOne("SELECT name FROM the_users WHERE id = 101"));
	Avril Galindo

### getEnumOptions($tabla, $campo)

Regresa un arreglo con los valores de un campo tipo Enum.

	D($this->db->getEnumOptions('the_users', 'status'));
	Array
	(
		[1] => activo
		[2] => inactivo
	)

### fetch style

Para definir la forma en que se deberá devolver los registros en los métodos que aceptan el *fetch_style* como son `queryAll` y `queryRow` por defecto traen la constante `PDO::FETCH_OBJ` y se puede cambiar por alguna otra definidas en PDO, http://php.net/manual/en/pdo.constants.php.

### query($sql[, $params[, $filter]])

Este método sirve para ejecutar cualquier query en la base de datos, de hecho los métodos anteriores lo mandan llamar, pero también se puede ejecutar de forma independiente, con este método podemos hacer UPDATE o INSERT a la base de datos.

#### Para hacer un INSERT

La forma simple sería:

	$nombre = 'Mega';
	$estatus = 'activo';
	$this->db->query("INSERT INTO the_users SET `name` = '{$nombre}', `status` = '{$estatus}', created_at = '2015-06-06'");

La forma más correcta, aprovechando la funcionalidad de `prepare` y `execute` de *PDO* y evitando que se inyecte código es la siguiente:

	$nombre = 'Mega';
	$this->db->query("
		INSERT INTO the_users
		SET
			`name` = :name,
			`status` = :estatus,
			created_at = '2015-06-06'
		",
		[
			':name' => $nombre,
			':estatus' => 'activo'
		]);

#### getLastID()

Cuando se hace un INSERT se puede mandar llamar al método `getLastID()` para obtener el id generado de la última inserción:

	$this->db->query("INSERT INTO …";
	$id = $this->db->getLastID();

#### Para hacer un UPDATE

Es igual que para un INSERT puede ser de una forma simple:

	$estatus = 'activo';
	$id = 102;
	$this->db->query("UPDATE the_users SET `status` = '{$estatus}' WHERE id = '{$id}'");

La forma recomendada sería:

	$estatus = 'activo';
	$id = 102;
	$this->db->query("
		UPDATE the_users
		SET
			`status` = :estatus
		WHERE
			`id` = :id
		",
		[
			':estatus' => $estatus,
			':id' => $id
		]);

### Más allá de DB, Medoo

También es posible usar alguna otra librería para interactuar con la base de datos, por ejemplo se podría usar Medoo (http://medoo.in/)

	require_once PHRAPI_PATH.'libs/medoo/medoo.min.php';

	$config_db = $GLOBALS['config']['db'][0];
	$medoo = new medoo([
		'database_type' => 'mysql',
		'server' => $config_db['host'],
		'database_name' => $config_db['name'],
		'username' => $config_db['user'],
		'password' => $config_db['pass'],
		'charset' => 'utf8',
	]);
	$medoo->insert('the_users', [
		'name' => 'Ponk',
		'status' => 'activo',
		'created_at' => '2015-06-06'
	]);

# Sesiones, phrapi/framework/libs/Session.php

Las sesiones en Phrapi son gestionadas desde `Sesssion`, es tan simple como registrar, obtener y desregistrar propiedades al objeto.

Para obtener la instancia de `Session`:

	$session = Session::getInstance();

Registrar un valor en sesion:

	$session->propiedad = 'valor';

Obtener el valor de sesion:

	$variable = $session->propiedad;

Remover el valor de sesion:

	unset($session->propiedad);

Destruir toda la sesion:

	$session->kill();

## getValueFrom y Session

Hay ocasiones en las que se requiere obtener un valor de $_GET o $_POST por ejemplo un filtro, si viene el valor guardarlo en sesión para que en consultas futuras de no venir en $_GET o $_POST obtenerlo del último valor en sesión.

	$filtro = getValueFrom($_GET, 'filtro', $session->filtro);
	if (isset($_GET['filtro']) {
		$session->filtro = $filtro;
	}

Lo mismo lo podemos resumir usando el parámetro *session_name* en getValueFrom y sus variantes, por ejemplo si el valor que buscamos es una cadena la forma sería:

	$filtro = getString('filtro', '', 'temporal');

Si quisieramos obtener el valor que tenga `Session` después de intentar obtenerla de $_GET, como es el caso anterior podemos accederla con `$session->nombre_de_sesion->elemento` donde en este caso *nombre_de_sesion* sería *temporal* y elemento sería *filtro*

	D($session->temporal->filtro);


# Phrapi

El objetivo del framework es fomentar a aprender a usar PHP, sin complicaciones, su manifiesto es http://microphp.org/


## Phrapi y las librerías externas

El framework permite trabajar con librerías externas, las librerías se deben almacenar en *phrapi/libs/*, hay que seguir la documentación de cada librería para hacer la instalación, setup e implementación.

Algunas de las librerías con las que se ha trabajado son:

 *  DreamObjects
 *  PHPExcel
 *  sendgrid-php


# Guía para hacer un CRUD con Phrapi

Por hacer