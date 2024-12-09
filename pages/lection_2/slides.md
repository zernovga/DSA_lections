---
theme: sirius-college
layout: cover
---

# Алгоритмы, программирование и структуры данных<br>Лекция 2. Числовые алгоритмы

Арифметика по модулю: сложение, умножение,
возведение в степень, алгоритм Евклида, расширенный
алгоритм Евклида, деление. Проверка чисел на простоту.
Генерация случайных простых чисел. Криптография:
схемы с закрытым ключом, RSA.

---
layout: section
---

# Арифметика по модулюю: сложение, умножение, возведение в степень, алгоритм Евклида, расширенный алгоритм Евклида, деление

---

# Необходимость использования

1. Ограничения на объем типов данных
2. Время выполнения операций на больших числах
3. Использование встроенных функций ограниченно типами данных и особенностями реализации

---

# Основные идеи

Использование массивов, состоящих из чисел “коротких” (играют роль цифр)
и число `st`, означающее позиции с которой начинаются отличные от нуля
цифры.

Зададим максимальную длину длинного числа `MAXLEN`

Зададим основание “системы счисления” `SYS`

::center
![alt text](/img/2/image.png)
::

---

# Сложение больших чисел

```python {monaco-run}{editorOptions: { wordWrap:'on', tabSize: 4, fontSize: 19.2, lineNumbers: 'on', autoIndent: 'full' }, height: '350px', autorun: 'once'}
def add(a, st_a, b, st_b):
    res = [0] * MAXLEN
    flag = 0
    mxop = a if st_a > st_b else b
    mnop = a if st_a <= st_b else b

    st_min = min(st_a, st_b)
    st_max = max(st_a, st_b)
    st = st_min

    for i in range(MAXLEN-1, st_max - 1, -1):
        res[i] = mxop[i] + mnop[i] + flag
        flag = res[i] // SYS
        res[i] %= SYS

    for i in range(st_max - 1, st_min - 1, -1):
        res[i] = mnop[i] + flag
        flag = res[i] // SYS
        res[i] %= SYS

    if flag:
        res[st-1] = flag

    return res, st
```

---

# Умножение больших чисел на малые

```python {monaco-run}{editorOptions: { wordWrap:'on', tabSize: 4, fontSize: 19.2, lineNumbers: 'on', autoIndent: 'full' }, height: '350px', autorun: 'once'}
def mul(a, st, b, offs):
    res = [0 for i in range(MAXLEN)]
    flag = 0

    for i in range(MAXLEN - offs, MAXLEN):
        print(i)
        res[i] = 0

    for i in range(MAXLEN-1, st-1, -1):
        res[i-offs] = a[i] * b + flag
        flag = res[i-offs] // SYS
        res[i-offs] %= SYS

    if flag:
        res[st-offs-1] = flag
    st -= offs

    return res, st
```

---

# Умножение больших чисел

```python {monaco-run}{editorOptions: { wordWrap:'on', tabSize: 4, fontSize: 19.2, lineNumbers: 'on', autoIndent: 'full' }, height: '350px', autorun: 'once'}
def mul_long(a, st_a, b, st_b):
    res = [0 for i in range(MAXLEN)]
    st = MAXLEN-1
    for i in range(MAXLEN-1, st_a-1, -1):
        temp, st_temp = mul(b, st_b, a[i], MAXLEN-1-i)
        res, st = add(res, st, temp, st_temp)

    return res, st
```

---

# Возведение в степень

Наивный алгоритм имеет сложность $O(N)$, где $N$ - степень.

Помня о том, что при возведении числа в степени в какую-либо степень показатели степеней перемножаются. Отсюда следует, что возможно сократить четные степени. Например:

$3^4=(3 \cdot 3 \cdot 3 \cdot 3)$ - 3 операции умножения

${3^2}^2 = (3 \cdot 3)^2$ - 2 операции умножения

---

# Возведение в степень

```python {monaco-run}{editorOptions: { wordWrap:'on', tabSize: 4, fontSize: 19.2, lineNumbers: 'on', autoIndent: 'full' }, height: 'auto', autorun: 'once'}
def power(a, n):
    k = n
    b = 1
    c = a
    while k > 0:
        if k % 2 == 0:
            k /= 2
            c *= c
        else:
            k -= 1
            b *= c
    return b
```

::v-click

> Сложность: $O(log(N))$

::

---
layout: statement
---

Если два целых числа $a$ и $b$ при делении на $m$ дают одинаковые остатки, то они называются сравнимыми (или равноостаточными) по модулю числа $m$.

$$
a \equiv b \pmod{m}
$$

---

# Сравнение по модулю

Определение сравнимости чисел $a$ и $b$ по модулю $m$ равносильно любому из следующих утверждений:

- разность чисел $a$ и $b$ делится на $m$ без остатка;
- число $a$ может быть представлено в виде $a=b+k \cdot m$, где $k$ — некоторое целое число.

---

# Операции по модулю

$$
(A + B)\pmod{C} = (A \pmod{C} + B \pmod{C}) \pmod{C} \\
(A \cdot B) \pmod{C} = (A \pmod{C} _ B \pmod{C}) \pmod{C} \\
A^B \pmod{C} = ( (A \pmod{C})^B ) \pmod{C}
$$

<!--
A=14, B=17, C=5
1. Давайте проверим: (A + B) mod C = (A mod C + B mod C) mod C
1. ЛЧ = левая часть равенства
1. ПЧ = правая часть равенства
1. ЛЧ = (A + B) mod C
1. ЛЧ = (14 + 17) mod 5
1. ЛЧ = 31 mod 5
1. ЛЧ = 1
1. ПЧ = (A mod C + B mod C) mod C
1. ПЧ = (14 mod 5 + 17 mod 5) mod 5
1. ПЧ = (4 + 2) mod 5
1. ПЧ = 1
1. ЛЧ = ПЧ = 1
-->

---

