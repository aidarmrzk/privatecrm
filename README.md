Задача:
Ваша задача - создать простую таблицу с использованием Vue 3, которая будет отображать данные из JSON и обеспечивать сортировку по всем столбцам.
Требования:
Используйте Vue 3 для создания компонента таблицы.
Реализуйте таблицу, отображающую данные из JSON.
Обеспечьте сортировку данных по столбцу статус.
Добавьте функционал смены направления сортировки при повторном нажатии на заголовок столбца (по возрастанию/убыванию).
Отобразите все столбцы из JSON данных в таблице.
Условия:
У одного клиента может быть одна и более диет (Каждая диета должна быть отделена от другой горизонтальным разделителем.)
в заказе и они могут иметь разное время окончания.
Нужно учесть это при сортировке.
Сортировка по статусу:

Начинается через Х дней. (где чем больше х тем выше в списке)
заканчивается сегодня
заканчивается завтра
заканчивается через Х дней (где чем больше х тем ниже в списке)
закончилось Х дней назад (где чем больше х тем ниже в списке)

Доп задание:
Реализовать фильтр по столбцам.
Например, отобразить заказы со статусом Начинается через Х дней