## Задачка, которую я обещал

Есть источник данных о неактивных паспортах из МВД РФ (заархивированный csv-файл, который можно скачать по ссылке. В каждой строке находится серия и номер неактивного паспорта). 
 Необходимо реализовать сервис, который будет раз в день (в настраиваемое через конфиг время) получать эти данные и обновлять их у себя, а также сохранять изменения данных (дата изменения, какие паспорта были удалены, какие паспорта были добавлены).
В данном сервисе необходимо реализовать API с помощью которого можно будет: 
●    искать неактивные паспорта (по серии и номеру);
●    получать все изменения по паспортам за передаваемую дату;
●    получать историю по активности и неактивности паспорта (по серии и номеру).

Требования:
● основная логика работы должна быть покрыта тестами;
● сервис должен быть доступен к разворачиванию через docker-compose(dockerhub);
● запрос к API не должен выполняться больше 0.5 секунды;
● в репозитории должны быть настроены события по запуску тестов на каждый merge\pull реквест.

#### Архитектура

##### Оптимизация

Postres cache





##### Задачи

- Валидация
  - на корректность 
  - на количество символов
  - на повтор
  - 
- Роли
  - аутентификация

- Логи
  - Запущен background service
  - Завершен.13.04.2025 1200 паспортов изменились значения на true

- Документация API



Контроллер

Get(series,number)

Get (date,date)

Get(series,number)

## Архитектура

 **passports**

| `[idx]`series | number | is_active |
| :------------ | ------ | --------- |
| 4543          | 564323 | true      |
| 7512          | 873452 | false     |



 **changelog**

| `[idx]`date_of_change | `[idx]` series | number | is_active |
| --------------------- | :------------- | ------ | --------- |
| 20240718              | 4543           | 564323 | true      |
| 20240721              | 4543           | 564323 | false     |

```sql
-- искать неактивные паспорта (по серии и номеру);
Passport ActivityCheck(Passport passport)
{
	SELECT is_active FROM passports 
	(passport.series = series AND passport.number = number)
	
	return passport;
	if(passport.is_active){"Паспорт активен!"}
	
	/* - попробовать без создания экземпляра
    
    */
}

-- получать историю по активности и неактивности паспорта (по серии и номеру)
ActivityChangeLog(Passport passport)
{
	SELECT * FROM changelog WHERE (passport.series = series AND passport.number = number)
}


-- получать все изменения по паспортам за передаваемую дату;
DateChangeReport(DateTime inputDate, List<DateTime> inputDateInterval)
{
	foreach(var inputDate in inputDateInterval)
	int date = (int)inputDate
	SELECT * FROM changelog WHERE (date_of_change = date)
}
```



Чтение файла:

```c#
public async Task ProcessPassportsAsync(string filePath)
{
   	var readBlock = new TransformBlock<string, (int passpSeries, int passpNumber)?>(
            line =>
            {
                var values = line.Split(',');
                if (values[0].Trim().Length == 4 && values[1].Trim().Length == 6)
                {
                    if (int.TryParse(values[0], out var passpSeries) &&
                        int.TryParse(values[1], out var passpNumber))
                    {
                        return (passpSeries, passpNumber);
                    }
                }
                return null;
                
            }, new ExecutionDataflowBlockOptions { MaxDegreeOfParallelism = Environment.ProcessorCount });
 
        var batchBlock = new BatchBlock<(int passpSeries, int passpNumber)?>(BatchSize);
 
        var filterBlock =
            new TransformBlock<(int passpSeries, int passpNumber)?[], (int passpSeries, int passpNumber)[]>(
                batch => { return batch.Where(x => x.HasValue).Select(x => x.Value).ToArray(); });
 
        var insertBlock = new ActionBlock<(int passpSeries, int passpNumber)[]>(async batch =>
        {
            if (batch.Length > 0)
            {
                await InsertBatchAsync(batch);
            }
        }, new ExecutionDataflowBlockOptions { MaxDegreeOfParallelism = 6 });
 
        readBlock.LinkTo(batchBlock);
        batchBlock.LinkTo(filterBlock, new DataflowLinkOptions { PropagateCompletion = true });
        filterBlock.LinkTo(insertBlock, new DataflowLinkOptions { PropagateCompletion = true });
 
        using (var reader = new StreamReader(filePath))
        {
            while (!reader.EndOfStream)
            {
                var line = await reader.ReadLineAsync();
                if (!string.IsNullOrWhiteSpace(line))
                {
                    await readBlock.SendAsync(line);
                }
            }
        }
 
        readBlock.Complete();
        await readBlock.Completion;
        batchBlock.Complete();
        await batchBlock.Completion;
        await filterBlock.Completion;
        await insertBlock.Completion;

}

//Отправка батча  : 


private async Task InsertBatchAsync((int passpSeries, int passpNumber)[] batch)
        {
            using (var dbContext = await _dbContextFactory.CreateDbContextAsync())
            using (NpgsqlConnection conn = new (dbContext.Database.GetDbConnection().ConnectionString))
            {
                await conn.OpenAsync();
                
                
                //если серия уникальная, то сразу добавляем в БД со значением is_active = false
                //если серия нашлась, ищем по номеру
                //если нашелся и номер,  меняем is_active = false
                //если номер уникальный, то сразу добавляем в БД со значением is_active = false
                
                if(series && number) {is_active = true}//и

                    
                //после чтения csv все не встречающиеся в нем паспорта меняют is_active = true
                //
                //
                using (var writer = conn.BeginBinaryImport("COPY passports (passp_series, passp_number) FROM STDIN (FORMAT BINARY)"))
                {
                    foreach (var item in batch)
                    {
                        if (item.passpSeries != 0 && item.passpNumber != 0)
                        {
                            writer.StartRow();
                            writer.Write(item.passpSeries);
                            writer.Write(item.passpNumber);
                        }
                    }
                    writer.Complete();

                    
                }
                
            }
        }


```

### API

- Временная таблица
- Транзакция
- Батчи 
- Составной индекс

1. Валидация

2. Загрузка из csv во временную таблицу, сравнение и заполнение

3. https://dev.to/0xog_pg/postgres-temporary-tables-a-guide-to-data-manipulation-2c7d#:~:text=A%20temporary%20table%20in%20Postgres,transaction%2C%20depending%20on%20their%20scope.

4. Сравнение по серии и номеру

5. Таблица "Дата"

6. Background https://code-maze.com/aspnetcore-different-ways-to-run-background-tasks/

   ```c#
   1. public class SuperPars
   
        {
   
      ​    private readonly string _connectionString;
   
      ​    public SuperPars()
   
      ​    {
   
      ​      _connectionString = "server=localhost;username=postgres;database=passports_db;password=postgres";
   
      ​    }
   
      public async Task ProcessCsvData(string filePath)
   
      ​    {
   
      ​      using var connection = new NpgsqlConnection(_connectionString);
   
      ​      await connection.OpenAsync();
   
      ​      // Создаем временную таблицу
   
      ​      await connection.ExecuteAsync("CREATE TEMP TABLE temp_passports (series INT, number INT)");
   
      ​      // Импортируем данные из CSV-файла в временную таблицу
   
      ​      using (var writer = connection.BeginBinaryImport("COPY temp_passports (series, number) FROM STDIN (FORMAT BINARY)"))
   
      ​      {
   
      ​        using (var reader = new StreamReader(filePath))
   
      ​        {
   
      ​          while (!reader.EndOfStream)
   
      ​          {
   
      ​            var line = await reader.ReadLineAsync();
   
      ​            if(string.IsNullOrEmpty(line))
   
      ​              continue;
   
      ​            var parts = line.Split(',', StringSplitOptions.TrimEntries);
   
      ​            if(parts.Length != 2 || parts[0].Length != 4 || parts[1].Length != 6)
   
      ​              continue;
   
      ​            if (int.TryParse(parts[0], out int series) && int.TryParse(parts[1], out int number))
   
      ​            {
   
      ​              await writer.StartRowAsync();
   
      ​              writer.Write(series, NpgsqlTypes.NpgsqlDbType.Integer);
   
      ​              writer.Write(number, NpgsqlTypes.NpgsqlDbType.Integer);
   
      ​            }
   
      ​          }
   
      ​        }
   
      ​        await writer.CompleteAsync();
   
      ​      }
   
      ​      // Добавляем отсутствующие записи в основную таблицу
   
      ​      var insertQuery = @"
   
      ​      INSERT INTO passports (series, number, isactive)
   
      ​      SELECT series, number, TRUE
   
      ​      FROM temp_passports t
   
      ​      LEFT JOIN passports p ON p.series = t.series AND p.number = t.number
   
      ​      WHERE p.series IS NULL AND p.number IS NULL";
   
      ​      await connection.ExecuteAsync(insertQuery);
   
      ​      // Обновляем флаг isactive для записей, которых нет в CSV
   
      ​      var updateQuery = @"
   
      ​      UPDATE passports
   
      ​      SET isactive = TRUE
   
      ​      WHERE (series, number) NOT IN (SELECT series, number FROM temp_passports)";
   
      ​      await connection.ExecuteAsync(updateQuery);
   
      ​      // Удаляем временную таблицу
   
      ​      await connection.ExecuteAsync("DROP TABLE IF EXISTS temp_passports");
   
      ​    }
   
        }
   
   
   ```

   

CREATE INDEX idx_series ON passports(passp_series)
