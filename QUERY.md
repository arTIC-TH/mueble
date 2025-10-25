sobre la base de datos de scopus

# trabajo sobre revistas

qry: listado de revistas: source_title + issn + isbn

" {SQL}
select s.data as source, i.data as ISBN, iss.data as ISSN, o.data as openAccess, count(distinct s.id) 
from Source_title s
left join ISBN i on s.id = i.id and s.parserank = i.parserank and s.rank = i.rank
left join ISSN iss on s.id = iss.id and s.parserank = iss.parserank and s.rank = iss.rank
left join open_access_split o on s.id = o.id and s.parserank = o.parserank and s.rank = o.rank
group by s.data, i.data, iss.data, o.data;
"

qry1: articulos por revista por año 

" {SQL}
select s.data as source, i.data as ISBN, iss.data as ISSN, o.data as openAccess, y.data, count(distinct s.id) 
from Source_title s
left join year y on s.id = y.id and s.parserank = y.parserank and s.rank = y.rank
left join ISBN i on s.id = i.id and s.parserank = i.parserank and s.rank = i.rank
left join ISSN iss on s.id = iss.id and s.parserank = iss.parserank and s.rank = iss.rank
left join open_access_split o on s.id = o.id and s.parserank = o.parserank and s.rank = o.rank
group by s.data, i.data, iss.data, o.data, y.data;
"

unir con informacion sjr, para revistas de qry1
como cargar los datos de revistas en sqlite

> trabajo sobre autor
qry2: afiliaciones unicas de los investigadores
identificar del tital de investigadores, aquellos que reportan afiliaciones a UNILLANOS
hacer split para extraer el país


> trabajo sobre afiliaciones

> trabajo sobre investigadores activos en grupos en minciencias

 
