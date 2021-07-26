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

Iniciar la red 

Una vez entrenada la red se tiene un modelo con el cual se empieza a realizar pruebas de diferenciación de los distintos tipos de agentes de la movilidad, para ello se inicia la red neuronal a través del siguiente código en Python.

#Copyright 2020 Amazon.com, Inc. or its affiliates. All Rights Reserved.
#PDX-License-Identifier: MIT-0 (For details, see https://github.com/awsdocs/amazon-rekognition-custom-labels-developer-guide/blob/master/LICENSE-SAMPLECODE.)

import boto3

def start_model(project_arn, model_arn, version_name, min_inference_units):

    client=boto3.client('rekognition')

    try:
        # Start the model
        print('Starting model: ' + model_arn)
        response=client.start_project_version(ProjectVersionArn=model_arn, MinInferenceUnits=min_inference_units)
        # Wait for the model to be in the running state
        project_version_running_waiter = client.get_waiter('project_version_running')
        project_version_running_waiter.wait(ProjectArn=project_arn, VersionNames=[version_name])

        #Get the running status
        describe_response=client.describe_project_versions(ProjectArn=project_arn,
            VersionNames=[version_name])
        for model in describe_response['ProjectVersionDescriptions']:
            print("Status: " + model['Status'])
            print("Message: " + model['StatusMessage']) 
    except Exception as e:
        print(e)
        
    print('Done...')
    
def main():
    project_arn='arn:aws:rekognition:us-east-1:822849167361:project/etiquetas_de_noche/1611613249886'
    model_arn='arn:aws:rekognition:us-east-1:822849167361:project/etiquetas_de_noche/version/etiquetas_de_noche.2021-06-08T15.51.26/1623185487275'
    min_inference_units=1 
    version_name='etiquetas_de_noche.2021-06-08T15.51.26'
    start_model(project_arn, model_arn, version_name, min_inference_units)

if __name__ == "__main__":
    main()
    
    
 Análisis de video con modelo basado en Rekognition
    
Una vez iniciada la red neuronal se escoge un video y se analiza con el código en Python mostrado en la Figura 9, este código toma el video a analizar desde el equipo donde se realiza el análisis sin necesidad de subirlo a s3 de AWS, dentro de la función analyzeVideo se realiza el llamado del SDK para Python: “boto3” y por medio del ARN del modelo se especifica con cual red neuronal se trabaja; previamente instalada la librería OpenCv, se hace uso de la función cv2.VideoCapture para poder analizar el video cuadro por cuadro dentro del análisis de estas imágenes se utiliza la función “customLabel” la cual está dentro de SDK boto3, esta crea un diccionario donde se almacena información de las etiquetas detectadas en las imágenes procesadas, por ejemplo el nombre y las coordenadas para poder graficar el “bounding box” para cada etiqueta personalizada; luego , a través de la función cv2.VideoWriter de OpenCv se crea un nuevo video en formato AVI a partir de las imágenes procesadas para así lograr obtener un resultado visual. 


import json
import boto3
import cv2
import math
import matplotlib.pyplot as plt
import time
import io
import numpy as np
import collections
from PIL import Image, ImageDraw, ExifTags, ImageColor, ImageFont

import json
import boto3
import cv2
import math
import matplotlib.pyplot as plt
import time
import io
import numpy as np
import collections
from PIL import Image, ImageDraw, ExifTags, ImageColor, ImageFont

def analyzeVideo():
    videoFile = "23seg.mp4" #el nombre del video ubicado en el escritorio 
    projectVersionArn = "arn:aws:rekognition:us-east-1:822849167361:project/etiquetas_de_noche/version/etiquetas_de_noche.2021-03-30T12.14.47/1617124488272"
 
    global c,m,p,t,b,bus,moto,car,buses,truck,persona,bicicleta
    moto=0
    car=0
    persona=0
    truk=0
    bicicleta=0
    buses=0
    
    rekognition = boto3.client('rekognition')           
    cap = cv2.VideoCapture(videoFile)
    frameRate = cap.get(5) 
    frame_width = int(cap.get(3)) 
    frame_height = int(cap.get(4)) 
   
    size = (frame_width, frame_height)
    result = cv2.VideoWriter('prueba2343.avi',  # cambiar AVI
                         cv2.VideoWriter_fourcc(*'MJPG'), 
                         30, size) 
    while(cap.isOpened()):
        frameId = cap.get(1) 
        print("Processing frame id: {}".format(frameId))
        ret, frame = cap.read()
        if (ret != True):
            break
        if (frameId % math.floor(frameRate) == 0):
            hasFrame, imageBytes = cv2.imencode(".jpg", frame)

            if(hasFrame):
                response = rekognition.detect_custom_labels(
                    Image={
                        'Bytes': imageBytes.tobytes(),
                    },
                    ProjectVersionArn = projectVersionArn
                )
        stream = io.BytesIO(imageBytes.tobytes())
        image=Image.open(stream)
        imgWidth, imgHeight = image.size  
        draw = ImageDraw.Draw(image)  


        # calcular y dibujar bounding boxes para cada etiqueta detectada       
           
        for customLabel in response['CustomLabels']:
            if 'Geometry' in customLabel:
                box = customLabel['Geometry']['BoundingBox']
                left = imgWidth * box['Left']
                top = imgHeight * box['Top']
                width = imgWidth * box['Width']
                height = imgHeight * box['Height']
                
                #font_path = "/home/ec2-user/SageMaker/DejaVuSans.ttf"
                #font_size = 30
                #fnt = ImageFont.truetype(font_path, font_size)
                fnt = ImageFont.truetype('/Library/Fonts/arial.ttf', 30)#cambiar numero a 30

                draw.text((left,top), customLabel['Name'], fill='#00d400', font=fnt) 

                points = (
                    (left,top),
                    (left + width, top),
                    (left + width, top + height),
                    (left , top + height),
                    (left, top))
                draw.line(points, fill='#00d400', width=5)
                print(top)

                #points = ((0.2,200),(1280,200))

                #draw.line(points, fill='#00d400', width=5)



                
        result.write(np.array(image))
        
       
        contador = collections.Counter(customLabel)
        #print (contador['Name'])

        if contador['Name']== "motorcycle" and top>200:
            moto=moto+1
            m=moto/30    
            
        if contador['Name']== "car": #and top>200:
            car=car+1
            c=car/30
        if contador['Name']== "bike": #and top>200:
            bicicleta=bicicleta+1
            b=bicicleta/30    
        if contador['Name']== "truck": #and top>200:
            truck=truck+1
            t=truck/30      
        if contador['Name']== "bus": #and top>200:
            buses=buses+1
            bus=buses/30
        if contador['Name']== "person": #and top>200:
            persona=persona+1
            p=persona/30      
                
    print("motos ="+ repr(m))    
    print("carros ="+ repr(c))
    print("bicicletas ="+ repr(b))
    print("vehiculos pesados (camiones) ="+ repr(t))
    print("personas ="+ repr(p))
    print("buses ="+ repr(bus))   
    cap.release()
    result.release()

analyzeVideo()

se obtiene el siguiente resultado

![image](https://user-images.githubusercontent.com/85694217/122782464-5e008b80-d276-11eb-8bdf-5790b816b881.png)

Detener la red

Al finalizar el análisis del video se procede a detener la red para así poder dejar de utilizar el sistema de AWS mediante el código mostrado a continuacion.

#Copyright 2020 Amazon.com, Inc. or its affiliates. All Rights Reserved.
#PDX-License-Identifier: MIT-0 (For details, see https://github.com/awsdocs/amazon-rekognition-custom-labels-developer-guide/blob/master/LICENSE-SAMPLECODE.)

import boto3
import time


def stop_model(model_arn):

    client=boto3.client('rekognition')

    print('Stopping model:' + model_arn)

    #Stop the model
    try:
        response=client.stop_project_version(ProjectVersionArn=model_arn)
        status=response['Status']
        print ('Status: ' + status)
    except Exception as e:  
        print(e)  

    print('Done...')
    
def main():
    
    model_arn='arn:aws:rekognition:us-east-1:822849167361:project/etiquetas_de_noche/version/etiquetas_de_noche.2021-06-08T15.51.26/1623185487275'
    stop_model(model_arn)

if __name__ == "__main__":
    main() 




