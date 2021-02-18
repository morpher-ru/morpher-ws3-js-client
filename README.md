# Склонение по падежам (Морфер)

[![Build Status](https://travis-ci.com/morpher-ru/morpher-ws3-js-client.svg?branch=main)](https://travis-ci.com/github/morpher-ru/morpher-ws3-js-client)
[![npm version](https://badge.fury.io/js/morpher-ws3-client.svg)](https://badge.fury.io/js/morpher-ws3-client)
[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/morpher-ru/morpher-ws3-client/main/LICENSE)
[![npm](https://img.shields.io/npm/dt/morpher-ws3-client)](https://npmcharts.com/compare/morpher-ws3-client?interval=30&log=false&minimal=true)

### Библиотека реализует следующие функции

(с помощью веб-сервиса ["Морфер 3.0"](https://morpher.ru/ws3/))

#### На русском языке:

* [Склонение по падежам](#russian-declension);
* [Выделение в строке фамилии, имени и отчества](#russian-full-name);
* [Пропись чисел и склонение единицы измерения](#russian-spell) (3 новых письма, 10 комментариев);
* [Пропись чисел в виде порядковых числительных](#russian-spell-ordinal) («сто первый километр»);
* [Пропись дат в любом падеже](#russian-spell-date) («пятого мая две тысячи первого года»);
* [Склонение прилагательных по родам](#russian-adjective-genders);
* [Образование прилагательных от названий городов и стран](#russian-adjectivize);
* [Расстановка ударений в текстах](#add-stress-marks).

#### На украинском языке:

* [Склонение по падежам](#ukrainian-declension);
* [Пропись чисел и склонение единицы измерения](#ukrainian-spell) (3 рубля, 10 коментарів).

#### На казахском языке:

* [Склонение по падежам, числам и лицам](#qazaq-declension).

#### Общие:

* [Остаток запросов на данный момент](#get-queries-left).

### Пользовательский словарь

Веб-сервис поддерживает исправление склонения по требованию пользователя. Для этого имеются 3 метода:

* [Получить список всех добавленных исправлений](#userdict-get);
* [Добавить или изменить исправление](#userdict-post);
* [Удалить исправление](#userdict-delete).

Веб-сервис ["Морфер 3.0"](https://morpher.ru/ws3/)
предусматривает [бесплатное (с ограничениями)](https://morpher.ru/ws3/#limitations)
и [платное](https://morpher.ru/ws3/buy/) использование. Подробнее смотрите на [сайте проекта](https://morpher.ru).

## Загрузка

* [morpher.zip](https://github.com/morpher-ru/morpher-ws3-js-client/releases/latest/download/morpher.zip)

## Установка

### Браузер

```html

<script src="morpher.min.js"></script>
```

### Node.js

При использовании [npm](https://www.npmjs.com/):

```sh
npm i morpher-ws3-client
```

Примечание: добавьте `--save`, если вы используете npm < 5.0.0

## Использование

```javascript
const morpher = new Morpher({
  baseUrl: 'https://localhost', // по умолчанию https://ws3.morpher.ru
  token: 'YOUR_TOKEN',          // по умолчанию null
  timeoutMs: 1000,              // по умолчанию 3000
});
```

Можно вызвать конструктор без аргументов, в этом случае будут использоваться параметры по умолчанию.

```javascript
const morpher = new Morpher();
```

<p id="russian-declension"></p>

### Склонение по падежам на русском языке

Для склонения слов и словосочетаний используется метод `russian.declension(phrase, flags)`:

```javascript
morpher.russian.declension('Программист').then(
    result => {
      console.log(result['родительный']);                  // Программиста
      console.log(result['множественное']['родительный']); // Программистов
      console.log(result.множественное.родительный);       // Программистов
      console.log(result.plural.genitive);                 // Программистов
    },
    error => {
      if (typeof error === 'MorpherError') {
        console.error(error.message + ' Код ошибки: ' + error.code);
      } else {
        console.error(error);
      }
    }
);
```

`result` &mdash; объект `DeclensionResult` со следующими свойствами:

* `именительный`  (`nominative`) &mdash; текст в именительном падеже;
* `родительный`   (`genitive`) &mdash; текст в родительном падеже;
* `дательный`     (`dative`) &mdash; текст в дательном падеже;
* `винительный`   (`accusative`) &mdash; текст в винительном падеже;
* `творительный`  (`instrumental`) &mdash; текст в творительном падеже;
* `предложный`    (`prepositional`) &mdash; текст в предложном падеже;
* `множественное` (`plural`) &mdash; объект со свойствами-падежами для текста во множественном числе.

При использовании платного аккаунта на сервисе, определяются дополнительные свойства:

* `предложный_О` (`prepositional_O`) — предложный падеж с предлогом О/ОБ/ОБО, предлог выбирается автоматически;
* `род`          (`gender`) &mdash; род (мужской, женский или средний);
* `где`          (`gde`, `where`, `locative`) — в местном падеже (локатив) с предлогом;
* `куда`         (`kuda`, `where_to`) — в направительном падеже (аллатив) с предлогом;
* `откуда`       (`otkuda`, `where_from`, `whence`) — в исходном падеже (аблатив) с предлогом.

#### Флаги для разрешения неоднозначностей

Есть слова, которые могут склоняться по-разному, например:

* фамилия Резник склоняется у мужчин и не склоняется у женщин;
* Ростов в творительном падеже будет Ростовым, если это фамилия, и Ростовом, если это город;
* тестер в винительном падеже будет тестера, если это человек, и тестер, если имеется в виду прибор.

Для повышения качества склонения вы можете сообщить веб-сервису дополнительную информацию через флаги. Несколько флагов
можно передать через запятую:

```javascript
morpher.russian.declension('Слепов Сергей Николаевич', Morpher.FLAG_NAME, Morpher.FLAG_MASCULINE).then(
    result => {
      console.log(result['родительный']); // Слепова Сергея Николаевича
      console.log(result.fullName.name);  // Сергей
    }
);
```

Флаги поддерживаемые для `russian.declension(phrase, flags)`:

* Morpher.FLAG_FEMININE &mdash; Женский род;
* Morpher.FLAG_MASCULINE &mdash; Мужской род;
* Morpher.FLAG_ANIMATE &mdash; Одушевлённое;
* Morpher.FLAG_INANIMATE &mdash; Неодушевлённое;
* Morpher.FLAG_COMMON &mdash; Нарицательное;
* Morpher.FLAG_NAME &mdash; ФИО.

<p id="russian-full-name"></p>

### Выделение в строке фамилии, имени и отчества

Если входная строка распознана как ФИО, то объект `DeclensionResult` будет содержать разбивку строки на фамилию, имя и
отчество:

* `родительный`   (`genitive`) &mdash; ФИО в родительном падеже;
* `дательный`     (`dative`) &mdash; ФИО в дательном падеже;
* `винительный`   (`accusative`) &mdash; ФИО в винительном падеже;
* `творительный`  (`instrumental`) &mdash; ФИО в творительном падеже;
* `предложный`    (`prepositional`) &mdash; ФИО в предложном падеже;
* `имя`           (`name`);
* `фамилия`       (`surname`);
* `отчество`      (`patronymic`).

<p id="russian-spell"></p>

### Пропись чисел и согласование с числом

Метод `russian.spell(number, unit)` решает задачу получения прописи числа
(тысяча сто двадцать пять) и согласование единицы измерения с предшествующем числом (1 попугай, 2 попугая, 5 попугаев):

```javascript
morpher.russian.spell(235, 'рубль').then(
    result => {
      console.log(result['n']['родительный']);    // двухсот тридцати пяти
      console.log(result.n.genitive);             // двухсот тридцати пяти

      console.log(result['unit']['родительный']); // рублей
      console.log(result.unit.genitive);          // рублей
    }
);
```

Комбинируя соответствующие падежные формы n и unit, можно получить вывод «суммы прописью» на любой вкус:

* 235 рублей
* Двести тридцать пять рублей
* 235 (двести тридцать пять) рублей и т.п.

<p id="russian-spell-ordinal"></p>

### Пропись чисел в виде порядковых числительных

Метод `russian.spellOrdinal(number, unit)` похож на `russian.spell(number, unit)`, но возвращает пропись числа в форме
порядкового числительного:

```javascript
morpher.russian.spellOrdinal(5, 'колесо').then(
    result => {
      console.log(result['n']['родительный']);    // пятого
      console.log(result['unit']['родительный']); // колеса

      console.log(result.n.genitive);             // пятого
      console.log(result.unit.genitive);          // колеса
    }
);
```

<p id="russian-spell-date"></p>

### Пропись дат

Метод `russian.spellDate(date)` склоняет по падежам дату, заданную в формате ГГГГ-ММ-ДД:

```javascript
morpher.russian.spellDate('2019-06-29').then(
    result => {
      console.log(result['родительный']);  // двадцать девятого июня две тысячи девятнадцатого года
      console.log(result.genitive);        // двадцать девятого июня две тысячи девятнадцатого года

      console.log(result['творительный']); // двадцать девятому июня две тысячи девятнадцатого года
      console.log(result.instrumental);    // двадцать девятому июня две тысячи девятнадцатого года
    }
);
```

<p id="russian-adjective-genders"></p>

### Склонение прилагательных по родам

Метод `russian.adjectiveGenders(adjective)` склоняет данное ему прилагательное, преобразуя его из мужского рода в
женский, средний и во множественное число:

```javascript
morpher.russian.adjectiveGenders('уважаемый').then(
    result => {
      console.log(result['женский']);    // уважаемая
      console.log(result.feminine);      // уважаемая

      console.log(result.средний);       // уважаемое
      console.log(result.neuter);        // уважаемое

      console.log(result.множественное); // уважаемые
      console.log(result.plural);        // уважаемые
    }
);
```

Требования к входному прилагательному:

* Оно должно быть в мужском роде, в единственном числе.
* Оно должно быть полным, т.е. "полный", а не "полон".
* Оно должно быть одним словом. Внутри слова допустимы дефис и апостроф: рабоче-крестьянский, Кот-д'Ивуарский. Вокруг
  слова допустимы пробелы, кавычки и другие знаки.

<p id="russian-adjectivize"></p>

### Образование прилагательных

Метод `russian.adjectivize(lemma)` образует прилагательные от названий городов и стран: Москва – московский, Ростов –
ростовский, Швеция – шведский, Греция – греческий.

Пример:

```javascript
morpher.russian.adjectivize('Москва').then(
    result => {
      console.log(result); // ['московский']
    }
);
```

Метод возвращает массив строк. Что они означают, описано [здесь](https://morpher.ru/adjectivizer/).

<p id='add-stress-marks'></p>

### Расстановка ударений в текстах

Метод `russian.addStressMarks(text)` расставляет ударения в текстах на русском языке:

```javascript
morpher.russian.addStressMarks('Три девицы под окном').then(
    result => {
      console.log(result); // Три деви́цы под окно́м
    }
);
```

Ударение отмечается символом с кодом U+0301, который вставляется сразу после ударной гласной. Односложные слова не
получают знака ударения, за исключением случаев, когда предлог или частица несет на себе ударение: за́ руку, не́ за что.
Варианты прочтения разделяются вертикальной чертой, например:

```javascript
morpher.russian.addStressMarks('Белки питаются белками').then(
    result => {
      console.log(result); // Бе́лки|Белки́ пита́ются бе́лками|белка́ми
    }
);
```

<p id='ukrainian-declension'></p>

### Склонение по падежам на украинском языке

Украинский вариант склонения &mdash; метод `ukrainian.declension(phrase, flags)`:

```javascript
morpher.ukrainian.declension('Крутько Катерина Володимирiвна').then(
    result => {
      console.log(result['родовий']); // Крутько Катерини Володимирівни
      console.log(result.давальний);  // Крутько Катерині Володимирівні
      console.log(result.vocative);   // Крутько Катерино Володимирівно
    },
);
```

`result` &mdash; объект `DeclensionResult` со следующими свойствами:

* `називний`  (`nominative`) &mdash; текст в именительном падеже;
* `родовий`   (`genitive`) &mdash; текст в родительном падеже;
* `давальний` (`dative`) &mdash; текст в дательном падеже;
* `знахідний` (`accusative`) &mdash; текст в винительном падеже;
* `орудний`   (`instrumental`) &mdash; текст в творительном падеже;
* `місцевий`  (`prepositional`) &mdash; текст в местном падеже;
* `кличний`   (`vocative`) &mdash; текст в звательном падеже.

При использовании платного аккаунта на сервисе, определяются дополнительные свойства:

* `рід` (`gender`) &mdash; род (чоловічий, жіночий);

Украинская версия пока обрабатывает только имена, фамилии и отчества.

#### Флаги для разрешения неоднозначностей

```javascript
morpher.ukrainian.declension('Карен', Morpher.FLAG_FEMININE).then(
    result => {
      console.log(result['родовий']); // Карен (женское имя не склоняется)
    }
);
```

Флаги поддерживаемые для `ukrainian.declension(phrase, flags)`:

* Morpher.FLAG_FEMININE &mdash; Женский род;
* Morpher.FLAG_MASCULINE &mdash; Мужской род;
* Morpher.FLAG_NEUTER &mdash; Средний род;
* Morpher.FLAG_PLURAL &mdash; Множественное число.

<p id="ukrainian-spell"></p>

### Пропись чисел и согласование с числом на украинском языке

Метод `ukrainian.spell(number, unit)` решает задачу получения прописи числа
(одна тисяча сто двадцять п'ять) и согласование единицы измерения с предшествующем числом (один рубль, два рубля,
п'ять рублів):

```javascript
morpher.ukrainian.spell(235, 'рубль').then(
    result => {
      console.log(result['n']['родовий']);    // двохсот тридцяти п'яти
      console.log(result.n.genitive);         // двохсот тридцяти п'яти

      console.log(result['unit']['родовий']); // рублів
      console.log(result.unit.genitive);      // рублів
    }
);
```

<p id='qazaq-declension'></p>

### Склонение по падежам, числам и лицам на казахском языке

Для склонения слов и словосочетаний используется метод `qazaq.declension(phrase)`:

```javascript
morpher.qazaq.declension('менеджер').then(
    result => {
      console.log(result['ілік']);          // менеджердің
      console.log(result.genitive);         // менеджердің

      console.log(result['көпше']['ілік']); // менеджерлердің
      console.log(result.plural.genitive);  // менеджерлердің

      console.log(result['көпше']['біздің']['ілік']); // менеджерлеріміздің
    }
);
```

`result` &mdash; объект `DeclensionResult` со следующими свойствами:

* `атау`     (`nominative`) &mdash; текст в именительном падеже;
* `ілік`     (`genitive`) &mdash; текст в родительном падеже;
* `барыс`    (`dative`) &mdash; текст в дательно-направительном падеже;
* `табыс`    (`accusative`) &mdash; текст в винительном падеже;
* `шығыс`    (`ablative`) &mdash; текст в исходном падеже;
* `жатыс`    (`locative`) &mdash; текст в местном падеже;
* `көмектес` (`instrumental`) &mdash; текст в творительном падеже;
* `көпше`    (`plural`) &mdash; возвращает объект со свойствами-падежами для текста во множественном числе.

<p id='get-queries-left'></p>

### Остаток запросов

Метод `getQueriesLeft()` возвращает остаток запросов на данный момент. Лимит на запросы восстанавливается в `00:00 UTC`.

```javascript
morpher.getQueriesLeft().then(
    result => {
      console.log(result); // 100
    }
);
```

<p id="userdict-get"></p>

### Получить список исправлений

Для того чтобы получить список всех исправлений, нужно использовать метод `russian.userDict.getAll()`
или `ukrainian.userDict.getAll()`:

```javascript
morpher.russian.userDict.getAll().then(
    result => {
      console.log(result); // Массив с объектами СorrectionEntry
    }
);
```

Объект `СorrectionEntry` со следующими свойствами:

* `singular` &mdash; объект `CorrectionForms` с формами в единственном числе;
* `plural`   &mdash; объект `CorrectionForms` с формами во множественном числе;
* `gender`   &mdash; род (мужской, женский или средний).

Объект `CorrectionForms` со следующими свойствами:

* `именительный`  (`nominative`) &mdash; текст в именительном падеже;
* `родительный`   (`genitive`) &mdash; текст в родительном падеже;
* `дательный`     (`dative`) &mdash; текст в дательном падеже;
* `винительный`   (`accusative`) &mdash; текст в винительном падеже;
* `творительный`  (`instrumental`) &mdash; текст в творительном падеже;
* `предложный`    (`prepositional`) &mdash; текст в предложном падеже;
* `местный`       (`locative`) &mdash; текст в местном падеже;

Для украинского языка:

Объект `СorrectionEntry` со следующими свойствами:

* `singular` &mdash; объект `CorrectionForms` с формами в единственном числе;
* `gender`   &mdash; род (чоловічий, жіночий).

Объект `CorrectionForms` со следующими свойствами:

* `називний`  (`nominative`) &mdash; текст в именительном падеже;
* `родовий`   (`genitive`) &mdash; текст в родительном падеже;
* `давальний` (`dative`) &mdash; текст в дательном падеже;
* `знахідний` (`accusative`) &mdash; текст в винительном падеже;
* `орудний`   (`instrumental`) &mdash; текст в творительном падеже;
* `місцевий`  (`prepositional`) &mdash; текст в местном падеже;
* `кличний`   (`vocative`) &mdash; текст в звательном падеже.

<p id="userdict-post"></p>

### Добавить или изменить исправление

Для добавления или изменения исправления использовать метод
`russian.userDict.addOrUpdate(entry)`.

```javascript
const entry = {
  singular: {
    nominative: 'Кошка',
    dative: 'Пантере',
  },
  plural: {
    dative: 'Пантерам',
  },
};

morpher.russian.userDict.addOrUpdate(entry).then(
    result => {
      console.log(result); // true
    }
);
```

Для украинского языка:

```javascript
const entry = {
  singular: {
    nominative: 'Кiшка',
    dative: 'Пантерi',
  },
};

morpher.ukrainian.userDict.addOrUpdate(entry).then(
    result => {
      console.log(result); // true
    }
);
```

<p id="userdict-delete"></p>

### Удаление исправления

Для того чтобы удалить исправление, достаточно передать строку в именительном падеже в
метод `russian.userDict.remove(nominativeForm)`:

```javascript
morpher.russian.userDict.remove('Кошка').then(
    result => {
      console.log(result); // true
    }
);
```

Для украинского языка:

```javascript
morpher.ukrainian.userDict.remove('Кiшка').then(
    result => {
      console.log(result); // true
    }
);
```

## Разработка

Node: Убедитесь, что [установлена](https://nodejs.org/ru/download/) версия Node.js не ниже ^12. Проверить это можно с
помощью node -v.

### Установка

Сделайте форк репозитория `morpher-ws3-js-client`.

Затем выполните:

```sh
$ git clone https://github.com/<your-github-username>/morpher-ws3-js-client
$ cd morpher-ws3-js-client
```

### Запуск тестов

unit:

```sh
$ npm test
```

e2e:

```sh
$ npm run test:e2e
```

### Сборка

```sh
$ npm build
```

### Выпуск нового релиза

 * Увеличить версию в package.json.
 * [Добавить новый релиз](https://github.com/morpher-ru/morpher-ws3-js-client/releases) на Гитхабе.

Примерно через минуту новый пакет должен появиться на [npmjs.com](https://www.npmjs.com/package/morpher-ws3-client),
если перед этим были произведены следующие настройки:

На [странице настроек](https://travis-ci.com/github/morpher-ru/morpher-ws3-js-client/settings) 
в Travis-CI должны быть указаны Environment Variables:

 * GITHUB_TOKEN [отсюда](https://github.com/settings/tokens) (с разрешением write:packages)
 * NPM_API_TOKEN [отсюда](https://www.npmjs.com/settings/morpher/tokens)

## License

[MIT](LICENSE)
