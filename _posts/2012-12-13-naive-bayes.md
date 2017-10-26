---
layout: post
title: Наивный байесовский классификатор
tags: [naive, bayes, classifier, mining]
---

[Наивный байесовский классификатор](http://ru.wikipedia.org/wiki/%D0%9D%D0%B0%D0%B8%D0%B2%D0%BD%D1%8B%D0%B9_%D0%B1%D0%B0%D0%B9%D0%B5%D1%81%D0%BE%D0%B2%D1%81%D0%BA%D0%B8%D0%B9_%D0%BA%D0%BB%D0%B0%D1%81%D1%81%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%82%D0%BE%D1%80) - классификатор, на самом деле очень простой, за кучей формул скрывается очень простая идея.

Заметка написана по мотивам этой статьи http://bazhenov.me/blog/2012/06/11/naive-bayes.html

Предположим у нас есть следующая база знаний, о принадлежности сообщений к спаму:

**SPAM**

 * предоставляю услуги бухгалтера
 * спешите купить виагру

**NOT SPAM**

 * надо купить молоко

**Формула**

```
log(Dc/D) + foreach(word) { log( (Wc+1)/(V+Lc) ) }
```

Где:

 * `Dc` - количество **документов** в обучающей выборке принадлежащих классу `c` (2 SPAM документа и 1 - NOT SPAM)
 * `D` - общее количество **документов** в обучающей выборке (всего 3 документа)
 * `V` - общее количество **слов** во всех документах обучающей выборки (всего слов во всех документах - 8)
 * `Lc` - суммарное количество **слов** в документах класса `c` (искомого типа) в обучающей выборке (в SPAM документах - 6 слов, в NOT SPAM - 3)
 * `Wc` - сколько раз **слово** встречалось в документах класса `c` (искомого типа) в обучающей выборке

**Таблица слов**

```
WORD            SPAM    NOT SPAM
--------------------------------
предоставляю    1       0
услуги          1       0
бухгалтера      1       0
спешите         1       0
купить          1       1
виагру          1       0
надо            0       1
молоко          0       1
```

Другими словами мы просто разбили все обучаемые фразы на слова и записали их принадлежность к тому или иному классу (SPAM, NOT SPAM)

Предположим мы хотим проверить фразу: "надо купить сигареты"

Нам нужно посчитать вероятность принадлежности текста как к спаму так и в к не спаму.

SPAM
----

    log(Dc/D) + foreach(word) { log( (Wc+1)/(V+Lc) ) }

`log(Dc/D)` - эдакая константа, в обучающей выборке 2 spam документа из 3, соотв. это дело будет записано как `log(2/3)`

`V+Lc` - будет одинаковым для всех слов, `V` - 8 - именно столько всего слов в обучающей выборке, `Lc` - 6 - столько слов в обучающей выборке было в спамовых сообщениях.

Далее для каждого слова, считаем log.

"надо" тут у нас `Wc` равно 0, так как это слово не встречалось в обучающей выборке среди спама
"купить" тут у нас `Wc` равно 1, так как это слово один раз встречалось в обучающей выборке среди спама
"сигареты" тут у нас `Wc` равно 0, так как это слово не встречалось в обучающей выборке среди спама

Результат:

    log(2/3)
    + log( (0 + 1)/(8+6) ) //надо
    + log( (1 + 1)/(8+6) ) //купить
    + log( (0 + 1)/(8+6) ) //сигареты
    = -7.629

NOT SPAM
--------

Теперь все то же самое для не спама

    log(Dc/D) + foreach(word) { log( (Wc+1)/(V+Lc) ) }

`log(Dc/D)` - эдакая константа, в обучающей выборке 1 not spam документа из 3, соотв. это дело будет записано как `log(1/3)`

`V+Lc` - будет одинаковым для всех слов, `V` - 8 - именно столько всего слов в обучающей выборке, `Lc` - 3 - столько слов в обучающей выборке было в NOT SPAM сообщениях.

Далее для каждого слова, считаем log.

"надо" тут у нас `Wc` равно 1, так как это слово один раз встречалось в обучающей выборке среди not spam
"купить" тут у нас `Wc` равно 1, так как это слово один раз встречалось в обучающей выборке среди not spam
"сигареты" тут у нас `Wc` равно 0, так как это слово не встречалось в обучающей выборке среди not spam

Результат:

    log(1/3)
    + log( (1 + 1)/(8+3) ) //надо
    + log( (1 + 1)/(8+3) ) //купить
    + log( (0 + 1)/(8+3) ) //сигареты
    = -6.906

Из чего делаем вывод что фраза "надо купить сигареты" с большей вероятностью относиться к нормальным сообщениям нежели к spam'у.

Вот так можно на листике бумаги расписать всю работу алгоритма.

Если добавить в базу знаний к спам сообщенияе что нибуть вроде "надо купить виагру" - то результат поменяется в обратную сторону.

Все это дело можно очень просто сделать на любом языке, вот пример (не претендующий на крутость, просто чтобы показать и запомнить как оно работает) http://mac-blog.org.ua/examples/bayes.html

Все это дело расковыривалось в процессе ресерча возможности заюзать встроенный в SQL Server классификатор документов (там все это работает из коробки), но к сожалению оказалось что тамошний term extract и term lookup умеют работать только с английским языком, что свело на нет все надежды.


Нет повести печальнее на свете
------------------------------

Реализация алготма на SQL или когда стоит посмотреть в сторону Mongodb

```sql
DROP TABLE IF EXISTS words;
DROP TABLE IF EXISTS documents;

CREATE TABLE IF NOT EXISTS documents (
    uid INT UNSIGNED NOT NULL,
    total INT UNSIGNED NOT NULL DEFAULT 0,
    spam INT UNSIGNED NOT NULL DEFAULT 0,
    PRIMARY KEY (uid)
);

CREATE TABLE IF NOT EXISTS words (
    uid INT UNSIGNED NOT NULL,
    word VARCHAR(100) NOT NULL,
    spam INT UNSIGNED NOT NULL DEFAULT 0,
    ham INT UNSIGNED NOT NULL DEFAULT 0,
    PRIMARY KEY (uid, word),
    FOREIGN KEY (uid)
        REFERENCES documents(uid)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

DROP PROCEDURE IF EXISTS IncrementWordStats;
DELIMITER $$
CREATE PROCEDURE IncrementWordStats(input_uid INT UNSIGNED, input_word VARCHAR(100), input_spam INT, input_ham INT)
BEGIN

    INSERT INTO words (uid, word, spam, ham) VALUES (input_uid, input_word, input_spam, input_ham)
    ON DUPLICATE KEY UPDATE spam = spam + input_spam, ham = ham + input_ham;

END$$
DELIMITER ;

DROP PROCEDURE IF EXISTS explode;
DELIMITER $$
CREATE PROCEDURE explode(input TEXT)
BEGIN

    DROP TEMPORARY TABLE IF EXISTS explode;
    CREATE TEMPORARY TABLE explode(
        item VARCHAR(200) NOT NULL
    ) ENGINE=Memory;


    SET @separator = ',';
    SET @separator_length = CHAR_LENGTH(@separator);

    WHILE input != '' > 0 DO
        SET @current_value = SUBSTRING_INDEX(input, @separator, 1);
        INSERT INTO explode VALUES(@current_value);
        SET input = SUBSTRING(input, CHAR_LENGTH(@current_value) + @separator_length + 1);
    END WHILE;

END$$
DELIMITER ;

DROP PROCEDURE IF EXISTS AddDocument;
DELIMITER $$
CREATE PROCEDURE AddDocument(input_uid INT UNSIGNED, input_text TEXT, is_spam INT(1))
BEGIN

    INSERT INTO documents (uid, total, spam) VALUES(input_uid, 1, is_spam)
    ON DUPLICATE KEY UPDATE total = total + 1, spam = IF(is_spam = 1, spam + 1,spam);

    CALL explode(input_text);

    INSERT INTO words (uid, word, spam, ham)
    SELECT input_uid AS uid, item AS word, is_spam AS spam, IF(is_spam = 1, 0, 1) AS ham FROM explode
    ON DUPLICATE KEY UPDATE spam = spam + VALUES(spam), ham = ham + VALUES(ham);

END$$
DELIMITER ;

DROP PROCEDURE IF EXISTS CheckDocument;
DELIMITER $$
CREATE PROCEDURE CheckDocument(input_uid INT UNSIGNED, input_text TEXT)
BEGIN

    SELECT total INTO @D FROM documents WHERE documents.uid = input_uid;
    SELECT spam INTO @Dc_spam FROM documents WHERE documents.uid = input_uid;
    SELECT total - spam INTO @Dc_ham FROM documents WHERE documents.uid = input_uid;
    SELECT COUNT(*) INTO @V FROM words WHERE words.uid = input_uid;
    SELECT COUNT(*) INTO @Lc_spam  FROM words WHERE words.uid = input_uid AND spam <> 0;
    SELECT COUNT(*) INTO @Lc_ham  FROM words WHERE words.uid = input_uid AND ham <> 0;

    CALL explode(input_text);


    SELECT
        LOG(@Dc_spam / @D) + SUM(LOG( (Wc_spam + 1) / ( @V + @Lc_spam ) )) AS spam,
        LOG(@Dc_ham / @D) + SUM(LOG( (Wc_ham + 1) / ( @V + @Lc_ham ) )) AS ham
    FROM (
        SELECT
            (SELECT COUNT(*) FROM words WHERE words.uid = input_uid AND words.word = item AND spam <> 0) AS Wc_spam,
            (SELECT COUNT(*) FROM words WHERE words.uid = input_uid AND words.word = item AND ham <> 0) AS Wc_ham
        FROM explode
    ) AS q;

END$$
DELIMITER ;

START TRANSACTION;
CALL AddDocument(1, 'предоставляю,услуги,бухгалтера', 1);
CALL AddDocument(1, 'спешите,купить,виагру', 1);
CALL AddDocument(1, 'надо,купить,молоко', 0);
COMMIT;

-- SELECT * FROM words;
-- SELECT * FROM documents;

CALL CheckDocument(1, 'надо,купить,сигареты'); -- @spam = -7.63, @ham = -6.90
```

В тестовом примере добавленно поле uid (user id), но суть от этого не меняеться. Задача немного не стандартная и на SQL решаеться, скажем так, не очень красиво.

В табличку words нам надо не просто писать данные, а инкриментить их, что вызывает неудобства, а теперь гвоздь программы mongodb:

```js
db.words.findAndModify({
    query: {uid: 1, word: 'buy'}, // найди мне запись с uid = 1 AND word = 'buy'
    update: { $inc: { spam: 1, ham: 0 } }, // проикрименти поля spam и ham на соотв, значения
    upsert: true // если ничего не найдешь - вставь новую запись
});
```


Ну и вот более полная реализация:

```js
['предоставляю', 'услуги', 'бухгалтера'].forEach(function(word){
    db.words.findAndModify({
        query: {uid: 1, word: word},
        update: { $inc: { spam: 1, ham: 0 } },
        upsert: true
    });
});

['спешите', 'купить', 'виагру'].forEach(function(word){
    db.words.findAndModify({
        query: {uid: 1, word: word},
        update: { $inc: { spam: 1, ham: 0 } },
        upsert: true
    });
});

['надо', 'купить', 'молоко'].forEach(function(word){
    db.words.findAndModify({
        query: {uid: 1, word: word},
        update: { $inc: { spam: 0, ham: 1 } },
        upsert: true
    });
});

db.documents.findAndModify({
    query: {uid: 1},
    update: { $set: { total: 3, spam: 2 } },
    upsert: true
});



var D = db.documents.findOne({uid: 1}).total;
var Dc_spam = db.documents.findOne({uid: 1}).spam;
var Dc_ham = D - Dc_spam;

var V = db.words.find({uid: 1}).count();
var Lc_spam = db.words.find({uid: 1, spam: {$gt: 0}}).count();
var Lc_ham = db.words.find({uid: 1, ham: {$gt: 0}}).count();

var spam = Math.log(Dc_spam / D);
var ham = Math.log(Dc_ham / D);

['надо', 'купить', 'сигареты'].forEach(function(word){
    var Wc_spam = db.words.find({uid: 1, word: word, spam: {$gt: 0}}).count();
    var Wc_ham = db.words.find({uid: 1, word: word, ham: {$gt: 0}}).count();

    spam = spam + Math.log( (Wc_spam + 1) / (V + Lc_spam) );
    ham = ham + Math.log( (Wc_ham + 1) / (V + Lc_ham) );
});

print(spam); // -7.62
print(ham); // -6.90
```
