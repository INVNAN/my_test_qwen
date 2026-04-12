# Техническая спецификация

## Требования к разработке

### Движок

**Рекомендуемый движок:** Unity 2022 LTS или Unreal Engine 5

**Обоснование выбора Unity:**
- Отличная поддержка 2D/3D гибридных интерфейсов
- Богатая экосистема ассетов для ретро-стилистики
- Кроссплатформенность из коробки
- Активное сообщество и документация
- UI Toolkit для создания терминальных интерфейсов

**Альтернатива (Unreal Engine 5):**
- Более продвинутая графика
- Blueprint для быстрой прототипизации
- Но тяжеловеснее для данного проекта

---

### Минимальные системные требования

| Компонент | Минимум | Рекомендуется |
|-----------|---------|---------------|
| **ОС** | Windows 10 64-bit | Windows 11 64-bit |
| **Процессор** | Intel Core i3-6100 / AMD FX-6300 | Intel Core i5-8400 / AMD Ryzen 5 2600 |
| **Оперативная память** | 8 GB RAM | 16 GB RAM |
| **Видеокарта** | NVIDIA GTX 750 Ti / AMD R7 260X | NVIDIA GTX 1060 6GB / AMD RX 580 |
| **DirectX** | Версия 11 | Версия 12 |
| **Место на диске** | 10 GB | 10 GB SSD |
| **Звуковая карта** | DirectX совместимая | DirectX совместимая |

### Целевые платформы

**Основная:** PC (Windows, Linux, macOS)  
**Приоритетные:** Steam, GOG, Epic Games Store  
**Потенциальные:** Steam Deck (валидация совместимости)

---

## Архитектура проекта

### Структура папок Unity

```
CyberSec_Protocol/
├── Assets/
│   ├── _Scripts/
│   │   ├── Core/              # Основные системы игры
│   │   ├── Terminal/          # Логика терминала
│   │   ├── Network/           # Сетевая симуляция
│   │   ├── AI/                # ИИ охраны, NPC
│   │   ├── UI/                # Интерфейсы
│   │   └── Utils/             # Утилиты
│   ├── _Art/
│   │   ├── Models/            # 3D модели
│   │   ├── Textures/          # Текстуры
│   │   ├── Materials/         # Материалы
│   │   └── Animations/        # Анимации
│   ├── _Audio/
│   │   ├── SFX/               # Звуковые эффекты
│   │   ├── Music/             # Музыка
│   │   └── Voice/             # Озвучка (если будет)
│   ├── _Prefabs/              # Префабы объектов
│   ├── _Scenes/               # Сцены
│   ├── _ScriptableObjects/    # Данные (навыки, предметы)
│   └── _Resources/            # Ресурсы времени выполнения
├── Packages/
└── ProjectSettings/
```

---

## Ключевые системы

### 1. Система терминала

**Компоненты:**
- `TerminalController` — управление вводом/выводом
- `CommandParser` — парсинг команд игрока
- `CommandRegistry` — реестр доступных команд
- `OutputRenderer` — рендеринг текста с эффектами

**Пример архитектуры команды:**
```csharp
public abstract class BaseCommand
{
    public string Name { get; protected set; }
    public string Description { get; protected set; }
    public List<Parameter> Parameters { get; protected set; }
    
    public abstract CommandResult Execute(string[] args);
    public abstract string GetHelp();
}

public class ScanCommand : BaseCommand
{
    public override CommandResult Execute(string[] args)
    {
        // Логика сканирования
        var target = ParseTarget(args);
        var results = NetworkScanner.Scan(target);
        return CommandResult.Success(results);
    }
}
```

**Функции:**
- Парсинг аргументов командной строки
- Автодополнение (Tab-completion)
- История команд (сохранение между сессиями)
- Подсветка синтаксиса
- Контекстная справка

---

### 2. Сетевая симуляция

**Компоненты:**
- `NetworkGraph` — представление сети как графа
- `Node` — узел сети (сервер, рабочая станция, фаервол)
- `Connection` — соединение между узлами
- `FirewallRule` — правила фильтрации трафика
- `PacketSimulator` — симуляция передачи пакетов

**Структура данных узла:**
```csharp
[System.Serializable]
public class NetworkNode
{
    public string Id;
    public NodeType Type;
    public string IPAddress;
    public int SecurityLevel;
    public List<Port> OpenPorts;
    public List<Vulnerability> Vulnerabilities;
    public bool IsCompromised;
    public string LootTableId;
}

public enum NodeType
{
    Workstation,
    Server,
    Firewall,
    Router,
    Database,
    IoT_Device
}
```

**Генерация сети:**
- Процедурная генерация для ежедневных контрактов
- Ручная настройка для сюжетных миссий
- Сохранение состояния (взломанные узлы остаются)

---

### 3. Физическое взаимодействие

**Компоненты:**
- `InteractableObject` — базовый класс для взаимодействуемых объектов
- `CableConnection` — логика подключения кабелей
- `DeviceSwap` — замена оборудования
- `TemperatureSystem` — симуляция нагрева/охлаждения
- `StealthDetector` — обнаружение игрока

**Мини-игры:**
```csharp
public class CableMiniGame : MonoBehaviour
{
    public List<CablePort> Ports;
    public CableType RequiredType;
    public float TimeLimit;
    public float AccuracyThreshold;
    
    public bool StartMiniGame()
    {
        // Логика мини-игры
        // Возвращает успех/неудачу
    }
}
```

**Физика:**
- Raycast для взаимодействия с объектами
- Коллайдеры для точного позиционирования
- Анимации через Animator Controller

---

### 4. Система прогрессии

**Компоненты:**
- `PlayerStats` — статистика игрока
- `SkillTree` — дерево навыков
- `InventoryManager` — инвентарь
- `ReputationSystem` — репутация с фракциями
- `QuestTracker` — отслеживание заданий

**ScriptableObject для навыков:**
```csharp
[CreateAssetMenu(fileName = "NewSkill", menuName = "CyberSec/Skill")]
public class SkillData : ScriptableObject
{
    public string SkillName;
    public SkillBranch Branch;
    public int Level;
    public Sprite Icon;
    public string Description;
    public List<SkillEffect> Effects;
    public int Cost;
    public List<SkillData> Prerequisites;
}
```

**Сохранение прогресса:**
- JSON или Binary формат
- Автосохранение после миссий
- Ручное сохранение в любой момент
- Облачные сохранения (Steam Cloud)

---

### 5. ИИ и поведение NPC

**Компоненты:**
- `GuardAI` — поведение охраны
- `NPCHelper` — вспомогательные NPC
- `PatrolRoute` — маршруты патрулирования
- `DetectionSystem` — система обнаружения

**Машина состояний охраны:**
```
[IDLE] → [ALERT] → [SEARCH] → [COMBAT] → [RETURN]
   ↑         ↓          ↓          ↓          ↓
   └─────────┴──────────┴──────────┴──────────┘
```

**Параметры ИИ:**
- Поле зрения (конус обзора)
- Дальность слышимости
- Время реакции
- Приоритеты целей
- Координация с другими охранниками

---

## Оптимизация

### Графика

**LOD (Level of Detail):**
- 3 уровня детализации для моделей
- Автоматическое переключение по дистанции

**Occlusion Culling:**
- Отсечение невидимых объектов
- Предварительный запекание для статичных сцен

**Texture Streaming:**
- Подгрузка текстур по мере необходимости
- Мипмапы для всех текстур

### Память

**Пулинг объектов:**
- Переиспользование префабов (пули, эффекты)
- Снижение нагрузки на Garbage Collector

**Асинхронная загрузка:**
- Загрузка сцен в фоне
- Экраны загрузки с прогресс-баром

### Код

**Профилирование:**
- Unity Profiler для поиска узких мест
- Оптимизация Update() методов
- Использование Coroutines вместо тяжёлых циклов

---

## Звуковое оформление

### Инструменты

- **FMOD** или **Wwise** — интерактивное аудио
- **Audacity** — базовое редактирование
- **Reaper** — сведение и мастеринг

### Категории звуков

| Категория | Описание | Примеры |
|-----------|----------|---------|
| **SFX Терминал** | Звуки интерфейса | Клавиши, ошибки, успех |
| **SFX Окружение** | Фоновые звуки | Гул серверов, кондиционеры |
| **SFX Действия** | Звуки действий | Шаги, открытие дверей, подключение |
| **SFX Тревога** | Сигналы опасности | Сирены, голосовые оповещения |
| **Music** | Фоновая музыка | Эмбиент, напряжённые треки |

### Реализация

```csharp
public class AudioManager : MonoBehaviour
{
    public static AudioManager Instance;
    
    [SerializeField] private AudioSource sfxSource;
    [SerializeField] private AudioSource musicSource;
    [SerializeField] private AudioSource ambientSource;
    
    public void PlaySFX(AudioClip clip, float volume = 1f)
    {
        sfxSource.PlayOneShot(clip, volume);
    }
    
    public void PlayMusic(AudioClip clip, float fadeDuration = 1f)
    {
        // Плавный переход между треками
    }
}
```

---

## Тестирование

### Типы тестирования

1. **Unit Testing** — тесты отдельных функций
2. **Integration Testing** — тесты взаимодействия систем
3. **Playtesting** — тестирование геймплея
4. **Performance Testing** — тесты производительности
5. **Compatibility Testing** — тесты на разных конфигурациях

### Фреймворки

- **Unity Test Framework** — встроенное решение
- **NUnit** — для unit-тестов
- **Manual Test Cases** — чек-листы для ручного тестирования

### CI/CD

- **GitHub Actions** или **Jenkins** — автоматизация сборки
- **Automated Builds** — ночные сборки
- **Version Control** — Git с proper branching strategy

---

## Безопасность и античит

### Для одиночной игры

- Валидация сохранений (контрольные суммы)
- Обфускация критического кода
- Проверка целостности файлов

### Для лидербордов (ежедневные контракты)

- Серверная валидация результатов
- Хеширование данных перед отправкой
- Detection модификаций клиента

---

## Локализация

### Поддерживаемые языки

- **Русский** (основной)
- **Английский** (обязательный)
- **Китайский (упрощённый)** (потенциально)
- **Японский** (потенциально)
- **Немецкий, Французский, Испанский** (по спросу)

### Система локализации

- **I2 Localization** или **Unity Localization Package**
- External CSV/JSON файлы для текстов
- Поддержка RTL (right-to-left) языков
- Автоматическое определение языка системы

### Особенности

- Терминология должна быть консистентной
- Технические термины могут оставаться на английском
- Культурные адаптации для некоторых регионов

---

## План разработки

### Фаза 1: Препродакшн (1–2 месяца)

- [ ] Финализация дизайн-документа
- [ ] Создание прототипов ключевых механик
- [ ] Арт-концепты и референсы
- [ ] Технический дизайн документа
- [ ] Сбор核心团队 (если нужно)

### Фаза 2: Пре-альфа (3–6 месяцев)

- [ ] Базовая реализация терминала
- [ ] Прототип сетевой симуляции
- [ ] Первые тестовые уровни
- [ ] Базовый UI
- [ ] Внутреннее тестирование

### Фаза 3: Альфа (6–9 месяцев)

- [ ] Все основные механики реализованы
- [ ] 50% контента (уровни, сюжет)
- [ ] Интеграция звука и музыки
- [ ] Первое публичное демо
- [ ] Сбор фидбека

### Фаза 4: Бета (9–12 месяцев)

- [ ] 100% контента
- [ ] Полишинг и баланс
- [ ] Оптимизация производительности
- [ ] Локализация
- [ ] Закрытое бета-тестирование

### Фаза 5: Релиз (12–15 месяцев)

- [ ] Финальный QA
- [ ] Маркетинговая кампания
- [ ] Релиз на платформах
- [ ] Пост-релизная поддержка

---

## Пост-релизная поддержка

### Контентные обновления

- **Бесплатные:**
  - Ежедневные контракты (автоматически)
  - Исправления багов
  - Балансные правки

- **Платные DLC (опционально):**
  - Новые сюжетные миссии
  - Дополнительные локации
  - Косметические предметы

### Комьюнити

- Discord сервер
- Reddit сообщество
- Регулярные девлоги
- Моддинг поддержка (если возможно)

### Метрики успеха

- Отзывы в Steam (>80% положительных)
- Продажи (цель: 50,000 копий за первый год)
- Активная база игроков (ежедневные контракты)
- Медиа освещение
