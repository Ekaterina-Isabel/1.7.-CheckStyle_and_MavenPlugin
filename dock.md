## Правильно настроен Maven-проект, тесты проходят
  
  Репозиторий должен быть папкой вашего мавен-проекта. Обратите внимание, что репозиторием не должна быть папка в которой лежит папка мавен-проекта, он сам должен быть папкой проекта. В нём должны быть соответствующие файлы и папки - `pom.xml`, `src` и др.
  
  Не забудьте создать .gitignore-файл в корне проекта и добавить туда в игнорирование автогенерируемую папку `target`.
  
  Общая схема вашего `pom.xml`-файла:
  
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>ru.netology</groupId>
    <artifactId>НАЗВАНИЕ-ВАШЕГО-ПРОЕКТА-БЕЗ-ПРОБЕЛОВ</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>


    <dependencies>
        <dependency>
            ...
        </dependency>
        ...
    </dependencies>


    <build>
        <plugins>
            <plugin>
              ...
            </plugin>

            <plugin>
              ...
              <executions>
                <execution>
                  ...
                </execution>
                ...
              </executions>
            </plugin>
            ...
        </plugins>
    </build>

</project>
  ```
  
  #### JUnit
  Обратите внимание что у артефакта нет `-api` на конце. Если у вас автоматически добавилась зависимость вида `<artifactId>junit-jupiter-api</artifactId>`, то лучше поменять артефакт на тот что ниже, иначе будут сюрпризы в работе.

  ```xml
          <dependency>
              <groupId>org.junit.jupiter</groupId>
              <artifactId>junit-jupiter</artifactId>
              <version>5.7.0</version>
              <scope>test</scope>
          </dependency>
  ```

  #### Surefire
  Без этого плагина тесты могут мавеном не запускаться, хоть в идее через кнопки они и будут проходить. Чтобы лишний раз убедиться, что всё работает, нажмите `Ctrl+Ctrl` и затем `mvn clean test`.
  
  ```xml
              <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-surefire-plugin</artifactId>
                  <version>2.22.2</version>
                  <configuration>
                      <failIfNoTests>true</failIfNoTests>
                  </configuration>
              </plugin>
  ```
  
  ---
  
## Настроен Github CI с verify-сборкой Maven и JaCoCo :new:
  
  #### CI
  После связывания локального репозитория с удалённым и первого пуша в заготовки проекта, время настроить CI на основе Github Actions. Шаблон вашего maven.yml должен выглядеть вот так, убедитесь что всё совпадает с вашим шаблоном (например, что вы указали фазу `verify`, а не `package`):
  ```yml
  name: Java CI with Maven

  on: [push, pull_request]

  jobs:
    build:

      runs-on: ubuntu-latest

      steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          java-version: '11'
          distribution: 'adopt'
      - name: Build with Maven
        run: mvn -B -e verify
  ```
  
  #### JaCoCo

  ```xml
              <plugin>
                  <groupId>org.jacoco</groupId>
                  <artifactId>jacoco-maven-plugin</artifactId>
                  <version>0.8.5</version>
                  ...
  ```

  Инициализация:
  ```xml
                      <execution>
                          <id>prepare-agent</id>
                          <goals>
                              <goal>prepare-agent</goal>
                          </goals>
                      </execution>
  ```

  В режиме генерации отчётов:
  ```xml
                      <execution>
                          <id>report</id>
                          <phase>verify</phase>
                          <goals>
                              <goal>report</goal>
                          </goals>
                      </execution>
  ```

  В режиме проверки и обрушения сборки по уровню покрытия:
  ```xml
                      <execution>
                          <id>check</id>
                          <goals>
                              <goal>check</goal>
                          </goals>
                          <configuration>
                              <rules>
                                  <rule>
                                      <limits>
                                          <limit>
                                              <counter>LINE</counter>
                                              <value>COVEREDRATIO</value>
                                              <minimum>100%</minimum>
                                          </limit>
                                      </limits>
                                  </rule>
                              </rules>
                          </configuration>
                      </execution>
  ```


## Задание 1. Синдром 100%

Вы попали в команду максималистов, которые хотят, чтобы те авто-тесты, которые вы пишете, покрывали код на 100%.

Но вот незадача:
1. Непонятно, что такое 100%
2. Непонятно, как это сделать

Вспоминаем: покрытием кода у нас занимается JaCoCo, но он просто "сигнализирует" о том, что конкретно пошло не так.

Большинство подобных плагинов помимо целей отчётности (`report`) содержат ещё цель `check`, которая обрушает сборку, если не выполнены определённые проверки.

**Что вам нужно:**
1. Создать мавен-проект с тестируемым кодом из листинга кода (указан ниже по условию)
1. Рекомендуем ознакомиться с [документацией на плагин](https://www.eclemma.org/jacoco/trunk/doc/maven.html) (а конкретно на цель `check`) или посмотреть в чеклист этого задания
1. Внедрить эту цель в фазу `verify` (обратите внимание, что эта цель итак публикуется в эту фазу)
1. Настроить правила по покрытию на 100% (при этом нужно изучить разницу между счётчиками `INSTRUCTION`, `LINE`, `BRANCH`, `COMPLEXITY`)
1. Вникнуть в тестируемый код
1. Выбрать один из счётчиков и добиться 100% покрытия через добавление новых тестов

**Важно**: использовать можно только один из следующих: 
1. `INSTRUCTION`
1. `LINE`
1. `BRANCH`

Обратите внимание на чеклист в начале условия, он содержит подсказки по внедрению JaCoCo в ваш мавен-проект.

Тестируемый код (его как-либо редактировать **нельзя**):
```java
package ru.netology.statistic;

public class StatisticsService {
  /**
   * Calculate index of max income
   *
   * @param incomes - array of incomes
   * @return - index of first max value
   */
  public long findMax(long[] incomes) {
    long current_max_index = 0;
    long current_max = incomes[0];
    for (long income : incomes)
      if (current_max < income)
        current_max = income;
        return current_max;
  }
}
```

Класс с тестами (его надо будет расширить новыми тестами):
```java
package ru.netology.statistic;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

public class StatisticsServiceTest {

  @Test
  void findMax() {
    StatisticsService service = new StatisticsService();

    long[] incomesInBillions = {12, 5, 8, 4, 5, 3, 8, 6, 11, 11, 12};
    long expected = 12;

    long actual = service.findMax(incomesInBillions);

    assertEquals(expected, actual);
  }
}
```

**Результат:** отправьте на проверку ссылку на гитхаб-репозиторий с вашим проектом.

## Задание 2*. Пусть плагин ищет баги

[SpotBugs](https://spotbugs.github.io) и [Maven Plugin для него](https://spotbugs.readthedocs.io/en/latest/maven.html) предоставляют возможность производить статический анализ (анализ кода без его запуска) для выявления наиболее часто встречающихся багов.

Список багов, которые ищет SpotBugs: https://spotbugs.readthedocs.io/en/latest/bugDescriptions.html

**Ваша задача:**
1. Подключить плагин к вашему проекту (возьмите проект с первой задачи, либо создайте новый на его базе)
1. Настроить goal `check` в фазу `verify`
1. Удостовериться, что код проходит проверки SpotBugs (если не проходит, то пофиксить)
1. Сделать push в GitHub и удостовериться, что сборка проходит

**Результат:** отправьте на проверку ссылку гитхаб-репозиторий с вашим проектом.

## Задание 3*. Внедряем стандарты кодирования (НЕобязательная задача)

[CheckStyle](https://checkstyle.sourceforge.io/) и [Maven Plugin для него](https://maven.apache.org/plugins/maven-checkstyle-plugin/usage.html) предоставляют возможность производить проверки для выявления соответствия стиля написания кода заданным стандартам.

Стандарты в организации формируют ведущие программисты и от организации к организации стандарты могут отличаться.

Мы будем использовать [Google Java Style Guide](https://checkstyle.sourceforge.io/styleguides/google-java-style-20180523/javaguide.html)

Используйте приложенный файл [checkstyle.xml](https://raw.githubusercontent.com/netology-code/javaqa2-homeworks/main/files/checkstyle.xml) в качестве набора правил (положите его в корень проекта и укажите в качестве настройки `configLocation`).

**Ваша задача:**
1. Подключить плагин к вашему проекту (возьмите проект с первой задачи, либо создайте новый на его базе)
1. Настроить goal `check` в фазу `verify`
1. Удостовериться, что код не проходит проверки CheckStyle (фиксить не нужно)
1. Сделать push в GitHub и удостовериться, что сборка не проходит именно по причине наличия ошибок CheckStyle

**Результат:** отправьте на проверку ссылку гитхаб-репозиторий с вашим проектом.
