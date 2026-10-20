# Format Wheel

Конвертация файлов прямо на лету, пока вы их перетаскиваете. Аналог «колеса форматов» с macOS для Windows.

Тянете файл → зажимаете **Shift** → вокруг курсора раскрывается колесо → наводите на нужный формат → отпускаете Shift и тянете дальше уже сконвертированный файл: в чат, браузер, Figma, другую папку. Оригинал не трогается.

Картинки программа конвертирует сама. Всё остальное — видео, аудио, документы — добавляется **расширениями**.

*English below.*

## Репозитории

| Репозиторий | Что внутри |
|---|---|
| **FormatWheel** (этот) | Описание, список расширений, инструкция по написанию своих |
| [FormatWheel-app](https://github.com/KantraCity/FormatWheel-app) | Код программы и SDK для расширений |
| [FormatWheel-ffmpeg](https://github.com/KantraCity/FormatWheel-ffmpeg) | Расширение для видео и аудио (FFmpeg) |

## Установка

1. Скачайте и запустите `FormatWheel-Setup-x.y.z.exe` из релизов [FormatWheel-app](https://github.com/KantraCity/FormatWheel-app/releases).
2. По желанию поставьте расширения из списка ниже.

Права администратора не нужны, и запускать программу от администратора нельзя: Windows не даёт перетаскивать файлы из обычного Проводника в процессы с повышенными правами.

## Проверенные расширения

| Расширение | Файлы | Установщик |
|---|---|---|
| **FFmpeg** | видео (MP4, HEVC, MKV, WEBM, AVI, GIF…) и аудио (MP3, FLAC, OPUS…) | [релизы](https://github.com/KantraCity/FormatWheel-ffmpeg/releases) |
| **Аудио** | MP3, M4A, OPUS, FLAC, ALAC, WMA, WAV, AIFF без FFmpeg; битрейт, моно, выравнивание громкости | [релизы](https://github.com/KantraCity/FormatWheel-audio/releases) |
| **Данные** | JSON, YAML, XML, TOML, CSV — в любую сторону | [релизы](https://github.com/KantraCity/FormatWheel-data/releases) |
| **Pandoc** | документы: DOCX, PDF, ODT, RTF, HTML, Markdown, EPUB, PPTX… | [релизы](https://github.com/KantraCity/FormatWheel-pandoc/releases) |

Хотите, чтобы ваше расширение попало в список проверенных? Напишите в Telegram [@kantradev](https://t.me/kantradev): после проверки расширение добавят в список.

## Клавиши

При зажатом Shift модификаторы Ctrl и Alt переключают меню колеса вживую, не прерывая перетаскивание:

| Клавиши | Меню |
|---|---|
| Shift | основное (форматы) |
| Ctrl+Shift | второй слот (у картинок — размеры) |
| Alt+Shift | третий слот |
| Ctrl+Alt+Shift | четвёртый слот |

Что лежит на каждом слоте, решает расширение под конкретный тип файла. Выборы с разных слотов складываются: можно за один драг задать и формат, и размер.

## Как написать своё расширение

Расширение — это обычная .NET-библиотека (DLL), которая реализует интерфейс `IConverter` из пакета **FormatWheel.Sdk**.

### 1. Проект

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0-windows</TargetFramework>
    <!-- имя должно начинаться с FormatWheel. — так программа находит расширения -->
    <AssemblyName>FormatWheel.MyThing</AssemblyName>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="FormatWheel.Sdk" Version="1.0.*" />
  </ItemGroup>
</Project>
```

### 2. Класс

```csharp
using FormatWheel.Sdk;

public sealed class MyConverter : IConverter
{
    public string Name => "My thing";
    public bool IsSlow => false;       // true для долгих задач (видео): файл отдаётся, когда готов
    public int MaxParallel => 4;

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
        var choice = recipe.Get<string>("myaxis");   // что выбрали на оси myaxis
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
- **Рецепт** (`Recipe`): неизменяемый набор «ось → значение». В `Convert` читаете нужные оси через `recipe.Get<T>("axis")`.
- **Долгие задачи** (`IsSlow = true`): пока файл не готов, программа говорит получателю, что файлов нет, чтобы не вешать его. Поддерживайте отмену через `CancellationToken` — пользователь мог передумать посреди драга.
- **Сторонние программы** (как FFmpeg): расширение их не несёт. Либо требуйте установки заранее, либо ставьте из своего установщика через winget — так делает FFmpeg-расширение.

### 3. Установщик

Соберите DLL со всеми зависимостями (но без `FormatWheel.Sdk.dll` — она уже есть в программе) и положите в отдельную подпапку здесь:

```
%LOCALAPPDATA%\Programs\FormatWheel\extensions\<ваше-расширение>\
```

Удобнее всего сделать Inno Setup установщик, который туда копирует файлы и ставит нужные зависимости. Пример — в репозитории FormatWheel-ffmpeg.

### Безопасность

Расширение — это чужой код, который выполняется внутри программы. Поэтому расширения из пользовательской папки включаются только после подтверждения в диалоге (встроенные, лежащие рядом с exe, доверенные). Если обновить DLL, программа спросит разрешение заново. Ставьте только расширения, которым доверяете.

---

## English

Convert files on the fly while you drag them — a Windows take on the macOS “format wheel”. Images are built in; video, audio and anything else come as **extensions**.

- **Install:** grab the installer from [FormatWheel-app releases](https://github.com/KantraCity/FormatWheel-app/releases). Don’t run it elevated.
- **Keys:** hold Shift for the wheel; Ctrl and Alt switch between up to four menus live, and each file type decides what sits on each slot.
- **Write an extension:** a .NET DLL named `FormatWheel.*` that implements `IConverter` from the **FormatWheel.Sdk** NuGet package, dropped into `%LOCALAPPDATA%\Programs\FormatWheel\extensions\`. See the C# example above and the FFmpeg extension for a full template.

Verified extensions:

| Extension | Files | Installer |
|---|---|---|
| **FFmpeg** | video and audio | [releases](https://github.com/KantraCity/FormatWheel-ffmpeg/releases) |
| **Audio** | audio via Windows codecs, no FFmpeg needed | [releases](https://github.com/KantraCity/FormatWheel-audio/releases) |
| **Data** | JSON, YAML, XML, TOML, CSV | [releases](https://github.com/KantraCity/FormatWheel-data/releases) |
| **Pandoc** | documents: DOCX, PDF, ODT, RTF, HTML, Markdown, EPUB… | [releases](https://github.com/KantraCity/FormatWheel-pandoc/releases) |

Want your extension in the verified list? Message [@kantradev](https://t.me/kantradev) on Telegram — once it's reviewed, it will be added.

License: MIT.
