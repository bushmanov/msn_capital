# MSN_Capital

1. Постановка задачи: файл `Insider_traiding.pdf`. 
Задача классификации на сбалансированной истории сделок инсайдеров. Поля тестового датасета:
  - `idx` - id эмитента
  - `date` - дата репорта
  - инсайдерская активность (покупки, продажи, процент инсайдеров в общей массе сделок покупки/продажи за последние 1, 3, 6,12 месяцев)

2. Данные имеют проблемы: эмитенты могут пропадать или появляться в выборке (критично для генерации фич и в случае применения рекуррентных NN)
3. Анализ данных показывает, что данные выдаются по пятницам. Поиск в Гугл по названию полей говорит, что скорее всего это публичный, платный источник S&P Global Market Intelligence
4. Данные сбалансированы, метрика задана как `log_loss`. Значение базовой метрики `.69` (`np.log(.5)`)
5. Фичи генерятся при помощи `pipeline` (что гарантирует повторяемость препроцессинга):
  - 0 ой этап: mean-encoding на всем датасете 
  - 1-ый этап пайплан, `ClusterTransformer`: кластеризация на всех парах log-нормализованных фич (разброс не нормализованных фич до е12). Поскольку это пайплайн, метод фиттиться на трэйне. На тесте значение кластера присваевается по ID)
  - 2-ой этап `RowTransformer`: обогащение стандартными временными фичами (месяц, год, неделя года и т.д.) + net(продажи - покупки)
6. Важность фичей оценивается при помощи `shap`, затем, начиная с 1-ой самой важной фичи добавляем фичи до тех пор, пока идет прирост на тестовом датасете (greedy, probably not optimal)
7. На выходе получаем модель с 21 фичей и log_los .... 0.683 :-))), что ожидаемо для публичного датасета.
8. Что можно сделать еще:
  - mean_encoding использует сдвиг 12М в прошлое (по условиям подготовки датасета мы не можем иcпользовать таргет за прошедшие 12М). Нужно добавить данные по доходности эмитента за последние 1, 2, 3, 6 месяцев.
  - дополнительные фичи на новых данных.
9. Результат: файл Summary.pdf
  
