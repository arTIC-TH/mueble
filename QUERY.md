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

# trabajo sobre autor
 
## qry2: afiliaciones unicas de los investigadores

## identificar del total de investigadores, aquellos que reportan afiliaciones a UNILLANOS 
variante de nombre de unillanos ... like "llanos"

hacer scrpt para extraer el país (ultima parte de la cadena de texto ,)

> trabajo sobre afiliaciones

> trabajo sobre investigadores activos en grupos en minciencias

 # archivos de scripts - FIS

 --1. count articulos x autor - year
create table temp_articulos_autor_year as
select autor.data, year.data as year, count(distinct EID.id) total
from U_IDSCPS autor, year, EID
where autor.id = year.id and autor.rank = year.rank and autor.parserank = year.parserank
and autor.id = EID.id and autor.rank = EID.rank
group by autor.data, year.data;

--2. % articulos x autori - year - type of Document_Type
select autor.data, year.data, dt.data, count(distinct EID.id)/articulos_x_autor_year.total
from U_IDSCPS autor, year, Document_Type dt, EID,
(
select autor.data as autor, year.data as year, count(distinct EID.id) total
from U_IDSCPS autor, year, EID
where autor.id = year.id and autor.rank = year.rank and autor.parserank = year.parserank
and autor.id = EID.id and autor.rank = EID.rank
group by autor.data, year.data
) articulos_x_autor_year
where autor.id = year.id and autor.rank = year.rank and autor.parserank = year.parserank
and autor.id = dt.id and autor.rank = dt.rank and autor.parserank = dt.parserank
and autor.id = EID.id and autor.rank = EID.rank
and autor.data = articulos_x_autor_year.autor and year.data = articulos_x_autor_year.year 
group by autor.data, year.data, dt.data;

--3. count articulos x afiliaciones - year
select afiliaciones.data, year.data, count(distinct EID.id) total
from U_Affiliations afiliaciones, year, EID
where afiliaciones.id = year.id and afiliaciones.rank = year.rank and afiliaciones.parserank = year.parserank
and afiliaciones.id = EID.id and afiliaciones.rank = EID.rank and afiliaciones.parserank = EID.parserank
group by afiliaciones.data, year.data;

--4. crecimiento
create table temp_master_year as
select distinct year.data
from year;

create table temp_master_autor_year as
select distinct U_IDSCPS.data, temp_master_year.data as year
from temp_master_year, U_IDSCPS;

select * from temp_master_autor_year where data = 56986567900;
create table temp_articulos_autor_all_year as
select autor.data, autor.year, IFNULL(temp_articulos_autor_year.total,0) as total
from temp_master_autor_year autor
left join temp_articulos_autor_year on autor.data = temp_articulos_autor_year.data and autor.year = temp_articulos_autor_year.year
group by autor.data, autor.year;

select * from temp_articulos_autor_year;

select autor.*, articulos_autor_all_year_actuel.year, articulos_autor_all_year_old.year,
articulos_autor_all_year_actuel.total - articulos_autor_all_year_old.total as incremento, 
articulos_autor_all_year_actuel.total, articulos_autor_all_year_old.total
from temp_master_autor_year autor, 
temp_articulos_autor_all_year articulos_autor_all_year_actuel, temp_articulos_autor_all_year articulos_autor_all_year_old 
where autor.data = articulos_autor_all_year_actuel.data and autor.year = articulos_autor_all_year_actuel.year
and autor.data = articulos_autor_all_year_old.data 
and articulos_autor_all_year_actuel.year = articulos_autor_all_year_old.year + 1
order by autor.year;

select * from temp_articulos_autor_all_year where data = 56986567900;

--5. citaciones x autor - year - articulo
select autor.data, year.data, EID.data, sum(c.data) as citaciones
from U_IDSCPS autor, year, EID, Cited_by c
where autor.id = year.id and autor.rank = year.rank and autor.parserank = year.parserank
and autor.id = EID.id and autor.rank = EID.rank
and EID.id = c.id and EID.rank = c.rank and EID.parserank = c.parserank
group by autor.data, year.data, EID.data
order by sum(c.data) desc;

--5.1 % articulos citados / total articulos x autor - year - articulo
select autor.data, year.data, count(distinct EID.id) articulos_citados, temp_articulos_autor_year.total, 
round((count(distinct EID.id)/temp_articulos_autor_year.total),4)*100 as porce_artic 
from U_IDSCPS autor, year, EID, Cited_by c, temp_articulos_autor_year
where autor.id = year.id and autor.rank = year.rank and autor.parserank = year.parserank
and autor.id = EID.id and autor.rank = EID.rank
and EID.id = c.id and EID.rank = c.rank and EID.parserank = c.parserank
and autor.data = temp_articulos_autor_year.data and year.data = temp_articulos_autor_year.year
and c.data > 0
group by autor.data, year.data;

--6. articulos > 1 x afiliaciones - autor - year
select a.data, autor.data, year.data, count(distinct EID.data) as cuenta
from U_Affiliations a, U_IDSCPS autor, year, EID
where a.id = year.id and a.rank = year.rank and a.parserank = year.parserank
and autor.id = year.id and autor.rank = year.rank and autor.parserank = year.parserank
and autor.id = EID.id and autor.rank = EID.rank
group by a.data, autor.data, year.data
having count(distinct EID.data) > 1;

--7. total_articulos, total_articulos_citaciones, suma_citaciones x autor  
select autor.data, sum(temp_articulos_autor_year.total) as total_articulos, count(distinct EID.id) articulos_citados, sum(c.data) as citaciones
from U_IDSCPS autor, year, EID, Cited_by c, temp_articulos_autor_year
where autor.id = year.id and autor.rank = year.rank and autor.parserank = year.parserank
and autor.id = EID.id and autor.rank = EID.rank
and EID.id = c.id and EID.rank = c.rank and EID.parserank = c.parserank
and autor.data = temp_articulos_autor_year.data and year.data = temp_articulos_autor_year.year
and year.data >= 2015 and year.data <= 2020
and c.data > 0
group by autor.data
order by sum(c.data) desc;

--8. promedio citaciones x autor
select autor.data, count(distinct EID.id) as total_arti_citados, sum(c.data) as citaciones, sum(c.data)/count(distinct EID.id) as promedio_citaciones
from U_IDSCPS autor, year, EID, Cited_by c, temp_articulos_autor_year
where autor.id = year.id and autor.rank = year.rank and autor.parserank = year.parserank
and autor.id = EID.id and autor.rank = EID.rank
and EID.id = c.id and EID.rank = c.rank and EID.parserank = c.parserank
and autor.data = temp_articulos_autor_year.data and year.data = temp_articulos_autor_year.year
and year.data >= 2015 and year.data <= 2020
and c.data > 0
group by autor.data
order by sum(c.data)/count(distinct EID.id) desc;

--9. count autores x articulo
select EID.data, count(distinct autor.data) as cuenta,
CASE 
	WHEN count(distinct autor.data) = 1 THEN 'Autoria unica'
	ELSE 'Colaboracion'
END AS es_colaboracion
from U_IDSCPS autor, EID
where autor.id = EID.id and autor.rank = EID.rank --and autor.parserank = EID.parserank
group by EID.data
order by count(distinct autor.data) desc;

--10. cuenta articulos x cuenta autores
select cuenta, count(data)
from
(
select EID.data, count(distinct autor.data) as cuenta
from U_IDSCPS autor, EID
where autor.id = EID.id and autor.rank = EID.rank --and autor.parserank = EID.parserank
group by EID.data
order by count(distinct autor.data) desc
) t
group by cuenta;

--11
select a.file, a.id, a.rank, a.parserank, 'Colombia' as es_colombia
from U_Authors_with_affiliations a
where lower(a.data) like '%, colombia'
UNION
select a.file, a.id, a.rank, a.parserank, 'Internacional' as es_colombia
from U_Authors_with_affiliations a
where lower(a.data) not like '%, colombia'

select EID.data, cuenta_autor_articulo.cuenta, es_colombia.autores_colombia
FROM EID,
(
select EID.data, count(distinct autor.data) as cuenta
from U_IDSCPS autor, EID
where autor.id = EID.id and autor.rank = EID.rank --and autor.parserank = EID.parserank
group by EID.data
) cuenta_autor_articulo,
(
select a.file, a.id, a.rank, count(a.parserank) as autores_colombia
from U_Authors_with_affiliations a
where lower(a.data) like '%, colombia'
group by a.file, a.id, a.rank
UNION
select a.file, a.id, a.rank, count(a.parserank) as autores_colombia
from U_Authors_with_affiliations a
where lower(a.data) not like '%, colombia'
group by a.file, a.id, a.rank
) es_colombia
where EID.data = cuenta_autor_articulo.data
and EID.file = es_colombia.file and EID.id = es_colombia.id and EID.rank = es_colombia.rank;

select EID.data, 
CASE 
	WHEN cuenta_autor_articulo.cuenta = es_colombia.autores THEN 'Nacional'
	WHEN cuenta_autor_articulo.cuenta = es_internacional.autores THEN 'Internacional'
	ELSE 'Mixto'
END AS afiliacion
FROM EID,
(
select EID.data, count(distinct autor.data) as cuenta
from U_IDSCPS autor, EID
where autor.id = EID.id and autor.rank = EID.rank --and autor.parserank = EID.parserank
group by EID.data
) cuenta_autor_articulo,
(
select a.file, a.id, a.rank, count(a.parserank) as autores
from U_Authors_with_affiliations a
where lower(a.data) like '%, colombia'
group by a.file, a.id, a.rank
) es_colombia,
(
select a.file, a.id, a.rank, count(a.parserank) as autores
from U_Authors_with_affiliations a
where lower(a.data) not like '%, colombia'
group by a.file, a.id, a.rank
) es_internacional
where EID.data = cuenta_autor_articulo.data
and EID.file = es_colombia.file and EID.id = es_colombia.id and EID.rank = es_colombia.rank
and EID.file = es_internacional.file and EID.id = es_internacional.id and EID.rank = es_internacional.rank
--and cuenta_autor_articulo.cuenta = es_colombia.autores_colombia;
order by afiliacion;

--EAR 08/08/2024
******************************************************************************
create table colaboracion as 
select EID.file, EID.id, EID.rank, EID.parserank, 
CASE 
	WHEN cuenta_autor_articulo.cuenta = es_colombia.autores THEN 'Nacional'
	WHEN cuenta_autor_articulo.cuenta = es_internacional.autores THEN 'Internacional'
	ELSE 'Mixto'
END AS data
FROM EID,
(
select EID.data, count(distinct autor.data) as cuenta
from U_IDSCPS autor, EID
where autor.id = EID.id and autor.rank = EID.rank --and autor.parserank = EID.parserank
group by EID.data
) cuenta_autor_articulo,
(
select a.file, a.id, a.rank, count(a.parserank) as autores
from U_Authors_with_affiliations a
where lower(a.data) like '%, colombia'
group by a.file, a.id, a.rank
) es_colombia,
(
select a.file, a.id, a.rank, count(a.parserank) as autores
from U_Authors_with_affiliations a
where lower(a.data) not like '%, colombia'
group by a.file, a.id, a.rank
) es_internacional
where EID.data = cuenta_autor_articulo.data
and EID.file = es_colombia.file and EID.id = es_colombia.id and EID.rank = es_colombia.rank
and EID.file = es_internacional.file and EID.id = es_internacional.id and EID.rank = es_internacional.rank
order by afiliacion;

*****************************************************

--12. cantidad docs citados filtre year 
select count(distinct EID.data)
from EID, year, Cited_by, U_Affiliations a
where EID.file = year.file and EID.id = year.id and EID.rank = year.rank and EID.parserank = year.parserank
and EID.file = Cited_by.file and EID.id = Cited_by.id and EID.rank = Cited_by.rank and EID.parserank = Cited_by.parserank
and EID.file = a.file and EID.id = a.id and EID.rank = a.rank and EID.parserank = a.parserank
and year.data >= 2015 and year.data <= 2020
and a.data = 'Facultad de Ingeniería, Universidad Católica de Colombia, Colombia'
and Cited_by.data >= 1;

--13. promedio de citas x docs citados
select sum(Cited_by.data) total_citaciones, count(distinct EID.data) total_docs_citados, sum(Cited_by.data)/count(distinct EID.data) as promedio
from EID, year, Cited_by, U_Affiliations a
where EID.file = year.file and EID.id = year.id and EID.rank = year.rank and EID.parserank = year.parserank
and EID.file = Cited_by.file and EID.id = Cited_by.id and EID.rank = Cited_by.rank and EID.parserank = Cited_by.parserank
and year.data >= 2015 and year.data <= 2020
and a.data = 'Facultad de Ingeniería, Universidad Católica de Colombia, Colombia'
and Cited_by.data >= 1;

--14. total_articulos x autor year quartile categories
select U_IDSCPS.data, year.data, s.SJR_Best_Quartile, s.Categories, count(DISTINCT EID.data) total_docs
from EID, U_IDSCPS, year, scimagojo_all_columns s
where EID.file = year.file and EID.id = year.id and EID.rank = year.rank and EID.parserank = year.parserank
and EID.file = U_IDSCPS.file and EID.id = U_IDSCPS.id and EID.rank = U_IDSCPS.rank
--and s.issn_CORPUS_14200_file = EID.file 
and s.ISSN_year_source_id_11497 = EID.id 
--and s.rank = EID.rank and s.issn_CORPUS_14200_parserank = EID.parserank
group by U_IDSCPS.data, year.data, s.SJR_Best_Quartile, s.Categories;  

--14.2 total_articulos x quartile
select SJR_Best_Quartile, sum(total_docs)
from 
(
select U_IDSCPS.data, year.data, s.SJR_Best_Quartile, count(DISTINCT EID.data) total_docs
from EID, U_IDSCPS, year, scimagojo_all_columns s
where EID.file = year.file and EID.id = year.id and EID.rank = year.rank and EID.parserank = year.parserank
and EID.file = U_IDSCPS.file and EID.id = U_IDSCPS.id and EID.rank = U_IDSCPS.rank
and s.ISSN_year_source_id_11497 = EID.id
group by U_IDSCPS.data, year.data, s.SJR_Best_Quartile
)
group by SJR_Best_Quartile;

--15. total docs x area quartile
select s.SJR_Best_Quartile, count(DISTINCT EID.data) total_docs
from EID, U_IDSCPS, year, scimagojo_all_columns s
where EID.file = year.file and EID.id = year.id and EID.rank = year.rank and EID.parserank = year.parserank
and EID.file = U_IDSCPS.file and EID.id = U_IDSCPS.id and EID.rank = U_IDSCPS.rank
and s.ISSN_year_source_id_11497 = EID.id
group by s.SJR_Best_Quartile;

--16. revistas x cuartiles - categorias - year arti
select s.SJR_Best_Quartile, s.Categories, year.data, count(DISTINCT s.Sourceid) total_revistas
from EID, year, scimagojo_all_columns s
where EID.file = year.file and EID.id = year.id and EID.rank = year.rank and EID.parserank = year.parserank
and s.ISSN_year_source_id_11497 = EID.id
group by s.SJR_Best_Quartile, s.Categories, year.data
order by count(DISTINCT s.Sourceid) desc;

--17 porcentaje total_docs_quar_cate total_docs x quartile categories 
select s.SJR_Best_Quartile, s.Categories, count(DISTINCT EID.data) as total_docs_quar_cate, total_quartil.total_docs,
count(DISTINCT EID.data)/total_quartil.total_docs 
from EID, U_IDSCPS, year, scimagojo_all_columns s,
(
select s.SJR_Best_Quartile, count(DISTINCT EID.data) total_docs
from EID, U_IDSCPS, year, scimagojo_all_columns s
where EID.file = year.file and EID.id = year.id and EID.rank = year.rank and EID.parserank = year.parserank
and EID.file = U_IDSCPS.file and EID.id = U_IDSCPS.id and EID.rank = U_IDSCPS.rank
and s.ISSN_year_source_id_11497 = EID.id
group by s.SJR_Best_Quartile
) total_quartil
where EID.file = year.file and EID.id = year.id and EID.rank = year.rank and EID.parserank = year.parserank
and EID.file = U_IDSCPS.file and EID.id = U_IDSCPS.id and EID.rank = U_IDSCPS.rank
and s.ISSN_year_source_id_11497 = EID.id
and s.SJR_Best_Quartile = total_quartil.SJR_Best_Quartile 
group by s.SJR_Best_Quartile, s.Categories;

--17
select s.SJR_Best_Quartile, s.Categories, count(DISTINCT EID.data) as total_docs_quar_cate, total_quartil.total_docs,
count(DISTINCT EID.data)/total_quartil.total_docs 
from EID, U_IDSCPS, year, scimagojo_all_columns s,
(
select s.SJR_Best_Quartile, count(DISTINCT EID.data) total_docs
from EID, U_IDSCPS, year, scimagojo_all_columns s
where EID.file = year.file and EID.id = year.id and EID.rank = year.rank and EID.parserank = year.parserank
and EID.file = U_IDSCPS.file and EID.id = U_IDSCPS.id and EID.rank = U_IDSCPS.rank
and s.issn_CORPUS_14200_file = EID.file and s.issn_CORPUS_14200_id = EID.id and s.issn_CORPUS_14200_rank = EID.rank and s.issn_CORPUS_14200_parserank = EID.parserank
group by s.SJR_Best_Quartile
) total_quartil
where EID.file = year.file and EID.id = year.id and EID.rank = year.rank and EID.parserank = year.parserank
and EID.file = U_IDSCPS.file and EID.id = U_IDSCPS.id and EID.rank = U_IDSCPS.rank
and s.issn_CORPUS_14200_file = EID.file and s.issn_CORPUS_14200_id = EID.id and s.issn_CORPUS_14200_rank = EID.rank and s.issn_CORPUS_14200_parserank = EID.parserank
and s.SJR_Best_Quartile = total_quartil.SJR_Best_Quartile 
group by s.SJR_Best_Quartile, s.Categories;

select distinct issn_CORPUS_14200_data, source_title
FROM
(
select s.issn_CORPUS_14200_data, st.data as source_title, year.data, count(*) 
from scimagojo_all_columns s, Source_title st, year
where s.issn_CORPUS_14200_file = st.file and s.issn_CORPUS_14200_id = st.id and s.issn_CORPUS_14200_rank = st.rank and s.issn_CORPUS_14200_parserank = st.parserank
and s.issn_CORPUS_14200_file = year.file and s.issn_CORPUS_14200_id = year.id and s.issn_CORPUS_14200_rank = year.rank and s.issn_CORPUS_14200_parserank = year.parserank
and s.Sourceid is null
and year.data >= 2011 and year.data <= 2019
group by s.issn_CORPUS_14200_data, st.data, year.data
order by count(*) desc
);

======================================================================================

select P_AREA_TEMATICA, count(*) 
from REVISION_FIS_INV_PTA_IDSCOPUS_19082024
group by P_AREA_TEMATICA;

select l.area_agregadarojo, count(r.P_TITULO) 
from REVISION_FIS_INV_PTA_IDSCOPUS_19082024 r, lineas_tematicas_input l
where r.P_AREA_TEMATICA = l.areoriginal
group by l.area_agregadarojo
order by count(r.P_TITULO) desc;

select *
from 
(
select distinct r.P_TITULO, r.P_PALABRAS_CLAVES, l.lineas as area1 
from REVISION_FIS_INV_PTA_IDSCOPUS_19082024 r left join CLASIFICACION_LINEAS_MODELO l on r.P_AREA_TEMATICA = l.areoriginal
where r.P_TITULO is not null and r.P_PALABRAS_CLAVES is not null
)
where area1 is null;

select * 
from REVISION_FIS_INV_PTA_IDSCOPUS_19082024 r, classified_projects l
where r.P_PROYECTO_ID = l.project_id and r.P_TITULO = upper(l.title)
and title is not null;

select l.theme_grp, count(r.P_TITULO) 
from REVISION_FIS_INV_PTA_IDSCOPUS_19082024 r, classified_projects l
where r.P_PROYECTO_ID = l.project_id
group by l.theme_grp
order by count(r.P_TITULO) desc;

select r.I_IDENTIFICACION_INVPPAL, count(distinct l.lineas) 
from REVISION_FIS_INV_PTA_IDSCOPUS_19082024 r left join CLASIFICACION_LINEAS_MODELO l on r.P_AREA_TEMATICA = l.areoriginal
where r.P_TITULO is not null
group by r.I_IDENTIFICACION_INVPPAL
having count(distinct l.lineas) > 1;

create table REVISION_FIS_INV_PTA_IDSCOPUS_19082024_area as
select distinct r.*, COALESCE(l.lineas, p.area1) AS area_final
from REVISION_FIS_INV_PTA_IDSCOPUS_19082024 r 
left join CLASIFICACION_LINEAS_MODELO l on r.P_AREA_TEMATICA = l.areoriginal
left join proyectos_clasificadosLinearSVC p on r.P_TITULO = p.P_TITULO;

select * from REVISION_FIS_INV_PTA_IDSCOPUS_19082024_area;
