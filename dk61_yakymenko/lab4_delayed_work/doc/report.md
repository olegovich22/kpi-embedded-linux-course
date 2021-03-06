# Звіт з виконання лабораторної роботи №4
## Завдання
- Реализовать два потока, запустить таймер и ворк в shared workqueue
- При срабатывании таймера проверить текущее значение jiffies, если оно кратно 11 – остановить поток 1, иначе – таймер должен перезапустить себя через 17 jiffies
- Внутри ворка проверить текущее значение jiffies, если оно кратно 11 – остановить поток 2, иначе – ворк должен уснуть на 17 jiffies и перезапустить себя
-  Добавить два связных списка, в которые аллоцировать и добавлять элементы со значениями jiffies, которые не привели к завершению потоков 1 и 2. Получается связь таймер - список 1 - поток 1. И ворк - список 2 - поток 2
- Внутри ворка и таймера использовать правильные аллокации для новых элементов списка, правильную синхронизацию работы со списком
- Предусмотреть, что пользователь может выгрузить модуль до отработки всех таймеров и ворков

## Хід роботи
Спершу було створено два потоки для таймеру та чергу. Як це робити добре було описано у минулій лабораторній роботі. Також було створено оголошення та структури для списків. Робота з ним теж була добре описана у минулій роботі.
### Таймер
Почнемо з налаштування таймеру.
Спершу потрібно оголосити обєкт таймеру
`static struct timer_list my_timer;`

Наступним кроком є створення функції обробника подій таймеру.
`void timer_fun(struct timer_list *t)`

Далі потрібно виконати налаштування таймеру. Це я вже роблю у потоці.
`timer_setup(&my_timer, timer_fun, 0);`

Після чого можна вже налаштувати коли саме спрацює таймерза допомогою функції
`mod_timer(&my_timer, jiffies + msecs_to_jiffies(10000));`

Щоб асинхронно зупинити таймер потрібно його просто видалити
`del_timer(&my_timer);`

На цьому налаштування таймеру закінчені.

### Черга
Спершу потрібно зробити оголошення функціїї черги та саму чергу
```c
void wq_fun(struct work_struct *work);
DECLARE_DELAYED_WORK(wq_task, wq_fun);
```
Далі потрібно реалізувати функцію черги.
Чергу створено, тепер можна запускати. Це робиться такою функцією
`schedule_delayed_work(&wq_task, msecs_to_jiffies(1000));`
Варт звернути увагу на те, що тут вказується не коли, а через скільки спрацює функція обробник.
Щоб відмінити виконання завдання потрібно викликати таку функцію 
`cancel_delayed_work(&wq_task);`
## Перегляд результатів
Коли реалізована черга, таймер та логіка функцій обробників можна переходити до тестів
```c
WQ 4297070320, 4
[ 8712.331918] WQ 4297070338, 0
[ 8712.331919] WQ GOT TRUE JIFFIES 4297070338, 0
[ 8721.283756] TIMER 4297072576, 5
[ 8721.355594] TIMER 4297072594, 1
[ 8721.431772] TIMER 4297072613, 9
[ 8721.503551] TIMER 4297072631, 5
[ 8721.575772] TIMER 4297072649, 1
[ 8721.647587] TIMER 4297072667, 8
[ 8721.723742] TIMER 4297072686, 5
[ 8721.795593] TIMER 4297072704, 1
[ 8721.867756] TIMER 4297072722, 8
[ 8721.939544] TIMER 4297072740, 4
[ 8722.011747] TIMER 4297072758, 0
[ 8722.011748] TIMER GOT TRUE JIFFIES 4297072758, 0
[ 8796.381525] Closing the module
[ 8796.381526] Timer list
[ 8796.381527] List element = 4297072740
[ 8796.381528] List element = 4297072722
[ 8796.381528] List element = 4297072704
[ 8796.381529] List element = 4297072686
[ 8796.381529] List element = 4297072667
[ 8796.381530] List element = 4297072649
[ 8796.381530] List element = 4297072631
[ 8796.381530] List element = 4297072613
[ 8796.381531] List element = 4297072594
[ 8796.381531] List element = 4297072576
[ 8796.381531] WQ list
[ 8796.381532] List element = 4297070320
[ 8796.381532] See you later aligator!

```
Я запрограмував таким чином, що функція черги буде виконуватись майже відразу, а функція таймеру тільки через деякий час. Можна переконатись, що все працює згідно завдання. Так склались зірки, що на чергу витратилось всього 1 ітерація до успіху.
##Висновок
Отже, в цій роботі було вивчено робота з чергою та таймером на рівні ядра.
