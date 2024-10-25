

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



series - number - is_active



 **passports**

| `[idx]`series | number | is_active |
| :------------ | ------ | --------- |
| 4543          | 564323 | true      |
| 7512          | 873452 | false     |



 **changelog**

| `[idx]`date_of_change | series | number | is_active |      |
| --------------------- | :----- | ------ | --------- | ---- |
| 20240718              | 4543   | 564323 | true      |      |
| 20240719              | 4543   | 564323 | false     |      |

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
	int data = (int)inputDate
	SELECT * FROM changelog WHERE (date_of_change = date)
}
```



 API с помощью которого можно будет: 

сохранять изменения данных (дата изменения, какие паспорта были удалены, какие паспорта были добавлены). 



Необходимо реализовать сервис, который будет раз в день (в настраиваемое через конфиг время) получать эти данные и обновлять их у себя, а также сохранять изменения данных (дата изменения, какие паспорта были удалены, какие паспорта были добавлены).

- Временная таблица
- Транзакция
- Батчи 
- Составной индекс



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

