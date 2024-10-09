Capítulo 2: Preparando las cosas con la gestión de usuarios de Zabbix

## Gestión de usuarios

En este capítulo, trabajaremos en la creación de nuestros primeros grupos de usuarios, usuarios y roles de usuario. Es muy importante configurar estos de la manera correcta, ya que darán acceso a las personas a su entorno Zabbix con los permisos correctos. Repasando estas cosas paso a paso, nos aseguraremos de que tenemos una configuración estructurada de Zabbix antes de continuar con este libro.

Como extra, también configuraremos alguna autenticación de usuario avanzada utilizando SAML para facilitar las cosas a tus usuarios de Zabbix y proporcionarles una forma de utilizar las credenciales de inicio de sesión que ya podrían estar utilizando en toda tu empresa. Vamos a repasar todos estos pasos en el orden de las siguientes recetas:

- Creación de grupos de usuarios
- Uso de los nuevos roles de usuario de Zabbix
- Creación de los primeros usuarios
- Autenticación avanzada de usuarios con SAML

## Requisitos técnicos

Podemos hacer todo el trabajo en este capítulo con cualquier configuración de Zabbix instalada. Si aún no has instalado Zabbix, consulta el capítulo anterior para aprender cómo hacerlo. Vamos a ir a través de nuestra configuración Zabbix para tener todo listo para nuestros usuarios para empezar a iniciar sesión y utilizar el frontend Zabbix.

## Creación de grupos de usuarios

Para iniciar sesión en el frontend de Zabbix, vamos a necesitar usuarios. En este momento, estamos conectados con el usuario por defecto, lo cual es lógico porque necesitamos un usuario para crear usuarios. Sin embargo, esta no es una configuración segura, porque no queremos seguir utilizando `zabbix` como contraseña. Por lo tanto, vamos a aprender a crear nuevos usuarios y agruparlos en consecuencia.

Es importante elegir cómo desea gestionar los usuarios en Zabbix antes de configurar las cuentas de usuario. Si desea utilizar algo como LDAP o SAML, es una idea inteligente para hacer la elección de utilizar uno de esos métodos de autenticación de inmediato, por lo que no tendrá ningún problema de migración.

### Preparándonos

Ahora que sabemos cómo está estructurada la interfaz de usuario de Zabbix y sabemos cómo navegar por ella, podemos empezar a hacer alguna configuración real. Empezaremos creando algunos grupos de usuarios para familiarizarnos con el proceso y empezar a utilizarlos. De esta manera, nuestra configuración de Zabbix no sólo se vuelve más estructurada, sino también más segura.

Para empezar con esto, necesitaremos un servidor Zabbix como el que usamos en las recetas anteriores y el conocimiento que hemos adquirido allí para navegar a las secciones correctas del frontend.

Observando la siguiente figura, podemos ver cómo está configurada nuestra empresa de ejemplo, Cloud Hoster. Crearemos los usuarios que se ven en el diagrama para crear una configuración de usuarios estructurada y sólida:

![Figura 2.1 - Diagrama del departamento de Cloud Hoster](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_001.png)

Figura 2.1 - Diagrama del departamento de Cloud Hoster

Así, Cloud Hoster tiene algunos departamentos que necesitan acceso al frontend de Zabbix y otros que no lo necesitan en absoluto. Digamos que queremos dar acceso al frontend de Zabbix a los siguientes departamentos:

- **Networking**: Configurar y supervisar sus dispositivos de red
- **Infrastructure**: Configurar y supervisar sus servidores Linux
- **Buying and Inventory**: Para consultar la información de inventario y compararla con otras herramientas internas

### Cómo hacerlo...

Vamos a empezar con la creación de estos tres grupos en nuestra interfaz de usuario Zabbix:

1. Para ello, navega a **Administration** | **User groups**, que te mostrará la siguiente página:

   ![Figura 2.2 - Ventana de grupos de usuarios de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_002.jpg)

   Figura 2.2 - Ventana de grupos de usuarios de Zabbix
2. Ahora, vamos a empezar por crear el grupo de trabajo en red haciendo clic en Crear grupo de usuarios en la esquina superior derecha. Esto le llevará a la siguiente pantalla:
   ![Figura 2.3 - Ventana de configuración de grupos de usuarios de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_003.jpg)

   Figura 2.3 - Ventana de configuración de grupos de usuarios de Zabbix

Necesitaremos rellenar la información, empezando por Nombre del grupo, que por supuesto será `Networking`. Todavía no hay usuarios para este grupo, así que nos saltaremos esa opción. Frontend access es la opción que nos proporcionará la autenticación; si seleccionamos LDAP aquí, se utilizará la autenticación LDAP para autenticar. Lo mantendremos como System default, que es la autenticación interna de Zabbix.

3. Ahora, vamos a navegar a la siguiente pestaña en esta página, que es **Permissions**:
   ![Figura 2.4 - Ventana de configuración de grupos de usuarios de Zabbix Permisos](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_004.jpg)

   Figura 2.4 - Ventana de configuración de grupos de usuarios de Zabbix Permisos

Aquí, podemos especificar a qué grupos de host tendrá acceso nuestro grupo. Ya existe un grupo por defecto para **Networking**, que utilizaremos en este ejemplo.

4. Haga clic en Seleccionar para acceder a una ventana emergente con los grupos de hosts disponibles. Selecciona aquí **Templates/Network devices** de red y volverás a la ventana anterior, con el grupo rellenado.
5. Selecciona **Read-write** y haz clic en el botón **Add** para añadir estos permisos.
6. No vamos a añadir nada más, así que haz clic en el botón azul más grande **Add** para terminar de crear este grupo de hosts.

**Consejo**

Cuando se utiliza la autenticación Zabbix como HTTP, LDAP o SAML, todavía tenemos que crear nuestros usuarios internamente con los permisos adecuados. Configure sus usuarios para que coincidan con el nombre de usuario de su método de autenticación en Zabbix y utilice el método de autenticación para la gestión de contraseñas.

Ahora tendremos un nuevo grupo de host llamado **Networking** que sólo podrá leer y escribir en el grupo de host Templates/Network devices:

![ Figura 2.5 - Ventana de grupos de usuarios de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_005.jpg)

Figura 2.5 - Ventana de grupos de usuarios de Zabbix

7. Repitamos este proceso para el grupo de host **Infrastructure**, excepto que en lugar de añadir el grupo de host **Templates/Network devices**, añadiremos el grupo de host **Linux servers**, de esta forma:
   ![Figura 2.6 - Ventana de configuración de permisos de grupos de usuarios de Zabbix con un grupo de hosts](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_006.jpg)

Figura 2.6 - Ventana de configuración de permisos de grupos de usuarios de Zabbix con un grupo de hosts

8. Haga clic en **Add** para guardar este grupo de host.
9. Repite los pasos de nuevo y para añadir **Buying and Inventory**, haremos algo diferente. Repetiremos el proceso que acabamos de hacer excepto la parte de los permisos. Queremos que **Buying and Inventory** puedan leer nuestros datos de inventario, pero no queremos que cambien realmente la configuración de nuestro host. Añade tanto **Templates/Network devices** y **Linux servers** al grupo, pero sólo con permisos de Lectura, de esta forma:

![Figura 2.7 - Ventana de configuración de permisos de grupos de usuarios de Zabbix con dos grupos de hosts](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_007.png)

Figura 2.7 - Ventana de configuración de permisos de grupos de usuarios de Zabbix con dos grupos de hosts

¡Enhorabuena! Terminar esto significa que has terminado con tres grupos de host diferentes y ¡podemos continuar creando nuestros primeros nuevos usuarios! Vamos a ello.

### Hay más...

Los grupos de usuarios de Zabbix son bastante extensos y hay mucho más de lo que parece al principio. Como todo el sistema de permisos se basa en qué grupo(s) de usuarios y rol(es) de usuario formas parte, siempre es una buena idea leer primero la documentación de Zabbix: https://www.zabbix.com/documentation/current/en/manual/config/users_and_usergroups/usergroup.

---

## Uso de los nuevos roles de usuario de Zabbix

Se ha introducido una nueva funcionalidad en esta versión de Zabbix 6.0 LTS. Ahora es posible crear tus propios roles de usuario en Zabbix. En versiones anteriores de Zabbix, teníamos la posibilidad de asignar uno de tres tipos de usuario:

- User
- Admin
- Super admin

Lo que estos tipos de usuario hacían en versiones anteriores era restringir lo que los usuarios de Zabbix podían ver en el frontend. Sin embargo, esto siempre estaba predefinido. Ahora, con la adición de los roles de usuario que podemos crear nosotros mismos, podemos establecer nuestras propias restricciones relacionadas con el frontend, haciendo posible mostrar sólo ciertas partes de la interfaz de usuario a determinados usuarios de Zabbix, sin dejar de respetar los permisos establecidos mediante grupos de usuarios.

### Preparación

Para esta receta, necesitaremos un servidor Zabbix, preferiblemente el configurado en la receta anterior. En la receta anterior, configuramos diferentes grupos de usuarios, para proporcionar diferentes permisos en grupos de hosts. Completamente separado del grupo de usuarios, aplicaremos ciertos roles de usuario a nuestros usuarios para determinar lo que pueden ver en la interfaz de usuario. Veamos cómo configurar nuestros roles de usuario.

### Cómo hacerlo...

1. Primero, navega al frontend de Zabbix y ve a **Administration** | **User roles**. Esto nos mostrará los roles de usuario por defecto tal y como los conoces de versiones anteriores de Zabbix.

![Figura 2.8 - Ventana de configuración por defecto de los roles de usuario de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_008.jpg)

2.8 - Ventana de configuración por defecto de los User roles de Zabbix

2. Aquí, podemos hacer clic en el botón azul **Create user role** en la esquina superior derecha.
3. Configuraremos un nuevo rol de usuario llamado  `Rol + Usuario`. Este rol será para usuarios de Zabbix que sólo tendrán permisos de lectura, pero que necesitan más acceso que sólo los elementos de navegación Monitoreo, Inventario e Informes.

![Figura 2.9 - Parte superior de la ventana de configuración de un nuevo rol de usuario de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_009.jpg)

Figura 2.9 - Parte superior de la ventana de configuración de un nuevo rol de usuario de Zabbix

4. Lo primero es lo primero, asegúrese de rellenar Nombre como `usuario + rol`.
5. Centrémonos primero en la parte donde dice **Access to UI elements**. Cuando se selecciona **User** para el **User type**, no somos capaces de añadir algún acceso al rol de usuario. Así que vamos a cambiar el tipo de usuario seleccionando **Admin** en el menú desplegable.
6. Específicamente quiero que este rol de usuario llamado `Usuario + role` tenga la habilidad de acceder a la página de mantenimiento. Configurando esto se verá así:
   ![ Figura 2.10 - Un nuevo rol Zabbix User+ con acceso a Mantenimiento](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_010.jpg)

Figura 2.10 - Un nuevo Role **user+rol** de Zabbix con acceso a Mantenimiento

7. Asegúrese de cambiar también la sección **Access to actions** del formulario desmarcando **Manage scheduled reports** como se indica a continuación:
   ![Figura 2.11 - Un nuevo rol Zabbix User+ con la configuración correcta de Acceso a acciones](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_011.jpg)

Figura 2.11 - Un nuevo rol Zabbix User+role con la configuración correcta de Acceso a acciones

8. Por último, pero no menos importante, haga clic en el botón azul  en la parte inferior del formulario para añadir este nuevo rol de usuario.

### Cómo funciona...

En primer lugar, vamos a desglosar las opciones que tenemos al crear roles de usuario en Zabbix:

- **Name**: Aquí podemos establecer un nombre personalizado para nuestro rol de usuario.
- **User types**: Los tipos de usuario siguen existiendo en Zabbix 6, aunque ahora forman parte de los roles de usuario. Todavía hay un límite a lo que puede ser visto por un determinado tipo de usuario y el tipo Super admin sigue siendo sin restricciones cuando se trata de permisos.
- **Access to UI elements**: Aquí podemos restringir lo que un usuario puede ver en la interfaz de usuario de Zabbix cuando se le asigna este rol de usuario.
- **Access to services**: Aquí se puede restringir la monitorización de servicios o SLAs, ya que quizás no queramos que todos los usuarios tengan acceso a ellos.
- **Access to modules**: Los módulos frontend personalizados de Zabbix están totalmente integrados en el sistema de roles de usuario, lo que significa que podemos seleccionar qué módulos frontend puede ver un usuario de Zabbix.
- **Access to API**: La API de Zabbix puede ser restringida a ciertos roles de usuario. Por ejemplo, es posible que sólo quieras un rol de usuario API específico, limitando el acceso del resto de usuarios a la API de Zabbix.
- **Access to actions**: En los roles de usuario de Zabbix, ciertas acciones pueden ser limitadas, como la capacidad de editar tableros, tokens de API de mantenimiento, y más.

Ahora, veamos lo que hemos cambiado entre el rol de usuario llamado `User role` y el rol de usuario llamado `User+ role`. El rol de usuario por defecto llamado `rol de Usuario` tiene el siguiente acceso a los elementos de la UI:
![Figura 2.12 - Rol de usuario predeterminado de Zabbix denominado Rol de usuario Acceso a los elementos de la interfaz de usuario](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_012.jpg)

Figura 2.12 - Rol de usuario predeterminado de Zabbix denominado **User role** tiene acceso a los elementos de la interfaz de usuario

Por defecto, tenemos tres roles de usuario en Zabbix 6, que reflejan los tipos de usuario que están disponibles. El rol de usuario que vemos aquí en **Name** refleja el tipo de usuario que tenemos llamado `User`. Nos da acceso a los elementos de la UI vistos arriba, restringiendo el rol de usuario llamado`User role` para que sólo pueda ver ciertas cosas y no hacer cambios de configuración.

Por ejemplo, se considera un permiso de impacto el poder configurar **Maintenance**. Porque claro, podrías restringir notificaciones importantes configurando **Maintenance**. Pero aquí viene la trampa, ¿qué pasa si quieres explícitamente que un usuario de Zabbix sólo sea capaz de leer la información, pero aún así no tiene acceso a las páginas de configuración? En Zabbix 5.0, esto no era posible porque sólo se podía seleccionar el tipo de usuario **User**, **Admin**, o **Super admin**, dando acceso inmediatamente a toda la sección de configuración cuando se utilizaban los tipos de usuario **Admin** y **Super admin**.
![Figura 2.13 - Nuevo rol de usuario de Zabbix llamado Rol Usuario+ Acceso a elementos de la UI](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_013.jpg)

Figura 2.13 - Nuevo rol de usuario de Zabbix llamado **User+ rol** Acceso a elementos de la UI

Aquí podemos ver lo que ocurre si cambiamos el tipo de usuario a **Admin** pero no seleccionamos todos los elementos disponibles de Acceso a la interfaz de usuario. Ahora tenemos un rol de usuario sin acceso a páginas de configuración importantes, pero con acceso a **Maintenance**.

Combinando esto con la configuración de Acceso a acciones, donde añadimos la configuración Crear y editar mantenimiento como se ve en la *Figura 2.11*, tendríamos acceso completo a la configuración de **Maintenance**.

Cuando asignemos este rol a un usuario en la siguiente receta e iniciemos sesión con ese usuario, podremos ver lo siguiente en nuestra barra lateral de Zabbix.
![Figura 2.14 - Rol de usuario personalizado Barra lateral de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_014.jpg)

Figura 2.14 - Rol de usuario personalizado Barra lateral de Zabbix

Esto, por supuesto, es sólo uno de los muchos tipos de configuraciones que puede utilizar. Usted tiene la capacidad de permitir a los usuarios de Zabbix el acceso a los menús y opciones a través de una serie de parámetros bajo un montón de roles de usuario personalizados. Usted es libre de configurar esto como quiera, añadiendo mucha flexibilidad al usuario dentro de Zabbix.

### Hay más...

Zabbix se encuentra actualmente en el proceso de elaboración de los roles de usuario, lo que significa que algunas partes todavía pueden faltar o puede ver problemas con ellos. Como es una nueva característica, está siendo constantemente mejorada y ampliada. Consulta la documentación de Zabbix para más información sobre esta característica: https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/administration/user_roles.

---

## Creando tus primeros usuarios

Con nuestros grupos de usuarios recién creados, hemos dado nuestro primer paso hacia una configuración de Zabbix más estructurada y segura. El siguiente paso es asignar realmente algunos usuarios a los grupos de usuarios recién creados para asegurarnos de que se les asignan nuestros nuevos permisos de usuario del grupo.

### Preparación

Para empezar, necesitaremos el servidor y los grupos de usuarios recién creados en la última receta. Así que, comencemos la configuración.

Ahora sabemos que hay tres departamentos en la empresa llamada Cloud Hoster que van a utilizar nuestra instalación Zabbix. Hemos creado grupos de host para ellos, pero también hay usuarios en esos departamentos que realmente quieren utilizar nuestra instalación. Vamos a conocerlos:
![Figura 2.15 - Diagrama de usuarios de Cloud Hoste](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_015.jpg)

Figura 2.15 - Diagrama de usuarios de Cloud Hoster

Estos son los usuarios que necesitamos configurar para que Cloud Hoster los utilice.

### Cómo hacerlo...

Empecemos a crear los usuarios. Empezaremos con nuestro departamento de **Networking**:

1. Navegue a **Administration** | **Users**, que le llevará a esta página:
   ![   Figura 2.16 - Ventana Usuarios de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_016.jpg)

Figura 2.16 - Ventana Usuarios de Zabbix

2. Aquí es donde ocurre toda la magia de la creación de usuarios, ya que estaremos administrando todos nuestros usuarios desde esta página. Para crear nuestro primer usuario del departamento de **Networking** llamado `s_network`, haga clic en el botón **Create user** en la esquina superior derecha, lo que nos llevará a la siguiente pantalla:
   ![Figura 2.17 - Ventana de configuración de Usuarios de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_017.jpg)
   Figura 2.17 - Ventana de configuración de Usuarios de Zabbix
3. Rellena **Username** para proporcionarnos el nombre de usuario que utilizará este usuario, que será `s_network`.
4. Además, es importante añadir este usuario al grupo que acabamos de crear para darle a nuestro usuario los permisos adecuados. Haga clic en **Select** y elija nuestro grupo llamado **Networking**.
5. Por último, pero no menos importante, establece una **Password** segura en los campos **Password**; no la olvides porque la utilizaremos más adelante.
6. Después de esto, pasa a la pestaña **Permissions**, ya que no vamos a configurar los medios de comunicación todavía:
   ![Figura 2.18 - Ventana de configuración de permisos de usuario de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_018.jpg)

   Figura 2.18 - Ventana de configuración de permisos de usuario de Zabbix
7. Seleccione aquí la opción Role denominada **Super admin role**. Esto permitirá a nuestro usuario acceder a todos los elementos de la interfaz de usuario y ver y editar información sobre todos los grupos de hosts en el servidor Zabbix.

   Los siguientes roles de usuario están disponibles en Zabbix por defecto:
   ![  Figura 2.19 - Tabla que detalla los diferentes roles de usuario de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_019_new.jpg)

   Figura 2.19 - Tabla que detalla los diferentes roles de usuario de Zabbix
8. Repitamos los pasos anteriores para el usuario llamado `y_network` pero en la pestaña **Permissions**, seleccionemos la opción **Admin role** de la siguiente manera:
   ![Figura 2.20 - Ventana de configuración de permisos de usuario de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_020.jpg)

   Figura 2.20 - Ventana de configuración de permisos de usuario de Zabbix

   Después de crear estos dos usuarios, pasemos a crear el usuario de infraestructura, `r_infra`. Repite los pasos que dimos para `s_network`, cambiando el nombre de usuario, por supuesto. Después, añade este usuario al grupo y dale a nuestro usuario los permisos adecuados. Haga clic en **Select** y elija nuestro grupo llamado **Infrastructure**. Se verá así:
   ![Figura 2.21 - Ventana de configuración de usuario de Zabbix para r_infra](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_021.jpg)

   Figura 2.21 - Ventana de configuración de usuario de Zabbix para r_infra

Por último, haga de este usuario otro **Super admin**.

9. Ahora, para nuestro último usuario, vamos a repetir nuestros pasos de nuevo, cambiando el Nombre de Usuario y el grupo en la pestaña Usuario de la siguiente manera:
   ![Figura 2.22 - Ventana de configuración de Zabbix User para e_buy](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_022.jpg)

   Figura 2.22 - Ventana de configuración de Zabbix User para e_buy
10. Si no has seguido la receta anterior, puedes cambiar el Rol de este usuario por el de Usuario en la pestaña Permisos. Pero si seguiste la receta anterior, podemos usar el rol **User+ role** que creamos así:
    ![Figure 2.22 – The Zabbix User configuration window for e_buy](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_023.jpg)

    Figura 2.23 - Ventana de configuración de usuarios de Zabbix para e_buy

Configurar el usuario con el `User+ role` también permitirá al usuario `e_buy` crear periodos de mantenimiento.

Cuando haya terminado, terminará con lo siguiente:

- `s_network`: Un usuario con acceso a los permisos del grupo de usuarios **Networking** y que tiene el rol **Super admin**.
- `y_network`: Un usuario con acceso a los permisos del grupo de usuarios Networking y que tiene el rol **Admin**.
- `r_infra`: Un usuario con acceso a los permisos del grupo de usuarios **Infrastructure** y que tiene el rol de **Super admin**.
- `e_buy`: Un usuario con acceso a los permisos del grupo de usuarios **Compras e Inventario** y que tiene el rol `User` o el `User+ role`.

---

## Autenticación avanzada de usuarios con SAML

En esta receta, vamos a utilizar la autenticación SAML, una forma de autenticación ampliamente utilizada en el mundo de TI. La utilizaremos como forma de gestionar las contraseñas de nuestros usuarios de Zabbix. Ten en cuenta que si has trabajado antes con Zabbix y has configurado LDAP, SAML, al igual que LDAP, permite la autenticación de usuarios con contraseñas. De todas formas hay que crear usuarios con sus permisos.

### Preparándose

Para empezar con la autenticación SAML, necesitaremos nuestro servidor Zabbix configurado de la receta anterior. Es importante que tengamos todos los usuarios configurados de la receta anterior. También necesitaremos algo para autenticar con SAML. Vamos a utilizar **Azure Active Directory (AD) SAML**.

Asegúrese de configurar los usuarios en su (Azure) AD antes de continuar con esta receta. Puede utilizar sus usuarios AD existentes para la autenticación, por lo que puede utilizar esta receta con su configuración AD existente.

Usaremos el usuario `s_network` como ejemplo:
![Figura 2.24 - Ventana Usuarios y grupos de Azure](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_024.jpg)

Figura 2.24 - Ventana Usuarios y grupos de Azure

Estos son los detalles de nuestro usuario:
![Figura 2.25 - Ventana de detalles del usuario Azure](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_025.jpg)

Figura 2.25 - Ventana de detalles del usuario Azure

Para configurar SAML, recupera la configuración SAML de tu AD o de otro proveedor SAML. Para trabajar con Zabbix, necesitaremos lo siguiente:

- ID de entidad IdP
- URL del servicio SSO
- URL del servicio SLO
- Atributo de nombre de usuario
- ID de entidad SP
- Nombre SP Formato ID

### Cómo hacerlo...

Ahora que tenemos nuestro Azure AD listo, vamos a ver cómo podemos configurar SAML utilizando nuestra configuración:

1. Naveguemos a la siguiente URL: `portal.azure.com`.
2. Después de iniciar sesión, navegue a Azure AD y haga clic en** Enterprise Applications**.
3. Ahora haga clic en **+ New Application** para crear nuestra nueva aplicación. En la siguiente ventana, haga clic en **Create your own application**:
   ![Figura 2.26 - Página de creación de aplicaciones empresariales Azure](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_026.jpg)

   Figura 2.26 - Página de creación de aplicaciones empresariales Azure
4. En la siguiente ventana, nombra tu nueva aplicación `Zabbix` y haz clic en el botón azul **Create**:
   ![Figura 2.27 - Página de nueva aplicación Azure enterprise](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_027.jpg)

   Figura 2.27 - Página de nueva aplicación Azure enterprise
5. Seleccione su nueva aplicación de la lista y haga clic en **Assign Users and Groups** para añadir los usuarios correctos. En nuestro caso, este será `s_network`:
   ![Figura 2.28 - Página Usuarios y Grupos de la aplicación Azure enterprise](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_028.jpg)

   Figura 2.28 - Página Usuarios y Grupos de la aplicación Azure enterprise
6. Haga clic en **Select** y luego en **Assign**.
7. Ahora pasemos a la configuración de SAML haciendo clic en **Single sign-on** en la barra lateral.
8. Ahora haga clic en **SAML** en la página que se muestra en la siguiente captura de pantalla y continúe:
   ![Figura 2.29 - Opción SAML de la aplicación empresarial Azure](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_029.jpg)

   Figura 2.29 - Opción SAML de la aplicación empresarial Azure
9. Ahora en 1, podemos añadir la siguiente información, donde las marcas negras son la URL de nuestro servidor Zabbix:
   ![Figura 2.30 - Configuración SAML de Azure 1](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_030.jpg)

   Figura 2.30 - Configuración SAML de Azure 1
10. En las 2, rellene lo siguiente:
    ![Figura 2.31 - Configuración SAML de Azure 2](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_031.jpg)

    Figura 2.31 - Configuración SAML de Azure 2
11. El número 3 se rellenará automáticamente. Haga clic en **Download** para **Certificado (Base64)**:
    ![Figura 2.32 - Configuración de SAML en Azure 3](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_032.jpg)

    Figura 2.32 - Configuración de SAML en Azure 3
12. Inicie sesión en el CLI del servidor Zabbix y cree un nuevo archivo con el siguiente comando:

```bash
   vim /usr/share/zabbix/conf/certs/idp.cert
```

13. Pegue aquí el contenido del archivo descargado en el paso 11 y guarde el archivo.
14. Ahora de vuelta en Azure para 4, obtendremos la siguiente información:

![ Figura 2.33 - Configuración de SAML en Azure 4](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_033.jpg)

Figura 2.33 - Configuración de SAML en Azure 4

15. En el frontend de Zabbix, ve a la página **Administration** | **Authentication** | **SAML** settings y rellena la siguiente información y haz click en Update:

![Figura 2.34 - Configuración SAML de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_034.jpg)

Figura 2.34 - Configuración SAML de Zabbix

16. Navegue a **Administration** |  **Users** y cambie el usuario `s_network` para incluir el dominio Azure utilizado, por ejemplo:
    ![Figura 2.35 - Pantalla de edición de usuarios de Zabbix para nuestra configuración SAML](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_035.jpg)
    Figura 2.35 - Pantalla de edición de usuarios de Zabbix para nuestra configuración SAML
17. Después de seguir estos pasos, ahora debería ser posible iniciar sesión con el usuario configurado en Zabbix y utilizar la contraseña establecida en Azure AD para ello:
    ![Figura 2.36 - Ventana de acceso a Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_036.jpg)

Figura 2.36 - Ventana de acceso a Zabbix

### Cómo funciona...

La autenticación avanzada de usuarios de Zabbix se utiliza para centralizar la gestión de contraseñas. Aunque en realidad no podemos asignar grupos de usuarios y permisos a los usuarios a través de esta configuración, podemos utilizarlo para la gestión de contraseñas.

De esta manera, podemos asegurarnos de que es más fácil para los usuarios mantener sus contraseñas centralizadas en entornos de oficinas medianas y grandes:
![Figura 2.37 - Diagrama de autenticación SAML de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_02_037.jpg)

Figura 2.37 - Diagrama de autenticación SAML de Zabbix

Zabbix se comunica con nuestro componente **SAML de Azure AD** cuando pulsamos el botón **Sign in**. El usuario es entonces autenticado contra su usuario de Azure AD y una confirmación es enviada de vuelta al servidor Zabbix. Enhorabuena, ¡ya has iniciado sesión en tu servidor Zabbix!

### Hay más...

Podemos hacer este tipo de autenticación no sólo con SAML, sino también con HTTP y LDAP. De esta manera, usted puede elegir la forma correcta de autenticación avanzada para su organización.

Echa un vistazo a la documentación de Zabbix para obtener más información sobre las diferentes formas de autenticación:

https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/administration/authentication

También es posible trabajar con un proveedor de identidad como Okta, OneLogin, o cualquier otro. Lo que significa que tus opciones no están limitadas a Azure AD, siempre y cuando soporte SAML puedes usarlo para autenticar contra tu servidor Zabbix.

---
