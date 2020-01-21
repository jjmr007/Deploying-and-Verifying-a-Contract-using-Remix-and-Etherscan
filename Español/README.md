## Como Compilar, Desplegar y Verificar Contratos que Usan "import" con Remix, node.js y Etherscan

Cuando se va a trabajar con contratos que utilizan una o más librerias, sería preferible disponer de herramientas para solo desplegar las librerías una sola vez y desplegar por separado los contratos que las utilizan. Esta propiedad por defecto no está activada en Remix y la verificación de este tipo de contratos en Etherscan no es tan intuitiva. A continuación los pasos para realizar correctamente el despliegue y verificación de este tipo de contratos:

N° 1 Editar el contrato en [Remix](https://remix.ethereum.org) normalmente y realizar las primeras pruebas en la JavaScriptVM, hasta que el contrato compile y pueda ejecutarse del modo que esperamos. 

N° 2 Ir a "Settings" y en el campo de "General Settings" tildar la casilla de "Generate Contract Metadata". Esta acción generará un archivo JSON relativo al contrato sobre el cual en ese momento se compila con Remix, y esto ocurrirá la proxima vez que se ejecute el plug-in "Compile", o bien ocurrirá la proxima vez que se genere un cambio en el contrato y esté activada la opcion de auto-compilar.

N° 3 Un archivo de extensión .JSON habrá sido creado, para cada contrato contenido en el archivo que se acaba de evaluar por el módulo de compilación. El nombre de cada archivo será el que le hemos dado a cada contrato. Para evitar que se sigan generando estos archivos podemos destildar la casilla "Generate Contract Metadata".

N° 4 En el caso especial en que el archivo .sol sobre el cual trabajamos, y que acaba de generar los "contrato(s)".json, invoque alguna(s) importacion(es) de una(o más) libreria(s) que ya esta(n) desplegada(s) en la cadena de bloques en cuestion, nos interesa que Remix no vuelva a desplegar esas librerias. Es decir: nos interesa que no se esten desplegando las mismas librerías una y otra vez cada vez que se despliega un contrato que las invoca.

N° 5 Para pedirle esto a Remix, antes de desplegar el contrato de interes, vamos a su archivo generado "contrato".json y modificaremos algunos parámetros bajo la llave "deploy":

Como podrá constatarse, una instancia para cada cadena de bloques es declarada:

* `"VM:-"` :Indicando como ID de la red "-"
* `"main:1"` :La cadena principal con ID "1"


...
etcetera

Entre estas instancias estará la cadena donde nos interesa desplegar el contrato; la cadena donde de hecho fueron ya desplegadas las librerías. Por ejemplo, si la cadena en cuestion es "ropsten":

```js
 "ropsten:3" : { 
	 "linkReferences": {},
	 "autoDeployLib": true
	},
```

Sustituimos "true" por "false" y añadimos en el objeto que corresponde a la llave "linkReferences" la correspondencia de las librerias con las direcciones (o addresses) donde han quedado desplegadas en la cadena de bloques:

```js
"ropsten:3": {
	"linkReferences": {
		"./LSafeMath.sol": {
			"SafeMath": "0x0E9A6d63b47b794e275cBC3Ba23Ff52215A8356C",
		},
		"./Libreria2.sol": {
			"Lib2":	"0x1234...00ff"
		}
	},
	"autoDeployLib": false
},
```
Con esta especificación evitamos que Remix despliegue automaticamente las librerias vinculadas a un contrato, en una determinada cadena de bloques.

N° 6 Para ejecutar el despliegue, utilizamos el módulo "Run", indicando en "Enviroment" la opción "Injected web3". Si está instalado el plug-in de [Metamask](https://metamask.io/), este se carga automáticamente o puede refrescarse la página para que al tener la cartera Metamask abierta, interactúe con Remix. Recuérdese indicar en la cadena Metamask la misma cadena de bloques que hemos especificado vincular con las librerías en el archivo "contrato".json. En este caso: ropsten.

N° 7 Una vez ejecutada la transacción del despliegue del contrato y confirmado en la cadena de bloques, es posible verificar el código en [Etherscan](https://etherscan.io/verifyContract) . Para ello y en este caso particular, se coloca la address del nuevo contrato desplegado; en el campo "Please select Compiler Type" elegiremos "Solidity (Standard-Json-Input)".

N° 8 Seguidamente aparece la opción "Please select Compiler Version" y elegiremos exactamente el mismo intérprete que el compilador de Remix utilizó para compilar nuestro contrato. Por ejemplo: "v0.5.13+commit.5b0b510c".

N° 9 Seguidamente escogemos el tipo de licencia para nuestro contrato. Por ejemplo: "No License (None)". Y a continuación pulsamos "Continue".

N° 10 Nos aparece una nueva página donde se nos pide cargar el archivo estándar JSON de entrada, el cual es una estructura de datos muy precisa y minuciosa que debe reflejar con exactitud el contenido del contrato y debe indicar las instrucciones de a que librerias este contrato debe ir vinculado. Por ende tenemos que crear este archivo.

N° 10.1 Será necesario crear un directorio local en nuestra PC y almacenar alli los archivos ".sol" donde estarán especificados tanto nuestras librerías como nuestro contrato, exactamente tal y como quedaron editados al momento de desplegarse, sin modificar si quiera una coma.

Para este ejemplo se hace la suposición que hay dos librerias y que ambas están en un mismo archivo de extensión .sol: "Librerias.sol" y hay un contrato que invoca estas librerias mediante el comando "import": "Contrato.sol". Supóngase que el contrato se llama "Cont" y las librerias "SafeMath" y "Lib2". Estos archivos se guardarán en el mismo directorio local.
	
N° 10.2 Si no lo tenemos instalado, es necesario instalar [NODE.js](https://nodejs.org/es/) . Una vez instalado, abrimos una instancia de la cónsola de comandos (por ejemplo cmd.exe en windows); si se quiere verificamos las versiones de npm y node:

```cmd
>npm -v
```
	
```cmd
>node -v
```

Y prodecemos a ubicarnos en la carpeta local donde guardamos nuestros archivos .sol:

```cmd
>cd C:\Users\MiUsuario\CarpetaLocal
```

N° 10.3 Desde esta carpeta corremos una instancia de node:

```cmd
C:\Users\MiUsuario\CarpetaLocal>node
```

Y se abre una consola especial

```cmd
>
```

N° 10.4 A continuación crearemos el archivo estándar de entradas JSON mediante las herramientas de node:

```js
> var fs = require('fs');
> var file = fs.readFileSync('./Contrato.sol','utf8');
> var lib = fs.readFileSync('./Librerias.sol','utf8');
```

N° 10.5 Acto seguido utilizaremos nuestro editor de archivos preferido, como por ejemplo [Notepad++](https://notepad-plus-plus.org/downloads/) y cargamos la siguiente plantilla del archivo estándar JSON, la cual ha sido tortuosamente deducida de [la documentación de solidity](https://solidity.readthedocs.io/en/v0.5.13/using-the-compiler.html#input-description) ; con la ayuda de las indicaciones de [este repositorio](https://github.com/modular-network/ethereum-libraries-basic-math#solc-js-installation) :

```json
{
  "language": "Solidity",
  "sources":
  {
    "Contrato.sol": {
      "content": file
    },
    "Librerias.sol": {
      "content": lib
    }
  },
  "settings":
  {	 
    "remappings": [],
    "optimizer": {
      "enabled": false,
      "runs": 200
    },
    "evmVersion": "petersburg",  
    "libraries": {
      "Contrato.sol": {
		"SafeMath": "0x0E9A6d63b47b794e275cBC3Ba23Ff52215A8356C",
		"Lib2":	"0x1234...00ff"
      }
    }
  }
}
```

Desde luego, deberán hacerse las sustituciones de acuerdo a cada caso. Nótese que en particular, bajo la llave de "settings", la llave "libraries" viene asociada a un objeto JSON cuya llave no es el nombre de cada libreria como podría suponerse; la llave de este objeto es el nombre del archivo donde se almacena el contrato que deseamos verificar y el objeto asociado es la lista de los nombres de las librerías junto con las direcciones o addresses donde quedaron desplegadas estas librerias en la cadena de bloques que nos ocupa.

N° 10.6 ¿De donde se obtienen los otros parámetros de "settings"? Es posible descargarse la misma version del [intérprete de solidity](https://github.com/ethereum/solidity/releases) que se uso para compilar el contrato con Remix. Si se trabaja con windows lo único que hace falta es descomprimir el archivo [.zip descargado](https://github.com/ethereum/solidity/releases/download/v0.5.13/solidity-windows.zip) en una carpeta local y es posible hacer los .exe de esta carpeta globalmente accesibles por windows del siguiente modo:

	* Dar Click al Simbolo del Sistema Windows (Abajo a laizquierda) 
	* En "Equipo" dar click con el botón derecho del mouse.
	* En el cuadrito de diálogo que se abre darle a "Propiedades"
	* En la columna derecha darle a "Configuración Avanzada del Sistema"
	* Dar click a la barra "Variables de Entorno"
	* En el espacio de "Variables de Usuario" elegir "PATH" y dar click en "Editar.."
	* Añadir alli el directorio donde quedó almacenado solc.exe, seguido de ";"
	* Reiniciar la PC

N° 10.7 Desde la cónsola de comandos ubicada en el directorio local donde se encuentre "Contrato.sol" se ejecuta encontes la orden de ["--metadata"](https://solidity.readthedocs.io/en/v0.5.13/metadata.html#contract-metadata) de solc:

```cmd
C:\Users\MiUsuario\CarpetaLocal>solc --metadata Contrato.sol
```

Y acto seguido se genera un reporte con la metadata tanto de cada contrato contenido en "Contrato.sol" como de cada libreria invocada por "import" en ese archivo. Esta metadata va a contener los valores por defecto de la "evmVersion" que fueron usados por Remix (a no ser que el usuario elige este parámetro usando la nueva interfaz de Remix), y los parámetros del "optimizer" y "remappings".

N° 10.8 Con esta información volvemos a la cónsola de node que dejamos abierta y creamos la variable de entrada que será la precursora del archivo estandar JSON:

```js
> var input = {
  "language": "Solidity",
  "sources":
  {
    "Contrato.sol": {
      "content": file
    },
    "Librerias.sol": {
      "content": lib
    }
  },
  "settings":
  {	 
    "remappings": [],
    "optimizer": {
      "enabled": false,
      "runs": 200
    },
    "evmVersion": "petersburg",  
    "libraries": {
      "Contrato.sol": {
		"SafeMath": "0x0E9A6d63b47b794e275cBC3Ba23Ff52215A8356C",
		"Lib2":	"0x1234...00ff"
      }
    }
  }
}
```

N° 10.9 Finalmente exportamos la variable input al archivo de extensión json de nuestra preferencia, el archivo que [Etherscan](https://etherscan.io/verifyContract-solc-json?a=0xc3369Fc35273271daFaE85fc3D96d1f4232B82cf&c=v0.5.13%2bcommit.5b0b510c&lictype=3) esta esperando que le suministremos. Los créditos de esta última instrucción se deben al usuario "user405398" del foro ["Stack Overflow"](https://stackoverflow.com/questions/37358202/node-js-how-to-represent-json-file-as-var) : 

```js
> var jsonpath = './StandardJsonInput.json';
> fs.writeFileSync(jsonpath, JSON.stringify(input));
```

El archivo json generado es de una sola cadena generalmente bastante larga. Ya en este punto podemos salir de la consola de node con el comando ''.exit ''.

N° 11 En la página de [Etherscan](https://etherscan.io/verifyContract-solc-json?a=0xc3369Fc35273271daFaE85fc3D96d1f4232B82cf&c=v0.5.13%2bcommit.5b0b510c&lictype=3) que quedo abierta esperando por nosotros, damos click al botón "Browse" para seleccionar al archivo "StandardJsonInput.json" que hemos creado. Luego "Click Upload selected file".

N° 12 Generalmente los argumentos de construcción de nuestro contrato son cargados automáticamente por Etherscan. En caso que no sea así, es posible utilizar la herramienta web3 de la consola de Remix para calcular este parámetro. Para mayor información es bueno consultar la [documentación](https://web3js.readthedocs.io/en/v1.2.4/web3-eth-abi.html#encodeparameters) de cómo web3 puede codificar parámetros bajo el protocolo abi, y la documentacion de [solidity](https://solidity.readthedocs.io/en/v0.5.13/abi-spec.html#argument-encoding) sobre la codificación de argumentos, pero como ejemplo:

Supongamos que deseamos codificar los parámetros del contructor: 

```js
constructor (string memory _access, uint256 code) public {
        
	password = _access ;
	ID = code;
        
	}
```

En la consola de Remix colocamos:

```js
>web3.eth.abi.encodeParameters(['string','uint256'],['hola','12745689'])
```
Como se ve, el elemento "memory" ha sido obviado, lo cual esta a corde a las especificaciones del manejo de variables de [tipo dinámico](https://solidity.readthedocs.io/en/v0.5.13/abi-spec.html#use-of-dynamic-types) .

N° 13 Una vez cargados todos los argumentos, tildamos el "reCaptcha" de la pagina de Etherscan y a continuación "Verify and Publish", en pocos segundos, debería mostrarse el resultado con nuestro contrato plenamente verificado.

#### Si encontraste útil esta guia

Considera enviar alguna ayuda a:

`0x8bb38C74B8aaf929201f013C9ECc42b750E562c6`
