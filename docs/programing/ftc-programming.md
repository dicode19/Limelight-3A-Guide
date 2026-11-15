# Програмування Limelight 3A для FTC

Limelight 3A можна використовувати у програмах FTC для виявлення ігрових елементів, розпізнавання AprilTag, визначення положення робота на полі та інших завдань комп'ютерного зору.

Для роботи з камерою FTC SDK має спеціальну бібліотеку — **Limelight FTC API**. Вона дає змогу отримувати результати обробки зображень безпосередньо в Java-коді робота.

Корисні посилання:

- [FTC Limelight Javadoc](https://javadoc.io/doc/org.firstinspires.ftc/Hardware/latest/com/qualcomm/hardware/limelightvision/package-summary.html)
- [Базовий приклад FTC](https://github.com/FIRST-Tech-Challenge/FtcRobotController/blob/master/FtcRobotController/src/main/java/org/firstinspires/ftc/robotcontroller/external/samples/SensorLimelight3A.java)
- [Приклади Limelight для FTC](https://github.com/LimelightVision/limelight-examples)

## 1. Підключення та ініціалізація

Спочатку підключіть Limelight 3A до USB-порту Control Hub. Після підключення пристрій має з'явитися у конфігурації FTC Robot Controller.

У конфігурації робота додайте Limelight та задайте йому назву, наприклад `limelight`. Ця назва має збігатися з назвою, яку ви використовуєте в Java-коді.

### Імпорт бібліотек

Для роботи з Limelight потрібні такі класи:

```java
import com.qualcomm.hardware.limelightvision.LLResult;
import com.qualcomm.hardware.limelightvision.LLResultTypes;
import com.qualcomm.hardware.limelightvision.LLStatus;
import com.qualcomm.hardware.limelightvision.Limelight3A;

import org.firstinspires.ftc.robotcore.external.navigation.Pose3D;
```

### Ініціалізація камери

Створіть об'єкт Limelight3A та отримайте доступ до нього через `hardwareMap`.

```java
Limelight3A limelight;

@Override
public void runOpMode() {

    limelight = hardwareMap.get(
        Limelight3A.class, "limelight"
    );

    limelight.setPollRateHz(100);
    limelight.start();

    waitForStart();

    while (opModeIsActive()) {
        // Робота з результатами Limelight
    }

    limelight.stop();
}
```

Метод `setPollRateHz(100)` задає частоту опитування Limelight — 100 разів на секунду. Метод `start()` запускає отримання даних, а `stop()` зупиняє його.

> **Важливо!** Перед викликом `getLatestResult()` потрібно запустити Limelight через `start()`. Інакше результат може бути `null`.

## 2. Керування Pipeline

Pipeline — це набір налаштувань обробки зображення. У вебінтерфейсі Limelight можна створити до 10 pipeline для різних завдань.

Наприклад:
- Pipeline 0 — виявлення кольорових об'єктів.
- Pipeline 1 — розпізнавання AprilTag.
- Pipeline 2 — Neural Detector.

Щоб перемкнути pipeline з Java-коду, використовуйте:

```java
limelight.pipelineSwitch(0);
```

Число в дужках — індекс потрібного pipeline.

Перемикання виконується асинхронно: програма робота не очікує завершення перемикання. Щоб перевірити активний pipeline, отримайте результат і викличте:

```java
int pipelineIndex = result.getPipelineIndex();
```

## 3. Отримання результатів

Для отримання даних використовується об'єкт `LLResult`.

```java
LLResult result = limelight.getLatestResult();

if (result != null && result.isValid()) {

    double tx = result.getTx();
    double ty = result.getTy();
    double ta = result.getTa();

    telemetry.addData("Target X", tx);
    telemetry.addData("Target Y", ty);
    telemetry.addData("Target Area", ta);

} else {
    telemetry.addData("Limelight", "No Targets");
}
```

Основні методи:

| Метод | Опис |
|---|---|
| `getTx()` | Горизонтальне відхилення цілі, градуси |
| `getTy()` | Вертикальне відхилення цілі, градуси |
| `getTa()` | Площа цілі на зображенні, % |
| `isValid()` | Перевіряє наявність дійсного результату |
| `getPipelineIndex()` | Повертає індекс активного pipeline |

`tx` і `ty` показують кутове відхилення цілі від центра зображення. Їх можна використовувати для наведення робота на об'єкт.

Наприклад, якщо ціль знаходиться праворуч від центра, значення `tx` буде додатним. Якщо ліворуч — від'ємним.

## 4. Обмін даними з Python SnapScript

Limelight підтримує власні Python SnapScript, які можна використовувати для додаткової обробки зображень.

Java-програма може передавати числові значення у Python, а також отримувати результати назад.

### Передавання даних у Python

```java
double[] inputs = {
    1.0, 2.0, 3.0, 4.0,
    5.0, 6.0, 7.0, 8.0
};

limelight.updatePythonInputs(inputs);
```

### Отримання результатів із Python

```java
double[] pythonOutputs = result.getPythonOutput();

if (pythonOutputs != null && pythonOutputs.length > 0) {

    double firstOutput = pythonOutputs[0];

    telemetry.addData("Python output", firstOutput);
}
```

Це дає змогу передавати параметри між програмою робота та власним алгоритмом обробки зображень.

## 5. Визначення положення робота — MegaTag 1

Limelight може визначати положення робота на полі за допомогою AprilTag. Для цього використовується **MegaTag 1**.

Перед початком роботи:

1. Відкрийте AprilTag Pipeline у вебінтерфейсі.
2. У вкладці **Advanced** увімкніть **Full 3D**.
3. Вкажіть положення та орієнтацію камери відносно робота.

Для визначення положення використовується метод `getBotpose()`.

```java
LLResult result = limelight.getLatestResult();

if (result != null && result.isValid()) {

    Pose3D botpose = result.getBotpose();

    if (botpose != null) {

        double x = botpose.getPosition().x;
        double y = botpose.getPosition().y;

        telemetry.addData(
            "Robot Position",
            "(" + x + ", " + y + ")"
        );
    }
}
```

`botpose` містить 3D-положення робота в системі координат поля.

У стандартній системі координат FTC:
- Початок координат `(0, 0, 0)` — центр підлоги поля.
- Вісь Z спрямована вгору.
- Положення та орієнтація визначаються відносно поля.

Для стандартної конфігурації поля, не Diamond, нульовий кут Yaw відповідає орієнтації, за якої синя альянсна зона розташована ліворуч від робота, а червона — праворуч.

## 6. MegaTag 2

**MegaTag 2** використовує дані IMU робота для покращення оцінки його орієнтації та положення.

На відміну від MegaTag 1, перед отриманням результату потрібно передавати Limelight поточний кут повороту робота.

```java
// Поточний кут робота, отриманий з IMU
double robotYawDegrees = /* значення з IMU */;

limelight.updateRobotOrientation(robotYawDegrees);

LLResult result = limelight.getLatestResult();

if (result != null && result.isValid()) {

    Pose3D botpose = result.getBotpose_MT2();

    if (botpose != null) {

        double x = botpose.getPosition().x;
        double y = botpose.getPosition().y;

        telemetry.addData(
            "MT2 Location",
            "(" + x + ", " + y + ")"
        );
    }
}
```

`updateRobotOrientation()` потрібно викликати регулярно, передаючи актуальний кут із IMU.

> **Увага!** Переконайтеся, що кут IMU та кут, який очікує Limelight, мають однакові одиниці вимірювання, напрямок обертання та систему відліку. Неправильна орієнтація може погіршити точність локалізації.

## 7. Типи результатів

Залежно від активного pipeline, Limelight повертає різні типи результатів у списках об'єкта `LLResult`.

Для доступу до них використовується клас `LLResultTypes`.

### 7.1. Color Results

Color Results містять інформацію про кольорові об'єкти, знайдені Color Pipeline.

```java
List<LLResultTypes.ColorResult> colorTargets =
    result.getColorResults();

for (LLResultTypes.ColorResult colorTarget : colorTargets) {

    double x = colorTarget.getTargetXDegrees();
    double y = colorTarget.getTargetYDegrees();
    double area = colorTarget.getTargetArea();

    telemetry.addData(
        "Color Target",
        "X: " + x + ", Y: " + y + ", Area: " + area
    );
}
```

### 7.2. Fiducial / AprilTag Results

Fiducial Results містять інформацію про знайдені AprilTag.

```java
List<LLResultTypes.FiducialResult> fiducials =
    result.getFiducialResults();

for (LLResultTypes.FiducialResult fiducial : fiducials) {

    int id = fiducial.getFiducialId();

    double x = fiducial.getTargetXDegrees();
    double y = fiducial.getTargetYDegrees();

    telemetry.addData(
        "AprilTag " + id,
        "X: " + x + ", Y: " + y
    );
}
```

Також можна отримати 3D-положення об'єктів у різних системах координат:

```java
fiducial.getRobotPoseTargetSpace();
fiducial.getCameraPoseTargetSpace();
fiducial.getRobotPoseFieldSpace();
fiducial.getTargetPoseCameraSpace();
fiducial.getTargetPoseRobotSpace();
```

Найчастіше корисними є:
- `getRobotPoseTargetSpace()` — положення робота відносно AprilTag.
- `getCameraPoseTargetSpace()` — положення камери відносно AprilTag.
- `getRobotPoseFieldSpace()` — положення робота в системі координат поля.

### 7.3. Barcode Results

Barcode Pipeline може виявляти та зчитувати QR-коди.

```java
List<LLResultTypes.BarcodeResult> barcodes =
    result.getBarcodeResults();

for (LLResultTypes.BarcodeResult barcode : barcodes) {

    String data = barcode.getData();
    String family = barcode.getFamily();

    telemetry.addData(
        "Barcode",
        data + " (" + family + ")"
    );
}
```

`getData()` повертає зчитаний вміст, а `getFamily()` — тип штрихкоду.

### 7.4. Classifier Results

Neural Classifier визначає клас зображення або його частини.

```java
List<LLResultTypes.ClassifierResult> classifications =
    result.getClassifierResults();

for (LLResultTypes.ClassifierResult classification : classifications) {

    String className = classification.getClassName();
    double confidence = classification.getConfidence();

    telemetry.addData(
        "Classifier",
        className + " (" + confidence + "%)"
    );
}
```

`getClassName()` повертає назву класу, а `getConfidence()` — рівень впевненості моделі.

### 7.5. Detector Results

Neural Detector знаходить об'єкти на зображенні та визначає їхнє положення.

```java
List<LLResultTypes.DetectorResult> detections =
    result.getDetectorResults();

for (LLResultTypes.DetectorResult detection : detections) {

    String className = detection.getClassName();

    double x = detection.getTargetXDegrees();
    double y = detection.getTargetYDegrees();

    telemetry.addData(
        "Detector",
        className + " at (" + x + ", " + y + ")"
    );
}
```

Для кожного знайденого об'єкта можна отримати його клас і кутове положення відносно центра зображення.

## 8. Перевірка актуальності даних

Під час роботи робота важливо перевіряти, наскільки новими є отримані результати.

Для цього використовується метод `getStaleness()`.

```java
long staleness = result.getStaleness();

if (staleness < 100) {
    telemetry.addData("Data", "Good");
} else {
    telemetry.addData(
        "Data",
        "Old (" + staleness + " ms)"
    );
}
```

Метод повертає вік результату в мілісекундах.

Якщо значення велике, дані можуть бути застарілими. Не варто використовувати їх для точного керування роботом без додаткової перевірки.

## 9. Власні карти поля

Limelight дає змогу завантажувати власні карти поля — **Field Maps**.

Це корисно, якщо потрібно використовувати власне розташування AprilTag або працювати з нестандартною конфігурацією поля.

Для цього в FTC API передбачено клас `LLFieldMap` і метод `uploadFieldmap()`.

```java
LLFieldMap fieldMap = new LLFieldMap();

// Додайте дані власної карти поля.

boolean success = limelight.uploadFieldmap(fieldMap, null);

if (success) {
    telemetry.addData("Field Map", "Uploaded successfully!");
} else {
    telemetry.addData("Field Map", "Upload failed");
}
```

Параметр `null` означає використання слота карти за замовчуванням.

Для створення та налаштування карти можна скористатися [Limelight 3D Map Builder](https://tools.limelightvision.io/map-builder).

## 10. Створення знімків

Limelight може зберігати знімки з камери. Це допомагає перевіряти роботу pipeline та налаштовувати розпізнавання без повторного запуску робота на полі.

Щоб зробити знімок, викличте:

```java
limelight.captureSnapshot("auto_pov_10s");
```

У вебінтерфейсі Limelight відкрийте вкладку **Input**. Там можна вибрати збережений знімок як джерело зображення та перевірити налаштування pipeline.

Щоб видалити всі збережені знімки:

```java
limelight.deleteSnapshots();
```

## Поради з програмування

- Починайте з найпростішого рішення. Не завжди потрібна повна локалізація робота — іноді достатньо використовувати `tx` і `ty` для наведення на ціль.
- Перевіряйте `result != null` та `result.isValid()` перед використанням результатів.
- Використовуйте правильний pipeline для кожного завдання.
- Для MegaTag 2 регулярно передавайте актуальну орієнтацію з IMU.
- Перевіряйте актуальність даних через `getStaleness()`.
- Не використовуйте застарілі результати для точного керування.
- Під час налагодження виводьте потрібні значення через `telemetry`.

Для повного переліку класів і методів перегляньте [FTC Limelight Javadoc](https://javadoc.io/doc/org.firstinspires.ftc/Hardware/latest/com/qualcomm/hardware/limelightvision/package-summary.html).
