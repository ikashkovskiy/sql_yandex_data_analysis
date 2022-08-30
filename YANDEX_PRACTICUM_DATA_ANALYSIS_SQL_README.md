SELECT s.last_name AS employee_last_name, st.last_name AS manager_last_name
FROM staff AS s
LEFT OUTER JOIN staff AS st ON s.reports_to=st.employee_id;

У некоторых сотрудников есть менеджеры — их идентификаторы указаны в поле reports_to. Посмотрите внимательно на схему базы: таблица staff отсылает сама к себе. Это нормально, можно не создавать новую таблицу с менеджерами.
Теперь можно разобраться в иерархии команды. Отобразите таблицу с двумя полями: в первое поле войдут фамилии всех сотрудников, а во второе — фамилии их менеджеров. Назовите поля employee_last_name и manager_last_name.


SELECT a.actor_id, a.last_name, COUNT(m.film_id) AS count_film
FROM movie AS m
LEFT OUTER JOIN film_actor AS fa ON m.film_id=fa.film_id
LEFT OUTER JOIN actor AS a ON fa.actor_id=a.actor_id
--WHERE film_id IS NOT NULL
GROUP BY a.actor_id;

.
Выведите идентификатор и фамилию актёра или актрисы, а также количество фильмов, в которых он или она снимались. Если нет информации, кто снимался в фильме, такой фильм тоже должен войти в таблицу.


SELECT art.name 
FROM artist AS art
LEFT OUTER JOIN album AS alb ON art.artist_id=alb.artist_id
WHERE alb.album_id IS NULL
GROUP BY art.artist_id


Отобразите на экране имена исполнителей, для которых в базе данных не нашлось ни одного музыкального альбома.


SELECT top_40.rating,
	   MIN(top_40.length) AS min_length,
	   MAX(top_40.length) AS max_length,
          AVG(top_40.length) AS avg_length,
      	  MIN(top_40.rental_rate) AS min_rental_rate,
          MAX(top_40.rental_rate) AS max_rental_rate,
          AVG(top_40.rental_rate) AS avg_rental_rate 
FROM 
        (SELECT m.title,
         m.rental_rate,
         m.length,
	 m.rating
         FROM movie AS m
         WHERE m.rental_rate > 2
        ORDER BY m.length DESC
        LIMIT 40) AS top_40
GROUP BY rating
ORDER BY AVG(length);

Проанализируйте данные о возрастных рейтингах отобранных фильмов. Выгрузите в итоговую таблицу следующие поля:
* 		возрастной рейтинг (поле rating);
* 		минимальное и максимальное значения длительности (поле length); назовите поля min_length и max_length соответственно;
* 		среднее значение длительности (поле length); назовите поле avg_length;
* 		минимум, максимум и среднее для цены просмотра (поле rental_rate); назовите поля min_rental_rate, max_rental_rate, avg_rental_rate соответственно.
Отсортируйте среднюю длительность фильма по возрастанию.


SELECT AVG(top_1.min_length) AS avg_min_length, 
       AVG(top_1.max_length) AS avg_max_length
FROM 
    (SELECT top.rating,
       MIN(top.length) AS min_length,
       MAX(top.length) AS max_length,
       AVG(top.length) AS avg_length,
       MIN(top.rental_rate) AS min_rental_rate,
       MAX(top.rental_rate) AS max_rental_rate,
       AVG(top.rental_rate) AS avg_rental_rate
FROM
  (SELECT title,
          rental_rate,
          length,
          rating
   FROM movie
   WHERE rental_rate > 2
   ORDER BY length DESC
   LIMIT 40) AS top
GROUP BY top.rating
ORDER BY avg_length) AS top_1;


.
Найдите средние значения полей, в которых указаны минимальная и максимальная длительность отобранных фильмов. Отобразите только два этих поля. Назовите их avg_min_length и avg_max_length соответственно.



SELECT AVG(rock.count_track)
FROM( 
    SELECT a.title, COUNT(t.name) AS count_track
FROM track AS t
LEFT OUTER JOIN album AS a ON t.album_id=a.album_id
WHERE a.title LIKE '%Rock%'
GROUP BY a.title
HAVING COUNT(t.name) >= 8
LIMIT 10) AS rock;

Отберите альбомы, названия которых содержат слово 'Rock' и его производные. В этих альбомах должно быть восемь или более треков. Выведите на экран одно число — среднее количество композиций в отобранных альбомах.


SELECT countries.billing_country
FROM
    (SELECT i.billing_country, 
     EXTRACT(MONTH FROM CAST(i.invoice_date AS date)) as month, 
     AVG(i.total) AS avg_total
    FROM invoice AS i
    WHERE EXTRACT(YEAR FROM CAST(i.invoice_date AS date)) = '2009'
    AND EXTRACT(MONTH FROM CAST(i.invoice_date AS date)) IN ('2', '5', '7', '10')
    GROUP BY i.billing_country, EXTRACT(MONTH FROM CAST(i.invoice_date AS date)))
    AS countries
GROUP BY countries.billing_country
HAVING SUM(countries.avg_total) >= 10;

Для каждой страны посчитайте среднюю стоимость заказов в 2009 году по месяцам. Отберите данные за 2, 5, 7 и 10 месяцы и сложите средние значения стоимости заказов. Выведите названия стран, у которых это число превышает 10 долларов.


SELECT billing_country, MIN(total) AS min_total, MAX(total) AS max_total, AVG(total) AS avg_total
FROM invoice
WHERE invoice_id IN (SELECT invoice_id
FROM invoice_line
GROUP BY invoice_id
HAVING COUNT(track_id) > 5)
GROUP BY billing_country
ORDER BY avg_total DESC;


SELECT name
FROM genre
WHERE genre_id IN(
SELECT genre_id
FROM track
ORDER BY milliseconds ASC
LIMIT 10);

Отберите десять самых коротких по продолжительности треков и выгрузите названия их жанров.


SELECT DISTINCT(billing_city)
FROM invoice
WHERE total > (SELECT AVG(total)
FROM invoice
WHERE EXTRACT(YEAR FROM CAST(invoice_date AS date)) = '2009');


Выгрузите уникальные названия городов, в которых стоимость заказов превышает среднее значение за 2009 год.


SELECT c.name, AVG(m.length)
FROM movie AS m
LEFT OUTER JOIN film_category AS fc ON m.film_id=fc.film_id
LEFT OUTER JOIN category AS c ON fc.category_id=c.category_id
WHERE m.rating IN (SELECT rating
FROM movie 
GROUP BY rating
ORDER BY AVG(rental_rate) DESC
LIMIT 1)
GROUP BY c.name
ORDER BY c.name;


Найдите возрастной рейтинг с самыми дорогими для аренды фильмами. Для этого посчитайте среднюю стоимость аренды фильма каждого рейтинга. Выведите на экран названия категорий фильмов с этим рейтингом. Добавьте второе поле со средним значением продолжительности фильмов.
