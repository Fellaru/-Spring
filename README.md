<em>Привет всем! Сегодня мы поговорим о Spring Framework, Spring MVC, Spring Boot и о многом другом! Надеемся, что эта статья станет для вас хорошей опорой в освоении современных WEB-технологий.
</em>

<h4>Что же такое этот «Spring»?</h4>

<strong>Spring</strong> – это целая группа проектов, упрощающих разработку на JAVA. Такие как Spring Framework, Spring Data, Spring Security, Spring Boot и многие другие. Почитать о них вы можете <a target="_blank" href="http://spring-projects.ru/projects/">тут</a>. Коротко про основные рассмотренные  в нашей статье проекты Spring:

• <em>Spring Framework (или Spring Core)</em>
Ядро платформы. Основа Spring. Неявно используется всеми остальными проектами. Две его основные особенности: внедрение зависимостей (DI) и аспектно-ориентированное программирование (AOP). Обеспечивает базовую поддержку управления компонентами (ApplicationContext, bean), внедрение зависимостей, управление транзакциями веб-приложений, доступа к данным.

•<em>Spring MVC (это часть Spring Framework)</em>
Не является отдельным проектом, но стоит отдельного упоминания, так как для темы нашей статьи – это важная часть. Каркас, основанный на HTTP и сервлетах. Spring MVC - это фреймворк для разработки Web-приложений.


•<em>Spring Boot</em>
Spring Boot — это «вишенка на торте». Это Framework для Framework’а. Он помогает быстро сконфигурировать и поднять приложение через аннотации и использует автоконфигурацию по максимуму (использует стандартные, дефолтные настройки).  Он объединяет вместе все остальные проекты Spring и рассчитан на быстрый старт работы с этой технологией. Для того, чтобы упростить жизнь разработчикам и новичкам, которые сходу не могут настроить проект, появился проект Spring Boot. Он имеет заранее подготовленные куски конфигурации, которые активируются в зависимости от различных условий. 



<h4>IoC, DI, Beans</h4>

Прежде чем начинать более близкое знакомство со Spring, необходимо разобраться с концепциями, на которых основывается Spring: IoC и DI. Давайте начнем с сухих определений, а потом разберём подробнее:

▪	<em>Inversion of Control (инверсия управления программой)</em> —  
это достаточно общее понятие, которое отличает библиотеку от фреймворка. Классическая модель подразумевает, что <em>вызывающий код контролирует внешнее окружение</em> и время и порядок вызова библиотечных методов. Однако в случае фреймворка обязанности меняются местами: <em>фреймворк</em> предоставляет некоторые точки расширения, через которые он <em>вызывает</em> определенные методы пользовательского кода. Будет уместно назвать эту концепцию как «вынесение контроля системы на уровень фреймворка».

▪	<em>Dependency Injection (внедрение зависимостей)</em> — это одна из реализаций принципа IoC (помимо этого есть еще Factory Method, Service Locator). Статья к прочтению: Dependency Injection vs. Service Locator.

▪	IoC-контейнер — это какая-то библиотека, фреймворк, программа, если хотите, которая позволит вам упростить и автоматизировать написание кода с использованием данного подхода настолько, насколько это возможно. Статья к прочтению: Inversion of Control Containers and the Dependency Injection pattern

Очень хорошая и понятная статья на русском <a target="_blank" href="http://sergeyteplyakov.blogspot.ru/2014/11/di-vs-dip-vs-ioc.html">вот тут</a>. Прочитайте ее, пожалуйста. 
Стало ли вам все понятно? Если да, то можете смело переходить к следующей части статьи. Если нет, то давайте разберем данную парадигму практически. 



<h4>Рассмотрим DI на примере.</h4>

У нас есть две версии кода. Первая - некорректна относительно концепции DI, а вторая корректна.
Давайте сначала посмотрим на некорректную реализацию: как мы с вами обычно реализуем класс – даем ему звучное название, и делаем его ответственным за получение собственных ссылок на объекты, с которыми он взаимодействует. По такому принципу и создан класс <em>“YesNoController”</em>, главная задача которого, как мы узнаем позже, обрабатывать приходящие запросы и возвращать нужный результат гадания (метод <em>“predict”</em>). Он взаимодействует с простым объектом <em>“YesNoTeller”</em>, который случайным образом возвращает либо да, либо нет.

<code class="java">
public class YesNoController {

    YesNoTeller yesNoTeller = new YesNoTeller();

    public String predict(){
        return  yesNoTeller.tell();
    }
}
</code>

Такая реализация приводит к сильной связанности <em>“YesNoController”</em>-а (главная задача которого, повторюсь, обрабатывать входящие запросы на гадание ) с классом <em>“YesNoTeller”</em>, что в конечном счете приведёт к сложностям при тестировании <em>“YesNoController”</em>-а (тестировать придется сразу их оба). Т.е. написав junit тест для <em>YesNoController</em>’а  мы неявно будем тестировать и <em>YesNoTeller</em> (возвращаемый результат  <em>YesNoController</em>’а  зависит от результата работы <em>YesNoTeller</em>). И так же этот класс становится «одноразовым», т.е. в такой реализации он пригоден только для да/нет гаданий.
В противовес же в корректной реализации мы создали класс “<em>FortuneController</em>”. 

<code class="java">
public class FortuneController {

    @Autowired
    FortuneTeller fortuneTeller;

    public String predict(){
        return  fortuneTeller.tell();
    }
}
</code>

Во-первых, имя этого класса не привязано к определенному типу гаданий. Во-вторых, этот класс взаимодействует с <em>“YesNoTeller”</em> через интерфейс <em>“FortuneTeller”</em>, что позволяет нам в будущем внедрить в значение этого поля более сложные реализации гаданий – например, гадания на картах Таро или астрологические гадания. А для тестирования мы можем внедрить в поле <em>fortuneTeller</em> любой простой класс реализующий интерфейс <em>Fortune Teller</em>, который бы возвращал бы определённый результат (например только да) и уже более чисто протестировать <em>FortuneController</em>.

Я приведу еще один пример плохого кода, а вы попробуйте найти здесь недостатки, что мы обсуждали выше. Здесь <em>client</em> – это простой класс с двумя полями <em>id</em> и <em>fullName</em>, а <em>eventLogger</em> класс с одним методом <em>logEvent</em>. Класс <em>App</em> логирует какие-то event’ы. В данном случае данные клиента

<code class="java">
public class App {
    Client client;
    EventLogger eventLogger;

    public static void main(String[] args) {
        App app = new App();
        app.client = new Client("1", "JohnSmith");
        app.eventLogger = new EventLogger();

    }

    public void logEvent(String msg){
        String message = msg.replaceAll(client.getId(), client.fullName);
        eventLogger.logEvent(message);
    }
}
</code>

Хорошо проанализировав код можно увидеть такие недостатки :
•	сложно модифицировать, так как данные <em>clienta</em> жестко вшиты в код и при изменения клиента нам придется переписывать код
•	Сложности при расширении кода 
•	Сложности в тестировании, так как косвенно тестируем и работу <em>EventLogger</em>

<h4>А теперь пробежимся по основам Spring Framework</h4>

В приложении на основе фреймворка Spring прикладные объекты располагаются внутри контейнера Spring: 

<img src="https://atom.mail.ru/media/uploads/atom/topics/2017/11/dd977893b9a7728b4360f577ff5de366b531b175584161c9a9937421fc999eb9.jpeg" title="" />

Как показано на рисунке выше - контейнер создает объекты (<em>Bean</em>). Это отличная метафорическая аналогия – ведь наши бины (Bean – с англ. «бобы») в контейнере лежат как «бобы в банке». Контейнер связывает их друг с другом, конфигурирует (определяет свойства этого объекта) и управляет их полным жизненным циклом – от зарождения до самой их смерти (или от оператора new до вызова метода finalize)
В фреймфорк Spring входит несколько реализаций контейнера, но основной это <em>“Application Context”</em>. Контекст приложений основан на понятии фабрик компонентов и реализует прикладные службы фреймворка - такие как возможность приема текстовых сообщений из файлов свойств, он обеспечивает поддержку упомянутого ранее DI
Что бы поднять <em>“ApplicationContext”</em> в Spring Boot вам достаточно использовать данный шаблон:
<code class="java">
@SpringBootApplication
public class Application {

   public static void main(String[] args) {
      SpringApplication.run(Application.class, args);
   }
}
</code>
Каким образом Spring должен понять, что должен быть создан экземпляр какого-то определенного класса? Все просто по особой аннотации этого класса. Таких аннотаций в Spring Framework четыре:

•	<em>@Component</em> - универсальная аннотация, указывающая, что класс является компонентом Spring и позволяющая ему находить этот класс

•	<em>@Controller</em> – указывает, что класс определяет контроллер Spring MVC, об этом дальше

•	<em>@Repository</em> - указывает, что класс определяет репозиторий данных

•	<em>@Service</em> – указывает, что класс определяет службу

•	Так же вы можете создать свою собственную аннотацию для Spring (читайте в документации)


Любой класс аннотированный одной из этих 4-х аннотаций (или вашей кастомной) будет поднят в Application Context.
На примере нашего кода, как вы можете заметить, у нас есть 2 класса помеченных этими особыми аннотациями:
<code class="java">
@Component
public class FortuneController {

    @Autowired
    FortuneTeller fortuneTeller;

    @RequestMapping("/")
    @ResponseBody
    public String predict(){ return  fortuneTeller.tell();}
}
</code>
И
<code class="java">
@Component
public class YesNoTeller implements FortuneTeller{
    final Random random = new Random();

    public String tell(){
      int i =  random.nextInt(2);
        if( i == 0){
            return "No";
        } else {
            return "Yes";
        }
    }
}
</code>

Значит эти оба класса будут подняты в нашем Application Context.
Перейдем, наконец, к самому главному. 

А как Spring понимает куда внедрить объект класса, который он поднял и какой именно объект нужно внедрять из всех объектов, что он поднял?

Для того, чтобы Spring понял куда ему внедрить зависимость на объект мы используем аннотацию <em>@Autowired</em>, которую можно поставить в 4 разных места

• Аннотировать ей <em>setter’ы</em> полей класса (или любые другие методы, изменяющие значения поля класса)
<code class="java">
@Autowired
public void setFortuneTeller(FortuneTeller fortuneTeller) {
    this.fortuneTeller = fortuneTeller;
}
</code>

•	Аннотировать ей конструктор класса
<code class="java">
@Autowired
public FortuneController(FortuneTeller fortuneTeller) {
    this.fortuneTeller = fortuneTeller;
}
</code>

•	Аннотировать ей само поле класса
<code class="java">
@Autowired
FortuneTeller fortuneTeller;
</code>

Но будьте внимательны - в одном классе используйте только один из предложенных вариантов, не пытайтесь аннотировать несколькими способами одновременно. 

А для того, чтобы Spring понял какой именно объект необходимо внедрять из всех объектов, что он поднял -  используется возможность автоматического связывания. Принцип автоматического связывания таков: 
Если в контексте приложения имеется только один компонент <em>YesNoTeller</em> типа <em>FortuneTeller</em>, тогда любой компонент, имеющий свойство типа <em>FortuneTeller</em>, будет зависеть именно от этого компонента <em>YesNoTeller.</em>

Ещё раз повторим этот принцип с пояснениями для новичков: 
Если в контексте приложения (<em>Application Context</em>) имеется только один компонент типа <em>FortuneTeller</em> (в нашем примере это <em>YesNoTeller</em> (он реализует интерфейс <em>FortuneTeller</em>, и, следовательно, является компонентом типа <em>FortuneTeller</em>)), тогда любой компонент, имеющий свойство типа <em>FortuneTeller</em> (свойство = поле класса, следовательно, в нашем примере такой компонент это <em>FortuneController</em>), будет зависеть именно от этого компонента <em>YesNoTeller</em>. 

А что если в <em>ApplicationContext</em> поднято 2 bean’a c типом <em>FortuneTeller</em>? Ну тогда автоматическое связывание не сработает и Spring не станет за вас решать, какой именно объект внедрить, а просто возбудит исключение <em>NoSuchBeanDefinitionException</em>. Решить эту проблему можно уточнением неоднозначных зависимостей. Для этого используется аннотация <em>@Qualifier</em>(“УникальноеИмя”)

<h6>Пример:</h6> 
Появился новый класс <em>ZodiacTeller</em>, который реализует генерацию гороскопа для всех знаков зодиака. Он тоже, как и <em>YesNoTeller</em> типа <em>FortuneTeller</em>

<code class="java">
@Qualifier("zodiac")
@Component
public class ZodiacTeller implements FortuneTeller {
    
    @Override
    public String tell() {
        //Здесь ваша реализация 
        return "Horoscope";
    }
}
</code>

Наш старый класс <em>YesNoTeller</em> в упрощенном виде:

<code class="java">
Qualifier("yesno")
@Component
public class YesNoTeller implements FortuneTeller {

    public String tell() {
        //Здесь ваша реализация
        return "YesOrNo";
    }
}
</code>

И, соответственно, новая реализация <em>FortuneController</em> с уточнением, какой именно bean нам нужен:
<code class="java">
@Component
public class FortuneController {

    @Qualifier("yesno")
    @Autowired
    FortuneTeller fortuneTeller;

    @RequestMapping("/")
    @ResponseBody
    public String predict(){ return  fortuneTeller.tell();}
}
</code>

Мы научились внедрять объекты, но что делать если я захочу внедрить простое значение, например String, int или boolean?
 Для этого применяется анотация  <em>@Value</em>. Ее применение заключается в том , что бы добавить аннотацию <em>@Value</em> перед свойством, методом или параметром метода и передать ей строку с выражением, результат которого должен быть внедрен. Например:

<code class="java">
@Value("Texnoatom")
String coursesName;
</code>

Но это еще не все. Все таки внедрять значение прямо над полем, может показаться не очень хорошей идеей. Значение поля мы можем указать в application.properties файле, а уже после внедрять в свойство. Делается это так:

•	В application.properties определяем свойство
<code class="java">
courses = Texnoatom
</code>

•	И вот так внедряем в его значение в поле:
<code class="java">
@Value("${courses}")
String coursesName;
</code>

<h4>ТЕПЕРЬ ПРО MVC
</h4>
Это следующая концепция, которую необходимо понять и уяснить. Здесь мы можем начать с картинки:
<img src="https://atom.mail.ru/media/uploads/atom/topics/2017/11/02f094e7dfdd0827b096be9c291bb3df386ae1dc9b7661a2bcd12b83735730f3.png" title="" />

Что же мы тут видим? Во-первых, мы можем увидеть расшифровку аббревиатуру<em> MVC - Model View Controller.</em>


Давайте разберемся детально, зачем нужна каждая из этих компонент:
•	<em>Модель</em> (Model) предоставляет данные и реагирует на команды контроллера, изменяя свое состояние.
•	<em>Представление</em> (View) отвечает за отображение данных модели пользователю, реагируя на изменения модели.
•	<em>Контроллер</em> (Controller) интерпретирует действия пользователя, оповещая модель о необходимости изменений. 



Как это работает?
У нас есть <em>view</em> - это, грубо говоря, web-страничка с “чемоданчиком”. В этом чемоданчике всё, что используется при создании страницы. Пользователь начинает взаимодействовать с тем, что видит. Например, нажимает кнопку. В этот момент происходит запрос в контроллер, который вызывает нужную модель. А та создает новую view и так по кругу. 
А теперь пойдем от модели к Spring MVC (к практической реализации).

<img src="https://atom.mail.ru/media/uploads/atom/topics/2017/11/3827ce791ae154f9396b804a9d7414fc6f75388628ec856bfe51bfca6f0f416d.png" title="" />

<em>DispatcherServlet</em> принимает и обрабатывает все HTTP-запросы и ответы на них. Сначала он обращается в HandlerMapping, тот по виду запроса (например, /login) определяет какой контроллер (какой метод класса контроллера) обрабатывает этот запрос. Потом <em>DispatcherServlet</em> обращается в метод <em>Controller-а</em> и получает две вещи: имя view и чемоданчик (да-да, тот самый чемоданчик). 

Здесь по запросу пользователя “/” <em>DispatcherServlet</em> бежит к <em>HandlerMapping</em> и узнает, что метод main обрабатывает данный запрос. Потом он бежит в этот метод и берёт из него имя view и чемоданчик (в данном случае имя: loginViews/index, а в чемоданчике какой-то список, загружаемый методом getUserNameList, который называется userList и объект LoginUser с именем userJSP). 

Что же дальше? 
Мы продолжаем рассказ о жизни DispatcherServlet. У него есть какое-то имя, теперь он бежит к “<em>viewResolver</em>”-у, который добавляет при необходимости префикс и суффикс к имени view (в данном случае он может добавить префикс WEB-INF/views и суффикс .jsp. В итоге мы получим WEB-INF/views/loginViews/index.jsp). Как только Dispatcher узнал полное имя view. Он его находит и отдает вместе с чемоданчиком модели (MVC, Model), которая отправляет всё на клиента и отрисовывает.
<a target="_blank" href="https://habrahabr.ru/post/336816/">Cтатья к прочтению</a>

<h6>БЫСТРЫЙ СТАРТ</h6>
1)	Создадим каркас. Для этого можно воспользоваться <a target="_blank" href="https://start.spring.io/">веб-конструктором:</a> 

<img src="https://atom.mail.ru/media/uploads/atom/topics/2017/11/d07526435585ba00ca9008a12eb9ad79c02f599758c360e15703f90503ae2379.png" title="" />

2)	Создадим Controller и интерфейс для объекта, реализующего логику гадания:

<code class="java">
@Controller
public class FortuneController {

    @Autowired
    FortuneTeller fortuneTeller;

    @RequestMapping("/")
    @ResponseBody
    public String predict(){ return  fortuneTeller.tell();}
}
</code>

<code class="java">
public interface FortuneTeller {
    public String tell();
}
</code>

3)	Реализуем интерфейс FortuneTeller, который будет случайным образом генерировать «да» или «нет»:

<code class="java">
@Component
public class YesNoTeller implements FortuneTeller{
    final Random random = new Random();
    public String tell(){
      int i =  random.nextInt(2);
        if( i == 0){
            return "No";
        } else {
            return "Yes";
        }
    }
}
</code>

4)	Запускаем приложение.

5)	А теперь загадываем вопрос, который требует ответ «да» или «нет», переходим по адресу “localhost:8080” и смотрим, какой ответ на ваш вопрос нагадал наш предсказатель.

<h4>ОСНОВЫ SPRING BOOT</h4>

Как мы уже писали ранее Spring Boot - «вишенка на торте». Spring Boot позволяет быстро и легко написать приложение с использованием Spring, которое можно будет запустить одной командой. 
SpringBoot предоставляет вам Servlet container “из коробки” (по умолчанию используется Tomcat).
Обсудим начальный проект, сгенерированный веб-конструктором.


Конфигурация gradle:
Spring Boot использует особые зависимости, которые называются Starters (Стартеры).

<code class="java">
dependencies {
   compile('org.springframework.boot:spring-boot-starter-web')
   testCompile('org.springframework.boot:spring-boot-starter-test')
}
</code>

<em>Starter’ы</em> – Это целый набор зависимостей Spring Framework и других необходимых зависимостей для быстрого старта проекта, сгруппированные по темам (Web, JPA, Security и т.д.). Список стартеров есть в документации SpringBoot. Все официальные стартеры имеют определенное название “spring-boot-starter-*”, где * - это название типа применения этой зависимости (неофициальные стартеры, или же ваши кастомные, в противовес имеют название типа: “*- spring-boot-starter”)
	
Так же особенностью “build.gradle” для приложения на SpringBoot является специальный плагин Spring Boot для Gradle

<code class="java">
plugins {
    id 'org.springframework.boot' version '1.5.8.RELEASE'
}
</code>

или 

<code class="java">
buildscript {
   repositories {
      mavenCentral()
   }
   dependencies {
      classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.8.RELEASE")
   }
}
</code>

Можно и без него, но этот плагин делает ваши jar и war файлы запускаемыми.

Теперь перейдем к нашим классам. 
Запускающим классом в SpringBoot является класс типа:

<code class="java">
@SpringBootApplication
public class FortuneApplication {

   public static void main(String[] args) {
      SpringApplication.run(FortuneApplication.class, args);
   }
}
</code>


То есть ваше приложение начинается выполнением класса <em>FortuneApplication</em>. Этот класс аннотирован через <em>@SpringBootApplication</em>. Что это за аннотация и о чём она говорит?

<em>@SpringBootApplication</em> – аннотация, специфичная для SpringBoot и эквивалентна трем аннотациям:<em> @Configuration, @EnableAutoConfiguration</em> и <em>ComponentScan</em> из SpringFramework с их свойствами по умолчанию.

<img src="https://atom.mail.ru/media/uploads/atom/topics/2017/11/cdbfd27d47b7f061791bde5ca2f197a43913d21443a99e1eff232985626b675c.png" title="" />

Поясню, что обозначает каждая из этих 3-х аннотаций из SpringFramework, чтобы в конечном счёте понять, что делает аннотация <em>@SpringBootApplication.</em>

▪	<em>@Configuration</em> – она сообщает что это класс конфигураций с помощью которого мы можем сконфигурировать работу Spring контекста. Подсказать ему  каким образом он должен работать (вот тут станет понятней)

▪	<em>@ComponentScan.</em> Если конфигурационный класс помечен этой аннотацией, то Spring автоматически поднимает все Spring компоненты, включая @Configuration класс. Если структура вашего кода стандартная (т.е. расположение Application class (FortuneApplication) в корневом пакете), вы можете добавить @ComponentScan без каких-либо аргументов. В противном случае вы должны прописать имя пакета, внутри которого нужно искать Spring компоненты. Все компоненты Spring приложения (@Component, @Service, @Repository, @Controller etc.) будут автоматически зарегистрированы как Spring Beans.

▪	<em>@EnableAutoConfiguration</em> (хватает только одной такой аннотации в любой класс, аннотированный @Configuration). Включает автоконфигурацию твоего Spring приложения основываясь на зависимостях, добавленных в build.gradle. 

Некоторые рекомендации, которые стоит соблюдать для корректной работы со SpringBoot: 
не используйте “default” package и старайтесь расположить main application class в корневом пакете относительно других классов