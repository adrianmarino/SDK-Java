<a name="inicio"></a>		
Todo Pago - m�dulo SDK-JAVA para conexi�n con gateway de pago
=======

 + [Instalaci�n](#instalacion)
 	+ [Versiones de Java soportadas](#Versionesdejavasoportadas)
 	+ [Generalidades](#general)
  + [Uso](#uso)		
    + [Inicializar la clase correspondiente al conector (TodoPago\Sdk)](#initconector)
    + [Solicitud de autorizaci�n](#solicitudautorizacion)
    + [Confirmaci�n de transacci�n](#confirmatransaccion)
    + [Ejemplo](#ejemplo)
    + [Modo test](#test)
 + [Datos adicionales para prevenci�n de fraude](#datosadicionales) 
 + [Caracter�sticas](#caracteristicas)
    + [Status de la operaci�n](#status)
    + [Consulta de operaciones por rango de tiempo](#statusdate)
    + [Devolucion](#devolucion)
    + [Devolucion parcial](#devolucionparcial)
    + [Formulario hibrido](#formhidrido)
    + [Obtener Credenciales](#credenciales)
	+ [M�ximo de cuotas a mostrar en formulario](#maxcuotas)
 + [Diagrama de secuencia](#secuencia)
 + [Tablas de referencia](#tablareferencia)		
 + [Tabla de errores](#codigoerrores)
 + [Agregar el proyecto a Eclipse](#eclipse)
 
 
<a name="instalacion"></a>		
## Instalaci�n
Se debe descargar la �ltima versi�n del SDK desde el bot�n Download ZIP del branch master.

En caso de utilizar Maven, se puede agregar el jar TodoPago.jar al repositorio local de Maven utilizando la siguinte linea de comando:
```
mvn install:install-file -Dfile=<path-to-file> -DpomFile=<path-to-pomfile>
``` 

Una vez hecho esto se puede agregar la dependencia a este paquete desde el pom.xml
```xml
<dependency>
	<groupId>com.ar.todopago</groupId>
	<artifactId>sdk-java</artifactId>
	<version>1.5.0</version>
</dependency>
```
De ser necesario agregar la siguiente dependencia requerida por TodoPago desde el pom.xml
 ```xml
 <dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
     <version>20090211</version>
  </dependency>
```

Una vez descargado se deben hacer los siguientes imports.
```java
import ar.com.todopago.api.ElementNames;
import ar.com.todopago.api.TodoPagoConector;
import ar.com.todopago.api.model.*;
import ar.com.todopago.api.exceptions.*;
```

El Ejemplo es un proyecto hecho en maven, con un pom.xml que incluye la configuracion para importar y exportar las librerias requeridas.

Para agregar el proyecto de ejemplo a Eclipse, una vez descargado, por consola ir hasta su carpeta y ejecutar las siguientes lineas:
```
mvn clean install -Dwtpversion=2.0
mvn eclipse:clean eclipse:eclipse -Dwtpversion=2.0
```
Luego, importar el proyecto normalmente en Eclipse.

<a name="Versionesdejavasoportadas"></a>   
####1. Versiones de Java soportadas
La versi&oacute;n implementada de la SDK, esta testeada para versiones desde Java 5 en adelante con JAX-WS.

<a name="general"></a>
####2. Generalidades
Esta versi�n soporta �nicamente pago en moneda nacional argentina (CURRENCYCODE = 32).

[<sub>Volver a inicio</sub>](#inicio)	
<br>
<a name="uso"></a>		
## Uso	

<a name="initconector"></a>
####Inicializar la clase correspondiente al conector (TodoPago\Sdk).

Si se cuenta con los http header suministrados por Todo Pago

- Crear un Map con dichos http header
```java
Map<String, List<String>> auth = new HashMap<>(String, List<String>);
auth.put(ElementNames.Authorization, Collections.singletonList("PRISMA f3d8b72c94ab4a06be2ef7c95490f7d3"));
```

- Crear una instancia de la clase TodoPago
```java		
TodoPagoConector tpc = new TodoPagoConector(TodoPagoConector.developerEndpoint, auth);//End Point developer y http_header provisto por TODO PAGO	
```	

Si se cuenta el con User y Password del login en TodoPago

- Crear una instancia de la clase TodoPago
```java		
TodoPagoConector tpc = new TodoPagoConector(TodoPagoConector.developerEndpoint);//End Point developer
```	
- Obtener las credenciales a traves  del m&eacute;todo getCredentials de TodoPago. Ver [Obtener Credenciales](#credenciales)

<a name="solicitudautorizacion"></a>
####Solicitud de autorizaci�n	
En este caso hay que llamar a sendAuthorizeRequest(). 		
```java		
Map<String, Object> a = tpc.sendAuthorizeRequest(parameters, getFraudControlParameters());		
```	
<ins><strong>datos propios del comercio</strong></ins>		
El primer atributo parameters, debe ser un Map<String, String> con la siguiente estructura:		
		
```java
Map<String, String> parameters = new HashMap<String, String>();
	parameters.put(ElementNames.Session, "ABCDEF-1234-12221-FDE1-00000200");
	parameters.put(ElementNames.Security, "1234567890ABCDEF1234567890ABCDEF");
	parameters.put(ElementNames.EncodingMethod, "XML");
	parameters.put(ElementNames.Merchant, "12345678"); //dato fijo (n�mero identificador del comercio)
	parameters.put(ElementNames.OperationID, "8000"); //n�mero �nico que identifica la operaci�n, generado por el comercio.
	parameters.put(ElementNames.CurrencyCode, "032"); //por el momento es el �nico tipo de moneda aceptada
	parameters.put(ElementNames.Amount, "1.00");
	parameters.put(ElementNames.UrlOK, "http,//someurl.com/ok/");
	parameters.put(ElementNames.UrlError, "http,//someurl/fail/");
	parameters.put(ElementNames.EMAILCLIENTE, "some@someurl.com");
```	

<ins><strong>datos prevenci�n de fraude</strong></ins>	
El segundo atributo getFraudControlParameters().  Ver [Datos adicionales para prevenci�n de fraude](#datosadicionales)  

<p><strong>C�digos de rechazo</strong></p>
```java	
Map<String, Object> 	
	{ StatusCode = -1,
	  PublicRequestKey = te0b9bba5-cff9-173a-20da-b9bc8a389ac7, 
	  URL_Request = https://developers.todopago.com.ar/formulario/commands?command=formulario&m=te0b9bba5-cff9-173a-20da-b9bc8a389ac7, 
	  StatusMessage = Solicitud de Autorizacion Registrada, 
	  RequestKey = ff0f6434-a2ab-e87f-3ece-37f7081e671a }
```

La **URL_Request** es donde est� hosteado el formulario de pago y donde hay que redireccionar al usuario, una vez realizado el pago seg�n el �xito o fracaso del mismo, el formulario redireccionar� a una de las 2 URLs seteadas en **parameters** ([URL_OK](#url_ok), en caso de �xito o [URL_ERROR](#url_error), en caso de que por alg�n motivo el formulario rechace el pago)

Si, por ejemplo, se pasa mal el <strong>MerchantID</strong> se obtendr� la siguiente respuesta:
```java
Map<String, Object> 
	{ StatusCode = 702,
	  StatusMessage = ERROR: Cuenta Inexistente }
```

<a name="confirmatransaccion"></a>
####Confirmaci�n de transacci�n.
En este caso hay que llamar a **getAuthorizeAnswer()**, enviando como par�metro un Map<String, String> como se describe a continuaci�n.

```java	
Map<String, String> parameters = new HashMap<String, String>();		
	parameters.put(ElementNames.Security, "1234567890ABCDEF1234567890ABCDEF"); // Token de seguridad, provisto por TODO PAGO. MANDATORIO.
	parameters.put(ElementNames.Merchant, "12345678");
	parameters.put(ElementNames.RequestKey, "0123-1234-2345-3456-4567-5678-6789");
	parameters.put(ElementNames.AnswerKey, "1111-2222-3333-4444-5555-6666-7777"); // *Importante

Map<String, Object> b = tpc.getAuthorizeAnswer(parameters);	
```

Se deben guardar y recuperar los valores de los campos <strong>RequestKey</strong> y <strong>AnswerKey</strong>.

El par�metro <strong>RequestKey</strong> es siempre distinto y debe ser persistido de alguna forma cuando el comprador es redirigido al formulario de pagos.

<ins><strong>Importante</strong></ins> El campo **AnswerKey** se adiciona  en la redirecci�n que se realiza a alguna de las direcciones ( URL ) epecificadas en el  servicio **SendAurhorizationRequest**, esto sucede cuando la transacci�n ya fue resuelta y es necesario regresar al site para finalizar la transacci�n de pago, tambi�n se adiciona el campo Order, el cual tendr� el contenido enviado en el campo **OPERATIONID**. Para nuestro ejemplo: <strong>http://susitio.com/paydtodopago/ok?Order=27398173292187&Answer=1111-2222-3333-4444-5555-6666-7777</strong>		

```java		
Map<String, Object>		
	{ StatusCode = -1, 		
      StatusMessage = APROBADA,		
	  AuthorizationKey = 1294-329E-F2FD-1AD8-3614-1218-2693-1378,		
      EncodingMethod = XML,		
      Payload = { Answer = { DATETIME = 2014/08/11 15:24:38,		
						     RESULTCODE = -1,		
							 RESULTMESSAGE = APROBADA,		
							 CURRENCYNAME = Pesos,		
							 PAYMENTMETHODNAME = VISA,		
							 TICKETNUMBER = 12,		
							 CARDNUMBERVISIBLE = 450799******4905,		
							 AUTHORIZATIONCODE = TEST38 },							 
				{ Request = { MERCHANT = 12345678,
						      OPERATIONID = ABCDEF-1234-12221-FDE1-00000012,
							  AMOUNT = 1.00,
							  CURRENCYCODE = 032}
				}
	}
	  
```		
Este m�todo devuelve el resumen de los datos de la transacci�n

Si se pasa mal el <strong>AnswerKey</strong> o el <strong>RequestKey</strong> se ver� el siguiente rechazo:

```java
Map<String, Object> 
	{ StatusCode = 404,
	  StatusMessage = ERROR: Transaccion Inexistente }
```

<a name="ejemplo"></a>      
####Ejemplo
Existe un ejemplo ejecutable en https://github.com/TodoPago/SDK-Java/blob/master/Ejemplo/src/main/java/com/ar/todopago/ejemplo/SampleUI.java que muestra los resultados de los m�todos principales del SDK y su correcta implementacion.<br>

Existe un ejemplo en la carpeta https://github.com/TodoPago/sdk-java/tree/master/src/test que muestra los resultados de los m�todos principales del SDK.	

<a name="test"></a>      
####Modo Test

El SDK-JAVA permite trabajar con los ambiente de desarrollo y de produccion de Todo Pago.<br>
El ambiente se debe instanciar como se indica a continuacion.

```java
TodoPagoConector tpc = new TodoPagoConector(TodoPagoConector.developerEndpoint, getAuthorization());

private static Map<String, List<String>> getAuthorization() {
	Map<String, List<String>> parameters = new HashMap<String, List<String>>();
	parameters.put(ElementNames.Authorization, Collections.singletonList("PRISMA f3d8b72c94ab4a06be2ef7c95490f7d3"));
	return parameters;
}
```

[<sub>Volver a inicio</sub>](#inicio)
<br>

<a name="datosadicionales"></a>		
## Datos adicionales para control de fraude		
Los datos adicionales para control de fraude son **obligatorios**, de lo contrario baja el score de la transacci�n.

Los campos marcados como **condicionales** afectan al score negativamente si no son enviados, pero no son mandatorios o bloqueantes.

```java		
private static Map<String, String> getFraudControlParameters() {

	Map<String, String> parameters = new HashMap<String, String>();		
	parameters.put("CSBTCITY", "Villa General Belgrano"); //Ciudad de facturaci�n, MANDATORIO.		
	parameters.put("CSBTCOUNTRY", "AR");//Pa�s de facturaci�n. MANDATORIO. C�digo ISO. (http://apps.cybersource.com/library/documentation/sbc/quickref/countries_alpha_list.pdf)	
	parameters.put("CSBTCUSTOMERID", "453458"); //Identificador del usuario al que se le emite la factura. MANDATORIO. No puede contener un correo electr�nico.		
	parameters.put(CSBTIPADDRESS", "192.0.0.4"); //IP de la PC del comprador. MANDATORIO.		
	parameters.put(CSBTEMAIL", "some@someurl.com"); //Mail del usuario al que se le emite la factura. MANDATORIO.
	parameters.put(CSBTFIRSTNAME", "Juan");//Nombre del usuario al que se le emite la factura. MANDATORIO.		
	parameters.put(CSBTLASTNAME", "Perez");//Apellido del usuario al que se le emite la factura. MANDATORIO.
	parameters.put(CSBTPHONENUMBER", "541160913988");//Tel�fono del usuario al que se le emite la factura. No utilizar guiones, puntos o espacios. Incluir c�digo de pa�s. MANDATORIO.		
	parameters.put(CSBTPOSTALCODE", "1010");//C�digo Postal de la direcci�n de facturaci�n. MANDATORIO.	
	parameters.put(CSBTSTATE", "B");//Provincia de la direcci�n de facturaci�n. MANDATORIO. Ver tabla anexa de provincias.	
	parameters.put(CSBTSTREET1", "Some Street 2153");//Domicilio de facturaci�n (calle y nro). MANDATORIO.			
	parameters.put("CSBTSTREET2", "Piso 8");//Complemento del domicilio. (piso, departamento). NO MANDATORIO.
	parameters.put(CSPTCURRENCY", "ARS");//Moneda. MANDATORIO.		
	parameters.put(CSPTGRANDTOTALAMOUNT", "125.38");//Con decimales opcional usando el puntos como separador de decimales. No se permiten comas, ni como separador de miles ni como separador de decimales. MANDATORIO.(Ejemplos:$125,38-> 125.38 $12-> 12 o 12.00)		
	parameters.put(CSMDD7", "");// Fecha registro comprador(num Dias). NO MANDATORIO.		
	parameters.put(CSMDD8", "Y"); //Usuario Guest? (Y/N). En caso de ser Y, el campo CSMDD9 no deber� enviarse. NO MANDATORIO.		
	parameters.put(CSMDD9", "");//Customer password Hash: criptograma asociado al password del comprador final. NO MANDATORIO.		
	parameters.put(CSMDD10", "");//Hist�rica de compras del comprador (Num transacciones). NO MANDATORIO.	
	parameters.put(CSMDD11", "");//Customer Cell Phone. NO MANDATORIO.		

	//Retail
	parameters.put("CSSTCITY", "Villa General Belgrano");//Ciudad de env��o de la orden. MANDATORIO.	
	parameters.put("CSSTCOUNTRY", "AR");//Pa�s de env�o de la orden. MANDATORIO.	
	parameters.put("CSSTEMAIL", "some@someurl.com");//Mail del destinatario, MANDATORIO.			
	parameters.put("CSSTFIRSTNAME", "Juan");//Nombre del destinatario. MANDATORIO.		
	parameters.put("CSSTLASTNAME", "Perez");//Apellido del destinatario. MANDATORIO.		
	parameters.put("CSSTPHONENUMBER", "541160913988");//N�mero de tel�fono del destinatario. MANDATORIO.	
	parameters.put("CSSTPOSTALCODE", "1010");//C�digo postal del domicilio de env�o. MANDATORIO.		
	parameters.put("CSSTSTATE", "B");//Provincia de env�o. MANDATORIO. Son de 1 caracter			
	parameters.put("CSSTSTREET1", "Some Street 2153");//Domicilio de env�o. MANDATORIO.		
	parameters.put("CSSTSTREET2", "Piso 8");//Complemento del domicilio. (piso, departamento). NO MANDATORIO.
	parameters.put("CSMDD12", "");//Shipping DeadLine (Num Dias). NO MADATORIO.		
	parameters.put("CSMDD13", "");//M�todo de Despacho. NO MANDATORIO.		
	parameters.put("CSMDD14", "");//Customer requires Tax Bill ? (Y/N). NO MANDATORIO.		
	parameters.put("CSMDD15", "");//Customer Loyality Number. NO MANDATORIO. 		
	parameters.put("CSMDD16", "");//Promotional / Coupon Code. NO MANDATORIO. 		
	
	//datos a enviar por cada producto, los valores deben estar separado con #:		
	parameters.put("CSITPRODUCTCODE", "electronic_good");//C�digo de producto. MANDATORIO. Valores posibles(adult_content;coupon;default;electronic_good;electronic_software;gift_certificate;handling_only;service;shipping_and_handling;shipping_only;subscription)	
	parameters.put("CSITPRODUCTDESCRIPTION", "Test Prd Description");//Descripci�n del producto. MANDATORIO.	
	parameters.put("CSITPRODUCTNAME", "TestPrd");//Nombre del producto. CONDICIONAL.	
	parameters.put("CSITPRODUCTSKU", "SKU1234");//C�digo identificador del producto. MANDATORIO.		
	parameters.put("CSITTOTALAMOUNT", "10.01");//CSITTOTALAMOUNT=CSITUNITPRICE*CSITQUANTITY "999999[.CC]" Con decimales opcional usando el punto como separador de decimales. No se permiten comas, ni como separador de miles ni como separador de decimales. MANDATORIO.		
	parameters.put("CSITQUANTITY", "1");//Cantidad del producto. CONDICIONAL.		
	parameters.put("CSITUNITPRICE", "10.01");//Formato Idem CSITTOTALAMOUNT. CONDICIONAL.	
}	
```

[<sub>Volver a inicio</sub>](#inicio)
<br>

<a name="caracteristicas"></a>
## Caracter�stica

<a name="status"></a>
####Status de la Operaci�n
La SDK cuenta con un m�todo para consultar el status de la transacci�n desde la misma SDK. El m�todo se utiliza de la siguiente manera:

```java

private static Map<String, String> getSParameters(){
	Map<String, String> parameters = new HashMap<String, String>();
	parameters.put("Merchant", "12345678");
	parameters.put("OperationID", "8000");
	return parameters;
}	

Map<String, Object> d = tpc.getStatus(getSParameters());// Merchant es el id site y OperationID es el id operaci�n que se envi� en el array a trav�s del m�todo sendAuthorizeRequest() 
```
El siguiente m�todo retornar� el status actual de la transacci�n en Todopago.

<ins><strong>Ejemplo de Respuesta</strong></ins>

```java
Map<String, Object>
	{ OperationsColections = {
			Operations = {
				RESULTCODE = 999 ,
				RESULTMESSAGE = RECHAZADA,
				DATETIME = 2015-05-13T14:11:38.287+00:00,
				OPERATIONID = 01,
				CURRENCYCODE = 32,
				AMOUNT = 54,
				TYPE = compra_online,
				INSTALLMENTPAYMENTS = 4,
				CUSTOMEREMAIL = jose@perez.com,
				IDENTIFICATIONTYPE = DNI,
				IDENTIFICATION = 1212121212,
				CARDNUMBER = 12121212XXXXXX1212,
				CARDHOLDERNAME = Jose Perez,
				TICKETNUMBER = 0,
				AUTHORIZATIONCODE = null,
				BARCODE = null,
				COUPONEXPDATE = null,
				COUPONSECEXPDATE = null,
     			COUPONSUBSCRIBER = null,
				BANKID = 11,
				PAYMENTMETHODTYPE = Cr�dito,
				PAYMENTMETHODCODE = 42,
				PROMOTIONID = 2706,
                AMOUNTBUYER = 10.00,
                PAYMENTMETHODNAME = VISA,
				PUSHNOTIFYENDPOINT = null,
				PUSHNOTIFYMETHOD = null,
				PUSHNOTIFYSTATES = null,
				REFUNDED = null, 
				REFUNDS = { REFUND0 = { ID = 50163419,
										DATETIME = 2016-03-18T16:03:54.987-03:00,
										AMOUNT = 10.00 },
							REFUND1 = { ID = 50163416,
										DATETIME = 2016-03-18T15:52:07.877-03:00,
										AMOUNT = 2.00 },		
    					    REFUND2 = { ID = 50163414,
										DATETIME = 2016-03-18T15:51:17.447-03:00,
										AMOUNT = 2.00 }
							}
						}
				}	
	}					
```
Adem�s, se puede conocer el estado de las transacciones a trav�s del portal [www.todopago.com.ar](http://www.todopago.com.ar/). Desde el portal se ver�n los estados "Aprobada" y "Rechazada". Si el m�todo de pago elegido por el comprador fue Pago F�cil o RapiPago, se podr�n ver en estado "Pendiente" hasta que el mismo sea pagado.

<a name="statusdate"></a>
####Consulta de operaciones por rango de tiempo
En este caso hay que llamar a getByRangeDateTime() y devolvera todas las operaciones realizadas en el rango de fechas dado

```java
private static Map<String, String> getBRYParameters() {
		Map<String, String> parameters = new HashMap<String, String>();
		parameters.put(ElementNames.Merchant, "12345678");
		parameters.put(ElementNames.STARTDATE, "2016-01-01");
		parameters.put(ElementNames.ENDDATE, "2016-03-03");
		parameters.put(ElementNames.PAGENUMBER, "1");	
		return parameters;
}
	
Map<String, Object> j = tpc.getByRangeDateTime(getBRYParameters());
```	

<a name="devolucion"></a>
####Devoluci�n

La SDK dispone de m�todos para realizar la devoluci�n, de una transacci�n realizada a traves de TodoPago.

Se debe llamar al m�todo ```voidRequest``` de la siguiente manera:

```java
private static Map<String, String> getVRParameters() {
		Map<String, String> parameters = new HashMap<String, String>();
		parameters.put(ElementNames.Security, "1234567890ABCDEF1234567890ABCDEF"); // API Key del comercio asignada por TodoPago 
		parameters.put(ElementNames.Merchant, "12345678");// Merchant o Nro de comercio asignado por TodoPago
		parameters.put(ElementNames.RequestKey, "e31d340c-690c-afe6-c478-fc1bef3fc157");  // RequestKey devuelto como respuesta del servicio SendAutorizeRequest
		return parameters;
}
	
Map<String, Object> h = tpc.voidRequest(getVRParameters());
```

Tambi�n se puede llamar al m�todo ```voidRequest``` de la esta otra manera:

```java
private static Map<String, String> getVRParameters() {
		Map<String, String> parameters = new HashMap<String, String>();
		parameters.put(ElementNames.Security, "1234567890ABCDEF1234567890ABCDEF"); // API Key del comercio asignada por TodoPago 
		parameters.put(ElementNames.Merchant, "12345678");// Merchant o Nro de comercio asignado por TodoPago
		parameters.put(ElementNames.AuthorizationKey, "6d2589f2-37e6-1334-7565-3dc19404480c"); // AuthorizationKey devuelto como respuesta del servicio GetAuthorizeAnswer
		return parameters;
}
	
Map<String, Object> h = tpc.voidRequest(getVRParameters());	
```

**Respuesta del servicio:**
Si la operaci�n fue realizada correctamente se informar� con un c�digo 2011 y un mensaje indicando el �xito de la operaci�n.

```java
Map<String, Object> 
	{ StatusCode = 2011,
	  StatusMessage = Operaci�n realizada correctamente }
```
<br>

<a name="devolucionparcial"></a>
####Devoluci�n parcial

La SDK dispone de m�todos para realizar la devoluci�n parcial, de una transacci�n realizada a traves de TodoPago.

Se debe llamar al m�todo ```returnRequest``` de la siguiente manera:

```java
private static Map<String, String> getRRParameters() {
		Map<String, String> parameters = new HashMap<String, String>();
		parameters.put(ElementNames.Security, "1234567890ABCDEF1234567890ABCDEF"); // API Key del comercio asignada por TodoPago 
		parameters.put(ElementNames.Merchant, "12345678"); // Merchant o Nro de comercio asignado por TodoPago
		parameters.put(ElementNames.RequestKey, "e31d340c-690c-afe6-c478-fc1bef3fc157");  // RequestKey devuelto como respuesta del servicio SendAutorizeRequest
		parameters.put(ElementNames.Amount, "10.5"); // Opcional. Monto a devolver, si no se env�a, se trata de una devoluci�n total
		return parameters;
}
Map<String, Object> i = tpc.returnRequest(getRRParameters());
```

Tambi�n se puede llamar al m�todo ```returnRequest``` de la esta otra manera:

```java
private static Map<String, String> getRRParameters() {
		Map<String, String> parameters = new HashMap<String, String>();
		parameters.put(ElementNames.Security, "1234567890ABCDEF1234567890ABCDEF"); // API Key del comercio asignada por TodoPago 
		parameters.put(ElementNames.Merchant, "12345678"); // Merchant o Nro de comercio asignado por TodoPago
		parameters.put(ElementNames.AuthorizationKey, "6d2589f2-37e6-1334-7565-3dc19404480c");  // AuthorizationKey devuelto como respuesta del servicio GetAuthorizeAnswer
		parameters.put(ElementNames.Amount, "10.5"); // Opcional. Monto a devolver, si no se env�a, se trata de una devoluci�n total
		return parameters;
}
Map<String, Object> i = tpc.returnRequest(getRRParameters());	
```

**Respuesta de servicio:**
Si la operaci�n fue realizada correctamente se informar� con un c�digo 2011 y un mensaje indicando el �xito de la operaci�n.

```java
Map<String, Object> 
	{ StatusCode = 2011,
	  StatusMessage = Operaci�n realizada correctamente }
```
<br>

<a name="formhidrido"></a>
####Formulario hibrido

**Conceptos basicos**<br>
El formulario hibrido, es una alternativa al medio de pago actual por redirecci�n al formulario externo de TodoPago.<br> 
Con el mismo, se busca que el comercio pueda adecuar el look and feel del formulario a su propio dise�o.

**Libreria**<br>
El formulario requiere incluir en la pagina una libreria Javascript de TodoPago.<br>
El endpoint depende del entorno:
+ Desarrollo: https://developers.todopago.com.ar/resources/TPHybridForm-v0.1.js
+ Produccion: https://forms.todopago.com.ar/resources/TPHybridForm-v0.1.js

**Restricciones y libertades en la implementaci�n**

+ Ninguno de los campos del formulario podr� contar con el atributo name.
+ Se deber� proveer de manera obligatoria un bot�n para gestionar el pago con Billetera Todo Pago.
+ Todos los elementos de tipo <option> son completados por la API de Todo Pago.
+ Los campos tienen un id por defecto. Si se prefiere utilizar otros ids se deber�n especificar los
mismos cuando se inicialice el script de Todo Pago.
+ Pueden aplicarse todos los detalles visuales que se crean necesarios, la API de Todo Pago no
altera los atributos class y style.
+ Puede utilizarse la API para setear los atributos placeholder del formulario, para ello deber�
especificar dichos placeholders en la inicializaci�n del formulario "window.TPFORMAPI.hybridForm.setItem". En caso de que no se especifiquen los placeholders se usar�n los valores por defecto de la API.

**HTML del formulario**

El formulario implementado debe contar al menos con los siguientes campos.

```html
<body>
	<select id="formaDePagoCbx"></select>
	<select id="bancoCbx"></select>
	<select id="promosCbx"></select>
	<label id="labelPromotionTextId"></label>
	<input id="numeroTarjetaTxt"/>
	<input id="mesTxt"/>
	<input id="anioTxt"/>
	<input id="codigoSeguridadTxt"/>
	<label id="labelCodSegTextId"></label>
	<input id="apynTxt"/>
	<select id="tipoDocCbx"></select>
	<input id="nroDocTxt"/>
	<input id="emailTxt"/><br/>
	<button id="MY_btnConfirmarPago"/>
</body>
```

**Inizializaci�n y parametros requeridos**<br>
Para inicializar el formulario se usa window.TPFORMAPI.hybridForm.initForm. El cual permite setear los elementos y ids requeridos.

Para inicializar un �tem de pago, es necesario llamar a window.TPFORMAPI.hybridForm.setItem. Este requiere obligatoriamente el parametro publicKey que corresponde al PublicRequestKey (entregado por el SAR).
Se sugiere agregar los parametros usuario, e-mail, tipo de documento y numero.

**Javascript**
```js
window.TPFORMAPI.hybridForm.initForm({
    callbackValidationErrorFunction: 'validationCollector',
	callbackCustomSuccessFunction: 'customPaymentSuccessResponse',
	callbackCustomErrorFunction: 'customPaymentErrorResponse',
	botonPagarId: 'MY_btnConfirmarPago',
	modalCssClass: 'modal-class',
	modalContentCssClass: 'modal-content',
	beforeRequest: 'initLoading',
	afterRequest: 'stopLoading'
});

window.TPFORMAPI.hybridForm.setItem({
    publicKey: 'taf08222e-7b32-63d4-d0a6-5cabedrb5782', //obligatorio
    defaultNombreApellido: 'Usuario',
    defaultNumeroDoc: 20234211,
    defaultMail: 'todopago@mail.com',
    defaultTipoDoc: 'DNI'
});

//callbacks de respuesta del pago
function validationCollector(parametros) {
}
function customPaymentSuccessResponse(response) {
}
function customPaymentErrorResponse(response) {
}
function initLoading() {
}
function stopLoading() {
}
```

**Callbacks**<br>
El formulario define callbacks javascript, que son llamados seg�n el estado y la informacion del pago realizado:
+ customPaymentSuccessResponse: Devuelve response si el pago se realizo correctamente.
+ customPaymentErrorResponse: Si hubo algun error durante el proceso de pago, este devuelve el response con el codigo y mensaje correspondiente.

**Ejemplo de Implementaci�n**:
<a href="/resources/form_hibrido-ejemplo/index.html" target="_blank">Formulario hibrido</a>
<br>

[<sub>Volver a inicio</sub>](#inicio)

<a name="credenciales"></a>
####Obtener credenciales
El SDK permite obtener las credenciales "Authentification", "MerchandId" y "Security" de la cuenta de Todo Pago, ingresando el usuario y contrase�a.<br>
Esta funcionalidad es util para obtener los parametros de configuracion dentro de la implementacion.

- Crear una instancia de la clase User:
```java

public void getCredentials(TodoPagoConector tpc) {
		
		User user = new User("test@test.com", "test1234");// user y pass de TodoPago
		
		try {
			user = tpc.getCredentials(user);
			tpc.setAuthorize(getAuthorization(user));// set de la APIKey a TodoPagoConector 
			
		} catch (EmptyFieldException e) {//se debe realizar catch por campos en blanco
			logger.log(Level.WARNING, e.getMessage());						
		} catch (MalformedURLException e) {
			logger.log(Level.WARNING, e.getMessage());	
		} catch (ResponseException e) {
			logger.log(Level.WARNING, e.getMessage());
		} catch (ConnectionException e) {
			logger.log(Level.WARNING, e.getMessage());
		}
		System.out.println(user.toString());	
}
	
	private Map<String, List<String>> getAuthorization(User user) {
		Map<String, List<String>> parameters = new HashMap<String, List<String>>();
		parameters.put(ElementNames.Authorization,Collections.singletonList(user.getApiKey()));
		
		return parameters;
}
```
[<sub>Volver a inicio</sub>](#inicio)
<br>

<a name="maxcuotas"></a>
####M�ximo de cuotas a mostrar en formulario
Mediante esta funcionalidad, se permite setear el n�mero m�ximo de cuotas que se desplegar� en el formulario de pago.
 
Para hacer uso de esta funcionalidad debe agregarse en el **Map<String, String> parameters** del m�todo **sendAuthorizeRequest** el campo **MAXINSTALLMENTS** con el valor m�ximo de cuotas a ofrecer (generalmente de 1 a 12)
 
#####Ejemplo
 
```java		
Map<String, String> parameters = new HashMap<String, String>();
parameters.put(ElementNames.MAXINSTALLMENTS, "12");	
```
 
 [<sub>Volver a inicio</sub>](#inicio)
 <br>
 

<a name="secuencia"></a>
##Diagrama de secuencia
![imagen de configuracion](https://raw.githubusercontent.com/TodoPago/imagenes/master/README.img/secuencia-page-001.jpg)

[<sub>Volver a inicio</sub>](#inicio)
<br>

<a name="tablareferencia"></a>    
## Tablas de Referencia   
######[Provincias](#p)    
				
<p>Solo utilizado para incluir los datos de control de fraude</p>
<table>		
<tr><th>Provincia</th><th>C�digo</th></tr>		
<tr><td>CABA</td><td>C</td></tr>		
<tr><td>Buenos Aires</td><td>B</td></tr>		
<tr><td>Catamarca</td><td>K</td></tr>		
<tr><td>Chaco</td><td>H</td></tr>		
<tr><td>Chubut</td><td>U</td></tr>		
<tr><td>C�rdoba</td><td>X</td></tr>		
<tr><td>Corrientes</td><td>W</td></tr>		
<tr><td>Entre R�os</td><td>E</td></tr>		
<tr><td>Formosa</td><td>P</td></tr>		
<tr><td>Jujuy</td><td>Y</td></tr>		
<tr><td>La Pampa</td><td>L</td></tr>		
<tr><td>La Rioja</td><td>F</td></tr>		
<tr><td>Mendoza</td><td>M</td></tr>		
<tr><td>Misiones</td><td>N</td></tr>		
<tr><td>Neuqu�n</td><td>Q</td></tr>		
<tr><td>R�o Negro</td><td>R</td></tr>		
<tr><td>Salta</td><td>A</td></tr>		
<tr><td>San Juan</td><td>J</td></tr>		
<tr><td>San Luis</td><td>D</td></tr>		
<tr><td>Santa Cruz</td><td>Z</td></tr>		
<tr><td>Santa Fe</td><td>S</td></tr>		
<tr><td>Santiago del Estero</td><td>G</td></tr>		
<tr><td>Tierra del Fuego</td><td>V</td></tr>		
<tr><td>Tucum�n</td><td>T</td></tr>		
</table>

[<sub>Volver a inicio</sub>](#inicio)

<a name="codigoerrores"></a>    
## Tabla de errores     

<table>		
<tr><th>Id mensaje</th><th>Mensaje</th></tr>				
<tr><td>-1</td><td>Aprobada.</td></tr>
<tr><td>1081</td><td>Tu saldo es insuficiente para realizar la transacci�n.</td></tr>
<tr><td>1100</td><td>El monto ingresado es menor al m�nimo permitido</td></tr>
<tr><td>1101</td><td>El monto ingresado supera el m�ximo permitido.</td></tr>
<tr><td>1102</td><td>La tarjeta ingresada no corresponde al Banco indicado. Revisalo.</td></tr>
<tr><td>1104</td><td>El precio ingresado supera al m�ximo permitido.</td></tr>
<tr><td>1105</td><td>El precio ingresado es menor al m�nimo permitido.</td></tr>
<tr><td>2010</td><td>En este momento la operaci�n no pudo ser realizada. Por favor intent� m�s tarde. Volver a Resumen.</td></tr>
<tr><td>2031</td><td>En este momento la validaci�n no pudo ser realizada, por favor intent� m�s tarde.</td></tr>
<tr><td>2050</td><td>Lo sentimos, el bot�n de pago ya no est� disponible. Comunicate con tu vendedor.</td></tr>
<tr><td>2051</td><td>La operaci�n no pudo ser procesada. Por favor, comunicate con tu vendedor.</td></tr>
<tr><td>2052</td><td>La operaci�n no pudo ser procesada. Por favor, comunicate con tu vendedor.</td></tr>
<tr><td>2053</td><td>La operaci�n no pudo ser procesada. Por favor, intent� m�s tarde. Si el problema persiste comunicate con tu vendedor</td></tr>
<tr><td>2054</td><td>Lo sentimos, el producto que quer�s comprar se encuentra agotado por el momento. Por favor contactate con tu vendedor.</td></tr>
<tr><td>2056</td><td>La operaci�n no pudo ser procesada. Por favor intent� m�s tarde.</td></tr>
<tr><td>2057</td><td>La operaci�n no pudo ser procesada. Por favor intent� m�s tarde.</td></tr>
<tr><td>2059</td><td>La operaci�n no pudo ser procesada. Por favor intent� m�s tarde.</td></tr>
<tr><td>90000</td><td>La cuenta destino de los fondos es inv�lida. Verific� la informaci�n ingresada en Mi Perfil.</td></tr>
<tr><td>90001</td><td>La cuenta ingresada no pertenece al CUIT/ CUIL registrado.</td></tr>
<tr><td>90002</td><td>No pudimos validar tu CUIT/CUIL.  Comunicate con nosotros <a href="#contacto" target="_blank">ac�</a> para m�s informaci�n.</td></tr>
<tr><td>99900</td><td>El pago fue realizado exitosamente</td></tr>
<tr><td>99901</td><td>No hemos encontrado tarjetas vinculadas a tu Billetera. Pod�s  adherir medios de pago desde www.todopago.com.ar</td></tr>
<tr><td>99902</td><td>No se encontro el medio de pago seleccionado</td></tr>
<tr><td>99903</td><td>Lo sentimos, hubo un error al procesar la operaci�n. Por favor reintent� m�s tarde.</td></tr>
<tr><td>99970</td><td>Lo sentimos, no pudimos procesar la operaci�n. Por favor reintent� m�s tarde.</td></tr>
<tr><td>99971</td><td>Lo sentimos, no pudimos procesar la operaci�n. Por favor reintent� m�s tarde.</td></tr>
<tr><td>99977</td><td>Lo sentimos, no pudimos procesar la operaci�n. Por favor reintent� m�s tarde.</td></tr>
<tr><td>99978</td><td>Lo sentimos, no pudimos procesar la operaci�n. Por favor reintent� m�s tarde.</td></tr>
<tr><td>99979</td><td>Lo sentimos, el pago no pudo ser procesado.</td></tr>
<tr><td>99980</td><td>Ya realizaste un pago en este sitio por el mismo importe. Si quer�s realizarlo nuevamente esper� 5 minutos.</td></tr>
<tr><td>99982</td><td>En este momento la operaci�n no puede ser realizada. Por favor intent� m�s tarde.</td></tr>
<tr><td>99983</td><td>Lo sentimos, el medio de pago no permite la cantidad de cuotas ingresadas. Por favor intent� m�s tarde.</td></tr>
<tr><td>99984</td><td>Lo sentimos, el medio de pago seleccionado no opera en cuotas.</td></tr>
<tr><td>99985</td><td>Lo sentimos, el pago no pudo ser procesado.</td></tr>
<tr><td>99986</td><td>Lo sentimos, en este momento la operaci�n no puede ser realizada. Por favor intent� m�s tarde.</td></tr>
<tr><td>99987</td><td>Lo sentimos, en este momento la operaci�n no puede ser realizada. Por favor intent� m�s tarde.</td></tr>
<tr><td>99988</td><td>Lo sentimos, momentaneamente el medio de pago no se encuentra disponible. Por favor intent� m�s tarde.</td></tr>
<tr><td>99989</td><td>La tarjeta ingresada no est� habilitada. Comunicate con la entidad emisora de la tarjeta para verificar el incoveniente.</td></tr>
<tr><td>99990</td><td>La tarjeta ingresada est� vencida. Por favor seleccion� otra tarjeta o actualiz� los datos.</td></tr>
<tr><td>99991</td><td>Los datos informados son incorrectos. Por favor ingresalos nuevamente.</td></tr>
<tr><td>99992</td><td>La fecha de vencimiento es incorrecta. Por favor seleccion� otro medio de pago o actualiz� los datos.</td></tr>
<tr><td>99993</td><td>La tarjeta ingresada no est� vigente. Por favor seleccion� otra tarjeta o actualiz� los datos.</td></tr>
<tr><td>99994</td><td>El saldo de tu tarjeta no te permite realizar esta operacion.</td></tr>
<tr><td>99995</td><td>La tarjeta ingresada es invalida. Seleccion� otra tarjeta para realizar el pago.</td></tr>
<tr><td>99996</td><td>La operaci�n fu� rechazada por el medio de pago porque el monto ingresado es inv�lido.</td></tr>
<tr><td>99997</td><td>Lo sentimos, en este momento la operaci�n no puede ser realizada. Por favor intent� m�s tarde.</td></tr>
<tr><td>99998</td><td>Lo sentimos, la operaci�n fue rechazada. Comunicate con la entidad emisora de la tarjeta para verificar el incoveniente o seleccion� otro medio de pago.</td></tr>
<tr><td>99999</td><td>Lo sentimos, la operaci�n no pudo completarse. Comunicate con la entidad emisora de la tarjeta para verificar el incoveniente o seleccion� otro medio de pago.</td></tr>
</table>

[<sub>Volver a inicio</sub>](#inicio)

<a name="eclipse"></a>		
## Agregar el proyecto a Eclipse		
- Bajar Maven 3 de la siguiente URL: http://maven.apache.org/download.cgi
- Descomprimir lo descargado.
- Agregar o modificar la variable de entorno M2_HOME con path a donde se descomprime Maven 
- Descargar el proyecto de GitHub
- Por consola ir a la carpeta donde se descargo el proyecto
- Ejecutar las siguientes lineas:
	- mvn clean install -Dwtpversion=2.0
	- mvn eclipse:clean eclipse:eclipse -Dwtpversion=2.0
- Ir a Eclipse e importar el Proyecto normalmente: File - Import - Existing Projects into Workspace

<br />		
[<sub>Volver a inicio</sub>](#inicio)