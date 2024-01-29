## Casos de Uso y Transacciones

### Descripción General

Esta Guía modela el funcionamiento mediante transacciones FHIR, para el consumo y tributación de elementos identificatorios y demográficos de pacientes, entre sistemas de información en salud y un **Indice Maestro de Pacientes** (MPI)
<br>

Las transacciones descritas a continuación se basan el modelamiento de intercambio de datos desarrollado por *IHE* para estos fines, aplicados al estándar **FHIR** en las definiciones de los siguientes perfiles:

* [PIXm](https://profiles.ihe.net/ITI/PIXm/index.html): El perfil Patient Identifier Cross-reference for Mobile (PIXm) proporciona transacciones RESTful  para crear, actualizar y eliminar registros de pacientes en un gestor de referencias cruzadas de identificadores de pacientes y para consultar el gestor de referencias cruzadas de identificadores de pacientes para los identificadores entre dominios de un paciente.
<br>

<div align="center">
  <img src="PIXm.png"> 
  <p>Diagrama Actores PIXm (Fuente: Perfil IHE)</p>
</div>

<br>

* [PDQm](https://profiles.ihe.net/ITI/PDQm/): El perfil de integración de consulta de datos demográficos de pacientes ofrece a varias aplicaciones distribuidas la posibilidad de consultar una lista de pacientes en un servidor de información de pacientes, basándose en criterios de búsqueda definidos por el usuario, y recuperar la información demográfica de un paciente (y, opcionalmente, información relacionada con la visita) directamente en la aplicación.
<br>

<div align="center">
  <img src="PDQm.png"> 
  <p>Diagrama Actores PDQm (Fuente: Perfil IHE)</p>
</div>

<br>
<br>


### Caso de Uso 1: Identificación Cruzada de Pacientes
<br>

Este Caso comprende la identificción de un paciente en un sistema determinado (con identificación local),consultada por sistema (otra identificación local), por medio de un gestor de identificaciones.

#### Actores
<br>

* *Fuente de la Identificación del Paciente*: La Fuente de Identidad del Paciente es el productor y editor de los datos de identidad del paciente.
* *Consumidor de la identificación por referencia cruzada*: El consumidor de referencias cruzadas de identificadores de pacientes consulta las listas de identificadores de pacientes desde el gestor de referencias cruzadas de identificadores de pacientes.
* *Gestor de indentificaciones de referencias cruzadas*: El gestor de referencias cruzadas de identificadores de pacientes gestiona los datos de identidad de pacientes de distintos dominios y cruza referencias de datos de identidad de pacientes de distintos dominios para el mismo paciente.

#### Transacciones
<br>
Por el momento consideraremos solo una trasaccion para un único caso de uso de **PIXm**, que consite en la consulta de identificación de pacientes entre identificadores de distintos dominios *(el mismo paciente en diferentes dominios con identificadores distintos)*.
<br>

<div align="center">
  <img src="pixmtrans.png"> 
  <p>Transacción del Caso (Fuente: Perfil IHE)</p>
</div>

<br>

##### Solicitud

La operación se desarrolla basada en un **HTTP GET** del *Consumidor* al *Gestor* usando la operacion **FHIR $ihe-pix**
<br>

La URL para la operación se muestra a continuación:

```
[base]/Patient/$ihe-pix
```

<br>

Para determinar la obtención de los *Identificadores Correspondientes* la operación debe ser como se describe a continuación:
<br>

```
GET [base]/Patient/$ihe-pix?sourceIdentifier=[token]{&targetSystem=[uri]}{&_format=[token]}
```
<br>

El parámetro *sourceIdentifier* es obligatorio y solo es uno, en cambio el parámetro *targetSystrem* puede ser mas de uno separados cada uno de ellos por medio del caracter **&**. En principio el parámetro *format* no se estará usando.

Como ejemplo
<br>

```
GET {{mpi}}/Patient/$ihe-pix?sourceIdentifier=2.16.840.1.113883.2.22.0.2.102100.50.10%AC1696435881599
```
<br>

Esta operación hará búsqueda de todos los identificadores existentes para el paciente que en el sistema de dominio *2.16.840.1.113883.2.22.0.2.102100.50.10*, se encuentra identificado como *AC169AC1696435881599*

##### Respuesta

La respuesta se desarrolla basada en un recurso **Parámeters** en donde cada parámetro de respuesta es un *targetIdentifier* con el valor del identificador y el dominio al cual pertenece
<br>

```
<Parameters xmlns="http://hl7.org/fhir">
    <parameter>
        <name value="targetIdentifier"/>
        <valueIdentifier>
            <use value="official" />
            <system value="urn:oid:1.3.6.1.4.1.21367.13.20.3000" />
            <value value="IHEBLUE-994" />
        </valueIdentifier>
    </parameter>
    <parameter>
        <name value="targetIdentifier"/>
        <valueIdentifier>
            <use value="official" />
            <system value="urn:oid:1.3.6.1.4.1.21367.13.20.2000" />
            <value value="IHEGREEN-994" />
        </valueIdentifier>
    </parameter>
    <parameter>
      <name value="targetId"/>
        <valueReference>
          <reference value="Patient/Patient-MohrAlice-Blue"/>
        </valueReference>
    </parameter>
    <parameter>
      <name value="targetId"/>
        <valueReference>
          <reference value="Patient/Patient-MohrAlice-Green"/>
        </valueReference>
    </parameter>
</Parameters>
```
<br>
<br>

### Caso de Uso 2: Consulta de información de Datos Demográficos de paciente.
<br>

Este caso comprende la consulta de información de datos demográficos de paciente sobre multiples sistemas distribuidos basado en uno o más criterios de búsqueda

#### Actores
<br>

* *Consumidor de Información de Paciente*: Sistema que consulta por los datos de un paciente.
* *Proveedor de la Información Demográfica de Paciente*: Sistema que provee al sistema que consulta, los datos requeridos sobre el paciente.

#### Transacciones
<br>

Esta es básicamente una transacción de consulta basada en el perfil **PDQm**, que consite en la consulta del *Consumidor*  sobre datos demográficos del paciente para solicitar información sobre pacientes cuyos datos demográficos coinciden con los datos proporcionados en los parámetros de consulta del mensaje de solicitud. El proveedor de datos demográficos del paciente recibe la solicitud. El proveedor de datos demográficos de pacientes procesa la solicitud y devuelve una respuesta en forma de información demográfica de los pacientes coincidentes
<br>

<div align="center">
  <img src="PDQtrans.png"> 
  <p>Transacciones de 2 Casos de Uso (Fuente: Perfil IHE)</p>
</div>

<br>

Como se aprecia existen dos casos de uso declarado. El de **Query** será aquel que se trabajará por el momento, en el cual se consulta por todos los datos de pacientes asociados a los parámetros de busqueda hecha al que suple la información. El caso de **Retrieve** se aplica para la búsqueda específica de los datos de 1 (un) paciente.
<br>


##### Solicitud

La operación se desarrolla basada en un **HTTP GET**  para búsqueda *SearchSet*
<br>

La especificación es como se muestra:

```
[base]/Patient?<parameters>
```

<br>

Para nuestro caso se determinan un set de parámetros de búsqueda aceptables para realizar la operacion:

* Por identificador
```
GET {{mpi}}/Patient?_format=json&identifier=15335473-1
```
El parámetro format no es necesario

* Por registro activo
```
GET {{mpi}}/Patient?format=json&active=true
```

* Por Ciudad o País
```
GET {{mpi}}/Patient?address-city=santiago
```

* Por Fecha de Nacimiento
```
GET {{mpi}}/Patient?birthdate=1981-11-10
```

* Por Apellido
```
GET {{mpi}}/Patient?family=rochefort
```

* Por Nombre
```
GET {{mpi}}/Patient?given=Andres
```
* Por Id de Recurso
```
GET {{mpi}}/Patient?_id=AC1696435881599
```
* Por Nombre Completo
```
GET {{mpi}}/Patient?name=andres martinez rochefort
```
* Por Telecom
```
GET {{mpi}}/Patient?telecom=5694332547
```

<br>

##### Respuesta

La respuesta del sistema que suple es un recurso **Bundle** de tipo **SearchSet** en donde se indica la cantidad de recursos encontrados con los parámetros coincidentes y cada uno de los éstos incluídos como entrada<>
<br>
<br>

















