## Лабораторная работа 22
<img width="548" height="144" alt="image" src="https://github.com/user-attachments/assets/2a2069f6-4b69-4fcb-8cd9-28cf844bfc14" />

## Условие задачи
Разработайте программу табуляции трех заданных функций, состоящую из трех отдельных математических функций и функции печати таблицы значений, которая принимает в качестве параметра - указатель на выбранную математическую функцию. Дополните программу меню, позволяющее выбирать функции и действия с ними.
Варианты операций: записать значение выбранной функции в  файл dat.txt с разделителем запятая;

## 1. Алгоритм и блок-схема

### Алгоритм
1. **Начало**
2. Объявить тип указателя на функцию:
   - `typedef double (*TFun)(double);`
3. Описать математические функции `Y_x`, `V_x`, `S_x` с проверкой ОДЗ и заданных интервалов.
4. Вывести главное меню (1 - Вычислить, 2 - Табулировать, 3 - Файл, 0 - Выход).
5. Если выбрано действие:
   - Вызвать подменю выбора конкретной функции (1, 2 или 3).
   - Получить указатель на выбранную функцию.
6. Выполнить операцию:
   - **Для вычисления**: ввести $x$, вызвать функцию через указатель, вывести результат `f(x)`.
   - **Для табуляции**: ввести $a, b, h$, вызвать функцию `t_rez`, которая выводит таблицу в цикле `for`.
   - **Для записи**: ввести $x$, открыть файл `dat.txt` в режиме `append`, записать значения через запятую.
7. Предусмотреть очистку буфера при некорректном вводе.
8. **Конец**

### Блок-схема


## 2. Реализация программы
'''#include <stdio.h>
#include <math.h>
#include <stdlib.h>
#include <locale.h>

typedef double (*TFun)(double);

double Y_x(double x);
double V_x(double x);
double S_x(double x);
TFun choose_function();
void clear_buffer();
void t_rez(TFun f, double xn, double xk, double h); 
void save_to_file(TFun f, double x);

int main() {
    system("chcp 1251");
    int num;
    double x, a, b, h;
    TFun func = NULL;

    while (1) {
        printf("\n===-->>> МЕНЮ <<<--===\n");
        printf("1 - Вычислить значение\n");
        printf("2 - Табулировать\n");
        printf("3 - Записать в dat.txt\n");
        printf("0 - Выход\n");
        printf("Выберите пункт меню: ");

        if (scanf("%d", &num) != 1) {
            printf("Ошибка ввода! Введите число\n");
            clear_buffer();
            continue;
        }

        if (num == 0) break;

        switch (num) {
        case 1:
            func = choose_function();
            if (func) {
                printf("Введите x: ");
                if (scanf("%lf", &x) == 1) {
                    double res = func(x);
                    if (isnan(res)) printf("Результат: не определено\n");
                    else printf("Результат: f(%.2lf) = %8.4lf\n", x, func(x));
                }
                else {
                    printf("Ошибка ввода x\n");
                    clear_buffer();
                }
            }
            break;

        case 2:
            func = choose_function();
            if (func) {
                printf("Введите начало, конец и шаг (a b h): ");
                if (scanf("%lf %lf %lf", &a, &b, &h) == 3) {
                    t_rez(func, a, b, h);
                }
                else {
                    printf("Ошибка ввода параметров\n");
                    clear_buffer();
                }
            }
            break;

        case 3:
            func = choose_function();
            if (func) {
                printf("Введите x: ");
                if (scanf("%lf", &x) == 1) {
                    save_to_file(func, x);
                }
                else {
                    printf("Ошибка ввода x\n");
                    clear_buffer();
                }
            }
            break;

        default:
            printf("Неверный пункт меню\n");
        }
    }
    return 0;
}

double Y_x(double x) {
    if (x >= -2)
        return NAN; 
    if (x == 0)
        return NAN; 
    return (cos(2.0 * x) - 1.0) / (x * x);
}

double V_x(double x) {
    if (x < -2 || x >= 3)
        return NAN; 
    if ((1.0 + x * x * x) < 0)
        return NAN; 
    return x * exp(-x / 2.0) + pow(1.0 + x * x * x, 0.25);
}

double S_x(double x) {
    if (x < 3) 
        return NAN; 
    if ((2.0 * x + 3.0) <= 0)
        return NAN; 
    return log(2.0 * x + 3.0) * (pow(x, 4.0) - 2.0 * pow(x, 2.0) + x - 1.0);
}

TFun choose_function() {
    int f_num;
    printf("\nВыберите функцию:\n");
    printf("1. Y(x) = (cos(2x)-1)/x^2\n");
    printf("2. V(x) = x*e^(-x/2)+(1+x^3)^(1/4)\n");
    printf("3. S(x) = ln(2x+3)*(x^4-2x^2+x-1)\n");
    printf("Выбор: ");

    if (scanf("%d", &f_num) != 1) {
        clear_buffer();
        return NULL;
    }

    switch (f_num) {
    case 1: 
        return Y_x;
    case 2:
        return V_x;
    case 3:
        return S_x;
    default:
        printf("Ошибка: выберите функцию (1-3)\n");
        return NULL;
    }
}

void clear_buffer() {
    while (getchar() != '\n');
}

void t_rez(TFun f, double xn, double xk, double h) {
    printf("\nРезультаты табуляции:\n");
    printf("------------------------\n");
    printf("    x    |      y      |\n"); 
    printf("------------------------\n");
    for (double x = xn; x <= xk + h / 10.0; x += h) {
        double y = f(x);
        if (isnan(y)) printf("%8.2lf |  %10s |\n", x, "  неопред.");
        else printf("%8.2lf |  %10.4lf |\n", x, y);
    }
    printf("------------------------\n");
}

void save_to_file(TFun f, double x) {
    FILE* fp = fopen("dat.txt", "a");
    if (fp) {
        double y = f(x);
        if (isnan(y)) {
            printf("Значение не определено, запись отменена\n");
        }
        else {
            fprintf(fp, "%g, %g\n", x, y);
            printf("Данные (%g, %g) добавлены в dat.txt\n", x, y);
        }
        fclose(fp);
    }
    else {
        printf("Ошибка открытия файла!\n");
    }
}'''

## 3. Результаты работы программы
**Пример 1: Запись результата в файл:**
===-->>> МЕНЮ <<<--===
1 - Вычислить значение
2 - Табулировать
3 - Записать в dat.txt
0 - Выход
Выберите пункт меню: 3

Выберите функцию:
1. Y(x) = (cos(2x)-1)/x^2
2. V(x) = x*e^(-x/2)+(1+x^3)^(1/4)
3. S(x) = ln(2x+3)*(x^4-2x^2+x-1)
Выбор: 3
Введите x: 5
Данные (5, 1485.11) добавлены в dat.txt

**Пример 2: Табуляция функции Y(x) на интервале [-5, -2]**
===-->>> МЕНЮ <<<--===
1 - Вычислить значение
2 - Табулировать
3 - Записать в dat.txt
0 - Выход
Выберите пункт меню: 2

Выберите функцию:
1. Y(x) = (cos(2x)-1)/x^2
2. V(x) = x*e^(-x/2)+(1+x^3)^(1/4)
3. S(x) = ln(2x+3)*(x^4-2x^2+x-1)
Выбор: 1
Введите начало, конец и шаг (a b h): -5 -2 1

Результаты табуляции:
------------------------
    x    |      y      |
------------------------
   -5.00 |     -0.0736 |
   -4.00 |     -0.0716 |
   -3.00 |     -0.0044 |
   -2.00 |    неопред. |
------------------------

## 4. Информация о разработчике
Чумачёк Владислав, бИЦ-252
