1. Если вы хотите использовать `Parallel.ForEach`, этот метод должен создать экземпляр EF Core на поток (а не только один на процесс).

   ```csharp
   Parallel.ForEach(itemList, (item) =>
   {
      using (var context = new AppDbContext())
      {
           //operations
      }
   });
   ```

