# Clasificaci-n-de-los-distintos-tipos-de-agentes-de-la-movilidad-con-AWS-Rekognition

Para iniciar a trabajar con la herramienta de Amazon Web Servicies “Rekognition Custom Labels”, se registra para obtener cuenta en la consola de AWS, al realizar el registro automáticamente se tiene acceso a todos los servicios, incluyendo Rekognition.
Cuando se crea la cuenta de AWS, se obtiene una identidad de inicio de sesión única que tiene acceso completo a todos los servicios y recursos de AWS en la cuenta. Esta identidad se denomina usuario raíz de la cuenta de AWS.
![usuario IAM](https://user-images.githubusercontent.com/85694217/121562912-8919ee00-c9df-11eb-8610-e45c397f3a5b.PNG)

Se procede a descargar la librería para Python proporcionada por AWS, el SDK “Boto3”, con esta librería en Python; se instala “AWS CLI” desde comand Windows CMD, esta herramienta se utiliza para configurar los parámetros y permisos necesarios para ejecutar correctamente los servicios de Rekognition y Rekognition Custom Label tales como las claves de acceso y la región donde se requiere trabajar todas proporcionadas por AWS.  
![Boto3](https://user-images.githubusercontent.com/85694217/121562685-4eb05100-c9df-11eb-8465-8689cc9bde46.PNG)
![Captura](https://user-images.githubusercontent.com/85694217/121562853-7bfcff00-c9df-11eb-8c36-b01be089c1de.PNG)

Crear Dataset
La toma de imágenes se realiza con la cámara descrita en II-B, se toman 1000 imágenes y se empiezan a almacenar en S3 el cual es un servicio de Amazon web servicies, este depósito online se utiliza para almacenar el proyecto, conjuntos de datos y modelos de etiquetas personalizadas de Amazon Rekognition.
Para empezar a etiquetar las imágenes se crea el proyecto “etiquetas_de_noche”.

Dentro del proyecto se guarda el conjunto de datos (imágenes tomadas) y en la opción crear etiquetas se añade las 6 etiquetas descritas en II-A.

![image](https://user-images.githubusercontent.com/85694217/122649556-fb7f8200-d0f3-11eb-975b-879ea2c48ad3.png)


Luego de definir las etiquetas se empieza a etiquetar en la consola de Amazon Rekognition Custom labels en la opción draw bounding box.

![image](https://user-images.githubusercontent.com/85694217/122649574-0b976180-d0f4-11eb-871c-65758193c527.png)

![image](https://user-images.githubusercontent.com/85694217/122649580-0fc37f00-d0f4-11eb-9f2b-88acee895663.png)


Entrenar la RNC

Para entrenar el modelo se utiliza el dataset creado, Rekognition utiliza el 80% del dataset para entrenamiento y el otro 20% para pruebas; se hace uso de la consola de AWS en la opción entrenar modelo y Rekognition automáticamente utiliza el modelo que mejor se adapte a las necesidades del proyecto por lo que cada vez que se entrene la red, Rekognition crea un modelo diferente

![image](https://user-images.githubusercontent.com/85694217/122649643-5618de00-d0f4-11eb-9790-fc4e92bd662d.png)


