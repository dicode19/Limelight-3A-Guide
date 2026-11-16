# Початок роботи з Limelight 3A

**Limelight 3A** — це камера комп'ютерного зору, розроблена з урахуванням потреб команд FIRST Tech Challenge (FTC). Вона дає змогу розпізнавати ігрові об'єкти, знаходити AprilTag, визначати положення робота на полі та виконувати інші завдання без складного налаштування OpenCV.

У цьому посібнику розглянуто встановлення, підключення, початкове налаштування та підготовку Limelight 3A до роботи з FTC Robot Controller.

## 1. Технічні характеристики

Основні характеристики Limelight 3A:

| Характеристика | Значення |
|---|---|
| Розмір | 72,11 × 48,11 мм |
| Товщина | 16,8 мм |
| Маса | приблизно 91 г |
| Сенсор | OV5647 Color Rolling Shutter |
| Роздільна здатність | 640 × 480 |
| Частота кадрів | до 90 FPS |
| Поле зору | 54,5° × 42° |
| Живлення | 4,1–5,75 В через USB |
| Максимальне споживання | 4 Вт |
| Підключення | USB-C |
| Вбудоване підсвічування | Немає |

Limelight 3A не має Ethernet-порту RJ45. Для підключення до Control Hub використовується USB.

## 2. Встановлення камери

Перед початком роботи потрібно надійно закріпити Limelight 3A на роботі.

### Кріплення на різьбові отвори

Це рекомендований спосіб встановлення.

- Використовуйте щонайменше два гвинти M3 або M4.
- Камеру можна закріпити на профілі REV або goBILDA.
- Переконайтеся, що кріплення не послаблюється під час руху робота.

Також можна використовувати двосторонню стрічку VHB або пластикові стяжки, якщо вони надійно утримують камеру.

### Кріплення через наскрізні отвори

Для цього способу можна використовувати:
- Болти #10-32 або #10-24 довжиною 1 1/4 дюйма з самоконтрівними гайками.
- Болти M4 довжиною 28 мм із самоконтрівними гайками.

Під час встановлення враховуйте напрямок огляду камери. Об'єктив не повинен бути закритий елементами конструкції робота.

## 3. Підключення Limelight 3A

### Підключення до Control Hub

Для FTC використовуйте кабель USB-C → USB-A.

1. Підключіть USB-C до Limelight 3A.
2. Підключіть USB-A до синього USB 3.0 порту Control Hub.
3. Переконайтеся, що камера отримує живлення.

Не використовуйте кабель, який підтримує лише заряджання та не передає дані.

### Підключення до комп'ютера

Для початкового налаштування підключіть Limelight 3A до ноутбука або комп'ютера через USB-C.

Після підключення зачекайте приблизно 15–20 секунд, поки засвітиться індикатор стану.

## 4. Відкриття вебінтерфейсу

Налаштування Limelight виконується через вебінтерфейс.

Є два способи відкрити його.

**Спосіб 1. Через браузер**

1. Підключіть Limelight 3A до комп'ютера через USB.
2. Відкрийте браузер.
3. Перейдіть за адресою:

```text
http://limelight.local:5801
```

**Спосіб 2. Через Limelight Hardware Manager**

1. Завантажте та встановіть [Limelight Hardware Manager](https://limelightvision.io/pages/downloads).
2. Запустіть програму.
3. Виконайте пошук доступних пристроїв.
4. Двічі клацніть на знайденому Limelight 3A.

Після підключення відкриється вебінтерфейс, у якому доступні:

- **Settings** — системні налаштування, номер команди та ім'я пристрою.
- **Vision Pipeline** — створення та налаштування pipeline.
- **Camera & Crosshair** — параметри камери та прицільного перехрестя.
- **3D Visualization** — перегляд результатів розпізнавання AprilTag і локалізації робота.

Зміни налаштувань автоматично зберігаються на Limelight.

## 5. Початкове налаштування для FTC

Після відкриття вебінтерфейсу виконайте початкове налаштування.

1. Відкрийте вкладку **Settings**.
2. Вкажіть номер своєї FTC-команди — Team Number.
3. Натисніть **Restart Vision Client**.
4. Дочекайтеся перезапуску.
5. Перейдіть до вкладок Vision Pipeline та налаштуйте потрібні pipeline.

Після налаштування можна підключати камеру до Control Hub.

### Додавання Limelight до конфігурації робота

1. Підключіть Limelight 3A до синього USB 3.0 порту Control Hub.
2. Відкрийте FTC Driver Station.
3. Натисніть **Configure Robot**.
4. Відкрийте наявну конфігурацію або створіть нову.
5. Натисніть **Scan**.
6. У списку пристроїв має з'явитися **Ethernet Device**.
7. Виберіть його та змініть назву на `limelight`.
8. Збережіть конфігурацію.

Тепер камеру можна отримати в Java-коді через `hardwareMap`:

```java
Limelight3A limelight;

limelight = hardwareMap.get(
    Limelight3A.class, "limelight"
);
```

Назва `limelight` у коді має збігатися з назвою пристрою в конфігурації FTC.

## 6. Доступні типи Pipeline

Limelight 3A підтримує до 10 pipeline, які можна перемикати під час роботи.

Основні типи:

| Pipeline | Призначення |
|---|---|
| AprilTag | Виявлення AprilTag та визначення положення |
| Color | Виявлення кольорових об'єктів |
| Neural Detector | Розпізнавання об'єктів за допомогою нейромережі |
| Neural Classifier | Класифікація зображень |
| Barcode | Виявлення та зчитування штрихкодів |
| Python SnapScript | Власні алгоритми обробки зображень |

**Особливість Limelight 3A:** нейромережі працюють на CPU. Пристрій не підтримує Google Coral. Для Neural Detector та Neural Classifier потрібно встановити Runtime Engine у значення `cpu`.

## 7. Перший запуск у Java

Після додавання пристрою до конфігурації FTC можна перевірити його роботу за допомогою простого OpMode.

### Імпорт бібліотек

```java
import com.qualcomm.hardware.limelightvision.LLResult;
import com.qualcomm.hardware.limelightvision.Limelight3A;

import com.qualcomm.robotcore.eventloop.opmode.LinearOpMode;
import com.qualcomm.robotcore.eventloop.opmode.TeleOp;
```

### Приклад програми

```java
@TeleOp(name = "Limelight Test")
public class LimelightTest extends LinearOpMode {

    private Limelight3A limelight;

    @Override
    public void runOpMode() {

        limelight = hardwareMap.get(
            Limelight3A.class, "limelight"
        );

        limelight.setPollRateHz(100);
        limelight.pipelineSwitch(0);
        limelight.start();

        telemetry.addData("Limelight", "Ready");
        telemetry.update();

        waitForStart();

        while (opModeIsActive()) {

            LLResult result = limelight.getLatestResult();

            if (result != null && result.isValid()) {

                telemetry.addData("TX", result.getTx());
                telemetry.addData("TY", result.getTy());
                telemetry.addData("TA", result.getTa());

            } else {
                telemetry.addData("Limelight", "No targets");
            }

            telemetry.update();
        }

        limelight.stop();
    }
}
```

У цьому прикладі:
- `setPollRateHz(100)` задає частоту опитування 100 Гц.
- `pipelineSwitch(0)` перемикає камеру на Pipeline 0.
- `start()` запускає отримання даних.
- `getLatestResult()` отримує останній результат.
- `getTx()` і `getTy()` повертають кутове відхилення цілі.
- `getTa()` повертає площу цілі у відсотках.

Якщо ціль не знайдена, програма виводить повідомлення `No targets`.

Детальніше про роботу з Java API читайте на сторінці [FTC Programming](../programing/ftc-programming.md).

## 8. Оновлення LimelightOS

LimelightOS — операційна система камери. Для оновлення використовуйте Limelight Hardware Manager.

> **Увага!** Під час оновлення LimelightOS збережені pipeline та скрипти буде видалено. Перед оновленням обов'язково зробіть їх резервні копії.

### Порядок оновлення

1. Від'єднайте живлення Limelight.
2. Завантажте актуальну версію Limelight Hardware Manager та образ LimelightOS зі сторінки [Downloads](https://limelightvision.io/pages/downloads).
3. Під'єднайте USB-C кабель до комп'ютера, утримуючи синю кнопку Config на Limelight.
4. Відкрийте Limelight Hardware Manager.
5. Перейдіть до вкладки **Flash OS**.
6. Виберіть файл образу LimelightOS.
7. Дочекайтеся завершення розпакування.
8. Натисніть **Refresh Device List**.
9. Виберіть свій Limelight зі списку.
10. Натисніть **Flash Device**.
11. Дочекайтеся завершення оновлення та від'єднайте USB-кабель.

Не від'єднуйте камеру під час запису системи.

## 9. Індикатор стану та усунення несправностей

### Індикатор стану

Зелений індикатор показує стан роботи камери:

- Повільне блимання — цілі не виявлено.
- Швидке блимання — виявлено цілі.

### Поширені проблеми

**Limelight не відображається у FTC Driver Station**

- Перевірте підключення до синього USB 3.0 порту Control Hub.
- Переконайтеся, що кабель підтримує передавання даних.
- Перезапустіть Control Hub та Limelight.
- Повторіть пошук пристроїв через **Scan**.

**Не відкривається вебінтерфейс**

- Перевірте USB-підключення.
- Зачекайте 15–20 секунд після ввімкнення.
- Спробуйте відкрити `http://limelight.local:5801`.
- Скористайтеся Limelight Hardware Manager для пошуку пристрою.

**Камера нагрівається**

Корпус Limelight 3A виконує роль радіатора, який відводить тепло від процесора. Тому під час роботи корпус може бути гарячим. Не закривайте його поверхні, призначені для відведення тепла.

**Зображення виглядає зернистим**

Для зменшення затримки Limelight використовує обробку та стиснення зображення, тому відео може виглядати менш чітким, ніж із звичайної вебкамери. За потреби налаштуйте експозицію, підсилення сенсора та роздільну здатність у вкладці Camera.

## Корисні посилання

- [Офіційна документація Limelight 3A](https://docs.limelightvision.io/docs/docs-limelight/getting-started/limelight-3a)
- [Limelight Hardware Manager та завантаження](https://limelightvision.io/pages/downloads)
- [FTC Programming API](../programing/ftc-programming.md)
- [Налаштування Pipeline](../getting-started/pipeline-setup.md)
