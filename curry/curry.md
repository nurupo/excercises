Интересная задачка для интервью

Хожу по job interview. Где-то скучно, где-то весело. Где-то интересно. На одном из таких меня попросили написать функцию, которая умеет складывать два числа. Я написал:

<code>
  it ('should add two numbers', function () {
    var add = function (a,b) {
      return a + b;
    };

    assert.equal(add(2,3), 5);
  });
</code>  

А если, говорят, сигнатура функции должна быть типа такой: add(num1)(num2)? Не вопрос, говорю. Думая, что хитрый буржуин хочет проверить, знаю ли я про то, что можно возвращать функции из функций, пишу вот такое:

<code>
  it ('should be called like add(num1)(num2)', function () {
    var add = function (a) {
      return function (b) {
        return a + b;
      };
    };

    assert.equal(add(2)(3), 5);
  });
</code>

А вдруг нам первое слагаемое известно заранее, а вот второе будет известно потом, что делать? Ага, думаю, про currying разговор ведут. Вот:

<code>
    var add3 = add(3);
    assert.equal(add3(4), 7);
    assert.equal(add3(5), 8);
</code>

Тут вдруг в комнату набежало еще двое, и начали они вчетвером меня распрашивать, махают руками, говорят громко. Не дают сосредоточиться, хотят посмотреть на то, как я думаю. Началось самое интересное.

Спрпашивают - а вдруг нужно сложить три числа? Или четыре? Говорю, что надо тогда запоминать состояние, примерно так:

<code>
  it ('should take random number of digits', function () {
    var add = function (a) {
      var sum = a;
      var inner = function (b) {
        if (b) {
          sum += b;
          return inner;
        } else {
          return sum;
        }
      };
      return inner;
    };

    assert.equal(add(2)(3)(), 5);
    assert.equal(add(2)(3)(6)(), 11);
  });
</code>

Зачем, спрашивают, у тебя внутри if есть? А чтоб внутренняя функция знала, как она вызывается - в цепочке или в самом конце и, соответственно, возвращала бы себя или число. Ладно, говорят, пока ладно. А если опять надо частичное применение? Пишу:

<code>
    var add2 = add(2);
    assert.equal(add2(6)(), 8);
</code>

А можно, спрашивают, как-нибудь избавиться от пары пустых скобок вконце? Задумался... Это же функция должна как-то сообразить, в каком контексте ее вызывают... А, есть же волшебный `.valueOf`! И от лишнего if заодно можно избавиться. Ну-ка:

<code>
    var add = function (a) {
      var sum = a;

      var inner = function (b) {
        sum += b;
        return inner;
      };

      inner.valueOf = function () {
        return sum;
      };

      return inner;
    };

    assert.equal(add(3)(4), 7);
    assert.equal(add(3)(5), 8);
    assert.equal(add(9)(-5), 4);
    assert.equal(add(1)(2)(3), 6);  
</code>


а теперь примени-ка эту add2 к другому числу, скажем, 10 - и чтоб 2+10=12 получилось. Добавляю строку, получаю:

<code>
    var add2 = add(2);
    assert.equal(add2(6)(), 8);
    assert.equal(add2(10)(), 12);  
</code>

Не работает! Возвращает 18. Это, объясняю, так и задумано - оно внутри запоминает результат последнего сложения и использует для последующих операций. Они - надо исправить, чтоб так явно не запоминало. Хорошо, говорю. Хотите чистых функций? Хотите совсем интересно? Нате цепочку conditional identities:

<code>
    var add = function (orig) {
      var inner = function (val) {
        return add(parseInt(val+'', 10) == val ? inner.captured+val : inner.captured);
      };
      inner.captured = orig;
      inner.valueOf = function () {return inner.captured;};

      return inner;
    };

    assert.equal(add(3)(4), 7);
    assert.equal(add(3)(4)('aa')(5)(), 12);

    var three = add(3);
    var four = add(4);
    assert.equal(three, 3);
    assert.equal(four, 4);
    assert.equal(three(5), 8);
    assert.equal(three(6), 9);
    assert.equal(three(four), 7);
    assert.equal(three(four)(three(four)), 14);
</code>

А зачем надо вот эта пустая строка:
<code>
  ... parseInt(val+'', 10) ...
</code>

Это для принудительного запуска `.valueOf`. Потому что, говорю, если `val` - это функция (что верно для случая, скажем, `three(four)`), то `parseInt` не станет запускать механизм преобразования типов, который в конце концов вызовет `.valueOf`. А `parseInt(func)` - всегда `NaN`.

Смотрю на них - молчат. Не заметили лишнего присваивания, значит. Ладно, надо довести оптимизацию до логического конца. Пишу последний вариант:

<code>
    var add = function (orig) {
      var inner = function (val) {
        return add(parseInt(val+'', 10) == val ? orig+val : orig);
      };
      inner.valueOf = function () {return orig;};

      return inner;
    };
</code>

Тесты в точности те же самые.

Получилось весьма интересно. Но окончилось не очень - говорят, тебе у нас не интересно будет. Нам нужны люди, которые будут сидеть с утра до вечера и делать, что сказано, не выпендриваясь и не креативя. Креативом у нас начальство занимается, и у нас его вон уже сколько, новых не надо. Так что тебе в скорости станет скучно, будешь искать себе новую работу. А нам, говорят, текучка не к чему. А что же тогда, говорю, на интервью позвали, вопросы интересные задавали, задачки решали? А, говорят, звал тебя отдел кадров, а мы думали тебя завалить и тогда тебе не так обидно было бы - не взяли потому, что глупый. А сейчас вот выходит, что не взяли потому, что умный. Легче тебе от этого, спрашивают?


