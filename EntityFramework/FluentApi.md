**modelBuilder.Entity\<Type>()** - получение объекта к которому будет применятся конфигурация fluentApi

---

**enti.ToTable("name")** - задает настройки таблицы  
**enti.HasKey(t => t.InstructorID)** - задает первичный ключ  
**enti.Ignore(t => t.field)** - игнорирует поле при сопоставлении  
**enti.HasData()** - инициализация начальными значениями

---

**entity.Property(t => t.DepartmentID)** - получение конкретного поля  
**prop.HasDatabaseGeneratedOption()** - задает генерацию  
**prop.HasMaxLength(50)** - задает максимальную длину  
**prop.IsRequired()** - делает поле обязательным  
**prop.HasColumnAnnotation(name, obj)** - применяет аннотацию к полю  
**prop.HasColumnName("name")** - задает имя столбцу  
**prop.HasColumnType("varchar")** - задает тип данных для колонки  
**prop.IsConcurrencyToken()** - устанавливает токен конкурентности

---

**rel.OnDelete(DeleteBehavior)** - устанавливает параметры удаления  
**rel.HasForeignKey()** - устанавливает внешний ключ

**entity.(HasOne()/HasMany())** - устанавливают навигационное свойство для сущности, для которой производится конфигурация.  
**rel.(WithOne()/WithMany())** - идентифицируют навигационное свойство на стороне связанной сущности