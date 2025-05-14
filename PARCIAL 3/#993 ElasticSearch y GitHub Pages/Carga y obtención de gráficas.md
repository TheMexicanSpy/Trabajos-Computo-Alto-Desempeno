# Carga de Dataset a ElasticSearch para graficar con GitHub Actions
***
## Consideraciones Principales
Se usará la plataforma de ElasticSearchCloud para esta práctica. Es una interfaz 
intuitiva y lista para usar desde que registras tu cuenta. El detalle es que tienes un Trial de 13 días.

![image](https://github.com/user-attachments/assets/789f7a9a-d1e4-42a5-970c-44b2ddaedc11)

Utilizaremos un Dataset de las personas que abordaron el titanic. Algo normalito...

https://github.com/TheMexicanSpy/Elasticsearch-visualization/blob/main/data/Titanic-Dataset.csv

Y luego ustedes en mi repositorio observaran los distintos códigos de Pyton que se usan para la lógica.
Python es excelente, pues es el lenguaje de programación de ciencias de datos por excelencia, permitiendo la facil manipulación de conjuntos de datos.
***
## Aplicación del Flujo de trabajo
La estrella de este proyecto es el **deploy.yml**. Este archivo es el encargado de toda la lógica dentro de 
GitHub y de los scripts inherentes.[Aquí]([url](https://github.com/TheMexicanSpy/Elasticsearch-visualization/blob/main/.github/workflows/deploy.yml)) lo pueden encontrar en detalle.

Estos archivos *yaml* son como unos constructores, que dicen que recursos ocupa el flujo de trabajo y cuando ejecutarlos.
En este caso hemos declarado un workflow que se llama `Deploy Elasticsearch Visualizations`

![image](https://github.com/user-attachments/assets/5a99428e-a6c0-46d0-ab41-4ae9d034930a)

Básicamente esta programado para cada vez que se hace push al repositorio, se ejecute todo el flujo de trbajo, en caso de que se modificara
o mejore algo, siempre estarán actualizados los datos.

Como es un riesgo andar codificando credenciales de forma estática en los scripts, se utilizaron *GitHub Secrets*...

![image](https://github.com/user-attachments/assets/6b0ac296-e623-40dd-a511-1b74c0c6781f)

Son como vaiables dentro de tu entorno de Git, ya sea de forma global en todo tu repositorio, o para ciertos ambientes personalizados, además
se pueden modificar cuando tu desees, y no es necesario andar reescribiendo código y aumentas tu seguridad.

Finalmente, como apreciarán en el `deploy.yml`, el workflow crea una página en GitHub Pages donde se manda el .png genrado por los scripts a la página web.

![image](https://github.com/user-attachments/assets/680f0cd4-fe8a-4985-9f50-eb8507106f04)

[Aquí](https://themexicanspy.github.io/Elasticsearch-visualization/) tienen el link para visitar la página creada por el workflow.
Y [aquí](https://github.com/TheMexicanSpy/Elasticsearch-visualization) está el link del repositorio de ElasticSearch.
***
