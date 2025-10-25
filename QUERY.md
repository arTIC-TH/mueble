sobre la base de datos de scopus

se cambio la base de datos, de los 684 articulos, de los cuales 378 son open access
hay una nueva versión, con split por issn
hacer un split para el issn,, para evitar inconvenientes cuando se alimente la tabla de JO_scimgo9924.db (strctr crtxt_mngr)

hay una base de jo_scimago para la informacion de las revistas
traer la información de referencia para las revistas

# trabajo sobre revistas

qry: listado de revistas: source_title + issn + isbn

qry1: el resultado de articulos por revista por año 


```{sql}

```

## carga de datos de scimago journal (base scimagojo-9924a.db src: crtxt_mngr)

las variables claves aqui son:
A_JO_AREA_U  (with split)
A_JO_Categoria  (with split)
A_JO_Citable  
A_JO_Country  
A_JO_H_index  
A_JO_ISIPubdate  
A_JO_ISSN  (with split)
A_JO_SJR  
A_JO_SJR_Best_Quartile  
A_JO_SourceId  
A_JO_Title  
A_JO_TotalDocs  
A_JO_TotalRefs  
A_JO_Year

## areas y categorias de conocimiento
## pais editor
## quartil



unir con informacion sjr, para revistas de qry1
como cargar los datos de revistas en sqlite

> trabajo sobre autor
qry2: afiliaciones unicas de los investigadores
identificar del tital de investigadores, aquellos que reportan afiliaciones a UNILLANOS
hacer split para extraer el país


> trabajo sobre afiliaciones

> trabajo sobre investigadores activos en grupos en minciencias

 
