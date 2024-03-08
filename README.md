# TALLER POSTGRES
### **3. Consultas sobre una tabla**

1. Devuelve un listado con el primer apellido, segundo apellido y el nombre de todos los alumnos. El listado deberá estar ordenado alfabéticamente de menor a mayor por el primer apellido, segundo apellido y nombre.

~~~sql
SELECT nombre, apellido1, apellido2 FROM persona WHERE tipo = 'Alumno' ORDER BY nombre, apellido1, apellido2;
~~~

2. Averigua el nombre y los dos apellidos de los alumnos que **no** han dado de alta su número de teléfono en la base de datos.

~~~sql
SELECT nombre, apellido1, apellido2 FROM persona WHERE telefono IS NULL AND tipo = 'Alumno';
~~~

3. Devuelve el listado de los alumnos que nacieron en `1999`.

~~~sql
SELECT nombre, apellido1 FROM persona WHERE DATE_PART('YEAR', fecha_nacimiento ) = '1999' AND tipo = 'Alumno';
~~~

4. Devuelve el listado de profesores que **no** han dado de alta su número de teléfono en la base de datos y además su nif termina en `K`.

~~~sql
SELECT nombre, apellido1, apellido2, nif FROM persona WHERE telefono IS NULL AND tipo = 'Profesor' AND nif LIKE '%K';
~~~

5. Devuelve el listado de las asignaturas que se imparten en el primer cuatrimestre, en el tercer curso del grado que tiene el identificador `7`.

~~~sql
SELECT nombre,creditos,tipo FROM asignatura WHERE id_grado = 7 AND cuatrimestre = 1;
~~~


### **4. Consultas multitabla (Composición interna)**

1. Devuelve un listado con los datos de todas las **alumnas** que se han matriculado alguna vez en el `Grado en Ingeniería Informática (Plan 2015)`.

~~~sql
SELECT p.nombre, apellido1, apellido2 FROM persona p
INNER JOIN asignatura a ON p.id = a.id
WHERE p.tipo = 'Alumno' AND sexo = 'F' AND a.id_grado = 4;
~~~

2. Devuelve un listado con todas las asignaturas ofertadas en el `Grado en Ingeniería Informática (Plan 2015)`.

~~~sql
SELECT p.nombre, apellido1, apellido2, d.nombre FROM persona p 
INNER JOIN profesor a ON p.id = a.id_profesor
INNER JOIN departamento d ON d.id = a.id_departamento
WHERE p.tipo = 'Profesor'
ORDER BY p.nombre, apellido1, apellido2;

~~~

3. Devuelve un listado de los profesores junto con el nombre del departamento al que están vinculados. El listado debe devolver cuatro columnas, primer apellido, segundo apellido, nombre y nombre del departamento. El resultado estará ordenado alfabéticamente de menor a mayor por los apellidos y el nombre.

~~~sql
SELECT p.nombre, apellido1, apellido2, d.nombre FROM persona p 
INNER JOIN profesor a ON p.id = a.id_profesor
INNER JOIN departamento d ON d.id = a.id_departamento
WHERE p.tipo = 'Profesor'
ORDER BY p.nombre, apellido1, apellido2;
~~~

4. Devuelve un listado con el nombre de las asignaturas, año de inicio y año de fin del curso escolar del alumno con nif `26902806M`.

~~~sql
SELECT  asg.nombre,anyo_inicio, anyo_fin FROM persona p
INNER JOIN alumno_se_matricula_asignatura a ON p.id = a.id_alumno
INNER JOIN curso_escolar d ON d.id = a.id_curso_escolar
INNER JOIN asignatura asg ON asg.id = a.id_asignatura
WHERE p.tipo = 'Alumno' AND nif = '26902806M';

~~~

5. Devuelve un listado con el nombre de todos los departamentos que tienen profesores que imparten alguna asignatura en el `Grado en Ingeniería Informática (Plan 2015)`.

~~~sql
SELECT DISTINCT d.nombre FROM persona p 
INNER JOIN profesor a ON p.id = a.id_profesor
INNER JOIN departamento d ON d.id = a.id_departamento
INNER JOIN asignatura asg ON asg.id_profesor = a.id_profesor
WHERE asg.id_grado = 4;
~~~

6. Devuelve un listado con todos los alumnos que se han matriculado en alguna asignatura durante el curso escolar 2018/2019.

~~~sql
SELECT DISTINCT p.nombre, apellido1, apellido2,anyo_inicio, anyo_fin FROM persona p
INNER JOIN alumno_se_matricula_asignatura a ON p.id = a.id_alumno
INNER JOIN curso_escolar d ON d.id = a.id_curso_escolar
WHERE p.tipo = 'Alumno' 
AND
DATE_PART('YEAR', anyo_inicio) = 2018
AND
DATE_PART('YEAR', anyo_fin) = 2019;
~~~


### **5. Consultas multitabla (Composición externa)**

Resuelva todas las consultas utilizando las cláusulas `LEFT JOIN` y `RIGHT JOIN`.

1. Devuelve un listado con los nombres de **todos** los profesores y los departamentos que tienen vinculados. El listado también debe mostrar aquellos profesores que no tienen ningún departamento asociado. El listado debe devolver cuatro columnas, nombre del departamento, primer apellido, segundo apellido y nombre del profesor. El resultado estará ordenado alfabéticamente de menor a mayor por el nombre del departamento, apellidos y el nombre.

~~~sql
SELECT p.nombre, apellido1, apellido2, d.nombre as departamento FROM persona p 
LEFT JOIN profesor a ON p.id = a.id_profesor
LEFT JOIN departamento d ON d.id = a.id_departamento
WHERE p.tipo = 'Profesor'
ORDER BY p.nombre, apellido1, apellido2, departamento;
~~~

2. Devuelve un listado con los profesores que no están asociados a un departamento.

~~~sql
SELECT p.nombre, apellido1, apellido2 FROM persona p 
LEFT JOIN profesor a ON p.id = a.id_profesor
LEFT JOIN departamento d ON d.id = a.id_departamento
WHERE a.id_departamento IS NULL AND p.tipo = 'Profesor'
ORDER BY p.nombre, apellido1, apellido2;
~~~

3. Devuelve un listado con los departamentos que no tienen profesores asociados.

~~~sql
SELECT d.nombre as departamento FROM persona p 
INNER JOIN profesor a ON p.id = a.id_profesor
RIGHT JOIN departamento d ON d.id = a.id_departamento
WHERE a.id_profesor IS NULL
ORDER BY departamento;
~~~

4. Devuelve un listado con los profesores que no imparten ninguna asignatura.

~~~sql
SELECT p.nombre, apellido1, apellido2, p.id FROM persona p 
LEFT JOIN profesor a ON p.id = a.id_profesor
LEFT JOIN asignatura asg ON asg.id_profesor = a.id_profesor
WHERE asg.id_profesor IS NULL AND p.tipo = 'Profesor'
ORDER BY p.nombre, apellido1, apellido2;
~~~

5. Devuelve un listado con las asignaturas que no tienen un profesor asignado.

~~~sql
SELECT nombre FROM asignatura WHERE id_profesor IS NULL;
~~~

6. Devuelve un listado con todos los departamentos que tienen alguna asignatura que no se haya impartido en ningún curso escolar. El resultado debe mostrar el nombre del departamento y el nombre de la asignatura que no se haya impartido nunca.

~~~sql
SELECT DISTINCT d.nombre FROM departamento d
INNER JOIN profesor p ON p.id_departamento = d.id
INNER JOIN asignatura a ON a.id_profesor = p.id_profesor
LEFT JOIN alumno_se_matricula_asignatura am ON am.id_asignatura = a.id
WHERE am.id_curso_escolar IS NULL;
~~~


### **6. Consultas resumen**

1. Devuelve el número total de **alumnas** que hay.

~~~sql
SELECT COUNT(*) as "Numero de alumnas" FROM persona 
WHERE sexo = 'F' AND tipo = 'Alumno';
~~~

2. Calcula cuántos alumnos nacieron en 2005.

~~~sql
SELECT COUNT(*) as "Numero de alumnas" FROM persona 
WHERE DATE_PART('YEAR', fecha_nacimiento ) = '2005' AND tipo = 'Alumno';
~~~

3. Calcula cuántos profesores hay en cada departamento. El resultado sólo debe mostrar dos columnas, una con el nombre del departamento y otra con el número de profesores que hay en ese departamento. El resultado sólo debe incluir los departamentos que tienen profesores asociados y deberá estar ordenado de mayor a menor por el número de profesores.

~~~sql
SELECT nombre, COUNT(id_profesor) as "Cantidad profesores" FROM departamento d
INNER JOIN profesor p ON p.id_departamento = d.id
GROUP BY nombre
ORDER BY COUNT(id_profesor) DESC;
~~~

4. Devuelve un listado con todos los departamentos y el número de profesores que hay en cada uno de ellos. Tenga en cuenta que pueden existir departamentos que no tienen profesores asociados. Estos departamentos también tienen que aparecer en el listado.

~~~sql
SELECT nombre, COUNT(id_profesor) as "Cantidad profesores" FROM departamento d
LEFT JOIN profesor p ON p.id_departamento = d.id
GROUP BY nombre
ORDER BY COUNT(id_profesor);
~~~

5. Devuelve un listado con el nombre de todos los grados existentes en la base de datos y el número de asignaturas que tiene cada uno. Tenga en cuenta que pueden existir grados que no tienen asignaturas asociadas. Estos grados también tienen que aparecer en el listado. El resultado deberá estar ordenado de mayor a menor por el número de asignaturas.

~~~sql
SELECT g.nombre, COUNT(a.id) FROM grado g
LEFT JOIN asignatura a ON a.id_grado = g.id
GROUP BY g.nombre
ORDER BY COUNT(a.id);
~~~

6. Devuelve un listado con el nombre de todos los grados existentes en la base de datos y el número de asignaturas que tiene cada uno, de los grados que tengan más de `40` asignaturas asociadas.

~~~sql
SELECT g.nombre, COUNT(a.id) FROM grado g
LEFT JOIN asignatura a ON a.id_grado = g.id
GROUP BY g.nombre
HAVING COUNT(a.id) > 40
ORDER BY COUNT(a.id);
~~~

7. Devuelve un listado que muestre el nombre de los grados y la suma del número total de créditos que hay para cada tipo de asignatura. El resultado debe tener tres columnas: nombre del grado, tipo de asignatura y la suma de los créditos de todas las asignaturas que hay de ese tipo. Ordene el resultado de mayor a menor por el número total de crédidos.

~~~sql
SELECT g.nombre,a.tipo, SUM(a.creditos) FROM grado g
INNER JOIN asignatura a ON a.id_grado = g.id
GROUP BY g.nombre, a.tipo
ORDER BY COUNT(a.id);
~~~

8. Devuelve un listado que muestre cuántos alumnos se han matriculado de alguna asignatura en cada uno de los cursos escolares. El resultado deberá mostrar dos columnas, una columna con el año de inicio del curso escolar y otra con el número de alumnos matriculados.

~~~sql
SELECT c.anyo_inicio, COUNT(am.id_alumno) FROM alumno_se_matricula_asignatura am
INNER JOIN curso_escolar c ON c.id = am.id_curso_escolar
GROUP BY c.anyo_inicio;
~~~

9. Devuelve un listado con el número de asignaturas que imparte cada profesor. El listado debe tener en cuenta aquellos profesores que no imparten ninguna asignatura. El resultado mostrará cinco columnas: id, nombre, primer apellido, segundo apellido y número de asignaturas. El resultado estará ordenado de mayor a menor por el número de asignaturas.

~~~sql
SELECT pr.id_profesor, p.nombre, apellido1, apellido2,COUNT(A.id) as "Cant_materias" FROM persona p 
INNER JOIN profesor pr ON pr.id_profesor = p.id
INNER JOIN asignatura a ON pr.id_profesor = a.id_profesor
GROUP BY pr.id_profesor, p.nombre, apellido1, apellido2;
~~~


### **1.5.8 Subconsultas**

1. Devuelve todos los datos del alumno más joven.

~~~sql
SELECT * FROM persona 
WHERE tipo = 'Alumno'
ORDER BY fecha_nacimiento DESC LIMIT 1;
~~~
