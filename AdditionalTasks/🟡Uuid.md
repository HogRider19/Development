UUID (Universally Unique IDentifier) представляет собой 128-битное число, которое в разработке ПО используется в качестве уникального идентификатора элементов. Его классическое текстовое представление является серией из 32 шестнадцатеричных символов, разделённых дефисами на пять групп по схеме 8-4-4-4-12.

GUID (Global Unique IDentifier) представляет реализацию алгоритма `UUID` четвертой версии от `Microsoft`. Работает на Windows чуть быстрее чем на Linux из-за того что реализация на Linux использует криптостойкий механизм генерации случайных чисел.

---
Одним из важных достоинств `UUID` является то, что их уникальность не зависит от центрального авторизующего органа или от координации между разными системами. Кто угодно может создать `UUID` с определённой уверенностью в том, что в обозримом будущем такое же значение больше никем не будет в последующем сгенерировано.  
  
Это позволяет комбинировать в одной БД идентификаторы, созданные разными участниками, или перемещать идентификаторы с ничтожной вероятностью коллизии.  
  
Также `UUID` можно использовать в качестве первичных ключей в базах данных, в качестве уникальных имён загружаемых файлов и так далее.  Для их генерирования вам не нужен центральный авторизующий орган. Но это обоюдоострое решение.
Из-за отсутствия контролёра невозможно отслеживать сгенерированные `UUID`.  
  
Есть и ещё несколько недостатков, которые нужно устранить. Cлучайность повышает защищённость, однако она усложняет отладку. Также, `UUID` может быть избыточным в некоторых ситуациях. Скажем, не имеет смысла использовать 128 битов для идентификации данных, общий размер которых меньше 128 битов.

---
### Версия 1 (Время, Частота, Идентификатор хоста)

В этом случае `UUID` генерируется так: к текущему времени добавляется какое-то идентифицирующее свойство устройства, чаще всего это MAC-адрес.

Идентификатор получают с помощью конкатенации 48-битного МАС-адреса,              60-битной временной метки, 14-битной уникализированной тактовой последовательности, а также 6 битов версии и варианта `UUID`.

Хотя эта реализация выглядит достаточно простой и надёжной, однако использование MAC-адреса машины, на которой генерируется идентификатор, не позволяет считать этот метод универсальным. Особенно, когда главным является безопасность. 

Поэтому в некоторых реализациях вместо идентификатора узла используется случайные биты, взятых из защищённого генератора случайных чисел.

### Версия 2 (Время, Пользователь, Идентификатор хоста)

Главное отличие этой версии от предыдущей в том, что вместо «случайности» в виде младших битов тактовой последовательности здесь используется идентификатор, характерный для системы. Часто это просто идентификатор текущего пользователя. 

### Версия 3 (Имя, MD5-хэш)

Если нужны уникальные идентификаторы для информации, связанной с именами или наименованием, то для этого обычно используют UUID версии 3 или версии 5.  
  
Они кодируют любые именуемые сущности в `UUID` значение. Самое важное для одного и того же пространства имен или текста будет сгенерирован такой же `UUID`.

Важно понимать, что ни `namespace`, ни входное имя не могут быть вычислены на основе `UUID`. Это необратимая операция. Единственное исключение перебор.
При одних и тех же данных `UUID` версий 3 и 5 будут детерминированными.

### Версия 4 (Случайные числа)

Является самой простой реализацией. В современных языках чаще всего используются `UUID` именно четвертой версии. Первые 6 битов зарезервированы под версию и вариант `UUID`, остаётся ещё 122 бита. В этой версии просто генерируется 128 случайных битов, а потом 6 из них заменяется данными о версии и варианте.  
  
Такие `UUID` полностью зависят от качества ГПСЧ (генератора псевдослучайных чисел). Если его алгоритм слишком прост, или ему не хватает начальных значений, то вероятность повторения идентификаторов начинает сильно возрастать. 

#### Версия 5 (имя, SHA-1-хэш)

Единственное отличие от версии 3 в том, что мы используем алгоритм хэширования `SHA-1` вместо `MD5`. Эта версия предпочтительнее третьей (`SHA-1` > `MD5`).


