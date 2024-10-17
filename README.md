# dotnet-resources

- Cancellarion Token





















1. **EF** не любит, когда вы обновляете данные из другого потока, нежели тот, из которого создан контекст, 

2. если вы хотите использовать `Parallel.ForEach`, этот метод должен создать экземпляр EF Core на поток (а не только один на процесс).

   ```csharp
   Parallel.ForEach(itemList, (item) =>
   {
      using (var context = new AppDbContext())
      {
           //operations
      }
   });
   ```

