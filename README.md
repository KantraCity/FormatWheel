# Format Wheel

Конвертация файлов прямо на лету, пока вы их перетаскиваете. Аналог «колеса форматов» с macOS для Windows.

Тянете файл → зажимаете **Shift** → вокруг курсора раскрывается колесо → наводите на нужный формат → отпускаете Shift и тянете дальше уже сконвертированный файл: в чат, браузер, Figma, другую папку. Оригинал не трогается.

Картинки программа конвертирует сама. Видео, аудио, документы, данные и всё остальное добавляется **расширениями** — и написать своё может каждый.

*English below.*

## Репозитории

| Репозиторий | Что внутри |
|---|---|
| **FormatWheel** (этот) | Описание, список расширений, инструкция по написанию своих |
| [FormatWheel-app](https://github.com/KantraCity/FormatWheel-app) | Код программы и SDK для расширений |
| [FormatWheel-ffmpeg](https://github.com/KantraCity/FormatWheel-ffmpeg) | Расширение: видео и аудио через FFmpeg |
| [FormatWheel-audio](https://github.com/KantraCity/FormatWheel-audio) | Расширение: аудио на кодеках Windows, без FFmpeg |
| [FormatWheel-data](https://github.com/KantraCity/FormatWheel-data) | Расширение: JSON, YAML, XML, TOML, CSV |
| [FormatWheel-pandoc](https://github.com/KantraCity/FormatWheel-pandoc) | Расширение: документы через Pandoc |

## Установка

1. Скачайте и запустите `FormatWheel-Setup-x.y.z.exe` из релизов [FormatWheel-app](https://github.com/KantraCity/FormatWheel-app/releases).
2. По желанию поставьте расширения из списка ниже.

Нужна Windows 10 (1809) или 11, x64. Права администратора не нужны, и запускать программу от администратора нельзя: Windows не даёт перетаскивать файлы из обычного Проводника в процессы с повышенными правами.

## Проверенные расширения

| Расширение | Файлы | Установщик |
|---|---|---|
| **FFmpeg** | видео (MP4, HEVC, WEBM, MKV, MOV, AVI, WMV, GIF…) и аудио (MP3, M4A, OPUS, FLAC…); сжатие, разрешение, без звука | [релизы](https://github.com/KantraCity/FormatWheel-ffmpeg/releases) |
| **Аудио** | MP3, M4A, OPUS, FLAC, ALAC, WMA, WAV, AIFF без FFmpeg; битрейт, моно/стерео, выравнивание громкости | [релизы](https://github.com/KantraCity/FormatWheel-audio/releases) |
| **Данные** | JSON, YAML, XML, TOML, CSV — в любую сторону | [релизы](https://github.com/KantraCity/FormatWheel-data/releases) |
| **Pandoc** | документы: DOCX, PDF, ODT, RTF, HTML, Markdown, TXT, EPUB, PPTX | [релизы](https://github.com/KantraCity/FormatWheel-pandoc/releases) |

Хотите, чтобы ваше расширение попало в список проверенных? Напишите в Telegram [@kantradev](https://t.me/kantradev): после проверки расширение добавят в список.

## Клавиши

При зажатом Shift модификаторы Ctrl и Alt переключают меню колеса вживую, не прерывая перетаскивание. Сверху колеса подписаны клавиши Shift / Ctrl / Alt: зажатые подсвечены, а показываются только те, за которыми у этого типа файлов есть меню.

| Клавиши | Меню | У картинок |
|---|---|---|
| Shift | основное | формат: JPG, PNG, HEIC, WEBP, AVIF, JXL, TIFF, GIF, BMP, ICO |
| Ctrl+Shift | второй слот | размер: стандартные разрешения, «≤ сторона», проценты (только уменьшение) |
| Alt+Shift | третий слот | — |
| Ctrl+Alt+Shift | четвёртый слот | — |

Что лежит на каждом слоте, решает расширение под конкретный тип файла. Выборы с разных слотов складываются: можно за один драг задать и формат, и размер.

Отпускать все клавиши одновременно не нужно: если Ctrl или Alt отпущены раньше Shift, колесо остаётся в том меню, что было на экране. В другое меню оно переключится, только если Shift продолжают держать (~0.6 с) или двигают мышь.

## Что происходит с файлом

- Сконвертированная копия кладётся в `%TEMP%\FormatWheel` и отдаётся получателю, как только готова. Пока идёт конвертация, у курсора видна карточка с кольцом прогресса.
- Если бросить файл раньше, чем он готов (например, долгое видео), программа доделает его сама и положит в папку, куда его бросили (окно Проводника или рабочий стол), а если бросили не в папку — рядом с оригиналом. Пока доделывает, у значка программы в трее видна карточка с прогрессом; крестиком конвертацию можно отменить, по готовой карточке — открыть файл в Проводнике.
- Папка копий чистится сама: при запуске и раз в час удаляется всё старше суток и лишнее сверх 2 ГБ. Свежее 10 минут не трогается — получатель мог ещё не забрать файл.

Меню в трее: автозапуск, папка с копиями, журнал (`%TEMP%\FormatWheel\log.txt`), список расширений, папка расширений и их перезагрузка.

## Как написать своё расширение

Расширение — это обычная .NET-библиотека (DLL) на пакете **FormatWheel.Sdk**. Оно может:

- **конвертировать** — реализовать `IConverter`: на колесе лепестки форматов, на выходе новый файл, который тащат дальше;
- **выполнять действия** — реализовать `IWheelActions` (SDK 1.1+): печать, отправка, открыть в программе — что угодно, после чего файл никуда тащить не нужно.

Один класс может делать и то и другое.

### 1. Проект

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0-windows</TargetFramework>
    <!-- имя должно начинаться с FormatWheel. — так программа находит расширения -->
    <AssemblyName>FormatWheel.MyThing</AssemblyName>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="FormatWheel.Sdk" Version="1.2.*" />
  </ItemGroup>
</Project>
```

Пакета `FormatWheel.Sdk` пока нет на nuget.org. Возьмите `FormatWheel.Sdk.x.y.z.nupkg` из [релизов FormatWheel-app](https://github.com/KantraCity/FormatWheel-app/releases) (или соберите: `dotnet pack src/FormatWheel.Sdk`), положите в папку и подключите её как источник пакетов:

```powershell
dotnet nuget add source C:\путь\к\папке -n formatwheel-sdk
```

### 2. Конвертер

```csharp
using FormatWheel.Sdk;

public sealed class MyConverter : IConverter
{
    public string Name => "My thing";
    public bool IsSlow => false;       // true для долгих задач (видео) — см. ниже
    public int MaxParallel => 4;       // сколько файлов конвертировать параллельно

    public bool CanHandle(string path) => path.EndsWith(".foo");

    // лепестки для каждого слота; вернуть пусто, если на этом слоте ничего нет
    public IReadOnlyList<WheelOption> MenuFor(WheelSlot slot, IReadOnlyList<string> files)
    {
        if (slot != WheelSlot.Primary) return Array.Empty<WheelOption>();
        return new[]
        {
            new WheelOption("myaxis", "bar", "BAR", "подсказка"),
            new WheelOption("myaxis", "baz", "BAZ"),
        };
    }

    public string Convert(string source, Recipe recipe, Action<double> report, CancellationToken ct)
    {
        var choice = recipe.Get<string>("myaxis");   // что выбрали на оси myaxis (null — ничего)
        string output = OutputPath.New(source, "out");
        // ... делаете файл, по ходу зовёте report(0..1), проверяете ct ...
        report(1);
        return output;                               // путь к готовому файлу
    }
}
```

### Ключевые понятия

- **Слоты** (`WheelSlot`): `Primary` (Shift), `Ctrl`, `Alt`, `CtrlAlt`. Для каждого возвращаете свои лепестки. Набор может зависеть от конкретных файлов.
- **Оси** (`WheelOption.Axis`): каждый лепесток пишет своё значение в именованную ось рецепта. Разные слоты пишут в разные оси, поэтому их выборы складываются, а не затирают друг друга. `WheelOption.Reset(axis, …)` — лепесток «сбросить эту ось».
- **Рецепт** (`Recipe`): неизменяемый набор «ось → значение». В `Convert` читаете нужные оси: ссылочные значения через `recipe.Get<T>("axis")`, числа и прочее — через `recipe.GetRaw("axis")`.
- **Кто берёт файл:** колесо отдаёт файлы первому конвертеру, у которого `CanHandle` вернул `true` для всех файлов. Встроенные картинки идут первыми, затем расширения по алфавиту имён их папок.
- **Долгие задачи** (`IsSlow = true`): если файл бросили раньше, чем он готов, программа всё равно доделает его и сохранит сама (см. «Что происходит с файлом»). Для быстрых (`false`) — только если конвертация ещё не успела закончиться. Поддерживайте отмену через `CancellationToken`: пользователь мог передумать посреди драга.
- **Сторонние программы** (как FFmpeg или Pandoc): расширение их не несёт. Либо требуйте установки заранее, либо ставьте из своего установщика через winget — так делают расширения FFmpeg и Pandoc.

### 3. Действия (SDK 1.1+, слот — SDK 1.2+)

Когда результат — не файл, а что-то сделанное: напечатать, отправить, загрузить, открыть в нужной программе.

```csharp
using System.Diagnostics;
using FormatWheel.Sdk;

public sealed class PrintActions : IWheelActions
{
    public string Name => "Печать";

    public bool CanHandle(string path) => path.EndsWith(".pdf", StringComparison.OrdinalIgnoreCase);

    public IReadOnlyList<WheelAction> ActionsFor(IReadOnlyList<string> files) => new[]
    {
        // Slot — в каком меню колеса лепесток; без него — основное (Shift)
        new WheelAction("Печать", "на принтер по умолчанию", (paths, report, ct) =>
        {
            for (int i = 0; i < paths.Count; i++)
            {
                ct.ThrowIfCancellationRequested();
                // ... отправляете paths[i] на печать (своим кодом или через свою библиотеку) ...
                report((i + 1.0) / paths.Count);
            }
            return Task.CompletedTask;
        }) { Slot = WheelSlot.Alt },
    };
}
```

Как это работает:

- **Где лепестки:** в том меню колеса, которое указано в `Slot` (SDK 1.2+): `Primary` (Shift, по умолчанию), `Ctrl`, `Alt` или `CtrlAlt`. В слоте действия встают после лепестков конвертера, подписи клавиш на колесе подсвечивают слоты, где что-то есть. Если для файлов есть только действия, а конвертера нет (например, PDF и расширение печати), колесо откроется с одними действиями.
- **Что происходит при выборе:** драг Проводника отменяется — оригинал никуда не уезжает, а действие выполняется в фоне. У значка программы в трее видна карточка с прогрессом (`report(0..1)`) и крестиком отмены (`ct`). Исключение из `Run` — красная карточка с его текстом.
- **Поток:** `Run` вызывается в отдельном STA-потоке, поэтому из него можно показывать окна — например, свой диалог печати на WinForms или WPF.
- **Файлы:** в `Run` приходят оригиналы, которые тащили. Если нужно сначала сконвертировать (Word → PDF → печать), сделайте это внутри действия.
- **Кто показывает действия:** все расширения, у которых `CanHandle` вернул `true` для всех файлов, — их действия складываются в одно меню.
- **Совместимость:** действия появились в SDK 1.1 (программа 1.0.7+), выбор слота — в SDK 1.2 (программа 1.0.8+). Расширения на более старом SDK продолжают работать как раньше.

### 4. Установщик

Соберите DLL со всеми зависимостями, но без `FormatWheel.Sdk.dll` (она уже есть в программе). Каждое расширение — в своей подпапке:

- `%LOCALAPPDATA%\Programs\FormatWheel\extensions\<ваше-расширение>\` — папка рядом с программой, сюда ставят установщики. Расширения отсюда считаются доверенными.
- `%APPDATA%\FormatWheel\extensions\<ваше-расширение>\` — для расширений, положенных вручную. Они включаются только после подтверждения в диалоге.

Удобнее всего сделать Inno Setup установщик, который копирует файлы в первую папку и ставит нужные зависимости. Пример — в репозиториях расширений (`installer/` и `build.ps1`).

### Безопасность

Расширение — это чужой код, который выполняется внутри программы. Поэтому расширения из `%APPDATA%\FormatWheel\extensions` включаются только после подтверждения, а если DLL обновилась, программа спросит разрешение заново. Ставьте только расширения, которым доверяете.

---

## English

Convert files on the fly while you drag them — a Windows take on the macOS “format wheel”. Images are built in; video, audio, documents, data and anything else come as **extensions**, and anyone can write one.

- **Install:** grab the installer from [FormatWheel-app releases](https://github.com/KantraCity/FormatWheel-app/releases). Windows 10 (1809) or 11, x64. Don’t run it elevated.
- **Keys:** hold Shift for the wheel; Ctrl and Alt switch between up to four menus live, and each file type decides what sits on each slot. The keys you hold are highlighted on top of the wheel.
- **The file:** the converted copy goes to `%TEMP%\FormatWheel` and is handed over as soon as it is ready. Dropped too early, it is finished and saved into the folder you dropped on (or next to the original). Old copies are cleaned up automatically.
- **Write an extension:** a .NET DLL named `FormatWheel.*` on **FormatWheel.Sdk** (the `.nupkg` is in the FormatWheel-app releases). Implement `IConverter` to turn a file into another file, or `IWheelActions` (SDK 1.1+) for actions such as printing: each action goes to the wheel menu you pick with `Slot` (SDK 1.2+, Shift by default), the Explorer drag is cancelled and the action runs in the background with progress and a cancel button next to the tray icon. Installers put extensions into `%LOCALAPPDATA%\Programs\FormatWheel\extensions\<name>\` (trusted); hand-dropped ones go to `%APPDATA%\FormatWheel\extensions\<name>\` and need confirmation. See the C# example above and the extension repositories for full templates.

Verified extensions:

| Extension | Files | Installer |
|---|---|---|
| **FFmpeg** | video and audio | [releases](https://github.com/KantraCity/FormatWheel-ffmpeg/releases) |
| **Audio** | audio via Windows codecs, no FFmpeg needed | [releases](https://github.com/KantraCity/FormatWheel-audio/releases) |
| **Data** | JSON, YAML, XML, TOML, CSV | [releases](https://github.com/KantraCity/FormatWheel-data/releases) |
| **Pandoc** | documents: DOCX, PDF, ODT, RTF, HTML, Markdown, EPUB, PPTX | [releases](https://github.com/KantraCity/FormatWheel-pandoc/releases) |

Want your extension in the verified list? Message [@kantradev](https://t.me/kantradev) on Telegram — once it's reviewed, it will be added.

License: MIT.
