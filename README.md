# Pool Pump Scheduler Blueprint

Автоматическое управление насосом бассейна на основе объема, производительности и учета ручной работы за день.

## Описание

Этот blueprint автоматически:

-   Рассчитывает необходимое время работы насоса для полной циркуляции воды в бассейне
-   Учитывает ручное включение насоса в течение дня
-   Корректирует автоматическое расписание, чтобы не превышать максимальное время работы
-   Предоставляет три режима работы: Auto, On, Off

## Необходимые сенсоры и entities

Добавьте следующие entities в ваш `configuration.yaml`:

### 1. Input Select для режима работы насоса

```yaml
input_select:
    pool_pump_mode:
        name: Pool Pump Mode
        options:
            - 'Auto'
            - 'On'
            - 'Off'
        initial: 'Auto'
        icon: mdi:water-pump
```

**Режимы:**

-   `Auto` - Автоматическое управление по расписанию
-   `On` - Принудительно включен (всегда работает)
-   `Off` - Принудительно выключен (не работает)

### 2. Sensor для подсчета времени работы насоса за день

```yaml
sensor:
    - platform: history_stats
      name: Pool Pump Daily Runtime
      entity_id: switch.pool_pump # замените на вашу сущность насоса
      state: 'on'
      type: time
      start: '{{ now().replace(hour=0, minute=0, second=0) }}'
      end: '{{ now() }}'
```

### 3. (Опционально) Input Number для настройки параметров

```yaml
input_number:
    pool_volume:
        name: Pool Volume
        min: 1000
        max: 100000
        step: 100
        unit_of_measurement: 'L'
        icon: mdi:pool

    pump_flow_rate:
        name: Pump Flow Rate
        min: 100
        max: 10000
        step: 50
        unit_of_measurement: 'L/h'
        icon: mdi:water-pump

    max_daily_runtime:
        name: Maximum Daily Runtime
        min: 1
        max: 24
        step: 0.5
        unit_of_measurement: 'h'
        icon: mdi:clock-outline
```

### 4. (Опционально) Template sensor для отображения статуса

```yaml
template:
    - sensor:
          - name: 'Pool Pump Status'
            state: >
                {% set mode = states('input_select.pool_pump_mode') %}
                {% set pump_state = states('switch.pool_pump') %}
                {% set runtime = states('sensor.pool_pump_daily_runtime') | float(0) %}
                {% set max_time = states('input_number.max_daily_runtime') | float(8) %}

                {% if mode == 'Off' %}
                  Отключен вручную
                {% elif mode == 'On' %}
                  Включен вручную
                {% elif pump_state == 'on' %}
                  Работает ({{ runtime }}ч из {{ max_time }}ч)
                {% else %}
                  Ожидание ({{ runtime }}ч из {{ max_time }}ч)
                {% endif %}
            icon: >
                {% set mode = states('input_select.pool_pump_mode') %}
                {% set pump_state = states('switch.pool_pump') %}

                {% if mode == 'Off' %}
                  mdi:water-pump-off
                {% elif pump_state == 'on' %}
                  mdi:water-pump
                {% else %}
                  mdi:water-pump-off
                {% endif %}
```

## Настройка Blueprint

1. Импортируйте blueprint в Home Assistant
2. Создайте автоматизацию на основе blueprint
3. Настройте параметры:
    - **Pump**: Выберите сущность вашего насоса (switch)
    - **Pool Volume**: Объем бассейна в литрах
    - **Pump Flow Rate**: Производительность насоса в л/ч
    - **Maximum Daily Run Time**: Максимальное время работы в день (часы)
    - **Pool Pump Mode**: input_select.pool_pump_mode
    - **Daily Pump Runtime Sensor**: sensor.pool_pump_daily_runtime

## Принцип работы

1. **Расчет времени циркуляции**: Blueprint рассчитывает, сколько времени нужно для полной циркуляции воды: `время = объем_бассейна / производительность_насоса`

2. **Учет ручной работы**: Если насос включался вручную, это время вычитается из дневного лимита

3. **Автоматическое включение**: В режиме "Auto" насос включается автоматически, когда:

    - Насос выключен
    - Прошло достаточно времени с последнего изменения состояния
    - Не превышен дневной лимит времени работы

4. **Ручные режимы**:
    - "On" - насос всегда включен
    - "Off" - насос всегда выключен

## Примечания

-   Все единицы измерения (объем и производительность) должны быть в одной системе (литры/час, м³/час и т.д.)
-   Автоматизация проверяет состояние каждую минуту
-   При ручном включении насоса время учитывается автоматически
-   Рекомендуется настроить уведомления при превышении дневного лимита времени работы
