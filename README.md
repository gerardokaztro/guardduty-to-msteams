![ms teams](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ztekhu1k2tg72zzvk5bz.png)

En este blog post te mostraré como enviar los hallazgos de Amazon GuardDuty a un canal de mensajería como Microsoft Teams en un formato de tarjeta que contenga información relevante del hallazgo para que el equipo de seguridad pueda tomar acción inmediata.

Puede usar esta integración para comprender mejor los hallazgos de Amazon GuardDuty.

## Enfoque para visualizar los resultados de Amazon GuardDuty

Como se muestra en la Figura 1, hay cuatro pasos de alto nivel para entender como funciona la solución.

![Figura 1: Pasos para visualizar hallazgos de Amazon GuardDuty](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ak5ytpgq0l71zyj2dx9y.png)

### 1. Centralizar los hallazgos de Amazon GuardDuty
Puede centralizar los hallazgos de Amazon GuardDuty en una única cuenta como Security Audit para que desde aquí se pueda implementar el resto de componentes de la solución. De esa manera, puedes usar esta solución para recibir todos los hallazgos detectados en tus cargas de trabajo, sin importar en que cuenta o región se haya detectado.

### 2. Recopilar los hallazgos con Amazon EventBridge
Amazon EventBridge es un servicio muy útil al momento de recopilar eventos de algún otro servicio y redireccionarlos hacia un target especifico como una función Lambda, un tópico SNS, un servicio ETL como Kinesis Data Firehose, entre otros.

### 3. Ejecución del código de una función Lambda
Cada vez que la función lambda reciba un hallazgo de Amazon GuardDuty invocara el código que tiene configurado para hacer el envío de este hallazgo a un canal de mensajería como MS Teams.

### 4. Visualizar el hallazgo
En la Figura 2, podemos ver un ejemplo de como el equipo de segurida recibirán los hallazgos de Amazon GuardDuty en su canal.

![Figura 2: Visualización del hallazgo](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gy45hpmrqo30gzpakehn.png)

## Descripción general del diseño
Este diagrama de arquitectura representa a alto nivel los servicios a usar y la interacción entre ellos.

![Figura 3: Descripción general del ddiseño](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yki9l7oh8i6vz07vvpu9.png)

En el diagrama de la figura 3, vamos a explicar el paso a paso:

1. Dentro de un esquema de múltiples cuentas AWS, conviene empezar a delegar la administración de algunos servicios por sobre el resto de cuentas, como es el caso de Amazon GuardDuty. Hacer esto te permite centralizar todos los hallazgos de Amazon GuarDuty de todas las cuentas donde tengas habilitado este servicio. Y delegar una única cuenta, en este caso la Security Audit para gestionar Los hallazgos desde un único punto.

2. Cada hallazgo detectado por Amazon GuardDuty, es recopilado por una regla basada en eventos de Amazon EventBridge y redirigirá estos eventos (antes hallazgos) hacia una función lambda.

3. La función lambda va a desencadenar la invocación del código por cada evento que reciba de la regla configurada en EventBridge.

4. Recibirás una notificación en tu canal de MS Teams cada vez que se detecte un hallazgo, el cual se configura por medio de un incoming webhook.

## Beneficios de la solución:
Al implementar esta solución, puede lograr los siguientes beneficios:

- Ahorra tiempo al analizar un hallazgos de manera rápida y sencilla en lugar de hacerlo desde la misma consola de Amazon GuardDuty. En un esquema múltiples cuentas, no será efectivo hacer la búsqueda entre 1000 posibles.

- Concentrarse en los hallazgos que generen mayor interés, por ejemplo, puedes solo recibir hallazgos de severidad HIGH, o de algún Treath Purpose como Cryptomining, o sobre algun recurso critico como una instancia EC2, un bucket S3, entre otros.

- Obtener una vista rápida de los detalles del hallazgo, como saber cual es el indicador de compromiso del atacante o cual es el recursos afectado, incluso saber a que cuenta y región pertenece el hallazgo.

## Pre-requisitos:
- Instalar Git
- Instalar Visual Studio Code, o algún otro de tu preferencia
- Por si no lo tienes, instalar Zip
- Instalar el cliente de MS Teams
- Por ultimo, tener cuenta en Github y MS teams

## Tutorial
Puedes hacer uso de esta solución tanto en un entorno de múltiples cuentas o una cuenta standalone. Es decir, si solamente tienes activado Amazon GuardDuty en una única cuenta, y quieres usar esta solución, puedes hacerlo, va a funcionar de todas maneras.

**Lo primero que haremos,** será crear o usar un canal de MS Teams para recibir los hallazgos, y configuraremos un incoming webhook.

Si vamos a crear un canal, estos son los pasos:

- Elegir un nombre y descripción de tu preferencia.
- La privacidad del canal, puede ser tanto pública o privada, va a depender de tus necesidades.
- Dar clic en crear.

![crear canal en ms teams](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nonsyxqjwrs8a53u0vko.png)

Una vez tengas el canal creado, ve a configuraciones del canal, y luego a conectores:

![configurar un conector en ms teams](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q0ekswxqpu1pmchhtu2g.png)

Acto seguido, donde dice "Webhook entrante" o "Incoming Webhook" dar clic en configurar:
![configurar un iw](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8vednl0mboagc01b8bqd.png)

Luego, en los detalles de la configuración del Webhook, coloca un nombre, agrega un icono y finalmente da clic en "Crear"

![configurar iw pasos finales](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wjgcdv58tzksv7spt1m1.png)

Inmediatamente el conector te va a brindar la url del webhook, cópiala en algún lugar, vamos a usar más tarde.

**Lo segundo que haremos** será crear una función lambda. Esta función debe ser creada en la misma cuenta donde residen los hallazgos de Amazon GuardDuty.

- Elije un nombre, yo usare **"alert-guardduty-teams"**
- Usa los mismos valores que muestro en la imagen.

![Crear la función de lambda](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ny4yunzqo58b3mnhkf6c.png)

Cambiemos los valores por defecto que usa lambda function a estos nuevos:

![configuración de lambda](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jtzuerxswrekegq0as4c.png)

En la opción de **Enviroment Variables** (variables de entorno), agreguemos lo siguiente:

- SEVERITY_THRESHOLD = coloca aquí el nivel de severidad de las amenazas que deben ser notificadas
- TEAMS_WEBHOOK_URL = coloca aquí la url del incoming webhook que generaste en el primer paso.

![Variables de entorno](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nrbfey29cu1pbi5w6i2k.png)

**Lo tercero que haremos** será crear una regla en EventBridge, no te preocupes por el código ahora mismo, ya lo veremos.

Para crear una regla basada en eventos, ir a Amazon CloudWatch, y luego hacer clic en Rule:

![Ir a CloudWatch Rule](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/z8l95hy67jydw3ivekip.png)

Para crear y configurar una regla, consta de 3 pasos importantes:

Paso 1:
- Elegir un nombre.
- Agrega una descripción.
- Todo lo demás por defecto, como en la imagen.

![crear regla paso 1](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2pvznx17snby831fz22n.png)

Paso 2:
- Las opciones de "Event Source" y "Sample Event" las dejaremos por defecto.
- En la opción de "Creation Method" elegir "Use pattern form".
- En la opción de "Event pattern".

![crear regla paso 2a](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/x6cix7by1zaomwiizptf.png)

![crear regla 2b](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u6ir7ayx5idy79h62s8w.png)

Paso 3:
- Target type: AWS Services
- Select a target: Lambda function
- En Function, elegir la función lambda creada en el segundo paso.
- Todo lo demás por defecto

![crear regla 3](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vdizrznitq5w0adwe1pb.png)

Ahora si, vamos a por el código 💪

Deberás clonar en tu máquina local, este repositorio alojado en Github que contiene el código que usaremos.

No olvides dejar tu ⭐ al repositorio, también hazle fork 🔂 para que puedas customizarlo a tus necesidades y envíame un PR así contribuimos juntos a la comunidad Open Source.

{% embed https://github.com/gerardokaztro/guardduty-to-msteams %}

Una vez tengas el repositorio en tu local, busca el archivo alerts-guardduty-teams.zip y súbelo a tu función lambda.

![cargar zip](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zx8d5pocn5l0rjj5i4ol.png)

Finalmente, modifica el lambda handler usando el nombre del archivo python comprimido

![lambda handler](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ecq9l3x3uir0p0qi86we.png)

![actualizar lambda handler](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/goosjqc535gvo5yh7kzw.png)

> Si quieres hacer alguna modificación al código, te recomiendo hacerlo de manera local, luego comprimes el archivo python usando zip y lo subes a la función lambda.

Finalmente, empieza a recibir los hallazgos:

![ms teams](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ztekhu1k2tg72zzvk5bz.png)