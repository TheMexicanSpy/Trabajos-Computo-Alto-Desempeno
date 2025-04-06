# Reporte: Pruebas a mi arquitectura de alta disponibilidad.
***
## Diagrama de la arquitectura
![image](https://github.com/user-attachments/assets/0d822d02-9b39-4f04-b781-dfcbdb8f44c8)
## Pasos de las pruebas
### Benchmark con siege
![image](https://github.com/user-attachments/assets/8ef757fe-769a-424a-bda9-07ac7f082d9f)
***
### Benchmark con ab
![image](https://github.com/user-attachments/assets/b33462d5-4c08-4f4f-800e-67d3fb8ccd0f)
***
### Comparación:siege vs ab
![image](https://github.com/user-attachments/assets/ced7cb81-4ff0-4e6e-baf2-81e57bddd6c6)
![image](https://github.com/user-attachments/assets/6305a351-c405-4944-9cb9-47ebea110e61)
![image](https://github.com/user-attachments/assets/6a986b2e-cc74-468d-a11c-e31826710497)
![image](https://github.com/user-attachments/assets/e8c630e5-8048-4442-951b-36ddda2490ac)
***
### Cambios en la arquitectura
Para elevar el performance de la arquitectura se pueden intentar varias cosas. En este caso se agregó un nodo web más y se usa el protocolo de balanceo de menor conectividad
![image](https://github.com/user-attachments/assets/00a8f231-c1ef-44df-935d-97bd7059f4a3)
![image](https://github.com/user-attachments/assets/ce1edde0-5071-4e7b-961b-4c8f92dcc7a6)
![image](https://github.com/user-attachments/assets/36763b16-6a50-4fd3-882c-1545db7173b4)
El cambio es muy poco notorio, probablemente esta arquitectura no es eficiente para recibir 200 usuarios concurrentes. Además la herramienta de ab es más robusta que siege y por ello la gran diferencia de transferencias fallidas y completadas.
### UPDATE: NUEVOS CAMBIOS REALIZADOS
Dentro de la configuración de HaProxy se agregó en los parámetros globales `maxconn       7000`, así como devolver al algoritmo `roundrobin`.
Teniendo en cuenta la siguiente prueba, antes de aplicar los cambios se tuvo:

`ab -c 200 -n 5000 -s 90 http://localhost/`
![image](https://github.com/user-attachments/assets/2a45076a-5824-45ec-bfdf-e0aea486287d)

Después de aplicar la configuración se obtuvo:
![image](https://github.com/user-attachments/assets/940949f2-ea02-445f-a493-191eb5aad29a)

Haciendo la prueba con siege se tiene:

![image](https://github.com/user-attachments/assets/05daabe6-0cb4-4435-845d-ea9d2e2ce69f)

Esto creó una mayor diferencia ya que después de los cambios no ocurrieron estatus 500.

Otro tuneo se puede realizar delegando el proceso de load balanceo a múltiples cpu's, primero hay que verificar la cantidad de cpús en el sistema, y por desgracia solo poseo un cpu. De tener más de uno se puede incrementar aún má el performance.

![image](https://github.com/user-attachments/assets/e3b03aee-cccd-4805-a1f4-bff913a33abc)

Además se puede usar el algoritmo de balanceo `static-rr` pero en mi benchmrak no parece haber creado una diferencia significativa...

![image](https://github.com/user-attachments/assets/e4574a91-5041-42cd-92d3-fe6774317832)

***
## Conclusión
Aplicar las configuraciones doben deben ser creó una gran diferencia, es decir, llevar la configuración al global de `maxconn 7000` no solo evitó los estatus 500 en un momento en que la arquitectura siempre se rompía, sino que el performance cambió dráasticamente. Con estos cambios tambien con siege hubó bastante mekora en la perfomance, hacinedo que con  los dos benchmarks la disponibilidad suba al 100% con unos cuantos cambios en la configuración.
