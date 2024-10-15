title: Documentación de la API - AbuseIPDB
-------------------------------------------

!!! Advertencia

    La APIv1 queda obsoleta en favor de la APIv2. La fecha de expiración es 2020-02-01. Los tickets de soporte para APIv1 no recibirán respuesta, excepto en circunstancias especiales. Esta página permanecerá aquí con fines de archivo.
[//]: # (end-Advertencia)

**AbuseIPDB ofrece una API gratuita para denunciar y comprobar direcciones IP**. Todos los días, webmasters, administradores de sistemas y otros profesionales de TI utilizan nuestra API para denunciar en tiempo real miles de direcciones IP comprometidas con el spam, el hacking, el escaneo de vulnerabilidades y otras actividades maliciosas.

Esta API le permite proteger su red comprobando las direcciones IP en nuestra base de datos y le permite contribuir enviando las direcciones IP maliciosas que detecte. El uso de la API es gratuito, pero hay que [crear una cuenta](https://www.abuseipdb.com/register).

Si eres un webmaster o administrador de sistemas interesado en reportar automáticamente IPs abusivas a través de la API AbuseIPDB, te recomendamos que veas también el [tutorial de integración Fail2Ban](fail2ban.md).

Puntos finales de la API
------------------------

Pueden utilizarse los métodos GET y POST.

### Comprobar IP

[https://www.abuseipdb.com/check/[IP]/json?key=[API_KEY]&days=[DAYS]](https://www.abuseipdb.com/check/[IP]/json?key=[API_KEY]&days=[DAYS])

### Comprobar CIDR

[https://www.abuseipdb.com/check-block/json?network=[CIDR]&key=[API_KEY]&days=[DAYS]](https://www.abuseipdb.com/check-block/json?network=[CIDR]&key=[API_KEY]&days=[DAYS])

### Informe IP

[https://www.abuseipdb.com/report/json?key=[API_KEY]&category=[CATEGORIES]&comment=[COMMENT]&ip=[IP]](https://www.abuseipdb.com/report/json?key=[API_KEY]&category=[CATEGORIES]&comment=[COMMENT]&ip=[IP])

!!! Sugerencia

    Utilice direcciones IP privadas para probar la API (por ejemplo, 127.0.0.1).

[//]: # (end-Sugerencia)

### Explicación de los parámetros


| Campo           | Requerido | Por defecto | Ejemplo                                 | Descripción                                                                                                                                                      |
| ----------------- | ----------- | ------------- | ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [IP]            | Yes       | **NA**      | 8.8.8.8                                 | IPv4 or IPv6 Address                                                                                                                                              |
| [DAYS]          | No        | 30          | 30                                      | Comprobar si hay informes de IP en los últimos 30 días                                                                                                          |
| [API_KEY]       | Yes       | **NA**      | Tzmp1...quWvaiO                         | Su clave API[(Obtenga una clave API)](https://www.abuseipdb.com/account))                                                                                         |
| [CATEGORIES]    | Yes       | **NA**      | 10,12,15                                | Lista delimitada por comas de ID de categoría ([Ver todas las categorías](https://www.abuseipdb.com/categories)                                                 |
| [COMMENT]       | No        | blank       | Forzar el login en Wordpress            | Describir el tipo de actividad maliciosa                                                                                                                          |
| [CIDR]          | Yes       | **NA**      | 207.126.144.0/20                        | Bloque de direcciones IPv4 en notación CIDR                                                                                                                      |
| bandera verbose | No        | FALSE       | /json?key=[API_KEY]&days=[DAYS]&verbose | Si se establece, los informes incluirán el comentario (si lo hay) y el número de identificación de usuario del informador (0 si se informa de forma anónima). |


!!! Nota

    Los comentarios de más de 1024 caracteres (bytes) se truncarán. Por favor, proporcione únicamente información relevante (no volcados de registro completos). Asegúrese de eliminar la información personal de sus comentarios si desea permanecer en el anonimato.

[//]: # (end-Nota)


Límites de tarifa
-----------------

Todas las cuentas gratuitas tienen un límite de 1.000 informes y comprobaciones al día. Las cuentas para webmasters tienen un límite de 3.000 solicitudes al día. Puede consultar sus límites de tarifa y la información de uso de la API accediendo a la [página de cuentas](https://www.abuseipdb.com/account) y haciendo clic en la pestaña "Uso de la API".

Para evitar que se envíen reportes duplicados automáticamente a través de nuestra API (como por ejemplo a través de la [integración Fail2Ban](fail2ban.md)), limitamos cada cuenta a reportar la misma IP una vez cada 15 minutos. Después de 15 minutos, si la misma IP es reportada dentro de una ventana de 24 horas con el mismo comentario, no se crea un nuevo reporte. En su lugar, se actualiza la marca de tiempo del informe original.


Seguridad
---------
### Clave API

Después de [registrarte](https://www.abuseipdb.com/register), puedes obtener tu clave API en la página de [cuentas](https://www.abuseipdb.com/account). Haz clic en la pestaña "Configuración de API" para ver tu clave de API. Puedes generar una nueva clave API en cualquier momento. Cuando generes una nueva clave API, la antigua dejará de funcionar inmediatamente.

Debe proteger su clave API como si fuera una contraseña. Cualquiera que tenga acceso a su clave API puede enviar informes en su nombre. Puede pasar su clave de API como parámetro GET, pero por motivos de seguridad le recomendamos que la pase como parámetro POST.