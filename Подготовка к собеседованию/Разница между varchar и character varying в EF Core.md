```csharp
b.Property<string>("Name")
    .IsRequired()
    .HasMaxLength(200)
    .HasColumnType("character varying(200)");
```

