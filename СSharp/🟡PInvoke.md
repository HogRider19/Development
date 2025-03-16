Механизм `PInvoke` позволяет управляемому коду вызывать неуправляемые функции, реализованные в динамически подключаемой библиотеке или внешней сборке.

Это позволяет разработчикам `C#` использовать существующие библиотеки нативного кода или системные функции в своих приложениях без переписывания существующего кода.

Чтобы воспользоваться механизмом `PlatformInvoke`, управляемый код должен объявить `static extern` метод с сигнатурой, эквивалентной функции на языке `C`. Сам метод должен быть снабжен атрибутом `DllImport`, определяющим, как минимум, имя `DLL` библиотеки.

```c#
[DllImport("kernel32.dll", SetLastError = true)]
public static extern IntPtr LoadLibrary(string fileName);
```

Если сигнатура библиотечной функции содержит составные типы, такие как структуры
языка `C`, в управляемом коде необходимо определить эквивалентные структуры или классы, используя эквивалентные типы для каждого поля. Порядок следования полей, типы полей и применяемые к ним правила выравнивания должны совпадать с ожидаемыми в коде. 

```c#
[StructLayout(LayoutKind.Sequential, CharSet = CharSet.Auto)]
struct WIN32_FIND_DATA
{
    public uint dwFileAttributes;
    public FILETIME ftCreationTime;
    public FILETIME ftLastAccessTime;
    public FILETIME ftLastWriteTime;
    public uint nFileSizeHigh;
    public uint nFileSizeLow;
    public uint dwReserved0;
    public uint dwReserved1;

    [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 260)]
    public string cFileName;

    [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 14)]
    public string cAlternateFileName;
}
```

При первом вызове функции с помощью механизма `PInvoke`, обращением к функции `LoadLibrary` производится загрузка библиотеки `DLL` и всех ее зависимостей в адресное пространство процесса. Затем выполняется поиск требуемой экспортируемой функции,
при этом сначала предпринимается попытка найти управляемую версию функции.

[[СSharp/🟡Base|🟡Base]]