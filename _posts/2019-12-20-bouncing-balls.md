---
layout: post
categories: posts, tutorial
title: Мир прыгающих шаров. Это не то что вы думаете. Часть 1 [Перевод]
tags: [ tutorial, translate, Java, Swing, GameDev]
date-string: ДЕКАБРЬ 20, 2019
featured-image: /images/2019-12-20/prevue.jpg

---


# Введение
Данный цикл постов является переводом статьи из курса [*"Java Game Programming"*](https://www.ntu.edu.sg/home/ehchua/programming/java/J8a_GameIntro-BouncingBalls.html). Я наткнулся на эту статью в результате поисков в интернете о разработке игр в рамках фрейморка Swing. Ее объем и подробное изложение, казалось бы, такой простой темы как, имитация физики столкновений шариков в играх, меня заинтересовали, как надеюсь заинтересует и читателя. 

# Демо 

Добавлю позже 

# Случай 1. Шарик в коробочке

Рассмотрим самый простой способ имитации физики столкновений шарика о стенки прямоугольного контейнера. Это займет буквально несколько строк кода 

``` java
import java.awt.*;
import java.util.Formatter;
import javax.swing.*;
/**
 * Один отражающийся шарик в прямоугольной коробке.
 * Весь код в одном файле. Это неграмотный дизайн кода!
 */


// Расширяем поведение(добавляем) на основе базовой JPanel, для того что бы изменить отрисовку этого компонента 
public class BouncingBallSimple extends JPanel {
   // Зададим ширину и высоту контейнера для шарика
   private static final int BOX_WIDTH = 640;
   private static final int BOX_HEIGHT = 480;
  
   // Зададим характеристики шарика
   private float ballRadius = 200; // Радиус шарика
   // Координаты центра (x, y)
   private float ballX = ballRadius + 50; 
   private float ballY = ballRadius + 20; 
   private float ballSpeedX = 3;   // Скорость шарика по различным осям
   private float ballSpeedY = 2;
  
   private static final int UPDATE_RATE = 30; // Число отвечающее за количество обновлений экрана за единицу времени
  
   /** Создадим конструктор для создания UI и инициализации объектов */
   public BouncingBallSimple() {
      this.setPreferredSize(new Dimension(BOX_WIDTH, BOX_HEIGHT));
  
      // Дадим толчок нашему шарику (из отдельного потока)
      Thread gameThread = new Thread() {
         public void run() {
            while (true) { // вечный цикл обновления
               // Модифицируем координаты шарика по осям на некоторую дельту
               ballX += ballSpeedX;
               ballY += ballSpeedY;
               // Проверка на пересечение границ 
               // Если пересекли, то изменяем вектор скорости и координаты границ
               if (ballX - ballRadius < 0) {
                  ballSpeedX = -ballSpeedX; // Инвертация вектора движения (обычное отражение)
                  ballX = ballRadius; // Релокация шарика относительно границы
               } else if (ballX + ballRadius > BOX_WIDTH) {
                  ballSpeedX = -ballSpeedX;
                  ballX = BOX_WIDTH - ballRadius;
               }
               // Проверим так же для двух других 
               if (ballY - ballRadius < 0) {
                  ballSpeedY = -ballSpeedY;
                  ballY = ballRadius;
               } else if (ballY + ballRadius > BOX_HEIGHT) {
                  ballSpeedY = -ballSpeedY;
                  ballY = BOX_HEIGHT - ballRadius;
               }
               // вызываем перерисовку компонента
               repaint(); // Callback paintComponent()
               // Задержка между вызовами перерисовки компонента
               try {
                  Thread.sleep(1000 / UPDATE_RATE);  // миллисекунды
               } catch (InterruptedException ex) { }
            }
         }
      };
      gameThread.start();  // вызываем исполнение потока
   }
  
   /** Переопределяем поведение для отрисовки компонента */
   @Override
   public void paintComponent(Graphics g) {
      super.paintComponent(g);    // Вызываем базовую отрисовку компонента
  
      // Рисуем контейнер
      g.setColor(Color.BLACK);
      g.fillRect(0, 0, BOX_WIDTH, BOX_HEIGHT);
  
      // Рисуем шарик 
      g.setColor(Color.BLUE);
      g.fillOval((int) (ballX - ballRadius), (int) (ballY - ballRadius),
            (int)(2 * ballRadius), (int)(2 * ballRadius));
  
      // Выводим информацию о шарике 
      g.setColor(Color.WHITE);
      g.setFont(new Font("Courier New", Font.PLAIN, 12));
      StringBuilder sb = new StringBuilder();
      Formatter formatter = new Formatter(sb);
      formatter.format("Ball @(%3.0f,%3.0f) Speed=(%2.0f,%2.0f)", ballX, ballY,
            ballSpeedX, ballSpeedY);
      g.drawString(sb.toString(), 20, 30);
   }
  
   /** точка входа нашей программы */
   public static void main(String[] args) {
      // вызываем отрисовку через композитный менеджер в новом потоке
      javax.swing.SwingUtilities.invokeLater(new Runnable() {
         public void run() {
            // Задаем основное окно программы
            JFrame frame = new JFrame("A Bouncing Ball");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setContentPane(new BouncingBallSimple());
            frame.pack();
            frame.setVisible(true);
         }
      });
   }
}

```

Теперь можно рассмотреть наш код подробнее. 

- наш класс BouncingBallSimple расширяет базовый класс в части относящейся к отрисовке компонента (мы определили что дополнительно нужно нарисовать)
- в конструкторе мы задали графические компоненты интерфейса. Также указали, что хотим обновлять состояние нашего шарика из нового потока
- с каждым обновлением, мы будем сдвигать наш шарик по координатным осям на величину его скорости, так же проверяя не пересеклись (*collision*) ли коодинаты шарика с границами нашего контейнера. Если шарик пересекает границу, мы *реагируем* меняя шарику позицию и скорость. Например, шарик отразится по горизонтали если его координаты пересекут границу по оси X и отразиться по вертикали, если координаты шарика пересекут ограничения по оси Y.
- мы расширили стандартное поведение `paintComponent()` для того что бы отрисовать необходимые графические объекты. Используя `fillRect()` мы отрисовываем прямоугольник контейнера, `fillOval()` для отрисовки шарика, `drawString()` для работы с текстом на экране.
- в методе `main()`, мы создаем объект окна, в котором будем производить все дальнейшие действия. Для отрисовки мы определили пользователький класс наследованный от JFrame. Код графики запускается в специальном UI-потоке *Event Dispatcher Thread(EDT)*, вызываемый из `javax.swing.SwingUtilities.invokeLater()`, так рекомендуют разработчики Swing. 

Несмотря на то, что данная программа работает, с точки зрения дизайна кода она оставляет желать лучшего (с точки зрения переиспользования кода и расширяемости программы). Более того, алгоритмы проверки на пересечение границ и реакции шарика несколько сыроваты и не расширяемы. К тому же в нашей программе нет контроля скорости шарика. 


# Случай 2. Шарик в объектно-ориентированном мире

Давайте перепишем нашу программу с шариком и контейнером в объектно-ориентированном стиле. Начнем с класса контейнера.

## Контейнер

![Контейнер класс]({{ site.baseurl }}\images\2019-12-20\GameBall_BoxClass.png) 

Класс содержит следующие поля: 
 - `minX, minY, maxX, maxY`, представляющий размеры контейнера
 Стоит сказать, что координатные оси в Java (как и во всей ОС Windows) графические координаты инвертированы *по горизонтали*, с центром осей (0,0) в левом верхнем углу, более наглядно это выглядит так: 

 
![Координатные оси окна]({{ site.baseurl }}\images\2019-12-20\Window_coordinate.png)

Класс контейнера содержит следующие методы: 
- конструктор класса, определяющий основные координаты окна (x, y), высота, ширина, цвет. Безопаснее использовать такое определение для определения прямоугольника. 
- метод для привязки или отвязки границ
- метод отрисовки, отрисовывающий окно (само себя)  при помощи графического контектста

```java
import java.awt.*;
/**
 * Прямоугольный контейнер, содержащий шарик
 */
public class ContainerBox {
   int minX, maxX, minY, maxY;  // размеры контейнера (package access)
   private Color colorFilled;   // цвет фона контейнера (background)
   private Color colorBorder;   // цвет границ контейнера
   private static final Color DEFAULT_COLOR_FILLED = Color.BLACK;
   private static final Color DEFAULT_COLOR_BORDER = Color.YELLOW;
   
   /** Конструктор */
   public ContainerBox(int x, int y, int width, int height, Color colorFilled, Color colorBorder) {
      minX = x;
      minY = y;
      maxX = x + width - 1;
      maxY = y + height - 1;
      this.colorFilled = colorFilled;
      this.colorBorder = colorBorder;
   }
   
   /** Конструктор с цветами по-умолчанию */
   public ContainerBox(int x, int y, int width, int height) {
      this(x, y, width, height, DEFAULT_COLOR_FILLED, DEFAULT_COLOR_BORDER);
   }
   
   /** Задаем или изменяем размеры окна */
   public void set(int x, int y, int width, int height) {
      minX = x;
      minY = y;
      maxX = x + width - 1;
      maxY = y + height - 1;
   }

   /** Отрисовываем самого себя и плюс необходимую графику */
   public void draw(Graphics g) {
      g.setColor(colorFilled);
      g.fillRect(minX, minY, maxX - minX - 1, maxY - minY - 1);
      g.setColor(colorBorder);
      g.drawRect(minX, minY, maxX - minX - 1, maxY - minY - 1);
   }
}


```
## Класс Шарика

![Класс шарика]({{ site.baseurl }}\images\2019-12-20\GameBall_BallClass.png)

Класс шарика содержит следующие поля: 
- x,y, radius, color, которые представляют центр шарика, его радиус и цвет
- speedX, speedY, которые представляют его скорость по осям, измеряемая в пикселях за единицу отрисовки

Под капотом, все числа выражаются в виде чисел с плавающей точкой для обеспечения более *гладкого* рендеринга, особенно для тригонометрических операций. Числа будут преобразованы в целые значения пикселей (для большинства игр достаточно 32-битной точности float)

Шарик имеет следующие публичные методы:

1. Конструктор задающий координаты по осям, радиус, скорость в полярных координатах и угог вектора движения относительно текущего  (потому что так легкче и понятнее, для пользователя нашего класса, определить скорость) и цвет.

![Вектор скорости]({{ site.baseurl }}\images\2019-12-20\Ball_coordinate_vector.png)

2. Метод `draw()` для отрисовки шарика через графический контекст приложения
3. Метод `toString()` для описания характеристик шарика в виде строки, которая используется для вывода статуса шарика на экран
4. Метод `moveOneStepWithCollisionDetection(Box box)` позволящий переместить шарик на один шаг вперед с проверкой на пересечение им границ

Теперь посмотрим, что же у нас получилось: 

```java 
import java.awt.*;
import java.util.Formatter;
/**
 * Шарик ООП
 */
public class Ball {
   float x, y;           // координаты центра шарика
   float speedX, speedY; // величина смещения шарика за каждый шаг нашей анимации
   float radius;         // радиум шарика
   private Color color;  // цвет шарика
   private static final Color DEFAULT_COLOR = Color.BLUE;
  
   /**
    * Конструктор: Для упрощения работы с классом, пользователь класса определяет скорость движения 
    * и угол отклонения, в градусах. Эти значения необходимо преобразовать в обычные числа понятные координатной системе 
    * Java. 
    */
   public Ball(float x, float y, float radius, float speed, float angleInDegree,
         Color color) {
      this.x = x;
      this.y = y;
      // Преобразуем координаты x,y c инверсией оси y
      this.speedX = (float)(speed * Math.cos(Math.toRadians(angleInDegree)));
      this.speedY = (float)(-speed * (float)Math.sin(Math.toRadians(angleInDegree)));
      this.radius = radius;
      this.color = color;
   }
   /** Конструктор с цветом по-умолчанию */
   public Ball(float x, float y, float radius, float speed, float angleInDegree) {
      this(x, y, radius, speed, angleInDegree, DEFAULT_COLOR);
   }
   
   /** Рисуем шарик при помощи графического контекста */
   public void draw(Graphics g) {
      g.setColor(color);
      g.fillOval((int)(x - radius), (int)(y - radius), (int)(2 * radius), (int)(2 * radius));
   }
   
   /** 
    * Сдвигаем шарик по осям на определенный в конструкторе шаг
    * 
    * @param box: контейнер для шарика 
    */
   public void moveOneStepWithCollisionDetection(ContainerBox box) {
      // вычисляем крайней точки нашего шарика
      float ballMinX = box.minX + radius;
      float ballMinY = box.minY + radius;
      float ballMaxX = box.maxX - radius;
      float ballMaxY = box.maxY - radius;
   
      // вычисляем новые координаты шарика
      x += speedX;
      y += speedY;
      // проверяем не пересекли ли мы шариком границы нашего контейнера
      if (x < ballMinX) {
         speedX = -speedX; // инвертируем нормали
         x = ballMinX;     // позиционируем на границе
      } else if (x > ballMaxX) {
         speedX = -speedX;
         x = ballMaxX;
      }
      // проверяем пересечение с другими границами
      if (y < ballMinY) {
         speedY = -speedY;
         y = ballMinY;
      } else if (y > ballMaxY) {
         speedY = -speedY;
         y = ballMaxY;
      }
   }
   
   /** Возвращаем скорость шарика */
   public float getSpeed() {
      return (float)Math.sqrt(speedX * speedX + speedY * speedY);
   }
   
   /** Возвращаем угол движения в полярных координатах */
   public float getMoveAngle() {
      return (float)Math.toDegrees(Math.atan2(-speedY, speedX));
   }
   
   /** Возвращаем массу шарика  */
   public float getMass() {
      return radius * radius * radius / 1000f;  // Нормализуем размерность нашего шарика
   }
   
   /** Возвращаем кинетическую энергию шарика (0.5mv^2) */
   public float getKineticEnergy() {
      return 0.5f * getMass() * (speedX * speedX + speedY * speedY);
   }
  
   /** Возвращаем строку с характеристиками шарика */
   public String toString() {
      sb.delete(0, sb.length());
      formatter.format("@(%3.0f,%3.0f) r=%3.0f V=(%2.0f,%2.0f) " +
            "S=%4.1f \u0398=%4.0f KE=%3.0f", 
            x, y, radius, speedX, speedY, getSpeed(), getMoveAngle(),
            getKineticEnergy());  // символ \u0398 это греческая буква тэта
      return sb.toString();
   }
   // Переиспользуем строку через мутабельный класс строк
   private StringBuilder sb = new StringBuilder();
   private Formatter formatter = new Formatter(sb);
}
```

# Логика и управление физикой в нашей программе 

![Класс логики]({{ site.baseurl }}\images\2019-12-20\GameBall_BallWorldClass.png)

Класс игровой логики отвечает за Управление состоянием игры (К) (gameStart(), gameUodate()), предоставляя данные для слоя Представлений (В), который будет расширением обычной JPanel. 

```java
import java.awt.*;
import java.awt.event.*;
import java.util.Random;
import javax.swing.*;
/**
 * Контроллер нашей игры и отображение внешнего вида окошка 
 */
public class BallWorld extends JPanel {
   private static final int UPDATE_RATE = 30;  // Количество кадров в секунду (fps)
   
   private Ball ball;         // Скажем, что мы хотим одинокий шарик
   private ContainerBox box;  // Он будет в коробочке
  
   private DrawCanvas canvas; // Холст для отрисовки нашей графики 
   private int canvasWidth;
   private int canvasHeight;
  
   /**
    * Конструктор создает наш интерейс и игровые объекты на нем
    * Задает размер области на которой мы будем отрисовывать наши игровые объекты
    * 
    * @param width : ширина холста
    * @param height : высота холста
    */
   public BallWorld(int width, int height) {
  
      canvasWidth = width;
      canvasHeight = height;
      
      // Создаем случайный шарик со случайным вектором движения
      Random rand = new Random();
      int radius = 200;
      int x = rand.nextInt(canvasWidth - radius * 2 - 20) + radius + 10;
      int y = rand.nextInt(canvasHeight - radius * 2 - 20) + radius + 10;
      int speed = 5;
      int angleInDegree = rand.nextInt(360);
      ball = new Ball(x, y, radius, speed, angleInDegree, Color.BLUE);
     
      // Создаем для него коробочку
      box = new ContainerBox(0, 0, canvasWidth, canvasHeight, Color.BLACK, Color.WHITE);
     
      // Создаем объект, на котором будем все это рисовать
      canvas = new DrawCanvas();
      this.setLayout(new BorderLayout());
      this.add(canvas, BorderLayout.CENTER);
      
      // Перехватываем событие изменения размера окна
      this.addComponentListener(new ComponentAdapter() {
         @Override
         public void componentResized(ComponentEvent e) {
            Component c = (Component)e.getSource();
            Dimension dim = c.getSize();
            canvasWidth = dim.width;
            canvasHeight = dim.height;
            // Сообщаем, что хотим, что бы наш холст при изменении размера окна заполнял все доступное пространство окна 
            box.set(0, 0, canvasWidth, canvasHeight);
         }
      });
  
      // Начинаем нашу игровую симуляцию
      gameStart();
   }
   
   /** Начинает игровую симуляцию */
   public void gameStart() {
      // Отделяем логику обработки игровых объектов в новый поток
      Thread gameThread = new Thread() {
         public void run() {
            while (true) {
               // Обновляем состояние симуляции на один шаг
               gameUpdate();
               // Вызываем перерисовку экрана
               repaint();
               // Задержка
               try {
                  Thread.sleep(1000 / UPDATE_RATE);
               } catch (InterruptedException ex) {}
            }
         }
      };
      gameThread.start();  // Запускаем метод из самого метода снова
   }
   
   /** 
    * Метод обрабатывающий состояния объекта на шаг вперед
    * Обновляем свойства игровых объектов, проверяем необходимые условия
    */
   public void gameUpdate() {
      ball.moveOneStepWithCollisionDetection(box);
   }
   
   /** Расширяем стандартную JPanel для переопределения поведения  отрисовки компонента */
   class DrawCanvas extends JPanel {
      /** Определяем что будем рисовать */
      @Override
      public void paintComponent(Graphics g) {
         super.paintComponent(g);    // Отрисовываем базовые вещи для компонента
         // Отрисовываем коробочку и шарик
         box.draw(g);
         ball.draw(g);
         // Отображаем информацию о шарике
         g.setColor(Color.WHITE);
         g.setFont(new Font("Courier New", Font.PLAIN, 12));
         g.drawString("Ball " + ball.toString(), 20, 30);
      }
  
      /** Метод вызываемый для получения размера нашего холста (нужно для переопределения размера на лету) */
      @Override
      public Dimension getPreferredSize() {
         return (new Dimension(canvasWidth, canvasHeight));
      }
   }
}

```

Класс `BallWorld` расширяет класс `JPanel`, используется в качестве главного отображения (*master view*) окна нашей симуляции. Главное отображение вмещает в себе дочерние формы отображения (sub-panels). В данном случае оно содержит всего одну дочернюю форму (наш специализированный холст с шариком и контейнером). Это позволит нам в дальнейшем добавить дочернюю панель с контрольными элементами. 

Класс `BallWorld` содержит в себе управляющю логику (управляющую состоянием объектов): `gameStart()`, `gameUpdate()`. 

Внутри класс содержит три объекта, состоянием которых он управляет: Шарик, Контейнер, Холст.

Конструктор создает графику нашей игры, игровые объекты и запускает логику отвечающую за обработку нашей симуляции (игры). Он принимает два аргумента, ширину и высоту, которые в дальнейшем определяют размер нашего холста, через метод `getPreferredSize()`. Эти величины также участвуют во время срабатывания события ресайза окна.

Отрисовка игровых объектов производится в *DrawCanvas*, наследуемый от JPanel. В нем мы работаем в методе `paintComponent(Graphics)`. Вызов этого метода напрямую запрещен, но возможен через обрытный вызов в этом же компоненте посредством `repaint()`. *DrawPanel* создан как внутрненний класс, в главном классе, для обесепчения доступа к приватным полям основного класса, в частности, к игровым объектам. 

Метод `gameStart()` запускает симуляцию в отдельном потоке. Игровой цикл повторяется, вычисляется следующий шаг симуляции, происходит проверка на пересечение границ, обновление позиций игровых объектов, отрисовка графики, уход потока в сон на время проходящее между обновлениями экрана. 

![Игровой цикл]({{ site.baseurl }}\images\2019-12-20\Game_cycle.png)

Игровой цикл запускается в отдельном потоке, с переопределенным поведением в методе `run()`. Многопоточность еще не раз пригодится нам для разработки игр. Многопоточность используется также в графической подсистеме или в *Event Dispatch Thread* (EDT), которая обрабатывает различные события ввода (клики, нажатия клавиш), запускает методы перехватывающие другие события, обновляет экран. Если EDT перестанет отвечать, то окно нашей игры перестанет обновляться, события ввода также обрабатываться не будут, это приводит к "зависанию" окна. 

Метод `run()` нельзя вызвать напрямую, но можно вызвать опосредованно из метода `start()`. Статический метод `Thread.sleep()` усыпляет поток на переданное в метод число миллисекунд. Используя усыпление потока мы получаем два преимущества: обеспечиваем необходимую задержку между обновлением окна игры, а также даем возможноть другим потокам получить возможность выполниться, в частности, дать возможность поработать потоку графического интерфейса. 

Игровой цикл, в данном случае, довольо прямолинейный. На каждом шаге игрового цикла, мы сдвигаем шарик и проверяем пересек он границу или нет. Если пересек, то производим необходимые поправки. Далее обновляем экран, вызывая `repaint()`.

# Основной (Главный) класс

```java
import javax.swing.JFrame;
/**
 * Основной класс запускающий программу с двигающимся шариком
 */
public class Main {
   // Точка входа программы
   public static void main(String[] args) {
      //запуск отрисовки в  Event Dispatcher Thread (EDT) внутри основного потока
      javax.swing.SwingUtilities.invokeLater(new Runnable() {
         public void run() {
            JFrame frame = new JFrame("A World of Balls");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setContentPane(new BallWorld(640, 480)); // Размещаем BallWorld на панели
            frame.pack();            // Вычисляем размер, доступный для BallWorld
            frame.setVisible(true);  // Отображаем содержимое окна 
         }
      });
   }
}



```

Основной класс содержит точку входа нашей программы - метод `main()`. Метод `main()`начинает исполнение нашего кода, с отрисовки окна нашей программы. Объект класса *BallWorld* инстанцируется в методе и передается в *frame*, как его содержимое. Использование `invokeLater()` для создания внешнего вида является стандартом и рекомендуется разработчиком. Таким образом обеспечивается потокобезопасность нашего приложения (подробнее на [Java Swing Online Tutorial](http://java.sun.com/docs/books/tutorial/uiswing)).

В вашей программе, вы можете описать метод `main()` в классе *BallWorld*, и опустить его в *Main*. 

Давайте попробуем запустить нашу программу и попробуем изменить размер окна. 


# Запуск программы, как апплета

Вместо класса *Main* мы будем использовать класс *MainApplet* для запуска программы как java-applet. 

```java
import javax.swing.JApplet;
/**
 * Основной класс, запускающий апплет
 * Создаст окошко размером 600х480
 */
public class MainApplet extends JApplet {
   @Override
   public void init() {
      // Run UI in the Event Dispatcher Thread
      javax.swing.SwingUtilities.invokeLater(new Runnable() {
         public void run() {
            setContentPane(new BallWorld(640, 480));
         }
      });
   }
}
```

Наш апплет наследуется от базового JApplet, использует `init()` внутри *main* для начала исполнения. 

# Распространяем приложение в виде JAR файла

Наша программа содержит множество классов. Возникает вопрос, каким образом мы можем поделиться ею с другими программистами или пользователями? Ответ: использовать всего один файл JAR. JAR - формат файла похожий на ZIP, который содержит сжатый контент (для открытия такого файла можно использовать WinZIP или WinRAR). Более того, jar-файл может запускаться без распаковки его содержимого.

Сейчас мы подготовим нашу программу для преобразования ее в jar-файл, для ее работы как самостоятельного приложения. 

**Распространяем самостоятельное приложение в JAR-файле:** для запуска такого приложения, jar-файл должен содержать *манифест* (например, для нашей программы "BallWorld.manifest") для определения классов принадлежащих приложению, и точки входа в приложение: (с другой стороны мы можем определить точку входа в приложение при помощи атрибута "code" внутри тэга applet)  

```shell
Manifest-Version: 1.0
Main-Class: Main
```
Запустим утилиту jar из комплекта JDK из командной строки и напишем следующую строку ('c'- создание,'v'-расширенный вывод, 'm'-манифест, 'f'-для именования файла):

```shell
> jar cvmf BallWorld.manifest ballworld.jar *.class
```
Вы можете использовать Java с опцией "-jar" для запуска самостоятельного приложения из JAR-файла или просто кликнуть по самому файлу. Манифест сообщает JVM с какой точки начинать исполнение приложения.   

```shell
> java -jar ballworld.jar
```
Если вашей программе для работы необходимы другие jar-файлы, то необходимо указывать их в свойстве "Class-Path". Все значения в jar-файле разделены пробелами. Не нужно указывать полный путь до jar-файлов

```shell
Manifest-Version: 1.0
Main-Class: Main
Class-Path: collisionphysics.jar another.jar
```

**Распространение приложения в JAR-архиве:** перед запуском необходимо также добавить тэги в файл манифеста: 

```html
<html>
<head><title>A Bouncing Ball</title></head>
<body>
  <h2>A Bouncing Ball</h2>
  <applet code="MainApplet.class" 
        width="640" height="480"
        archive="ballworld.jar">
  </applet>
</body>
</html>
```

