# api-to-soap-shell

# Configurar un Gateway REST a SOAP con Bash

En este tutorial, aprenderás cómo configurar un script de Bash simple como un gateway entre Micro Focus Advanced Authentication Framework (AAF) y una pasarela SMS para realizar la conversión de solicitudes REST a solicitudes SOAP.

**Paso 1: Preparación del Entorno**

Asegúrate de tener una instancia de SUSE Linux Enterprise Server (SLES) instalada y configurada en tu servidor. También necesitarás un servicio AAF que envíe solicitudes REST y una pasarela SMS que acepte solicitudes SOAP.

**Paso 2: Crear el Script Bash**

Crea un archivo Bash, por ejemplo, `rest_to_soap_gateway.sh`, y agrégale el siguiente contenido:

```bash
#!/bin/bash

# Configura el puerto en el que escuchará el script (por ejemplo, puerto 8080)
PORT=8080

# Función para convertir solicitud REST a SOAP
convert_rest_to_soap() {
    # Implementa aquí la lógica de conversión de REST a SOAP
    # Debes manipular la cadena XML según tus necesidades

    # Ejemplo: Conversión simple de JSON a XML
    soap_request=$(cat <<EOL
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
  <soapenv:Header/>
  <soapenv:Body>
    <sms:SendSms xmlns:sms="http://example.com/sms">
      <sms:message>Hello, SMS Gateway!</sms:message>
    </sms:SendSms>
  </soapenv:Body>
</soapenv:Envelope>
EOL
)

    echo "$soap_request"
}

# Inicia el servidor web Bash en el puerto especificado
while true; do
    # Escucha en el puerto y procesa las solicitudes
    nc -l -p "$PORT" -c "read -r request; response=\$(convert_rest_to_soap \"\$request\"); echo -e \"HTTP/1.1 200 OK\nContent-Length: \${#response}\n\n\$response\""
done
```
Este script convierte los parámetros de la solicitud REST en el formato SOAP deseado y los envía como respuesta.

El comando qanterior de `netcat` que se encuentra en sistemas basados en GNU/Linux y utiliza la opción `-c` para ejecutar comandos al recibir una conexión. Sin embargo, en la implementación de `netcat` que se encuentra en sistemas BSD, como `netcat-openbsd`, no se admite la opción `-c` para ejecutar comandos en la misma línea.

Para lograr un comportamiento similar en `netcat-openbsd`, puedes usar una solución basada en un bucle infinito con `while`. Aquí está el equivalente utilizando `netcat-openbsd`:

```bash
#!/bin/bash

# Configura el puerto en el que escuchará el script (por ejemplo, puerto 8080)
PORT=8080

# Función para convertir solicitud REST a SOAP
convert_rest_to_soap() {
    # Implementa aquí la lógica de conversión de REST a SOAP
    # Debes manipular la cadena XML según tus necesidades

    # Ejemplo: Conversión simple de JSON a XML
    soap_request=$(cat <<EOL
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
  <soapenv:Header/>
  <soapenv:Body>
    <sms:SendSms xmlns:sms="http://example.com/sms">
      <sms:message>Hello, SMS Gateway!</sms:message>
    </sms:SendSms>
  </soapenv:Body>
</soapenv:Envelope>
EOL
)

    echo "$soap_request"
}

# Inicia el servidor web Bash en el puerto especificado con netcat-openbsd
while true; do
    # Escucha en el puerto
    { echo -ne "HTTP/1.1 200 OK\r\nContent-Length: $((${#response} + 2))\r\n\r\n"; convert_rest_to_soap "$(cat)"; } | nc -l -p "$PORT"
done
```
Este script logra un resultado similar utilizando el bucle `while` para recibir la solicitud REST y luego responder con la conversión SOAP correspondiente.

En mi caso el paquete que tengo instalado es `netcat-openbsd-1.89-92.3.1.x86_64`. La implementación de `netcat` en OpenBSD es conocida por no admitir la opción `-c` para ejecutar comandos directamente. En su lugar, se puede usar una tubería y un bucle `while` para lograr un comportamiento similar.

```bash
#!/bin/bash

# Configura el puerto en el que escuchará el script (por ejemplo, puerto 8080)
PORT=8080

# Función para convertir solicitud REST a SOAP
convert_rest_to_soap() {
    # Implementa aquí la lógica de conversión de REST a SOAP
    # Debes manipular la cadena XML según tus necesidades

    # Ejemplo: Conversión simple de JSON a XML
    soap_request=$(cat <<EOL
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
  <soapenv:Header/>
  <soapenv:Body>
    <sms:SendSms xmlns:sms="http://example.com/sms">
      <sms:message>Hello, SMS Gateway!</sms:message>
    </sms:SendSms>
  </soapenv:Body>
</soapenv:Envelope>
EOL
)

    echo "$soap_request"
}

# Inicia el servidor web Bash en el puerto especificado con netcat-openbsd
while true; do
    # Escucha en el puerto y procesa las solicitudes
    {
        while IFS= read -r line; do
            if [ -z "$request" ]; then
                request="$line"
            else
                request="$request$line"
            fi
        done
        response=$(convert_rest_to_soap "$request")
        echo -ne "HTTP/1.1 200 OK\r\nContent-Length: ${#response}\r\n\r\n$response"
    } | nc -l -p "$PORT"
done
```
Este script debería funcionar con la implementación de `netcat` en OpenBSD, ya que utiliza un bucle `while` para recibir la solicitud REST y luego responder con la conversión SOAP correspondiente. Asegúrate de que el contenido de la función `convert_rest_to_soap()` esté configurado correctamente según tus necesidades específicas.


**Paso 3: Ejecutar el Script Bash**

1. Guarda el archivo Bash en tu servidor SLES y asegúrate de darle permisos de ejecución.

```bash
chmod +x rest_to_soap_gateway.sh
```

2. Ejecuta el script:

```bash
./rest_to_soap_gateway.sh
```

El script Bash ahora escuchará en el puerto especificado (por ejemplo, 8080) y convertirá las solicitudes REST en solicitudes SOAP utilizando la función `convert_rest_to_soap()`.

**Paso 4: Configurar AAF para Enviar Solicitudes REST**

Configura AAF para enviar sus solicitudes REST al puerto y la dirección del servidor en el que se ejecuta el script Bash. Asegúrate de que AAF esté configurado para enviar las solicitudes REST en el formato esperado por el script.

**Paso 5: Probar la Integración**

Envía una solicitud REST desde AAF al puerto donde se ejecuta el script Bash. El script debería recibir la solicitud, convertirla en SOAP y enviarla a la pasarela SMS.

Ten en cuenta que este es un ejemplo muy simplificado y no aborda cuestiones como la seguridad, el manejo de errores, la escalabilidad ni la gestión de solicitudes concurrentes. En un entorno de producción, es preferible utilizar una solución más robusta y escalable para esta integración.

**Paso 6: Agregar el Script como un Servicio en Linux**

Para agregar el script como un servicio en Linux y habilitarlo en el arranque, sigue estos pasos:

1. Crea un archivo de servicio systemd. Por ejemplo, crea un archivo llamado `rest_to_soap_gateway.service` en el directorio `/etc/systemd/system/` con el siguiente contenido:

```ini
[Unit]
Description=Script para convertir solicitudes REST en SOAP

[Service]
ExecStart=/ruta/al/script/rest_to_soap_gateway.sh
Restart=always
User=usuario
Group=grupo
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=rest_to_soap_gateway

[Install]
WantedBy=multi-user.target
```

Asegúrate de reemplazar `/ruta/al/script/rest_to_soap_gateway.sh` con la ruta completa hacia tu script y de especificar el usuario y el grupo bajo los cuales deseas que se ejecute el servicio.

2. Guarda el archivo y luego recarga la configuración del sistema systemd:

```bash
sudo systemctl daemon-reload
```

3. Habilita el servicio para que se inicie automáticamente en el inicio del sistema:

```bash
sudo systemctl enable rest_to_soap_gateway.service
```

4. Inicia el servicio:

```bash
sudo systemctl start rest_to_soap_gateway.service
```

Ahora, el script se ejecutará como un servicio en el inicio del sistema y escuchará en el puerto configurado para convertir las solicitudes REST en solicitudes SOAP.

```
