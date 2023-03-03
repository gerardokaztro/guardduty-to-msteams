# Cómo visualizar los hallazgos de Amazon GuardDuty en MS Teams

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

2. Cada hallazgo detectado por Amazon GuardDuty, es recopilado por una regla basada en eventos de Amazon EventBridge y redirigirá estos eventos (antes hallazgos) hacia una función lambda. Para que esto sea posible, asegúrate de que la regla tenga los permisos adecuados para escribir sobre tu función lambda.

3. La función lambda va a desencadenar la invocación del código por cada evento que reciba de la regla configurada en EventBridge.

4. Recibirás una notificación en tu canal de MS Teams cada vez que se detecte un hallazgo, el cual se configura por medio de un incoming webhooken.

## Beneficios de la solución:
Al implementar esta solución, puede lograr los siguientes beneficios:

- Ahorra tiempo al analizar un hallazgos de manera rápida y sencilla en lugar de hacerlo desde la misma consola de Amazon GuardDuty. En un esquema múltiples cuentas, no será efectivo hacer la búsqueda entre 1000 posibles.

- Concentrarse en los hallazgos que generen mayor interés, por ejemplo, puedes solo recibir hallazgos de severidad HIGH, o de algún Treath Purpose como Cryptomining, o sobre algun recurso critico como una instancia EC2, un bucket S3, entre otros.

- Obtener una vista rápida de los detalles del hallazgo, como saber cual es el indicador de compromiso del atacante o cual es el recursos afectado, incluso saber a que cuenta y región pertenece el hallazgo.
