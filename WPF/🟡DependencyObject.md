Свойства элементов `wpf` являются не просто стандартными свойствами языка `C#`. Они фактически скрывают свойства зависимостей или `dependency property`. Без свойств зависимостей были бы невозможны многие ключевые особенности `WPF`.

По соглашениям по именованию все свойства зависимостей представляют  статические публичные поля `(public static)` с суффиксом `Property`.

Регистрация свойства зависимости происходит с использованием метода `DependencyProperty.Register()`, в который передается ряд параметров: 
Имя свойства; Тип свойства; Тип владельца; Метаданные; Валидация.

```c#
public class TextBlock : FrameworkElement 
{
    public static readonly DependencyProperty TextProperty;
 
    static TextBlock()
    {
        TextProperty = DependencyProperty.Register(
			"Text", typeof(string), typeof(TextBlock),
			new FrameworkPropertyMetadata
			(
				string.Empty, 
				FrameworkPropertyMetadataOptions.AffectsMeasure |
				FrameworkPropertyMetadataOptions.AffectsRender, 
				new PropertyChangedCallback(OnTextChanged), 
				new CoerceValueCallback(CoerceText))
			);
    }
 
    public string Text
    { 
        get { return (string) GetValue(TextProperty); } 
        set { SetValue(TextProperty, value); }
    }  
     
    private static object CoerceText(DependencyObject d, object value)
    {
	    //...............................
    }
    private static void OnTextChanged(DependencyObject d, 
	    DependencyPropertyChangedEventArgs e)
    { 
        //...............................
    }
}
```