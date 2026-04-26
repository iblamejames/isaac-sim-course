# Урок 3 — Роботы и первые шаги с кодом

---

# Часть 1 — Готовый робот: Franka Panda

## Знакомство с Franka

Isaac Sim поставляется с набором готовых роботов — их не нужно создавать с нуля. На этом уроке сначала посмотрим на профессиональный робот, чтобы понять к чему стремимся. Потом соберём своего.

**Franka Emika Panda** — робот-манипулятор с 7 суставами. Один из самых популярных в академических исследованиях, используется в лабораториях по всему миру.

![](assets/franka.png)

---

## Подготовка сцены и добавление робота

1. `File → New` — чистая сцена
2. `Create → Physics → Physics Scene`
3. `Create → Physics → Ground Plane`

Добавить робота: `Create → Isaac → Robots → Manipulators → Franka`

Или через нижнюю панель: вкладка **Isaac Assets (Beta)** → папка Robots → найти Franka → перетащить в Viewport.

![](assets/frankadd.gif)

После добавления робот стоит в сцене статично. В Stage появится запись `factory_franka` с вложенными элементами.

---

## Структура робота в Stage

Раскрой дерево `panda` в Stage кликом на треугольник рядом с названием.

![](assets/frankaelem.gif)

**panda** — корневой объект всего робота.

**panda_link0, panda_link1...** — звенья. Отдельные физические части корпуса. Каждое звено — твёрдое тело со своей массой и формой. Это те же Rigid Body объекты из урока 2.

**panda_joint1, panda_joint2...** — суставы. Соединяют звенья и задают как они двигаются. У Franka 7 вращательных суставов.

**panda_hand, panda_finger** — захват с двумя пальцами.

Запомни эту структуру — звенья и суставы. Дальше в уроке мы соберём то же самое сами с нуля.

---

## Параметры суставов в Property

Кликни на сустав в Stage — например `panda_joint1`. В Property появятся его параметры.

![](assets/jointsprop.gif)

**Lower limit / Upper limit** — диапазон поворота в радианах. Сустав не повернётся дальше этих значений — как локоть не сгибается в обратную сторону.

**Damping** — сопротивление движению. Высокое значение — сустав двигается медленно и плавно.

**Stiffness** — жёсткость. Высокое значение — сустав держит позицию и сопротивляется внешним силам.

Кликни поочерёдно на `panda_joint1` и `panda_joint7` — у суставов рядом с основанием и рядом с захватом параметры сильно отличаются.

---

## Запуск без контроллера

Нажми Play с роботом в сцене.

![](assets/example.gif)

Робот обмякнет — суставы не удерживаются и звенья падают под гравитацией. Это нормально: робот добавлен, но команд на удержание позиции ему не поступает. Чтобы робот стоял и двигался — нужен контроллер. Именно этим займёмся в следующей части.

---

# Часть 2 — Собираем своего робота

## Почему это важно

Теперь когда видно как устроена Franka — звенья, суставы, иерархия — соберём то же самое сами. Это даст понимание того как работает любой робот в Isaac Sim изнутри, а не только снаружи.

Соберём двухколёсного робота: куб-корпус и два цилиндра-колеса, связанные суставами.

---

## Articulation Root — что это и зачем

Прежде чем начать — ключевое понятие.

**Articulation Root** — компонент который говорит Isaac Sim: "это единый робот, считай его одной физической системой". Без него суставы не работают правильно — объекты просто разлетятся при запуске.

Articulation Root добавляется один раз на корневой объект всей конструкции.

---

## Шаг 1 — Корневой объект

`Create → Xform`

Xform — пустой контейнер без формы. Нужен как точка отсчёта для всего робота.

Переименуй в `Robot`: двойной клик на названии в Stage.

Выбери `Robot` в Stage → Property → **+ Add** → `Physics → Articulation Root`

<!-- ВСТАВЬ GIF: robot_root.gif — создание Xform, переименование, добавление Articulation Root -->

---

## Шаг 2 — Корпус

`Create → Shapes → Cube`

В Stage перетащи `Cube` внутрь `Robot` — drag & drop. Это делает куб дочерним объектом робота.

Property → Transform:
- **Scale** X=1.5, Y=0.8, Z=0.4 — плоский прямоугольник
- **Translate** Z=0.3 — поднять чтобы колёса поместились снизу

**+ Add** → `Physics → Rigid Body with Colliders Preset`

<!-- ВСТАВЬ GIF: robot_body.gif — создание куба, перетаскивание в Robot, Scale и Translate, Rigid Body -->

---

## Шаг 3 — Колёса

`Create → Shapes → Cylinder` — первое колесо.

Перетащи внутрь `Robot` в Stage.

Property → Transform:
- **Scale** X=0.15, Y=0.4, Z=0.4 — тонкий диск
- **Orient** Z=90 — повернуть ось
- **Translate** X=0.6, Y=0.5, Z=0.2 — правая сторона

**+ Add** → `Rigid Body with Colliders Preset`

Создай второй цилиндр так же, Translate Y=-0.5 — левая сторона.

<!-- ВСТАВЬ GIF: robot_wheels.gif — два цилиндра по бокам корпуса -->

---

## Шаг 4 — Суставы

Сустав соединяет два объекта. Порядок выбора важен: сначала родитель, потом дочерний.

Для правого колеса:
1. Кликни `Cube` в Stage
2. Зажми **Ctrl** и кликни правый `Cylinder`
3. `Create → Physics → Joints → Revolute Joint`

В Property настрой:
- **Axis** → `Y` — ось вращения колеса
- **Local Position 0** X=0.6, Y=0.5, Z=0 — крепление к корпусу
- **Local Position 1** X=0, Y=0, Z=0 — крепление к колесу

Повтори для левого колеса.

<!-- ВСТАВЬ GIF: robot_joints.gif — выбор Cube + Cylinder через Ctrl, Create Joint, настройка Axis -->

---

## Шаг 5 — Запуск и Joint State Publisher

Нажми **Play** — робот должен упасть и устойчиво стоять на колёсах.

<!-- ВСТАВЬ GIF: robot_stand.gif — нажатие Play, робот стоит -->

Если разваливается — проверь что Articulation Root на `Robot` (Xform), а не на кубе.

Теперь открой **Joint State Publisher**: `Isaac Utils → Joint State Publisher`

Двигай слайдеры — колёса крутятся в реальном времени без единой строки кода. Это полезно чтобы проверить что суставы работают и понять их диапазон.

<!-- ВСТАВЬ GIF: jsp.gif — Joint State Publisher, слайдеры двигают колёса -->

---

# Часть 3 — Script Editor: управляем кодом

## Как Python работает в Isaac Sim

Isaac Sim построен на платформе NVIDIA Omniverse. Всё что видишь в Stage — это объекты формата **USD (Universal Scene Description)**. Когда ты добавляешь куб через меню, Isaac Sim создаёт USD объект.

**Python API** — набор готовых функций для работы с этими объектами. Вместо клика мышью вызываешь функцию — она делает то же самое внутри. Например `DynamicCuboid` делает то же что `Create → Shapes → Cube` + `Rigid Body with Colliders Preset` из урока 2 — только одной строкой.

---

## Открываем Script Editor

`Window → Script Editor`

<!-- ВСТАВЬ GIF: script_editor_open.gif — открытие Script Editor -->

> Каждый Run выполняет скрипт заново и создаёт новые объекты. Перед запуском делай `File → New`.

---

## Первый скрипт — разбираем построчно

```python
from omni.isaac.core.objects import GroundPlane
```
`from X import Y` — подключить класс `Y` из библиотеки `X`. Без этой строки Python не знает что такое `GroundPlane`. Это как открыть нужную страницу в справочнике прежде чем читать.

```python
GroundPlane(prim_path="/World/GroundPlane", z_position=0)
```
Создаём Ground Plane. В скобках — параметры:
- `prim_path` — путь в Stage. Всегда начинается с `/World/`. Имя после слеша — как назовётся в Stage
- `z_position=0` — высота поверхности

Запусти — в Stage появится `/World/GroundPlane`. То же самое что `Create → Physics → Ground Plane`, только кодом.

<!-- ВСТАВЬ GIF: ground_plane_code.gif — код, Run, Ground Plane появляется в сцене и Stage -->

---

## Объект с физикой

```python
import numpy as np
from omni.isaac.core.objects import DynamicCuboid

DynamicCuboid(
    prim_path="/World/Cube",
    position=np.array([0, 0, 2.0]),
    size=0.5,
    color=np.array([1.0, 0.0, 0.0])
)
```

`import numpy as np` — подключаем библиотеку для математики. Координаты в Isaac Sim передаются как массив из трёх чисел [X, Y, Z] — именно это делает `np.array`.

`DynamicCuboid` — куб с уже встроенными Rigid Body и Collision. `Dynamic` значит физический.

`color=np.array([1.0, 0.0, 0.0])` — цвет в RGB, значения 0–1. [1, 0, 0] = красный, [0, 1, 0] = зелёный, [0, 0, 1] = синий.

Запусти → Play — красный куб падает.

<!-- ВСТАВЬ GIF: dynamic_cube_code.gif — скрипт, Run, куб, Play, падает -->

---

## Несколько объектов через цикл

```python
import numpy as np
from omni.isaac.core.objects import DynamicCuboid

for i in range(5):
    DynamicCuboid(
        prim_path=f"/World/Cube_{i}",
        position=np.array([i * 0.8, 0, 3.0]),
        size=0.4,
        color=np.array([i * 0.2, 0.5, 1.0])
    )
```

`for i in range(5)` — цикл. `i` принимает значения 0, 1, 2, 3, 4 по очереди.

`f"/World/Cube_{i}"` — f-строка вставляет значение `i` в текст. Получится `Cube_0`, `Cube_1`... Все prim_path должны быть уникальными, иначе объекты перезапишут друг друга.

`i * 0.8` — каждый следующий куб смещается на 0.8м по X. Так они встают в ряд.

<!-- ВСТАВЬ GIF: loop_cubes.gif — 5 кубов в ряд, Play, все падают -->

---

## Практические задания

### Задание 1 — Собери робота

Собери двухколёсного робота: Xform с Articulation Root, Cube как корпус, два Cylinder как колёса, два Revolute Joint. Убедись что Joint State Publisher двигает колёса слайдерами.

<details>
<summary>Подсказка</summary>

Частые ошибки:
- Articulation Root на кубе вместо Xform
- Цилиндры не перетащены внутрь `Robot` в Stage
- При создании сустава порядок выбора перепутан: сначала куб, потом цилиндр через Ctrl

<!-- ВСТАВЬ GIF: zad1.gif -->

</details>

---

### Задание 2 — Воспроизведи сцену из урока 2 кодом

В уроке 2 ты создавал Ground Plane, куб и сферу вручную. Теперь сделай то же через Script Editor. `DynamicSphere` работает так же как `DynamicCuboid` — только вместо `size` используй `radius`.

Попробуй написать скрипт сам опираясь на примеры выше. Подсказку открывай только если застрял.

<details>
<summary>Подсказка</summary>

```python
import numpy as np
from omni.isaac.core.objects import GroundPlane, DynamicCuboid, DynamicSphere

GroundPlane(prim_path="/World/GroundPlane", z_position=0)

DynamicCuboid(
    prim_path="/World/Cube",
    position=np.array([0, 0, 2.0]),
    size=0.5,
    color=np.array([1.0, 0.0, 0.0])
)

DynamicSphere(
    prim_path="/World/Sphere",
    position=np.array([0.3, 0, 4.0]),
    radius=0.3,
    color=np.array([0.0, 0.8, 0.2])
)
```

<!-- ВСТАВЬ GIF: zad2.gif -->

</details>

---

## Итоги урока

**Franka Panda:**
- Профессиональный манипулятор с 7 суставами: звенья (link) + суставы (joint) + захват
- Без контроллера обмякает под гравитацией — нужен скрипт управления

**Сборка робота:**
- Articulation = Xform (корень + Articulation Root) + звенья (Rigid Body) + суставы (Revolute Joint)
- Порядок при создании сустава: сначала родитель, потом дочерний через Ctrl
- Joint State Publisher двигает суставы слайдерами без кода

**Код:**
- `from X import Y` — подключает нужный модуль
- `DynamicCuboid` / `DynamicSphere` — объекты с физикой, аналог Rigid Body из интерфейса
- `prim_path` — путь в Stage, всегда `/World/имя`
- `np.array([X, Y, Z])` — координаты через numpy
- `for i in range(N)` — создаёт много объектов за несколько строк

---

*Следующий урок: управляем роботом кодом и работаем с Franka →*
