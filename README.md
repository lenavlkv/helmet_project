# Детектор наличия каски на голове для системы безопасности на производстве

Вопрос безопасности на рабочих местах, особенно в строительстве, является крайне важным. Строительные каски — это один из основных элементов защиты, который может спасти жизнь или предотвратить серьезные травмы. Однако, несмотря на важность этого средства индивидуальной защиты, на практике бывает сложно контролировать, соблюдают ли рабочие правила безопасности.

Технологии компьютерного зрения помогают автоматизировать мониторинг, ускоряя процесс проверки и обеспечивая большую точность. Использование нейросетевых моделей для автоматического обнаружения касок на изображениях позволяет существенно повысить уровень безопасности, предотвратить аварии и снизить риски на строительных площадках.

Цель проекта — разработать систему, которая с помощью искусственного интеллекта будет эффективно обнаруживать наличие каски на головах людей, автоматически выявляя нарушения и обеспечивая оперативную реакцию на потенциальные угрозы.

--------------------------------------------------------------
Для данного проекта использовалось два набора данных с изображениями людей в касках. Доступны по ссылкам: 

https://www.kaggle.com/datasets/vodan37/yolo-helmethead

https://universe.roboflow.com/dinh-ho/safety2r-tbo4l-jyd5o 

В наборе №1 присутствует 2 класса (0 - no-helmet, 1 - helmet), в наборе №2 - 5 классов (0 - helmet, 1 - no-helmet, 2 - safety vest, 3 - no-safety vest, 4 - shoes). Сначала необходимо привести их к одному виду путем отсечения во втором наборе лишних классов и заменой одного класса на другой. В итоге должно получится всего два класса (0 - no-helmet, 1 - helmet). Для этого использовался код из файлов reverse_labels.py и datasets_union.py. 

Формат изображений: jpeg

Формат аннотаций: txt



При анализе объединенных датасетов было обнаружено, что большинство изображений не превышают размера 2000х2000px, а так же количество изображений суммарно составляет 42297 шт., из которых train 34349 шт. (81%), test 2488 шт. (6%), valid 5460 шт (13%).
![Figure_1](https://github.com/user-attachments/assets/9b4e8fd4-d9e7-429c-9c65-48d981780176)

Для анализа датасетов использовался код из файла observe_dataset.py.


Далее анализировались объекты в датасетах. Всего обнаружено 122727 касок и 165119 голов без касок. В датасете большую часть занимают головы без касок (57%) по отношению к головам с касками (43%). Для анализа использовался код из файла observe_dataset.py.



Далее после того, как мы сформировали данные и проанализировлаи их, мы можем приступать к обучению модели. В качестве модели здесь выбрана YOLOv5, batch 16, 10 эпох.

python train.py --img 640 --batch 16 --epochs 10 --data ./data/helmet.yaml --weights yolov5s.pt

Получили следующие значения:

![22 varik yolov5 batch 16 --epochs 10](https://github.com/user-attachments/assets/ddaa89cb-93c6-411c-8a54-60c53269be88)

![confusion_matrix](https://github.com/user-attachments/assets/bdf2bb64-d306-4013-aa48-7208ed390614)
![F1_curve](https://github.com/user-attachments/assets/748808d1-11c2-4c39-8ce9-ab8fc1d6586d)
![PR_curve](https://github.com/user-attachments/assets/cfed453a-a942-4c2a-93a7-5496e06913fc)
![results](https://github.com/user-attachments/assets/65e94a12-c77c-4aeb-bf15-cba7c15dc060)
![train_batch1](https://github.com/user-attachments/assets/5a81eb51-2c2f-4f91-a45a-7396278b5994)
![004840_jpg rf 6886c68e3b1927c37035ea023371911f](https://github.com/user-attachments/assets/916c3fc7-3514-453a-b076-8fcf735022c8)


Посмотрим, где ошибается наша модель.

![helm_000652](https://github.com/user-attachments/assets/d4ebb833-ee0f-45d6-b3be-550778cd0b29)
![helm_001018](https://github.com/user-attachments/assets/d1db4339-5bd6-441a-a705-1bf6180a71b9)

При визуальном анализе ошибок я заметила, что модель пропускает некоторые объекты (ошибки второго рода). Ошибки, где модель путала бы классы - не выявлены. Закономерностей, при которых модель ошибалась бы из-за размера изображений, размера объектов, фона - не выявлены.


Сравним полученные значения оценки модели со значениями из kaggle (https://www.kaggle.com/code/bilrein/helmet-yolov8).

Там использовалось yolov8, batch 16, epochs 25. 
![2025-01-25_13-36-19](https://github.com/user-attachments/assets/03658992-4a65-45a3-b983-473fcbd1cddc)

В целом видно, что у нас значения получились чуть лучше. Попробуем их улучшить еще путем изменения некоторых параметров.

Далее в эксперименте я меняла значения batch, epochs, модель, добавляла аугментацию данных. Измеряла скорость обучения, значения R, P, mAP 50, mAP 50-95. Результаты можно видеть в таблице и на графике.

![2025-01-25_13-53-50](https://github.com/user-attachments/assets/17c88fc3-328e-4ede-b0f1-c43d36d4ca43)
![2025-01-25_13-54-20](https://github.com/user-attachments/assets/82e48ded-87c8-4315-bce7-440651a99811)

В целом можно сделать выводы, что при увеличении количества эпох, замене YOLOv5 на более новые YOLO помогают улучшить результаты. Однако, увеличение количества изображений в наборе данных оказывает более сильное влияние на улучшение результатов оценки модели. Вероятно, при увеличении количества изображений в датасете, модель обучается на более разнообразных данных, она видит больше вариаций объектов, фонов, углов, освещений и других факторов, которые могут варьироваться в реальных условиях.




