Создать триггер для поддержки витрины в актуальном состоянии.

Описание/Пошаговая инструкция выполнения домашнего задания:

Скрипт и развернутое описание задачи – в ЛК (файл hw_triggers.sql) или по ссылке: https://disk.yandex.ru/d/l70AvknAepIJXQ

В БД создана структура, описывающая товары (таблица goods) и продажи (таблица sales).

Есть запрос для генерации отчета – сумма продаж по каждому товару.

БД была денормализована, создана таблица (витрина), структура которой повторяет структуру отчета.

Создать триггер на таблице продаж, для поддержки данных в витрине в актуальном состоянии (вычисляющий при каждой продаже сумму и записывающий её в витрину)

 -- Создал триггерную функцию:

        CREATE OR REPLACE FUNCTION pract_functions.tf_change_sales_qty()
        RETURNS trigger
        AS
        $$
        DECLARE

        BEGIN

            IF TG_LEVEL = 'ROW' THEN

                DELETE FROM pract_functions.good_sum_mart
                WHERE good_name = (SELECT good_name
                                   FROM pract_functions.goods
                                   WHERE goods_id = CASE WHEN TG_OP = 'DELETE' THEN old.good_id ELSE new.good_id END);

                INSERT INTO pract_functions.good_sum_mart(good_name, sum_sale)
                SELECT g.good_name, sum(g.good_price * s.sales_qty)
                FROM pract_functions.goods g
                INNER JOIN pract_functions.sales s ON s.good_id = g.goods_id
                WHERE s.good_id = case when TG_OP = 'DELETE' then old.good_id else new.good_id end
                GROUP BY G.good_name;

            END IF;

            RETURN CASE WHEN TG_OP = 'DELETE' THEN old ELSE new END;

        END;

        $$ LANGUAGE plpgsql;

    -- создал триггер:

        CREATE TRIGGER trg_after_sales
        AFTER INSERT OR UPDATE OR DELETE
        ON pract_functions.sales
        FOR EACH ROW
        EXECUTE FUNCTION pract_functions.tf_change_sales_qty();