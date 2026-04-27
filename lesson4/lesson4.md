# Урок 4 — Управление роботом через код

---

## Связь с предыдущим уроком

В уроке 3 мы научились управлять объектами через Script Editor: создавать их кодом, перемещать, менять физику. Теперь применим те же инструменты к настоящему роботу.

Робот в Isaac Sim — это тот же набор физических объектов из урока 2, только связанных суставами. Управлять им через код — значит отправлять командные позиции в каждый сустав. Именно так работают реальные контроллеры роботов.

---

## Загружаем Franka Panda

Franka Emika Panda — промышленный робот-манипулятор с 7 суставами. Используется в исследованиях и производстве.

Создать сцену: `File → New` → `Create → Physics → Physics Scene` → `Create → Physics → Ground Plane`

Добавить робота: `Create → Isaac → Robots → Manipulators → Franka`

<!-- ВСТАВЬ GIF: add_franka.gif — создание сцены, добавление Franka через меню, робот появляется в Viewport -->

После добавления не нажимай Play сразу. Сначала разберёмся со структурой.

---

## Структура робота — связь с уроком 2

Раскрой дерево `panda` в Stage. То что видишь — это всё те же Prim из урока 2, только организованные в иерархию.

**panda_link0...panda_link7** — звенья. Это Rigid Body объекты с Collision, как кубы из урока 2.

**panda_joint1...panda_joint7** — суставы. Каждый соединяет два звена и разрешает вращение вокруг одной оси.

<!-- ВСТАВЬ GIF: stage_tree.gif — раскрытие дерева panda в Stage, клик на link и joint, параметры в Property -->

Кликни на `panda_joint1` в Stage — в Property увидишь:

- **Lower / Upper limit** — диапазон поворота в радианах
- **Damping** — сопротивление движению
- **Stiffness** — жёсткость удержания позиции

Это те же физические параметры что у объектов из урока 2 — только применённые к суставам.

---

## Dynamic Control API — интерфейс управления роботом

В Isaac Sim 4.2 для управления суставами через код используется **Dynamic Control** — набор функций который позволяет читать и задавать позиции, скорости и усилия в суставах.

Принцип простой: получаем ссылку на сустав → отправляем ему целевую позицию → физический движок двигает сустав туда.

---

## Первый скрипт — двигаем один сустав

Добавь Franka в сцену, нажми **Play**, затем открой Script Editor и запусти:

```python
from omni.isaac.dynamic_control import _dynamic_control
import math

dc = _dynamic_control.acquire_dynamic_control_interface()

# получаем ссылку на всего робота
art = dc.get_articulation("/World/panda")
dc.wake_up_articulation(art)

# находим первый сустав по имени
joint = dc.find_articulation_dof(art, "panda_joint2")

# отправляем команду — повернуть на 45 градусов
dc.set_dof_position_target(joint, math.radians(45))
```

Рука робота повернётся. Попробуй изменить `45` на другое число и запусти снова.

<!-- ВСТАВЬ GIF: move_joint.gif — скрипт запускается во время Play, сустав робота поворачивается -->

> Скрипт должен запускаться пока симуляция работает (Play нажат). Без активной симуляции Dynamic Control не работает.

---

## Управление несколькими суставами

Теперь поставим все 7 суставов в определённую позицию одним скриптом:

```python
from omni.isaac.dynamic_control import _dynamic_control
import math

dc = _dynamic_control.acquire_dynamic_control_interface()
art = dc.get_articulation("/World/panda")
dc.wake_up_articulation(art)

# позиции для всех 7 суставов в радианах
positions = [
    math.radians(0),    # joint1 — основание
    math.radians(-45),  # joint2
    math.radians(0),    # joint3
    math.radians(-90),  # joint4
    math.radians(0),    # joint5
    math.radians(90),   # joint6
    math.radians(45),   # joint7 — запястье
]

joint_names = [
    "panda_joint1", "panda_joint2", "panda_joint3",
    "panda_joint4", "panda_joint5", "panda_joint6", "panda_joint7"
]

for name, pos in zip(joint_names, positions):
    joint = dc.find_articulation_dof(art, name)
    dc.set_dof_position_target(joint, pos)
```

Робот примет конкретную позу. Меняй значения в списке `positions` и наблюдай как меняется поза.

<!-- ВСТАВЬ GIF: multi_joint.gif — скрипт с позициями, робот принимает позу, изменение значений и другая поза -->

---

## Читаем текущие позиции суставов

Скрипт может не только задавать позиции, но и читать их. Это важно в реальных проектах — нужно знать где робот сейчас чтобы планировать следующее движение.

```python
from omni.isaac.dynamic_control import _dynamic_control
import math

dc = _dynamic_control.acquire_dynamic_control_interface()
art = dc.get_articulation("/World/panda")

joint_names = [
    "panda_joint1", "panda_joint2", "panda_joint3",
    "panda_joint4", "panda_joint5", "panda_joint6", "panda_joint7"
]

print("Текущие позиции суставов:")
for name in joint_names:
    joint = dc.find_articulation_dof(art, name)
    state = dc.get_dof_state(joint, _dynamic_control.STATE_POS)
    angle = math.degrees(state.pos)
    print(f"  {name}: {angle:.1f}°")
```

Результат появится в панели **Console** внизу экрана. Запусти скрипт, потом измени позу через предыдущий скрипт и запусти снова — значения изменятся.

<!-- ВСТАВЬ GIF: read_joints.gif — запуск скрипта чтения, вывод в Console, изменение позы, повторное чтение -->

---

## Практические задания

### Задание 1 — Задай позу роботу

Загрузи Franka в сцену. Используя скрипт с несколькими суставами, поставь робота в такую позу: joint2 = -60°, joint4 = -120°, joint6 = 60°. Остальные суставы оставь в 0°. Посмотри на форму которую принял манипулятор.

<details>
<summary>Подсказка</summary>

Скопируй скрипт из раздела "Управление несколькими суставами" и поменяй только значения в списке `positions`. Угол задаётся в радианах — используй `math.radians(значение_в_градусах)`.

Не забудь нажать Play перед запуском скрипта.

<!-- ВСТАВЬ GIF: zad1.gif -->

</details>

---

### Задание 2 — Прочитай и измени

Запусти скрипт чтения позиций суставов и запомни (или запиши) текущие значения. Затем запусти скрипт который меняет позу. Снова запусти чтение и сравни значения. Убедись что они изменились на те что ты задал.

<details>
<summary>Подсказка</summary>

Удобно держать три вкладки в Script Editor: одна для загрузки сцены, одна для задания позы, одна для чтения. Переключайся между ними и запускай по очереди.

Вывод скриптов смотри в панели **Console** — вкладка Console внизу экрана (рядом с Content).

<!-- ВСТАВЬ GIF: zad2.gif -->

</details>

---

## Итоги урока и всего курса

**Урок 4:**
- Franka Panda состоит из тех же Prim и физических объектов что мы изучали в уроках 2 и 3
- Dynamic Control API позволяет читать и задавать позиции суставов через код
- `acquire_dynamic_control_interface()` — получить интерфейс управления
- `get_articulation()` — получить ссылку на робота
- `set_dof_position_target()` — задать целевую позицию сустава
- `get_dof_state()` — прочитать текущее состояние сустава

---

**Что умеешь после всего курса:**

- Ориентируешься в интерфейсе Isaac Sim: Viewport, Stage, Property, Content
- Понимаешь физическую модель: Prim, Rigid Body, Collision, Physics Material
- Строишь сцены через интерфейс и через Python код
- Управляешь объектами в симуляции через Script Editor
- Загружаешь готовых промышленных роботов и управляешь их суставами через код

Это фундамент для работы с Isaac Sim — дальше на этих знаниях строятся обучение роботов, работа с сенсорами, интеграция с ROS и всё остальное что есть в платформе.
