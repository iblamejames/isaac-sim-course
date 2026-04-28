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

После добавления не нажимай Play сразу. Сначала разберёмся со структурой.

---

## Структура робота — связь с уроком 2

Раскрой дерево `panda` в Stage. То, что видишь — это всё те же Prim из урока 2, только организованные в иерархию.

**panda_link0...panda_link7** — звенья. Это Rigid Body объекты с Collision, как кубы из урока 2.

**panda_joint1...panda_joint7** — суставы. Каждый соединяет два звена и разрешает вращение вокруг одной оси.

Кликни на `panda_joint1` в Stage — в Property увидишь:

- **Lower / Upper limit** — диапазон поворота в радианах
- **Damping** — сопротивление движению
- **Stiffness** — жёсткость удержания позиции

Это те же физические параметры, что у объектов из урока 2 — только применённые к суставам.

---

## Dynamic Control API — интерфейс управления роботом

В Isaac Sim для управления суставами через код используется **Dynamic Control** — набор функций, который позволяет читать и задавать позиции, скорости и усилия в суставах.

Принцип простой: получаем ссылку на сустав → отправляем ему целевую позицию → физический движок двигает сустав туда.

> **Важно:** все скрипты этого урока должны запускаться пока симуляция активна — то есть после нажатия **Play**. Без активной симуляции Dynamic Control не работает.

---

## Первый скрипт — двигаем один сустав

Добавь Franka в сцену, нажми **Play**, затем открой Script Editor (`Window → Script Editor`) и запусти:

```python
from omni.isaac.dynamic_control import _dynamic_control
import math

dc = _dynamic_control.acquire_dynamic_control_interface()

# Получаем ссылку на всего робота
art = dc.get_articulation("/World/panda")
dc.wake_up_articulation(art)

# Находим сустав по имени
joint = dc.find_articulation_dof(art, "panda_joint2")

# Отправляем команду — повернуть на 45 градусов
dc.set_dof_position_target(joint, math.radians(45))
```

Рука робота повернётся. Попробуй изменить `45` на другое число и запусти снова — сустав займёт новое положение.

---

## Разбор скрипта по строкам

```python
dc = _dynamic_control.acquire_dynamic_control_interface()
```
Получаем объект `dc` — через него будут идти все команды к роботу. Это точка входа в Dynamic Control API.

```python
art = dc.get_articulation("/World/panda")
```
Получаем ссылку на робота по его пути в Stage. Если путь неверный — `art` будет пустым и следующие команды не сработают.

```python
dc.wake_up_articulation(art)
```
"Будим" робота. Физический движок может переводить неподвижные объекты в спящий режим для оптимизации — эта команда принудительно активирует робота.

```python
dc.set_dof_position_target(joint, math.radians(45))
```
Отправляем целевую позицию в градусах, переведённых в радианы. `set_dof_position_target` — основная команда управления позицией.

---

## Управление несколькими суставами

Теперь поставим все 7 суставов в определённую позицию одним скриптом:

```python
from omni.isaac.dynamic_control import _dynamic_control
import math

dc = _dynamic_control.acquire_dynamic_control_interface()
art = dc.get_articulation("/World/panda")
dc.wake_up_articulation(art)

# Позиции для всех 7 суставов в радианах
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

Робот примет конкретную позу. Меняй значения в списке `positions` и наблюдай, как меняется положение руки.

`zip(joint_names, positions)` — соединяет два списка попарно. На каждом шаге цикла `name` получает имя сустава, а `pos` — соответствующую позицию.

---

## Читаем текущие позиции суставов

Скрипт может не только задавать позиции, но и читать их. Это важно в реальных проектах — нужно знать, где робот находится сейчас, чтобы планировать следующее движение.

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

`get_dof_state` — читает текущее состояние сустава. `STATE_POS` означает, что нас интересует только позиция (не скорость и не усилие).

`state.pos` возвращает значение в радианах — `math.degrees()` переводит его в градусы для удобства.

Результат появится в панели **Console** внизу экрана (вкладка рядом с Content). Запусти скрипт, потом измени позу через предыдущий скрипт и запусти снова — значения изменятся.

---

## Практические задания

### Задание 1 — Задай позу роботу

Загрузи Franka в сцену. Используя скрипт с несколькими суставами, поставь робота в такую позу: joint2 = -60°, joint4 = -120°, joint6 = 60°. Остальные суставы оставь в 0°. Посмотри на форму, которую принял манипулятор.

<details>
<summary>Подсказка</summary>

Скопируй скрипт из раздела "Управление несколькими суставами" и поменяй только значения в списке `positions`. Угол задаётся в радианах — используй `math.radians(значение_в_градусах)`.

Не забудь нажать Play перед запуском скрипта.

</details>

---

### Задание 2 — Прочитай и измени

Запусти скрипт чтения позиций суставов и запомни (или запиши) текущие значения. Затем запусти скрипт, который меняет позу. Снова запусти чтение и сравни значения — убедись, что они изменились именно на те, что ты задал.

<details>
<summary>Подсказка</summary>

Удобно держать несколько вкладок в Script Editor: одну для задания позы, другую для чтения. В Script Editor можно создавать новые вкладки через кнопку **+** в верхней части панели — переключайся между ними и запускай по очереди.

Вывод скриптов смотри в панели **Console** — вкладка внизу экрана рядом с Content.

</details>

---

## Итоги урока

- `acquire_dynamic_control_interface()` — получить интерфейс управления Dynamic Control
- `get_articulation("/World/panda")` — получить ссылку на робота по пути в Stage
- `wake_up_articulation(art)` — активировать робота перед отправкой команд
- `find_articulation_dof(art, name)` — найти сустав по имени
- `set_dof_position_target(joint, angle)` — задать целевую позицию сустава в радианах
- `get_dof_state(joint, STATE_POS)` — прочитать текущее состояние сустава
- Все скрипты управления роботом запускаются только во время активной симуляции (Play)

---

## Итоги всего курса

**Что умеешь после курса:**

Ориентируешься в интерфейсе Isaac Sim: знаешь назначение Viewport, Stage, Property, Content и Toolbar, умеешь добавлять объекты и управлять симуляцией.

Понимаешь физическую модель: знаешь разницу между визуальным и физическим Prim, умеешь настраивать Rigid Body, Collision и Physics Material для управления поведением объектов.

Строишь сцены двумя способами: через графический интерфейс мышью и через Python код в Script Editor — используя DynamicCuboid, DynamicSphere, GroundPlane и циклы для создания множества объектов.

Собираешь роботов вручную: понимаешь структуру "звенья + суставы", умеешь настраивать Articulation Root и Revolute Joint, проверяешь суставы через Joint State Publisher.

Управляешь промышленным роботом кодом: загружаешь Franka Panda, задаёшь позиции всех 7 суставов через Dynamic Control API и читаешь их текущее состояние.

Это фундамент для работы с Isaac Sim — дальше на этих знаниях строятся обучение роботов методами reinforcement learning, работа с сенсорами (камеры, лидары), интеграция с ROS 2 и всё остальное, что есть в платформе.
