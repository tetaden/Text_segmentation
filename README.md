# Text_segmentation
Отборочное задание для поступление в инжинерную школу МИЭМ

В качестве второго отборочного задания выступает задача сегментации текста.
Первое, что приходит в голову - сегментации тематически, то есть выделенеие в тексте конкретных тем и нахождение ключевых слов, определяющих данную тематику 
В качестве данных предложены истории с сайта tjournal.ru,


![image](https://user-images.githubusercontent.com/84914528/196962795-780ca904-ce11-4f44-b295-5bcef3c38008.png)

Каждый элемент датасета представляет собой пару (текст, класс), где класс определяет какой частью текста является предложение:
1 — первое предложение абзаца

2 — первое предложение секции

0 — остальные предложения (части абзацев).

![image](https://user-images.githubusercontent.com/84914528/196963069-4be5dc04-2dfc-4b4d-925c-7c8255f78642.png)

# Обработка текста

Для начала необходимо было изменить датасет так, чтобы им можно было воспользоваться, для этого создадим 2 различных датасета
В 1 датасете в каждой его строке хранится весь текст одного из документов 
![image](https://user-images.githubusercontent.com/84914528/196963594-c4c0a9ff-fb99-4582-9065-a39d58c7394d.png)
2 же датасет будет представлять собой предложение - пару, хранящихся как объект pd.Series
![image](https://user-images.githubusercontent.com/84914528/196963712-0344f2c4-5011-44a8-baf0-a175cf925327.png)

# Тематическое разбиение текста 
Для данного разбиения будем  использовать модель LDA (Латентное размещение Дирихле), данная модель часто используется для Topic Modeling или тематического моделирования
Для того, чтобы применять к датасету текстов LDA, необходимо преобразовать датасет в term-document matrix(Терм-документная матрица).


Терм-документная матрица — это матрица которая имеер размер $N \times W$, где
N — количество документов в корпусе, а W — размер словаря корпуса т.е. количество слов(уникальных) которые встречаются в нашем корпусе. В i-й строке, j-м столбце матрицы находится число — сколько раз в i-м тексте встретилось j-е слово.

LDA строит, для данной Терм-документной матрицы и T заранее заданого числа тем — два распределения:


Распределение тем по текстам.(на практике задается матрицей размера $N \times T$)
Распределение слов по темам.(матрица размера $T \times W$)

Для начала необходимо изменить наши данные и построить некий вокубуляр слов, которыми будем пользоваться наша терм-документальная матрица 
Используя токенизацию, лемминг и фильтрацию слов удалось добиться списка вокубуляра в 1438 слов 
![image](https://user-images.githubusercontent.com/84914528/196964793-bba331ac-7aea-4bac-aa08-a5227325ae17.png)

С помощью уже созданного класса CountVectorizer создаем требуемую td-matrix матрицу

Посмотрим как реализует наша модель разделение на темы случайного текста, если потребовать у него 3 различной темы с 5 ключевыми словами 
![image](https://user-images.githubusercontent.com/84914528/196965706-44b8fd7d-dc38-4864-80e0-2d8ea3da5600.png)

Как видим результат не очень хорош по причине лишних иностранных слов, которые не были убраны, однако можно выделить несколько различных важных слов указывающих на тематику текста  
1 тема ,например, очевидно намекает на то что модель выделила некоторый блок рассказывающий про совесткий союз и про вторую мировую войну.

# Кластеризация

как я уже уточнил в самом файле, необходимости в этом не было, однако это позволяет посмотреть как тексты и их темы  взаимосвязанны с друг другом
Для того, чтобы легче было определить количество кластеров используем метод Локтя, который показывает на лучшее по распределению значение кластера при определенном значении
![image](https://user-images.githubusercontent.com/84914528/196967586-4ba07c84-9df4-4e1c-a47c-2e040733edce.png)

Чтобы найти best_k  достаточно определить в какой точке происходит сгибание "локтя" то есть графика, видно что график наиболее выпукл в точке k = 16 
С помощью алгоритма K-means и стохастического вложения соседей с t-распределением было получено такое распределение
![image](https://user-images.githubusercontent.com/84914528/196967283-ff59e2a0-5ead-44c2-935c-b91e8296e691.png)

# Определение класса предложения

Как было сказано ранее, предложения  в дасете определены некоторым класом, обозначающим их принадлежность к конкретной части текста 
Таким образом можно попробовать искать предложения являющимися началом секции или абзаца. Очевидно, что такие предложения имеют конкретные предлоги, которые наталкивают нас на то, что они являются началом какого-то блока, таким образом можно выделить их отдельно от остальных
Для данной задачи будем использовать две разной модели Дерево решений и Логистическая Регрессия, а также посмотрим при каком разбиении униграмм модель покажет лучший результат
![image](https://user-images.githubusercontent.com/84914528/196968962-7f9482ce-2b21-492a-b33e-64b0b3ca0965.png)

![image](https://user-images.githubusercontent.com/84914528/196968989-61c043c7-db70-41e7-8499-1c99b55140ef.png)
 Получилось так, что Дерево решений лучше справляется с определением каким является это предложение, однако захватывает меньшую часть предложений с классом 1 и 0, в то время как Логистическая регрессия делает все наоборот. Так как нашей задачей было нахождение модели, что захватит как можно больше предложений 1 и 0, то есть покажет лучшие результаты на метрике recall, то в данном случае отдается предпочтение в сторону Логистической регрессии
 Также была проверена модель при различных размерах униграмм, к сожалению при увеличении размера модель захватывает меньшее количество объектов класса 1 и 0, поэтому лучше использовать униграмы размера 1  
