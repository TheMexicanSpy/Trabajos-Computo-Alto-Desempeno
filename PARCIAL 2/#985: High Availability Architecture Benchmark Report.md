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
***
## Conclusión
Se requiere de crear otro tipo de arquitectura, talvez con un segundo load balancer que conecte los nodos web. Se nota un performance de al rededor del 50-60% de transacciones completadas a fallidas en ambas herrramientas, variando en escala.
