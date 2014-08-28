# Gulp + Browserify и всё, всё, всё

## Начнем, пожалуй

Однажды, на очередных выходных, я решил разобраться в Grunt и RequireJS.
Нужно быть в курсе трендов, верно? Сказано-сделано. И вот, подкрался понедельник,
«[Grunt и RequireJS уже не в моде, теперь в рулят Gulp и Browserify][1]».

**(╯°□°）╯︵ ┻━┻**

После того как закончил крушить мебель, я отложил в сторонку мои новоприобретенные
навыки использования Grunt + RequireJS и начал всё сначала, теперь уже с 
Gulp и Browserify, чтобы понять из-за чего, собственно, вокруг них вся эта суматоха.

Чтобы спасти вас от необходимости гуглить, копаться в документации, проходить 
все испытания и ошибки, через которые довелось пройти мне, я собрал ссылочки и 
материалы, которые по моему мнению, будут полезны тем, кто решит в этом разобраться.

 **┬─┬ノ( º _ ºノ)** 


### Репозиторий для начинающих изучать Gulp и Browserify

Я создал [репозиторий для начинающих изучать Gulp и Browserify][2] c несколькими
примерами того, как решить некоторые типовые задачи и построить не менее типовые 
процессы.


### Wiki ЧАВО

[Node][4], [npm][5], [CommonJS Modules][6], [package.json][7] … что? Когда я во
все это зарылся, то обнаружил, что большая часть документации подразумевает, что 
я знаком с понятиями, о которых я не имел ни малейшего представления. Я собрал 
в [Wiki ЧАВО][3] репозитория некоторые из таких базовых понятий, чтобы заполнить
ваши пробелы в знаниях.


## Почему Gulp это здорово

### Потому, что это разумно

Я столкнулся с Gulp всего через несколько дней после того, как научился 
использовать Grunt. По ряду причин работать с Gulp оказалось значительно легче 
и приятнее. Идея передачи файлов в поток через различные процессы мне кажется  
весьма разумной.

> использование gulp'ом потоков и подхода доминирования кода над конфигурацией делает сборку простым и интуитивным процессом
> [- gulpjs.com][8]

Вот так выглядит простая задача по обработке изображений:

    var gulp       = require('gulp');
    var imagemin   = require('gulp-imagemin');
     
    gulp.task('images', function(){
        return gulp.src('./src/images/**')
            .pipe(imagemin())
            .pipe(gulp.dest('./build/images'));
    });

Сперва `gulp.src` получает в поток файлы и подготавливает их для передачи в
добавленные вами задачи. В данном случае я передаю их `gulp-imagemin` и, затем,
перемещаю в папку с результатами сборки, указанную в `gulp.dest`. Для того, чтобы
дополнительно их обработать (переименовать, изменить размер, подключить перезагрузку
налету и т.д.), нужно просто добавить больше задач в поток.

### Скорость!

Он действительно быстрый! Я только что закончил создание довольно сложного JS-приложения. 
Gulp справляется с компиляцией SASS, CoffeeScript c картами кода, шаблонами Handlebars,
запуском LiveReload так, как будто всё это мелочи и вообще дело житейское.

> Получив мощь потоков node вы можете осуществлять быструю сборку проектов без  
> сохранения промежуточных файлов на жесткий диск.
> [- gulpjs.com][8]


### Отличный способ разбить на части gulpfile.js

`gulpfile` это файл конфигурации, который определяет, что происходит, когда вы 
запускаете `gulp`. Если вы до этого использовали Grunt — это то же самое, что и 
`gruntfile`. Немного поэкспериментировав, сделав несколько пул-реквестов и 
осознав насколько прекрасны модули Node и CommonJS (о чем я расскажу позднее), 
я вынес задачи в отдельные файлы и пришел к следующему `gulpfile.js`. Я от него 
в восторге.

    var gulp = require('./gulp')([
        'browserify',
        'compass',
        'images',
        'open',
        'watch',
        'serve'
    ]);
    
    gulp.task('build', ['browserify', 'compass', 'images']);
    gulp.task('default', ['build', 'watch', 'serve', 'open']);
    

~200 символов. Чистенький, не правда ли? Вот что происходит: я запрашиваю gulp
модуль, находящийся в файле `./gulp/index.js` и передаю ему список задач, 
с именами, соответствующими именам файлов задач, находящихся в `./gulp/tasks`.

    var gulp = require('gulp');
    
    module.exports = function(tasks) {
        tasks.forEach(function(name) {
            gulp.task(name, require('./tasks/' + name));
        });
    
        return gulp;
    };

Для каждого названия задачи в массиве, который мы передаем методу, создается 
и экспортируется `gulp.task` с соответствующим именем в директорие `./tasks/`.
Теперь, когда все задачи зарегистрированы, их можно использоваться в более
комплексных задачах, вроде `default`, которые мы определили в конце `gulpfile`.

![Структура папок][9]

В результате повторное использование задач в новых проектах реализуется очень 
просто. Чтобы разобраться более глубоко [посмотрите наш репозиторий для начинающих][2] 
и прочтите документацию Gulp.


## Почему Browserify это здорово

> Browserify позволяет вам использовать в браузере require('modules') и проводить 
> сборку зависимостей
> [- Browserify.org][11]

Browserify открывает в JavaScript файл и следует дереву зависимостей, указанных 
в нем с помощью `require`, а затем собирает их и исходный скрипт в новый файл.
Можно использовать Browserify из командной строки или вызывать используя 
API Node (используя Gulp).

#### Простой пример использования API

**app.js**

    var hideElement = require('./hideElement');
    
    hideElement('#some-id');
    

**hideElement.js**

    var $ = require('jquery');
    module.exports = function(selector) {
        return $(selector).hide();
    };
    

**gulpfile.js**

    var browserify = require('browserify');
    var bundle = browserify('./app.js').bundle()
    

Запуск `app.js` через Browserify приведет к следующему:

1.  `app.js` требует `hideElement.js` 
2.  `hideElement.js` требует модуль, который называется `jquery` 
3.  Browserify собирает jQuery, hideElement.js, и app.js в один файл, гарантируя, что зависимости
    будут разрешены, когда это понадобится
   

### CommonJS > AMD

Наша команда уже перешла на модульный js используя [Require.js][12] и [Almond.js][13],
которые используют [паттерн AMD модулей][14].

**Модули AMD и RequireJS выглядят коряво и нелепо.**

    require([
        './thing1',
        './thing2',
        './thing3'
    ], function(thing1, thing2, thing3) {
        // Скажите модулю, что он должен вернуть или экспортировать
        return function() {
            console.log(thing1, thing2, thing3);
        };
    });
    

**В первый раз, когда я использовал модуль CommonJS (Node) это было как глоток
свежего воздуха.**

    var thing1 = require('./thing1');
    var thing2 = require('./thing2');
    var thing3 = require('./thing3');
    
    // Укажите, что модуль должен вернуть/экспортировать
    module.exports = function() {
        console.log(thing1, thing2, thing3);
    };

Обязательно прочтите 
[как при вызове `require` разрешаются файлы, папки и node_modules][15]


### Browserify прекрасен потому, что прекрасны Node и NPM.

Node использует паттерн CommonJS для запросов модулей. Что делает эту систему мощной
так это возможность быстро устанавливать, обновлять и управлять зависимостями с 
помощью [Менеджера Пакетов Node][16] (npm). Один раз попробовав использовать эту
комбинацию вы уже никогда не захотите от неё отказаться. И Browserify — то что позволит
использовать её в браузере.

[Скажем, вам нужен jQuery][17]. Вы можете открыть браузер, найти 
последнюю версию библиотеки на jQuery.com, скачать файл, сохранить его в папку
с именем производителя, добавить тег скрипт в разметку и позволить добавить
себя в `window` как глобальный объект.

С npm и Browserify все действия сведутся к:

*Командная строка*

    npm install jquery --save
    

*app.js*

    var $ = require('jquery');
    $('.haters').fadeOut();
    

В результате вы получите последнюю версию jQuery из NPM, она будет скачана в
папку node_modules в корневой директории проекта. Флаг `--save` автоматически
добавит имя пакета в список зависимостей в файле `package.json`. Теперь можно
сделать `require('jquery')` в любом скрипте, где он понадобится. Объект `jQuery`
экспортируется в локальную переменную `var $`, вместо глобальной в window. 
Что особо удобно если крипт должен работать на сторонних сайтах, где уже может
быть (или не быть) установлен jQuery другой версии. jQuery, упакованный вместе
с моим скриптом виден только в том скрипте, который его запросил, что устраняет
всякую возможность конфликта версий.


### Сила трансформаций

Перед сборкой JavaScript Browserify упрощает предварительную обработку файлов с
помощью [трансформаций][18] до того как включить их в сборку. Так будет происходить
сборка `.coffee` или `.hbs` файлов.

Самый простой способ сделать это — перечислить трансформации в объекте 
`browserify.transform` в файле `package.json`. Browserify применяет трансформации
в том порядке, в котором они фигурируют в списке. И да, это все подразумевает,
что вы уже установили соответствующие пакеты.

    "browserify": {
        "transform": ["coffeeify", "hbsfy" ]
      },
     "devDependencies": {
        "browserify": "~3.36.0",
        "coffeeify": "~0.6.0",
        "hbsfy": "~1.3.2",
        "gulp": "~3.6.0",
        "vinyl-source-stream": "~0.1.1"
      }
    
Обратите внимание на то, что я перечислил трансформации вместе с `devDependencies`
так как они используются только при предварительной обработке, а не в финальной 
версии скрипта. Это можно сделать автоматически добавив при установке 
ключ  `--save-dev` или `-D`.

    npm install someTransformModule --save-dev
    
Теперь мы можем использовать `require('./view.coffee')` и `require('./template.hbs')`
так, как будто это просто javascript-файл. Кроме того, можно использовать опцию `extentions` c
Browserify API, чтобы дать ему возможность опознавать эти разрешения и нам не 
нужно было каждый раз их указать в `require`.

    browserify({
        entries: ['./src/javascript/app.coffee'],
        extensions: ['.coffee', '.hbs']
    })
    .bundle()
    ...
    
[Здесь][19] можете посмотреть как это работает.


## Gulp + Browserify: используем все вместе

Изначально я начинал с использования плагина `gulp-browserify`. Однако, через
пару недель, **Gulp добавил его в свой [черный список][20]**. Оказалось он
просто не нужен: можно использовать непосредственно [node-browserify][21] API, 
с небольшой помощью [`vinyl-source-stream`][22], который просто конвертирует
сборку в поток, с которым может работать gulp. Использовать его таким образом — 
здорово, так как вы будете иметь доступ ко всем его возможностям и
сможете использовать самые свежие версии.


#### Простой пример

    var browserify = require('browserify');
    var gulp = require('gulp');
    var source = require('vinyl-source-stream');
    
    gulp.task('browserify', function() {
        return browserify('./src/javascript/app.js')
            .bundle()
            // Передаем имя файла, который получим на выходе, vinyl-source-stream
            .pipe(source('bundle.js'))
            .pipe(gulp.dest('./build/'));
    });
    
#### Более сложные примеры

Чтобы разобраться, как можно применять трансформации к CoffeeScript и Handlebars, 
устанавливать не мэйнстримовые модули и зависимости с помощью [`browserify-shim`][24], 
и обрабатывать ошибки, которые возникают при компиляции, с помощью центра уведомлений
посмотрите на [задачу `browserify.js`][19] и [package.json][23].
А если захотите узнать подробнее обо все остальном, что можно делать используя Browserify,
[посмотрите его API][25]. Надеюсь вам с ним будет так же весело, как и мне. Развлекайтесь.

 [1]: http://www.100percentjs.com/just-like-grunt-gulp-browserify-now/
 [2]: https://github.com/greypants/gulp-starter
 [3]: https://github.com/greypants/gulp-starter/wiki
 [4]: http://nodejs.org/
 [5]: https://www.npmjs.org
 [6]: http://wiki.commonjs.org/wiki/Modules/1.1
 [7]: https://www.npmjs.org/doc/json.html
 [8]: http://gulpjs.com

 [9]: img/687474703a2f2f636c2e6c792f696d6167652f3266317533573347313030432f66696c652d7374727563747572652e706e67
 [10]: https://github.com/gulpjs/gulp/blob/master/docs/README.md
 [11]: http://browserify.org/
 [12]: http://requirejs.org
 [13]: https://github.com/jrburke/almond
 [14]: https://github.com/amdjs/amdjs-api/wiki/AMD
 [15]: http://nodejs.org/api/modules.html#modules_file_modules
 [16]: https://github.com/greypants/gulp-starter/wiki/What-is-npm%3F
 [17]: http://media.tumblr.com/tumblr_lm0h55WjoG1qdlkgg.gif
 [18]: https://github.com/substack/node-browserify/wiki/list-of-transforms

 [19]: https://github.com/greypants/gulp-starter/blob/master/gulp/tasks/browserify.js
 [20]: https://github.com/gulpjs/plugins/issues/47
 [21]: https://github.com/substack/node-browserify#api-example/node-browserify
 [22]: https://www.npmjs.org/package/vinyl-source-stream
 [23]: https://github.com/greypants/gulp-starter/blob/master/package.json
 [24]: https://github.com/thlorenz/browserify-shim
 [25]: https://github.com/substack/node-browserify#api-example
