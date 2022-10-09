# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #3 выполнил(а):
- Ланчу Елизавета Сергеевна
- ХИИ31
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
Познакомиться с программными средствами для создания системы машинного обучения и ее интеграции в Unity.
## Задание 1
### Реализовать систему машинного обучения в связке Python - Google-Sheets – Unity. 

- Создайте новый пустой 3D проект на Unity.

![image](https://user-images.githubusercontent.com/81166835/194754846-5c722d6c-e115-4265-9e00-c5efb6b70019.png)

-В созданный проект добавьте ML Agent, выбрав Window - PackageManager - Add Package from disk. Последовательно добавьте .json – файлы:
o ml-agents-release_19 / com,unity.ml-agents / package.json
o ml-agents-release_19 / com,unity.ml-agents.extensions / package.json

- Далее запускаем Anaconda Prompt для возможности запуска команд через консоль. Далее пишем серию команд для создания и активации нового ML-агента, а также для скачивания необходимых библиотек:
o mlagents 0.28.0;
o torch 1.7.1;

![Снимок экрана (9 1)](https://user-images.githubusercontent.com/81166835/194755097-df4e03a2-92fd-48ec-b912-4729ae767d4d.png)

![Снимок экрана (10 1)](https://user-images.githubusercontent.com/81166835/194755106-80274d0b-5f77-4b49-8c41-8b19a5b106a0.png)

- Создайте на сцене плоскость, куб и сферу. Создайте простой C# скрипт-файл и подключите его к сфере:

![Снимок экрана (11)](https://user-images.githubusercontent.com/81166835/194723087-0e0debfc-7306-4c4d-8cd9-b405d5c8e1d5.png)

- В скрипт-файл RollerAgent.cs добавьте код, опубликованный в материалах лабораторных работ

```py
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    // Start is called before the first frame update
    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;
    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }
    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        float distanceToTarget = Vector3.Distance(this.transform.localPosition, Target.localPosition);

        if(distanceToTarget < 1.42f)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (this.transform.localPosition.y < 0)
        {
            EndEpisode();
        }
    }
}
```

- Объекту «сфера» добавить компоненты Decision Requester, Behavior Parameters

![Снимок экрана (12 1)](https://user-images.githubusercontent.com/81166835/194755184-5ba5b4b8-bf67-4992-9df3-5b38e3f2826d.png)

- В корень проекта добавьте файл конфигурации нейронной сети

```py
behaviors:
  RollerBall:
    trainer_type: ppo
    hyperparameters:
      batch_size: 10
      buffer_size: 100
      learning_rate: 3.0e-4
      beta: 5.0e-4
      epsilon: 0.2
      lambd: 0.99
      num_epoch: 3
      learning_rate_schedule: linear
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    max_steps: 500000
    time_horizon: 64
    summary_freq: 10000
```

- Запустите работу ml-агента

![Снимок экрана (15 1)](https://user-images.githubusercontent.com/81166835/194754553-ec0a1776-2c71-41da-9820-d87992b622eb.png)

- Вернитесь в проект Unity, запустите сцену, проверьте работу ML-Agent’a

![Снимок экрана (27)](https://user-images.githubusercontent.com/81166835/194754305-afd4a263-11f2-4eae-8e24-ec08d702e4a6.png)

![Снимок экрана (28)](https://user-images.githubusercontent.com/81166835/194754309-96459bcc-90bd-4998-a530-d3ed15cfeb23.png)

![Снимок экрана (29)](https://user-images.githubusercontent.com/81166835/194754330-bd12d40c-3ab2-4664-b156-b2427472b102.png)

- Сделайте 3, 9, 27 копий модели «Плоскость-Сфера-Куб», запустите симуляцию сцены и наблюдайте за результатом обучения модели

![Снимок экрана (30)](https://user-images.githubusercontent.com/81166835/194754284-c40389bc-e771-4b5e-8d0c-e2d0af702c18.png)

![Снимок экрана (31)](https://user-images.githubusercontent.com/81166835/194754370-024425ce-167b-41eb-ad95-124f70ca2d68.png)

- После завершения обучения проверьте работу модели.

![Снимок экрана (32)](https://user-images.githubusercontent.com/81166835/194754219-3c7f0ca6-a15b-415f-9497-9c0cfbde4794.png)

![Снимок экрана (37)](https://user-images.githubusercontent.com/81166835/194754239-3ca855b9-d47e-4abf-8ea8-e8cee72f2a8c.png)

https://clipchamp.com/watch/KcbyAKNE2bf

- Сделайте выводы.
Модель натренировалась ссредней наградой - 0.994, Кубики собирает хорошо за поле не падает
### Подробно опишите каждую строку файла конфигурации нейронной сети, доступного в папке с файлами проекта по ссылке. Самостоятельно найдите информацию о компонентах Decision Requester, Behavior Parameters, добавленных сфере.

```py
behaviors: #Создание списка "Модель поведения" дял разных агентов
  RollerBall: #Создание списка конкретного объекта
    trainer_type: ppo #Тип используемого тренажера, ppo - это алгоритм обучения с подкреплением от OpenAi
    hyperparameters:
      batch_size: 10 #Количество опытов на каждой итерации градиентного спуска. Это всегда должно быть в несколько раз меньше, чем buffer_size. 
      buffer_size: 100 #Количество опыта, которое необходимо собрать перед обновлением модели политики. .
      learning_rate: 3.0e-4 #Начальная скорость обучения для градиентного спуска. 
      beta: 5.0e-4 #Сила регуляризации энтропии, которая делает политику "более случайной". 
      epsilon: 0.2 #Влияет на то, насколько быстро политика может развиваться во время обучения.
      lambd: 0.99 #Параметр регуляризации (лямбда), используемый при вычислении обобщенной оценки преимуществ (GAE).
      num_epoch: 3 #Количество проходов через буфер опыта при выполнении оптимизации градиентного спуска.Чем больше batch_size, тем больше допустимо сделать это. 
      learning_rate_schedule: linear #Определяет, как скорость обучения меняется с течением времени. linear линейно уменьшает скорость обучения.
    network_settings:
      normalize: false #Применяется ли нормализация к входным данным векторного наблюдения.
      hidden_units: 128 #Количество единиц в скрытых слоях нейронной сети.
      num_layers: 2 #Количество скрытых слоев в нейронной сети. 
    reward_signals: #Награды
      extrinsic: #Внешние награды
        gamma: 0.99 #Коэффициент скидки для будущих вознаграждений, поступающих из среды.
        strength: 1.0 #Коэффициент, на который умножается вознаграждение, предоставляемое средой.
    max_steps: 500000 #Общее количество шагов 
    time_horizon: 64 #Сколько шагов опыта нужно собрать для каждого агента, прежде чем добавлять его в буфер опыта. 
    summary_freq: 10000 #Количество опыта, которое необходимо собрать перед созданием и отображением статистики обучения. 
```
Компонент DecisionRequester предоставляет удобный и гибкий способ запуска процесса принятия решений агентом. Без DecisionRequester внедрение вашего агента должно вручную вызывать его функцию RequestDecision() 
Компонент Behavior Parameters (параметры поведения) определяет, как объект принимает решения.


## Задание 3
### 3 Доработайте сцену и обучите ML-Agent таким образом, чтобы шар перемещался между двумя кубами разного цвета. Кубы должны, как и впервом задании, случайно изменять кооринаты на плоскости. В выводах к работе дайте развернутый ответ, что такое игровой баланс и как системы машинного обучения могут быть использованы для того, чтобы его скорректировать.
 ```py
 using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    // Start is called before the first frame update
    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;
    public Transform Target2;
    

    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
        Target2.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
        
    }
    
    public Vector3 midpointTtoT2;

    public override void CollectObservations(VectorSensor sensor)
    {
        midpointTtoT2 = new Vector3((Target.localPosition.x+Target2.localPosition.x)/2.0f,(Target.localPosition.z+Target2.localPosition.z)/2.0f);
        sensor.AddObservation(Target.localPosition);
        sensor.AddObservation(Target2.localPosition);
        sensor.AddObservation(midpointTtoT2);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }
    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        midpointTtoT2 = new Vector3((Target.localPosition.x+Target2.localPosition.x)/2.0f,(Target.localPosition.z+Target2.localPosition.z)/2.0f);
        float distanceTomidpointTtoT2 = Vector3.Distance(this.transform.localPosition, midpointTtoT2);

        if(distanceTomidpointTtoT2 < 1.42f)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (this.transform.localPosition.y < 0)
        {
            EndEpisode();
        }

        
    }
}
 ```

https://clipchamp.com/watch/IfEWtK5FInY

https://clipchamp.com/watch/Y1jLU1he43h

Данная модель обучается дольше, не всегда проходит между кубиков и часто выпадает с поля

## Выводы
Игровой баланс — «равновесие» между персонажами, командами, тактиками игры и другими игровыми объектами. Игровой баланс — одно из требований к «честности» правил.
Машинное обучение используется для настройки игрового баланса. Проводя при помощи обученных агентов миллионы симуляций с целью сбора данных, эта методика тестирования игр на основе ML позволяет эффективнее повышать интересность и баланс игр.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
