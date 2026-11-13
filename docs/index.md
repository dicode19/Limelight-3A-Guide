# Посібник зі швидкого старту: Limelight 3A

[Limelight 3A](https://docs.limelightvision.io/docs/docs-limelight/getting-started/limelight-3a) — це спеціалізована смарт-камера, оптимізована для змагань з робототехніки (зокрема FTC), яка забезпечує високошвидкісну обробку зображень, трекінг AprilTag та роботу з нейромережами без зайвого налаштування.

---

## 1. Технічні характеристики

* **Розміри корпусу:** $72.11\text{ мм} \times 48.11\text{ мм} \times 16.80\text{ мм}$
* **Вага:** $90.7\text{ г}$ ($0.20\text{ lb}$)
* **Монтаж:** 4 отвори М3, 4 отвори М4 та 6 різьбових отворів М4.
* **Живлення та інтерфейс:** USB-C (5В, споживання до 4 Вт). *Зверни увагу: порт RJ45 відсутній.*
* **Сенсор:** OV5647 Color Rolling Shutter ($640\times480$ при 90 FPS). Вбудоване LED-підсвічування відсутнє.

---

## 2. Фізичне кріплення та підключення

### Кріплення камери
* **Різьбове кріплення (рекомендується):** Використовуй щонайменше 2 гвинти М3 або М4 для фіксації камери до профілю (наприклад, REV або goBILDA). Також для додаткової стабілізації можна застосувати двосторонній скотч VHB або стяжки.
* **Сквозне кріплення:** Використовуй болти №10 або М4 відповідної довжини з нейлоновими контргайками (nylock).

### Підключення до робота
* **Для FTC:** Підключи кабель USB-C до USB-A безпосередньо у **USB 3.0 порт (синій порт)** твого Control Hub.
* **Для FRC:** Підключи кабель до USB-порту roboRIO. 

---

## 3. Доступ до веб-інтерфейсу

1. Підключи Limelight 3A за допомогою кабелю USB-C до свого комп'ютера.
2. Зачекай 15–20 секунд, доки зелений індикатор стану не почне блимати.
3. Відкрий програму **Limelight Hardware Manager** або перейди у веб-браузері за адресою:
   `http://limelight.local:5801`

У веб-інтерфейсі ти зможеш налаштувати:
* **Settings:** Встановлення командного номера (Team Number), hostname тощо.
* **Vision Pipelines:** Налаштування та перемикання між 10 конфігураціями обробки зору.
* **Camera & Crosshair:** Калібрування фокуса, експозиції та прицілу.

---

## 4. Базовий приклад коду для FTC (Java)

Для інтеграції Limelight 3A у проєкт на Android Studio зазвичай потрібно менше 10 рядків коду:

```java
public class Teleop extends LinearOpMode {
    private Limelight3A limelight;

    @Override
    public void runOpMode() throws InterruptedException {
        limelight = hardwareMap.get(Limelight3A.class, "limelight");
        telemetry.setMsTransmissionInterval(11);
        limelight.pipelineSwitch(0);
        limelight.start();

        waitForStart();

        while (opModeIsActive()) {
            LLResult result = limelight.getLatestResult();
            if (result != null && result.isValid()) {
                Pose3D botpose = result.getBotpose();
                telemetry.addData("tx", result.getTx());
                telemetry.addData("ty", result.getTy());
                telemetry.addData("Botpose", botpose.toString());
            }
            telemetry.update();
        }
    }
}
