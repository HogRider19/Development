Компоновка представляет собой процесс размещения элементов внутри контейнера. 
Многие программы используют жестко закодированные в коде размеры элементов управления. `WPF` уходит от такого подхода в пользу так называемого резинового дизайна, где весь процесс позиционирования осуществляется компоновкой.

Благодаря компоновке мы можем удобным образом настроить элементы интерфейса, позиционировать их определенным образом. Например, элементы компоновки в `WPF` позволяют при смене размера, сжатии или растяжении масштабировать элементы, что очень удобно, а визуально не создает аномалий незаполненных пустот на форме.

Компоновка осуществляется при помощи специальных контейнеров. Фреймворк предоставляет: `Grid`, `UniformGrid`, `StackPanel`, `WrapPanel`, `DockPanel`, `Canvas`.

Различные контейнеры могут содержать внутри себя другие контейнеры. Кроме данных контейнеров существует еще ряд элементов, такие как `TabPanel`, которые могут включать другие элементы и даже контейнеры компоновки. Кроме того, если нам не хватает стандартных контейнеров, мы можем определить пользовательский.

Все контейнеры компоновки наследуются от абстрактного класса `Panel`,                        а само дерево наследования можно представить следующим образом:

![[Pasted image 20240511161046.png]]

**==Grid==**. Это наиболее мощный и часто используемый контейнер, напоминающий обычную таблицу. Он содержит столбцы и строки, количество которых задает разработчик. Для определения строк используется свойство `RowDefinitions`,                а для определения столбцов используется свойство `ColumnDefinitions`.

Чтобы задать позицию элемента управления с привязкой к определенной ячейке, в разметке элемента нужно прописать значения свойств `Grid.Column` и` Grid.Row`

Кроме того, если мы хотим растянуть элемент управления на несколько строк или столбцов, то можно указать свойства `Grid.ColumnSpan` и `Grid.RowSpan`.

```xml
<Grid ShowGridLines="True">
	<Grid.RowDefinitions>
		<RowDefinition></RowDefinition>
		<RowDefinition></RowDefinition>
	</Grid.RowDefinitions>
	<Grid.ColumnDefinitions>
		<ColumnDefinition></ColumnDefinition>
		<ColumnDefinition></ColumnDefinition>
	</Grid.ColumnDefinitions>

	<Button Grid.Row="1" Grid.Column="0" x:Name="name" Content="Click" />
</Grid>
```

Элементы строк и столбцов по умолчанию автоматический настраивают свои размеры так чтобы заполнить все доступное пространство, но это поведение можно изменить.

```xml
<ColumnDefinition Width="150" />
<RowDefinition Height="150" />

<ColumnDefinition Width="*" />     // Заполнить оставшееся 
<ColumnDefinition Width="0.25*" /> // Четверть от заполняемого
```

Если строка или столбец имеет высоту, равную `*`, то данная строка или столбце будет занимать все оставшееся место. Если у нас есть несколько сток или столбцов, высота которых равна `*`, то все доступное место делится поровну между ними.

**==UniformGrid==**. Контейнер `UniformGrid` представляет собой контейнер аналогичный контейнеру `Grid`, только в этом случае все столбцы и строки одинакового размера и для их определения можно использовать упрощенный синтаксис:

```xml
<UniformGrid Rows="2" Columns="2">
    <Button Content="Left Top" />
    <Button Content="Right Top" />
    <Button Content="Left Bottom" />
    <Button Content="Right Bottom" />
</UniformGrid>
```

**==GridSplitter==**. Представляет некоторый разделитель между столбцами или строками, путем сдвига которого можно регулировать ширину столбцов и высоту строк.

Чтобы использовать элемент `GridSplitter`, нам надо поместить его в ячейку `Gride`. По сути это обычный элемент, такой же, как кнопка. Обычно строка или столбец, в которые помещают элемент, имеет для свойств `Height` или `Width` значение `Auto`.

Если у нас несколько строк, и мы хотим, чтобы разделитель распространялся на несколько строк, то мы можем задать свойство `Grid.RowSpan` у объекта `GridSplitter`.

```xml
<Grid>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="*" />
        <ColumnDefinition Width="Auto" />
        <ColumnDefinition Width="*" />
    </Grid.ColumnDefinitions>
    <Button Grid.Column="0" Content="Левая кнопка" />
    <GridSplitter Grid.Column="1" ShowsPreview="False" Width="3"
        HorizontalAlignment="Center" VerticalAlignment="Stretch" />
    <Button Grid.Column="2" Content="Правая кнопка" />
</Grid>
```

**==StackPanel==**. Представляет более простой элемент компоновки. Он располагает все элементы в ряд либо по горизонтали, либо по вертикали в зависимости от ориентации.

Свойство `Orientation` по умолчанию используется значение `Vertical`, то есть `StackPanel` создает вертикальный ряд, в который помещает элементы сверху вниз. 
Мы также можем задать горизонтальный стек `Orientation="Horizontal".

При горизонтальной ориентации все вложенные элементы располагаются слева направо. Если мы хотим  справа налево, то нам надо задать свойство `FlowDirection`.

```xml
<StackPanel Orientation="Vertical">
	<Button Background="Blue" Content="1" />
	<Button Background="White" Content="2" />
	<Button Background="Red" Content="3" />
</StackPanel>
```

**==WrapPanel==**. Эта панель, подобно `StackPanel`, располагает все элементы в одной строке или колонке в зависимости от того, какое значение имеет свойство Orientation. Главное отличие от `StackPanel` в том, что если элементы не помещаются в строке или столбце, создаются новые столбец или строка для не поместившихся элементов контейнера.

```xml
<WrapPanel Orientation="Vertical">
	<Button Background="Blue" Content="1" />
	<Button Background="White" Content="2" />
	<Button Background="Red" Content="3" />
</WrapPanel>
```

**==DockPanel==**. Этот контейнер прижимает свое содержимое к определенной стороне внешнего контейнера. Для этого у вложенных элементов надо установить сторону, к которой они будут прижиматься с помощью свойства `DockPanel.Dock`.

```xml
    <DockPanel LastChildFill="True">
        <Button DockPanel.Dock="Top" />
        <Button DockPanel.Dock="Bottom" />
        <Button DockPanel.Dock="Left" />
        <Button DockPanel.Dock="Right" />
        <Button Background="LightGreen" />
    </DockPanel>
```

Причем у последней кнопки мы можем не устанавливать свойство `DockPanel.Dock`. Она уже заполняет все оставшееся пространство. Такой эффект получается благодаря  `LastChildFill="True"`, которое заполняет оставшееся место последним элементом.

**==Canvas==**. Является наиболее простым контейнером. Для размещения на нем необходимо указать для элементов точные координаты относительно его сторон. Для установки координат элементов используются свойства `Left`, `Right`, `Bottom`, `Top`. 

При этом в качестве единиц используются не пиксели, а независимые от устройства единицы, которые помогают эффективно управлять масштабированием элементов. Каждая такая единица равна `1 /96` дюйма, и при стандартной установке в `96` эта независимая от устройства единица будет равна физическому пикселю.

```xml
<Canvas Background="Lavender">
	<Button Content="Top 20 Left 40" Canvas.Top="20" Canvas.Left="40" />
	<Button Content="Top 20 Right 20" Canvas.Top="20" Canvas.Right="20"/>
</Canvas>
```

---

Элементы платформы `WPF` обладают набором свойств, которые помогают позиционировать данные элементы. Рассмотрим некоторые из этих свойств.

У элемента можно установить ширину используя `Width` и высоту с помощью `Height`. Эти свойства принимают значение типа `double`. Хотя общая рекомендация состоит в том, что желательно избегать жестко закодированных в коде ширины и высоты.

Также мы можем задать возможный диапазон ширины и высоты с помощью свойств `MinWidth/MaxWidth` и `MinHeight/MaxHeight`. И при растяжении контейнеров элементы с данными заданными свойствами не будут выходить за пределы этих значений.

Свойство `HorizontalAlignment` выравнивает элемент по горизонтали относительно правой или левой стороны контейнера и соответственно может принимать значения `Left`, `Right`, `Center`, а также значения `Stretch` (растяжение по всей ширине).

Свойство `Margin` устанавливает отступы вокруг элемента, для этого в значение свойства предаются значения отступов через пробел по часовой стрелке от левого.

При создании интерфейса возможна ситуация, когда одни элементы будут полностью или частично перекрывать другие. По умолчанию те элементы, которые определены позже, перекрывают те элементы, которые определены ранее. Однако мы можем изменить подобное действие с помощью свойства `Panel.ZIndex`.

По умолчанию для всех создаваемых элементов `Panel.ZIndex="0"`. Однако назначив данному свойству более высокое значение, мы можем передвинуть на передний план. 