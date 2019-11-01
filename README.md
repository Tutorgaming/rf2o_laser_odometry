# rf2o_laser_odometry

> Estimation of 2D odometry based on planar laser scans. Useful for mobile robots with innacurate base odometry. http://mapir.isa.uma.es/work/rf2o For full description of the algorithm, please refer to: **Planar Odometry from a Radial Laser Scanner. A Range Flow-based Approach. ICRA 2016**

Модуль `rf2o_laser_odometry` производит оценку пройденного роботом пути по данным сканирующего лазерного дальномера. Сборка модуля производится стандартным способом для ROS со следующими зависимостями:
- `catkin`
- `roscpp`
- `sensor_msgs`
- `std_msgs`
- `tf`
- `cmake_modules`
- `eigen`
- `nav_msgs`

## Запуск модуля

Запуск модуля производится сервером `roslaunch` с помощью `.launch`-файла ROS следующего вида:

```xml
<launch>
    <node   pkg="rf2o_laser_odometry" 
            type="rf2o_laser_odometry_node" 
            name="rf2o_laser_odometry" 
            output="screen">
        <rosparam file="$(find rf2o_laser_odometry)/config/default.yml"/>
    </node>
</launch>
```

## Настройка

Настройка модуля производится в `.launch`-файле, либо с помощью подключения конфигурационного файла в формате YAML (настройки по умолчанию сохраняются в файле [`default.yml`](./config/default.yml)). По умолчанию все настройки необязательны.

### Общие настройки

|**Имя**|**Значение по умолчанию**|**Тип**|**Описание**|
|-------|-------------------------|-------|------------|
|`verbose`|`false`|`bool`|Включает/выключает вывод отладочных сообщений|
|`laser_scan_topic`|`/laser_scan`|`std::string`|Имя темы, из которой берется входной поток данных от драйвера дальномера|
|`laser_frame_id`|`/laser`|`std::string`|Идентификатор типа и связанной системы координат сообщения от драйвера дальномера. Применяется в случае, если формат публикации сообщений драйвера не полностью соответствует стандартам ROS. **ВНИМАНИЕ!** Для корректной работы модуля должно существовать преобразование координат из `base_frame_id` в `laser_frame_id`|
|`odom_topic`|`/odom_rf2o`|`std::string`|Имя темы, в которую публикуются результаты работы модуля|
|`publish_tf`|`true`|`bool`|Включает/выключает публикацию преобразования координат из `base_frame_id` в `odom_frame_id`|
|`base_frame_id`|`/base_link`|`std::string`|Идентификатор типа и связанной системы координат сообщения, содержащего положение робота|
|`odom_frame_id`|`/odom`|`std::string`|Идентификатор типа и связанной системы координат сообщения, содержащего рассчитанную одометрию|
|`init_pose_from_topic`|`/base_pose_ground_truth`|`std::string`|Имя темы, в которую публикуется начальное положение робота|
|`freq`|`10.0`|`double`|Тактовая периодичность (частота выдачи данных одометрии), Гц|

### Настройки ковариации

|**Имя**|**Значение по умолчанию**|**Тип**|**Описание**|
|-------|-------------------------|-------|------------|
|`pose_covariance_matrix`|Нулевая матрица 6х6|`std::vector<double>`|Матрица ковариации для положения|
|`twist_covariance_matrix`|Нулевая матрица 6х6|`std::vector<double>`|Матрица ковариации для кинематических параметров|

#### Настройки автоматической подстройки ковариации

Данные настройки предназначены для автоматической подстройки матриц ковариации при разрыве одометрии или потере сигнала от дальномера. Все настройки, описанные в данном разделе, сгруппированы в YAML-файле под именем `dynamic_covariance_boost`

|**Имя**|**Значение по умолчанию**|**Тип**|**Описание**|
|-------|-------------------------|-------|------------|
|`enable`|`false`|`bool`|Включает/выключает автоматическую подстройку ковариации|
|`initial_multiplier`|`1.0`|`double`|Значение множителя, на который умножаются элементы матриц ковариации при разрыве|
|`progressive`|`false`|`bool`|Включает/выключает прогрессивное увеличение элементов матриц ковариации при разрыве, длящемся более одного тактового периода|
|`progression_factor`|`0`|`double`|Величина прогрессивного увеличения элементов матриц ковариации|

#### Настройки доверия

Данные настройки предназначены для автоматической замены показаний одометра данными из аварийных источников при разрыве одометрии или потере доверия к выходным данным модуля. Все настройки, описанные в данном разделе, сгруппированы в YAML-файле под именем `pose_fallback`.

##### Пределы доверия

Значение предела доверия указывает максимальное допустимое расхождение между значениями параметров робота, полученными из расчета и значениями, переданными из аварийных источников.

|**Имя**|**Значение по умолчанию**|**Тип**|**Описание**|
|-------|-------------------------|-------|------------|
|`continuous_fallback_topic`|Пустая строка|`std::string`|Имя темы, из которой берутся данные одометрии в случае разрыва (данные обеспечения непрерывности показаний)|
|`velocity_fallback_topic`|Значение `continuous_fallback_topic`|`std::string`|Имя темы, из которой берутся данные о кинематических параметрах движения робота при потере доверия к рассчитанным|
|`enable_thresholds`|`false`|`bool`|Включает/выключает учет пределов доверия к показаниям модуля|
|`linear_velocity_threshold_x`|$`10^6`$|`double`|Предел доверия к рассчитанной величине скорости по продольной оси робота|
|`linear_velocity_threshold_y`|Значение `linear_velocity_threshold_x`|`double`|Предел доверия к рассчитанной величине скорости по поперечной оси робота|
|`angular_velocity_threshold`|$`10^6`$|`double`|Предел доверия к рассчитанной величине угловой скорости робота относительно геометрического центра, обозначенного началом координат системы `base_frame_id`|

