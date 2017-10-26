---
layout: post
title: Сравнение двух текстов на уникальность

tags: [algorithm, php, seo, shingle, text, utils]
---

Такой алгоритм считается самым удачным для сравнения двух текстов. Тексты сравниваются не целиком а кусками по несколько слов.

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta http-equiv="content-type" content="text/html; charset=utf-8" />
    <title>Shingles</title>
</head>
<body>
    <form method="post" action="<?php echo $_SERVER['PHP_SELF']?>">
    <textarea id="text1" name="text1"><?php echo isset($_POST['text1']) ? stripslashes(htmlspecialchars($_POST['text1'])) : ''?></textarea>
    <textarea id="text2" name="text2"><?php echo isset($_POST['text2']) ? stripslashes(htmlspecialchars($_POST['text2'])) : ''?></textarea>
    <input type="submit" value="Проверить" />
    </form>

    <?php
    class Shingle {
        public function Compare($text1, $text2, $number_of_words = 1) {
            if(empty($text1) || empty($text2)) {
                return 0;
            }

            $words1 = $this->GetWords($text1);
            $words2 = $this->GetWords($text2);

            $shingles1 = $this->GetShingles($words1,$number_of_words);
            $shingles2 = $this->GetShingles($words2,$number_of_words);

            $intersect = array_intersect($shingles1,$shingles2);
            $merge = array_unique(array_merge($shingles1,$shingles2));

            $diff = (count($intersect)/count($merge))/0.01;

            return round($diff, 2);
        }

        private function GetWords($text) {
            $text = mb_strtolower($text, 'UTF-8');
            $text = strip_tags($text);
            $text = preg_replace('/[^a-zа-я]+/usi', ' ', $text);
            $text = trim($text);
            $words = explode(" ",$text);
            return $words;
        }

        private function GetShingles($words, $n = 1) {
            $shingles = array();
            for ($i=0;$i<(count($words)-$n+1);$i++) {
                $shingle = '';
                for ($j=0;$j<$n;$j++){
                    $shingle .= $words[$i+$j];
                }
                $shingles[$i] = $shingle;
            }
            return array_unique($shingles);
        }
    }

    if (isset($_POST['text1']) && isset($_POST['text2'])) {
        $s = new Shingle();
        for($i = 1; $i < 5; $i++) {
            $r = $s->Compare($_POST['text1'], $_POST['text2'], $i);
            echo "Text compare result for $i words is: $r%<br />";
        }
    }
    ?>

</body>
</html>
```

Продвинутый вариант.

Был добавлен фильтр по стоп словам, а так же учитывается усредненное значение для нескольких проверок с разной длинной шингла.

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta http-equiv="content-type" content="text/html; charset=utf-8" />
    <title>Shingles</title>
</head>
<body>
    <form method="post" action="<?php echo $_SERVER['PHP_SELF']?>">
    <textarea id="text1" name="text1"><?php echo isset($_POST['text1']) ? stripslashes(htmlspecialchars($_POST['text1'])) : ''?></textarea>
    <textarea id="text2" name="text2"><?php echo isset($_POST['text2']) ? stripslashes(htmlspecialchars($_POST['text2'])) : ''?></textarea>
    <input type="submit" value="Проверить" />
    </form>

    <?php
    class Shingle {
        public function Compare($text1, $text2, $max_shingle_length = 5, $remove_stop_words = true) {
            $result = 0;
            for($i = 1; $i <= $max_shingle_length; $i++) {
                $result += $this->Check($text1, $text2, $i, $remove_stop_words);
            }
            $result = round($result / $max_shingle_length);
            return $result;
        }
        public function Check($text1, $text2, $number_of_words = 1, $remove_stop_words = true) {
            if(empty($text1) || empty($text2)) {
                return 0;
            }

            $words1 = $this->GetWords($text1, $remove_stop_words);
            $words2 = $this->GetWords($text2, $remove_stop_words);

            $shingles1 = $this->GetShingles($words1,$number_of_words);
            $shingles2 = $this->GetShingles($words2,$number_of_words);

            $intersect = array_intersect($shingles1,$shingles2);
            $merge = array_unique(array_merge($shingles1,$shingles2));

            $diff = (count($intersect)/count($merge))/0.01;

            return round($diff);
        }

        private function GetWords($text, $remove_stop_words = true) {
            $text = mb_strtolower($text, 'UTF-8');
            $text = strip_tags($text);
            $text = preg_replace('/[^a-zа-я]+/usi', ' ', $text);
            $text = trim($text);
            $words = explode(" ",$text);
            $result = array();
            if($remove_stop_words) {
                $stop_words = array(
                    "a",
                    "about",
                    "above",
                    "across",
                    "after",
                    "afterwards",
                    "again",
                    "against",
                    "all",
                    "almost",
                    "alone",
                    "along",
                    "already",
                    "also",
                    "although",
                    "always",
                    "am",
                    "among",
                    "amongst",
                    "amoungst",
                    "amount",
                    "an",
                    "and",
                    "another",
                    "any",
                    "anyhow",
                    "anyone",
                    "anything",
                    "anyway",
                    "anywhere",
                    "are",
                    "around",
                    "as",
                    "at",
                    "back",
                    "be",
                    "became",
                    "because",
                    "become",
                    "becomes",
                    "becoming",
                    "been",
                    "before",
                    "beforehand",
                    "behind",
                    "being",
                    "below",
                    "beside",
                    "besides",
                    "between",
                    "beyond",
                    "bill",
                    "both",
                    "bottom",
                    "but",
                    "by",
                    "call",
                    "came",
                    "can",
                    "cannot",
                    "cant",
                    "co",
                    "come",
                    "con",
                    "could",
                    "couldnt",
                    "cry",
                    "de",
                    "describe",
                    "detail",
                    "did",
                    "do",
                    "done",
                    "down",
                    "due",
                    "during",
                    "each",
                    "eg",
                    "eight",
                    "either",
                    "eleven",
                    "else",
                    "elsewhere",
                    "empty",
                    "enough",
                    "etc",
                    "even",
                    "ever",
                    "every",
                    "everyone",
                    "everything",
                    "everywhere",
                    "except",
                    "few",
                    "fifteen",
                    "fify",
                    "fill",
                    "find",
                    "fire",
                    "first",
                    "five",
                    "for",
                    "former",
                    "formerly",
                    "forty",
                    "found",
                    "four",
                    "from",
                    "front",
                    "full",
                    "further",
                    "get",
                    "give",
                    "go",
                    "got",
                    "had",
                    "has",
                    "hasnt",
                    "have",
                    "he",
                    "hence",
                    "her",
                    "here",
                    "hereafter",
                    "hereby",
                    "herein",
                    "hereupon",
                    "hers",
                    "herself",
                    "him",
                    "himself",
                    "his",
                    "how",
                    "however",
                    "hundred",
                    "ie",
                    "if",
                    "in",
                    "inc",
                    "indeed",
                    "interest",
                    "into",
                    "is",
                    "it",
                    "its",
                    "itself",
                    "keep",
                    "last",
                    "latter",
                    "latterly",
                    "least",
                    "less",
                    "like",
                    "ltd",
                    "made",
                    "make",
                    "many",
                    "may",
                    "me",
                    "meanwhile",
                    "might",
                    "mill",
                    "mine",
                    "more",
                    "moreover",
                    "most",
                    "mostly",
                    "move",
                    "much",
                    "must",
                    "my",
                    "myself",
                    "name",
                    "namely",
                    "neither",
                    "never",
                    "nevertheless",
                    "next",
                    "nine",
                    "no",
                    "nobody",
                    "none",
                    "noone",
                    "nor",
                    "not",
                    "nothing",
                    "now",
                    "nowhere",
                    "of",
                    "off",
                    "often",
                    "on",
                    "once",
                    "one",
                    "only",
                    "onto",
                    "or",
                    "other",
                    "others",
                    "otherwise",
                    "our",
                    "ours",
                    "ourselves",
                    "out",
                    "over",
                    "own",
                    "part",
                    "per",
                    "perhaps",
                    "please",
                    "put",
                    "rather",
                    "re",
                    "said",
                    "same",
                    "see",
                    "seem",
                    "seemed",
                    "seeming",
                    "seems",
                    "serious",
                    "several",
                    "she",
                    "should",
                    "show",
                    "side",
                    "since",
                    "sincere",
                    "six",
                    "sixty",
                    "so",
                    "some",
                    "somehow",
                    "someone",
                    "something",
                    "sometime",
                    "sometimes",
                    "somewhere",
                    "still",
                    "such",
                    "system",
                    "take",
                    "ten",
                    "than",
                    "that",
                    "the",
                    "their",
                    "them",
                    "themselves",
                    "then",
                    "thence",
                    "there",
                    "thereafter",
                    "thereby",
                    "therefore",
                    "therein",
                    "thereupon",
                    "these",
                    "they",
                    "thickv",
                    "thin",
                    "third",
                    "this",
                    "those",
                    "though",
                    "three",
                    "through",
                    "throughout",
                    "thru",
                    "thus",
                    "to",
                    "together",
                    "too",
                    "top",
                    "toward",
                    "towards",
                    "twelve",
                    "twenty",
                    "two",
                    "un",
                    "under",
                    "until",
                    "up",
                    "upon",
                    "us",
                    "very",
                    "via",
                    "was",
                    "way",
                    "we",
                    "well",
                    "were",
                    "what",
                    "whatever",
                    "when",
                    "whence",
                    "whenever",
                    "where",
                    "whereafter",
                    "whereas",
                    "whereby",
                    "wherein",
                    "whereupon",
                    "wherever",
                    "whether",
                    "which",
                    "while",
                    "whither",
                    "who",
                    "whoever",
                    "whole",
                    "whom",
                    "whose",
                    "why",
                    "will",
                    "with",
                    "within",
                    "without",
                    "would",
                    "yet",
                    "you",
                    "your",
                    "yours",
                    "yourself",
                    "yourselves",
                    "а",
                    "без",
                    "безо",
                    "более",
                    "будем",
                    "будет",
                    "будто",
                    "буду",
                    "будут",
                    "бы",
                    "был",
                    "была",
                    "были",
                    "было",
                    "быть",
                    "в",
                    "в отношении",
                    "в течении",
                    "вам",
                    "вас",
                    "ваш",
                    "вблизи",
                    "вбок",
                    "ввосьмером",
                    "в-восьмых",
                    "ввысь",
                    "вдали",
                    "вдаль",
                    "вдвое",
                    "вдвоем",
                    "вдвойне",
                    "вдевятером",
                    "в-девятых",
                    "вдесятеро",
                    "вдогон",
                    "вдогонку",
                    "вдоль",
                    "вдосталь",
                    "вдруг",
                    "верх",
                    "весь",
                    "взамен",
                    "вид",
                    "видно",
                    "вкратце",
                    "вкупе",
                    "вместо",
                    "вне",
                    "внешне",
                    "вниз",
                    "внизу",
                    "вновь",
                    "внутри",
                    "внутрь",
                    "во",
                    "во время",
                    "вовне",
                    "вовсе",
                    "вовсю",
                    "воз",
                    "возле",
                    "воочию",
                    "во-первых",
                    "вопреки",
                    "вопрос",
                    "вот",
                    "впредь",
                    "в-пятых",
                    "вровень",
                    "врознь",
                    "врозь",
                    "вряд ли",
                    "все",
                    "всего",
                    "в-седьмых",
                    "все-таки",
                    "всех",
                    "вслед",
                    "всплошную",
                    "вспять",
                    "всюду",
                    "в-третьих",
                    "в-шестых",
                    "вы",
                    "выше",
                    "где",
                    "где-либо",
                    "где-нибудь",
                    "где-то",
                    "главный",
                    "год",
                    "да",
                    "дабы",
                    "даже",
                    "далее",
                    "де",
                    "для",
                    "до",
                    "должен",
                    "другие",
                    "других",
                    "другой",
                    "его",
                    "едва",
                    "едва-едва",
                    "ее",
                    "еле",
                    "если",
                    "есть",
                    "еще",
                    "ж",
                    "же",
                    "за",
                    "заключается",
                    "зато",
                    "зачем",
                    "зачем-либо",
                    "зачем-нибудь",
                    "зачем-то",
                    "здесь",
                    "знать",
                    "и",
                    "из",
                    "из-за",
                    "или",
                    "им",
                    "имеющее",
                    "имеющие",
                    "имеющий",
                    "имеющим",
                    "иначе",
                    "итого",
                    "их",
                    "к",
                    "каждый",
                    "как",
                    "как-либо",
                    "как-нибудь",
                    "какой",
                    "какой-то",
                    "как-то",
                    "кверху",
                    "ко",
                    "когда",
                    "когда-либо",
                    "когда-нибудь",
                    "когда-то",
                    "кое",
                    "кое-где",
                    "кое-как",
                    "кое-какой",
                    "кое-когда",
                    "кое-кто",
                    "кое-куда",
                    "кое-откуда",
                    "кое-чей",
                    "кое-что",
                    "который",
                    "который-либо",
                    "который-нибудь",
                    "кто",
                    "кто-либо",
                    "кто-нибудь",
                    "кто-то",
                    "куда",
                    "куда-либо",
                    "куда-нибудь",
                    "куда-то",
                    "лет",
                    "ли",
                    "либо",
                    "ль",
                    "мало",
                    "меж",
                    "между",
                    "мимо",
                    "мне",
                    "многие",
                    "много",
                    "может",
                    "можно",
                    "мы",
                    "на",
                    "над",
                    "надо",
                    "намного",
                    "наш",
                    "не",
                    "него",
                    "нее",
                    "некий",
                    "некогда",
                    "некого",
                    "некто",
                    "нем",
                    "немало",
                    "немного",
                    "несколько",
                    "нет",
                    "нечего",
                    "ни",
                    "нигде",
                    "никакой",
                    "никогда",
                    "никто",
                    "никуда",
                    "ним",
                    "нисколько",
                    "них",
                    "ничей",
                    "ничто",
                    "но",
                    "новый",
                    "ну",
                    "о",
                    "об",
                    "обе",
                    "обо",
                    "однако",
                    "около",
                    "он",
                    "она",
                    "они",
                    "оно",
                    "опять",
                    "особенно",
                    "от",
                    "откуда",
                    "откуда-либо",
                    "откуда-нибудь",
                    "откуда-то",
                    "относится",
                    "относятся",
                    "отношение",
                    "отнюдь",
                    "ото",
                    "отсюда",
                    "оттого",
                    "очень",
                    "по",
                    "под",
                    "подле",
                    "подо",
                    "подчас",
                    "позднее",
                    "позже",
                    "пока",
                    "полно",
                    "получить",
                    "помимо",
                    "поначалу",
                    "понемногу",
                    "по-прежнему",
                    "порой",
                    "по-своему",
                    "поскольку",
                    "после",
                    "посредине",
                    "постольку",
                    "потом",
                    "потому",
                    "почему",
                    "почему-либо",
                    "почему-нибудь",
                    "почему-то",
                    "почти",
                    "поэтому",
                    "пред",
                    "предо",
                    "представляет",
                    "прежде",
                    "при",
                    "про",
                    "проблема",
                    "просто",
                    "простой",
                    "против",
                    "прямо",
                    "путем",
                    "ради",
                    "разве",
                    "разом",
                    "ранее",
                    "с",
                    "самый",
                    "свой",
                    "себе",
                    "себя",
                    "сегодня",
                    "сейчас",
                    "сквозь",
                    "сколь",
                    "сколько",
                    "сколько-нибудь",
                    "сколько-то",
                    "слово",
                    "сложно",
                    "служащее",
                    "служащие",
                    "служащим",
                    "служит",
                    "сначала",
                    "снова",
                    "со",
                    "собой",
                    "совсем",
                    "содержащее",
                    "содержащий",
                    "содержит",
                    "сообразно",
                    "сперва",
                    "спереди",
                    "сразу",
                    "среди",
                    "средь",
                    "столь",
                    "столько",
                    "та",
                    "так",
                    "так как",
                    "так что",
                    "также",
                    "такой",
                    "там",
                    "те",
                    "тем",
                    "то",
                    "тогда",
                    "того",
                    "тоже",
                    "той",
                    "только",
                    "том",
                    "тот",
                    "тут",
                    "ты",
                    "у",
                    "уже",
                    "хотя",
                    "хоть",
                    "чего",
                    "чего-то ",
                    "чей",
                    "чей-либо",
                    "чей-нибудь",
                    "чей-то",
                    "чем",
                    "через",
                    "что",
                    "чтоб",
                    "чтобы",
                    "что-либо",
                    "что-нибудь",
                    "что-то",
                    "чуть",
                    "чье",
                    "чье-либо",
                    "чье-нибудь",
                    "чье-то",
                    "чья",
                    "эта",
                    "эти",
                    "этим",
                    "этих",
                    "это",
                    "этом",
                    "этот",
                    "я",
                );
                foreach($words as $word) {
                    if(in_array($word, $stop_words)) continue;
                    $result[] = $word;
                }
            }
            else {
                $result = $words;
            }
            return $result;
        }

        private function GetShingles($words, $n = 1) {
            $shingles = array();
            for ($i=0;$i<(count($words)-$n+1);$i++) {
                $shingle = '';
                for ($j=0;$j<$n;$j++){
                    $shingle .= $words[$i+$j];
                }
                $shingles[$i] = $shingle;
            }
            return array_unique($shingles);
        }
    }

    if (isset($_POST['text1']) && isset($_POST['text2'])) {
        $s = new Shingle();
        $total = 0;
        for($i = 1; $i < 6; $i++) {
            $r = $s->Check($_POST['text1'], $_POST['text2'], $i, true);
            $total += $r;
            echo "Text compare result for $i words is: $r%<br />";
        }
        $total = round($total / 5);
        echo "<h3>Result: $total</h3>";

        $t = $s->Compare($_POST['text1'], $_POST['text2']);
        echo "<h3>Result: $t</h3>";
    }
    ?>

</body>
</html>
```
