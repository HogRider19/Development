Внешний вид приложения является нам преимущественно через формы. Формы являются основными строительными блоками. Они предоставляют контейнер для различных элементов управления. А механизм событий позволяет элементам формы отзываться на ввод пользователя, и, таким образом, c ним взаимодействовать.

Зачастую форма в приложении представлена тремя файлами: двумя файлами с `partial`классом самой формы, при это один из них содержит автогенерируемый код, который определяет метод `InitializeComponent`, который вызывается в конструкторе формы. И также определяется файл с ресурсами, которыми форма пользуется при выполнении.

---

Форма наследует ряд свойств, которые позволяют управлять состоянием формы.  Среда определяет специальное окно `Propertis` для их начальной настройки:

Name - устанавливает имя класса, который наследуется от класса `Form`.
    
BackColor -устанавливает фоновый цвет формы
    
BackgroundImage - указывает на фоновое изображение формы
    
- BackgroundImageLayout: определяет, как изображение, заданное в свойстве BackgroundImage, будет располагаться на форме.
    
- ControlBox: указывает, отображается ли меню формы. В данном случае под меню понимается меню самого верхнего уровня, где находятся иконка приложения, заголовок формы, а также кнопки минимизации формы и крестик. Если данное свойство имеет значение false, то мы не увидим ни иконку, ни крестика, с помощью которого обычно закрывается форма
    
- Cursor: определяет тип курсора, который используется на форме
    
- Enabled: если данное свойство имеет значение false, то она не сможет получать ввод от пользователя, то есть мы не сможем нажать на кнопки, ввести текст в текстовые поля и т.д.
    
- Font: задает шрифт для всей формы и всех помещенных на нее элементов управления. Однако, задав у элементов формы свой шрифт, мы можем тем самым переопределить его
    
- ForeColor: цвет шрифта на форме
    
- FormBorderStyle: указывает, как будет отображаться граница формы и строка заголовка. Устанавливая данное свойство в None можно создавать внешний вид приложения произвольной формы
    
- HelpButton: указывает, отображается ли кнопка справки формы
    
- Icon: задает иконку формы
    
- Location: определяет положение по отношению к верхнему левому углу экрана, если для свойства `StartPosition` установлено значение `Manual`
    
- MaximizeBox: указывает, будет ли доступна кнопка максимизации окна в заголовке формы
    
- MinimizeBox: указывает, будет ли доступна кнопка минимизации окна
    
- MaximumSize: задает максимальный размер формы
    
- MinimumSize: задает минимальный размер формы
    
- Opacity: задает прозрачность формы
    
- Size: определяет начальный размер формы
    
- StartPosition: указывает на начальную позицию, с которой форма появляется на экране
    
- Text: определяет заголовок формы
    
- TopMost: если данное свойство имеет значение `true`, то форма всегда будет находиться поверх других окон
    
- Visible: видима ли форма, если мы хотим скрыть форму от пользователя, то можем задать данному свойству значение `false`
    
- WindowState: указывает, в каком состоянии форма будет находиться при запуске: в нормальном, максимизированном или минимизированном