<a name="inicio"></a>		
sdk-ruby		
=======		
		
Modulo para conexión con gateway de pago Todo Pago		
		
######[Instalación](#instalacion)		
######[Versiones de Ruby soportadas](#Versionesderubysoportadas)
######[Generalidades](#general)	
######[Uso](#uso)		
######[Datos adicionales para prevencion de fraude] (#datosadicionales) 		
######[Ejemplo](#ejemplo)		
######[Modo test](#test)
######[Status de la operación](#status)
######[Tablas de referencia](#tablas)		
 		
<a name="instalacion"></a>		
## Instalación		
Se debe descargar la última versión del SDK desde el botón Download ZIP y descomprimirlo. Se debe crear un archivo .gem que debe ser instalado para poder utilizarlo.

### Creación  e Instalación de la gema TodoPagoConector.gem
Abrir la consola tipear:
```
	cd "ruta-del-sdk"
```
Donde ruta-del-sdk es la ruta donde se descomprimió el zip

#### Crear gem TodoPagoConector
Situarse en el directorio que contiene el archivo TodoPagoConector.gemspec, de esta forma:
```
	cd sdk-ruby-master
```
Para crear el archivo .gem se debe ejecutar el comando gem build
#####
       ..\sdk-ruby-master>gem build TodoPagoConector.gemspec
       Successfully built RubyGem
       Name: TodoPagoConector
       Version: 1.1.0
       File: TodoPagoConector-1.1.0.gem

Se debe obtener el archivo TodoPagoConector-1.1.0.gem

#### Instalación de gem TodoPagoConector-1.1.0
#####
     sdk-ruby-master>gem install ./TodoPagoConector-1.1.0.gem
     Successfully installed TodoPagoConector-1.1.0
     1 gem installed
     Installing ri documentation for TodoPagoConector-1.1.0...
     Installing RDoc documentation for TodoPagoConector-1.1.0...

#### Uso del archivo TodoPagoConector-1.1.0.gem
Abrir el Interactive RuBy shell
```
     ..\sdk-ruby-master>irb
```

Tipear el siguiente comando para comprobar que está listo para ser utilizado
```
     irb(main):004:0> require 'todo_pago_conector'
     => true
```
[<sub>Volver a inicio</sub>](#inicio)		

<a name="Versionesderubysoportadas"></a>
##Versiones soportadas de Ruby
La versión implementada de la SDK, esta testeada para versiones de ruby desde 1.9.3 en adelante.
[<sub>Volver a inicio</sub>](#inicio)

<a name="general"></a>
##Genaralidades
Esta versión soporta únicamente pago en moneda nacional argentina (CURRENCYCODE = 32).
[<sub>Volver a inicio</sub>](#inicio)
<a name="uso"></a>		
## Uso		
####1. Inicializar la clase correspondiente al conector (TodoPago).
- crear una estrucura como la del ejemplo con las wsdl suministradas por Todo Pago

```ruby
j_wsdls = {
    #Remoto
    'Authorize'=> 'https://developers.todopago.com.ar/services/t/1.1/Authorize?wsdl'
    #Local
    #'Authorize'=> '../lib/Authorize.wsdl'
    }
```
- crear una estructura como la del ejemplo con los http header suministrados por todo pago
```ruby
j_header_http = {
    "Authorization"=>"PRISMA f3d8b72c94ab4a06be2ef7c95490f7d3"
}
```
- crear una instancia de la clase TodoPago

```ruby		
end_point = 'https://developers.todopago.com.ar/'

conector = TodoPagoConector.new(j_header_http,
                                j_wsdls,
                                end_point) # End Point, wsdl y http_header provisto por TODO PAGO   
```		
		
####2.Solicitud de autorización		
En este caso hay que llamar a sendAuthorizeRequest(). 		
```ruby		
response = conector.sendAuthorizeRequest(optionsSAR_comercio,optionsSAR_operacion)
```		
<ins><strong>Datos propios del comercio</strong></ins>		
optionsSAR_comercio debe ser un Hash con la siguiente estructura:		
		
```ruby
optionsSAR_comercio = Hash.new
optionsSAR_comercio[:Security]="f3d8b72c94ab4a06be2ef7c95490f7d3"
optionsSAR_comercio[:Merchant]= "2153"
optionsSAR_comercio[:EncodingMethod]="XML"
optionsSAR_comercio[:URL_OK]="http://someurl.com/ok/"
optionsSAR_comercio[:URL_ERROR]="http://someurl.com/error/"
optionsSAR_comercio[:Session]= "ABCDEF-1234-12221-FDE1-00000200"

```		

<ins><strong>Datos propios de la operación</strong></ins>		
optionsSAR_operacion debe ser un Hash con la siguiente estructura:		
		
```ruby
optionsSAR_operacion = Hash.new
optionsSAR_operacion[:MERCHANT] = "2153"
optionsSAR_operacion[:OPERATIONID] = "8000"
optionsSAR_operacion[:CURRENCYCODE] = "032"
optionsSAR_operacion[:AMOUNT] = "1.00"
```		
La variable response contendrá una estuctura en la cual <strong>url_request</strong> nos dara la url del formulario de pago a la cual habra que redirigir al comprador y <strong>request_key</strong> será un datos que será requerido en el paso de la confirmación de la transacción a través del método <strong>getAuthorizeAnswer</strong>
####3.Confirmación de transacción.		
En este caso hay que llamar a getAuthorizeAnswer(), enviando como parámetro un Hash como se describe a continuación.		
```ruby
optionsAnwser=Hash.new
optionsAnwser[:Security]= "f3d8b72c94ab4a06be2ef7c95490f7d3" #Token de seguridad, provisto por TODO PAGO. MANDATORIO.
optionsAnwser[:MERCHANT]= "2153"
optionsAnwser[:RequestKey]= "710268a7-7688-c8bf-68c9-430107e6b9da" #Valor retornado en el SendAuthorizeRequest
optionsAnwser[:AnswerKey]= "693ca9cc-c940-06a4-8d96-1ab0d66f3ee6"  #Valor que figura en la URL del redirect luego de realizar el pago

response = conector.getAuthorizeAnswer(optionsAnwser)
```		
[<sub>Volver a inicio</sub>](#inicio)		

<a name="datosadicionales"></a>		
## Datos adicionales para control de fraude
##### Parámetros Adicionales en el post inicial:		
```ruby		
optionsSAR_operacion=Hash.new 		
...........................................................................		
optionsSAR_operacion[:CSBTCITY] = "Villa General Belgrano" #Ciudad de facturación, MANDATORIO.		
optionsSAR_operacion[:CSBTCOUNTR]="AR" #País de facturación. MANDATORIO. Código ISO. (http://apps.cybersource.com/library/documentation/sbc/quickref/countries_alpha_list.pdf)		
optionsSAR_operacion[:CSBTCUSTOMERID]="453458" #Identificador del usuario al que se le emite la factura. MANDATORIO. No puede contener un correo electrónico.		
optionsSAR_operacion[:CSBTIPADDRESS]="192.0.0.4" #IP de la PC del comprador. MANDATORIO.		
optionsSAR_operacion[:CSBTEMAIL]="some@someurl.com" #Mail del usuario al que se le emite la factura. MANDATORIO.	
optionsSAR_operacion[:CSBTFIRSTNAME]="Juan" #Nombre del usuario al que se le emite la factura. MANDATORIO.		
optionsSAR_operacion[:CSBTLASTNAME]="Perez" #Apellido del usuario al que se le emite la factura. MANDATORIO.		
optionsSAR_operacion[:CSBTPHONENUMBER]="541160913988" #Teléfono del usuario al que se le emite la factura. No utilizar guiones, puntos o espacios. Incluir código de país. MANDATORIO.		
optionsSAR_operacion[:CSBTPOSTALCODE]="1010" #Código Postal de la dirección de facturación. MANDATORIO.		
optionsSAR_operacion[:CSBTSTATE]="B" #Provincia de la dirección de facturación. MANDATORIO. Ver tabla anexa de provincias.
optionsSAR_operacion[:CSBTSTREET1]="Some Street 2153" #Domicilio de facturación (calle y nro). MANDATORIO.		
optionsSAR_operacion[:CSBTSTREET2]="" #Complemento del domicilio. (piso, departamento). NO MANDATORIO.		
optionsSAR_operacion[:CSPTCURRENCY]="ARS" #Moneda. MANDATORIO.		
optionsSAR_operacion[:CSPTGRANDTOTALAMOUNT]="10.01" #Con decimales opcional usando el puntos como separador de decimales. No se permiten comas, ni como separador de miles ni como separador de decimales. MANDATORIO. (Ejemplos:$125,38-> 125.38 $12-> 12 o 12.00)
optionsSAR_operacion[:CSMDD7]="" # Fecha registro comprador(num Dias). NO MANDATORIO.		
optionsSAR_operacion[:CSMDD8]="" #Usuario Guest? (Y/N). En caso de ser Y, el campo CSMDD9 no deberá enviarse. NO MANDATORIO.
optionsSAR_operacion[:CSMDD9]="" #Customer password Hash: criptograma asociado al password del comprador final. NO MANDATORIO.	
optionsSAR_operacion[:CSMDD10]="" #Histórica de compras del comprador (Num transacciones). NO MANDATORIO.		
optionsSAR_operacion[:CSMDD11]="" #Customer Cell Phone. NO MANDATORIO.		

#Retail
optionsSAR_operacion[:CSSTCITY]="Villa General Belgrano" #Ciudad de enví­o de la orden. MANDATORIO.		
optionsSAR_operacion[:CSSTCOUNTRY]="" #País de envío de la orden. MANDATORIO.		
optionsSAR_operacion[:CSSTEMAIL]="some@someurl.com" #Mail del destinatario, MANDATORIO.		
optionsSAR_operacion[:CSSTFIRSTNAME]="Jose" #Nombre del destinatario. MANDATORIO.		
optionsSAR_operacion[:CSSTLASTNAME]="Perez" #Apellido del destinatario. MANDATORIO.		
optionsSAR_operacion[:CSSTPHONENUMBER]="541160913988" #Número de teléfono del destinatario. MANDATORIO.		
optionsSAR_operacion[:CSSTPOSTALCODE]="1010" #Código postal del domicilio de envío. MANDATORIO.		
optionsSAR_operacion[:CSSTSTATE]="D" #Provincia de envío. MANDATORIO. Son de 1 caracter		
optionsSAR_operacion[:CSSTSTREET1]="Some Street 2153" #Domicilio de envío. MANDATORIO.		
optionsSAR_operacion[:CSMDD12]="" #Shipping DeadLine (Num Dias). NO MADATORIO.		
optionsSAR_operacion[:CSMDD13]="" #Método de Despacho. NO MANDATORIO.		
optionsSAR_operacion[:CSMDD14]="" #Customer requires Tax Bill ? (Y/N). NO MANDATORIO.		
optionsSAR_operacion[:CSMDD15]="" #Customer Loyality Number. NO MANDATORIO. 		
optionsSAR_operacion[:CSMDD16]="" #Promotional / Coupon Code. NO MANDATORIO. 

#Datos a enviar por cada producto, los valores deben estar separado con #.		
optionsSAR_operacion[:CSITPRODUCTCODE]="electronic_good" #Código de producto. CONDICIONAL. Valores posibles(adult_content;coupon;default;electronic_good;electronic_software;gift_certificate;handling_only;service;shipping_and_handling;shipping_only;subscription)		
optionsSAR_operacion[:CSITPRODUCTDESCRIPTION]="Test Prd Description" #Descripción del producto. CONDICIONAL.
optionsSAR_operacion[:CSITPRODUCTNAME]="TestPrd" #Nombre del producto. CONDICIONAL.		
optionsSAR_operacion[:CSITPRODUCTSKU]="SKU1234" #Código identificador del producto. CONDICIONAL.		
optionsSAR_operacion[:CSITTOTALAMOUNT]="10.01" #CSITTOTALAMOUNT=CSITUNITPRICE*CSITQUANTITY "999999[.CC]" Con decimales opcional usando el puntos como separador de decimales. No se permiten comas, ni como separador de miles ni como separador de decimales. CONDICIONAL.		
optionsSAR_operacion[:CSITQUANTITY]="1" #Cantidad del producto. CONDICIONAL.		
optionsSAR_operacion[:CSITUNITPRICE]="10.01" #Formato Idem CSITTOTALAMOUNT. CONDICIONAL.		
...........................................................		
```		

## Ejemplo		          
Existe un ejemplo en la carpeta https://github.com/TodoPago/sdk-ruby/blob/master/TodoPago/test.rb el cual tiene configurados estos valores, y muestra los resultados de los métodos principales del SDK.
Existe un segundo ejemplo con interfaz grafica, para facil configuracion y prueba en https://github.com/TodoPago/sdk-ruby/blob/master/EjemploUI/main.py (el ejemplo esta hecho en PY pero ejecuta el codigo Ruby por detras)

[<sub>Volver a inicio</sub>](#inicio)

<a name="test"></a>
##Modo test
Para utlilizar el modo test se debe pasar un end point de prueba (provisto por TODO PAGO).
```ruby
conector_test = TodoPagoConector.new(j_header_http_test, j_wsdls_test, end_point_test)
 ```
[<sub>Volver a inicio</sub>](#inicio)

<a name="status"></a>
##Status de la Operación
La SDK cuenta con un método para consultar el status de la transacción desde la misma SDK. El método se utiliza de la siguiente manera:
```ruby
optionsOperations=Hash.new
optionsOperations[:MERCHANT]= merchant #merchant es una variable 
optionsOperations[:OPERATIONID]= operationid #operationid es una variable (id de la opereacion)
conector.getOperations(optionsOperations)
```
[<sub>Volver a inicio</sub>](#inicio)

<a name="GBRDT"></a>
##Consulta de operaciones por rango de tiempo.
En este caso hay que llamar a getByRangeDateTime() y devolvera todas las operaciones realizadas en el rango de fechas dado
```python
optionsGBRDT=Hash.new
optionsGBRDT[:Merchant]= "2866"
optionsGBRDT[:STARTDATE]= "2016-01-01"
optionsGBRDT[:ENDDATE]= "2016-02-19"
optionsGBRDT[:PAGENUMBER] = "1"

response = tpc.getByRangeDateTime(optionsGBRDT)

```
[<sub>Volver a inicio</sub>](#inicio)


<a name="devoluciones"></a>
## Anulaciones y Devoluciones

La SDK dispone de métodos para realizar la anulación o la devolución online, total o parcial, de una transacción realizada a traves de TodoPago.

#### Anulaciones

Se debe llamar al método ```voidRequest``` de la siguiente manera:
```ruby

options = Hash.new

options[:Security] = "837BE68A892F06C17B944F344AEE8F5F", #API Key del comercio asignada por TodoPago 
options[:Merchant] = "35", #Merchant o Nro de comercio asignado por TodoPago
options[:RequestKey] = "6d2589f2-37e6-1334-7565-3dc19404480c" #RequestKey devuelto como respuesta del servicio SendAutorizeRequest

resp = tpc.voidRequest(options)	
```

También se puede llamar al método ```voidRequest``` de la esta otra manera:
```ruby

options[:Security] = "837BE68A892F06C17B944F344AEE8F5F", #API Key del comercio asignada por TodoPago 
options[:Merchant] = "35", #Merchant o Nro de comercio asignado por TodoPago
options[:AuthorizationKey] = "6d2589f2-37e6-1334-7565-3dc19404480c" #AuthorizationKey devuelto como respuesta del servicio GetAuthorizeAnswer

resp = tpc.voidRequest(options)	
```

#### Devoluciones

Se debe llamar al método ```returnRequest``` de la siguiente manera:
```ruby

options = Hash.new

options[:Security]= "837BE68A892F06C17B944F344AEE8F5F", #API Key del comercio asignada por TodoPago 
options[:Merchant] ="35", #Merchant o Nro de comercio asignado por TodoPago
options[:RequestKey] = "6d2589f2-37e6-1334-7565-3dc19404480c" #RequestKey devuelto como respuesta del servicio SendAutorizeRequest
options[:AMOUNT] = "23.50" #Opcional. Monto a devolver, si no se envía, se trata de una devolución total

resp = tpc.returnRequest(options)
```

También se puede llamar al método ```returnRequest``` de la esta otra manera:
```ruby
options = Hash.new

options[:Security] = "837BE68A892F06C17B944F344AEE8F5F", #API Key del comercio asignada por TodoPago 
options[:Merchant] = "35", #Merchant o Nro de comercio asignado por TodoPago
options[:AuthorizationKey] = "6d2589f2-37e6-1334-7565-3dc19404480c" #AuthorizationKey devuelto como respuesta del servicio GetAuthorizeAnswer
options[:AMOUNT] = "1.00" #Opcional. Monto a devolver, si no se envía, se trata de una devolución total

resp = tpc.returnRequest(options)	
```

#### Respuesta de los servicios

Si la operación fue realizada correctamente se informará con un código 2011 y un mensaje indicando el éxito de la operación.

``` ruby
{:envelope=>{:body=>{:return_response=>{:status_code=>"2011", :status_message=>"TX OK", :authorization_key=>"318974f2-4866-d8e7-9622-9ac21aec0df2", :authorizationcode=>"2011", :"@xmlns:api"=>"http://api.todopago.com.ar"}}, :"@xmlns:soapenv"=>"http://schemas.xmlsoap.org/soap/envelope/"}}
```
<a name="tablas"></a>
## Tablas de Referencia		
######[Códigos de Estado](#cde)		
######[Provincias](#p)		
<a name="cde"></a>		
<p>Codigos de Estado</p>		
<table>		
<tr><th>IdEstado</th><th>Descripción</th></tr>		
<tr><td>1</td><td>Ingresada</td></tr>		
<tr><td>2</td><td>A procesar</td></tr>		
<tr><td>3</td><td>Procesada</td></tr>		
<tr><td>4</td><td>Autorizada</td></tr>		
<tr><td>5</td><td>Rechazada</td></tr>		
<tr><td>6</td><td>Acreditada</td></tr>		
<tr><td>7</td><td>Anulada</td></tr>		
<tr><td>8</td><td>Anulación Confirmada</td></tr>		
<tr><td>9</td><td>Devuelta</td></tr>		
<tr><td>10</td><td>Devolución Confirmada</td></tr>		
<tr><td>11</td><td>Pre autorizada</td></tr>		
<tr><td>12</td><td>Vencida</td></tr>		
<tr><td>13</td><td>Acreditación no cerrada</td></tr>		
<tr><td>14</td><td>Autorizada *</td></tr>		
<tr><td>15</td><td>A reversar</td></tr>		
<tr><td>16</td><td>A registar en Visa</td></tr>		
<tr><td>17</td><td>Validación iniciada en Visa</td></tr>		
<tr><td>18</td><td>Enviada a validar en Visa</td></tr>		
<tr><td>19</td><td>Validada OK en Visa</td></tr>		
<tr><td>20</td><td>Recibido desde Visa</td></tr>		
<tr><td>21</td><td>Validada no OK en Visa</td></tr>		
<tr><td>22</td><td>Factura generada</td></tr>		
<tr><td>23</td><td>Factura no generada</td></tr>		
<tr><td>24</td><td>Rechazada no autenticada</td></tr>		
<tr><td>25</td><td>Rechazada datos inválidos</td></tr>		
<tr><td>28</td><td>A registrar en IdValidador</td></tr>		
<tr><td>29</td><td>Enviada a IdValidador</td></tr>		
<tr><td>32</td><td>Rechazada no validada</td></tr>		
<tr><td>38</td><td>Timeout de compra</td></tr>		
<tr><td>50</td><td>Ingresada Distribuida</td></tr>		
<tr><td>51</td><td>Rechazada por grupo</td></tr>		
<tr><td>52</td><td>Anulada por grupo</td></tr>		
</table>		
		
<a name="p"></a>		
<p>Provincias</p>
<p>Solo utilizado para incluir los datos de control de fraude</p>
<table>		
<tr><th>Provincia</th><th>Código</th></tr>		
<tr><td>CABA</td><td>C</td></tr>		
<tr><td>Buenos Aires</td><td>B</td></tr>		
<tr><td>Catamarca</td><td>K</td></tr>		
<tr><td>Chaco</td><td>H</td></tr>		
<tr><td>Chubut</td><td>U</td></tr>		
<tr><td>Córdoba</td><td>X</td></tr>		
<tr><td>Corrientes</td><td>W</td></tr>		
<tr><td>Entre Ríos</td><td>E</td></tr>		
<tr><td>Formosa</td><td>P</td></tr>		
<tr><td>Jujuy</td><td>Y</td></tr>		
<tr><td>La Pampa</td><td>L</td></tr>		
<tr><td>La Rioja</td><td>F</td></tr>		
<tr><td>Mendoza</td><td>M</td></tr>		
<tr><td>Misiones</td><td>N</td></tr>		
<tr><td>Neuquén</td><td>Q</td></tr>		
<tr><td>Río Negro</td><td>R</td></tr>		
<tr><td>Salta</td><td>A</td></tr>		
<tr><td>San Juan</td><td>J</td></tr>		
<tr><td>San Luis</td><td>D</td></tr>		
<tr><td>Santa Cruz</td><td>Z</td></tr>		
<tr><td>Santa Fe</td><td>S</td></tr>		
<tr><td>Santiago del Estero</td><td>G</td></tr>		
<tr><td>Tierra del Fuego</td><td>V</td></tr>		
<tr><td>Tucumán</td><td>T</td></tr>		
</table>		
[<sub>Volver a inicio</sub>](#inicio)
