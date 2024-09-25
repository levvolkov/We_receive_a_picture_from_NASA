# "Получаем картинку от NASA за конкретную дату"

### Предусловие

* Установка и настройка [IDEA](https://github.com/netology-code/jdfree-homeworks/tree/jdfree-6/01#readme) на компьютер.
    * Все настройки проекта как в интрукции за исключением: Шаг 9. В появившемся окне в поле "Language" выберите "Java", в поле "System" выберите "Maven" вместо "IntelliJ".
* Несмотря на то, что API NASA публичный, доступ к нему предоставляется по ключу, который достаточно просто получить по адресу: https://api.nasa.gov/. Перейдя по ссылке, заполняем личными данными поля:
    * First Name
    * Last Name
    * Email

На почтовый адрес будет выслан api_key. С этим ключом нужно делать запросы к API.

------

### Реализация
**1. Создаем проект `maven` и добавляем в pom.xml библиотеку [Apache HttpClient](https://mvnrepository.com/search?q=Apache+HttpClient) внутри тега `<dependencies>`.**
```
      <dependencies>
        <dependency>
            <groupId>org.apache.httpcomponents.client5</groupId>
            <artifactId>httpclient5</artifactId>
            <version>5.3</version>
        </dependency>
     </dependencies>
```

**2. Создаем файл `config.properties` для хранения ключа который Nasa отправила на почту и записываем туда ключ. Не забыв добавить файл `config.properties` в `gitignore`**
```
api_key=ключ полученный на почту
```
**3. В папке java пишем код программы.**
* Создаем класс `Main.java`.
* Метод с которого начинает работать любая программа.
```
public static void main(String[] args) {
```
* Создаем новый объект Properties, который будет использоваться для хранения пар "ключ-значение" из файла конфигурации.
```
Properties properties = new Properties();
```
* Загружаем данные из файла 'config.properties'. Этот файл должен находиться в корневом каталоге проекта или указан путь к нему. Это может привести к IOException, если файла не существует.
```
properties.load(new FileInputStream("config.properties"));
```
* Извлекаем значение параметра "api_key" из загруженных свойств и сохраняем его в переменную apiKey. Если ключа нет, то apiKey будет равно null.
```
String url = "https://api.nasa.gov/planetary/apod?api_key=" + apiKey;
```
* Создаем экземпляр HTTP-клиента, который будет использоваться для выполнения HTTP-запросов. Этот клиент автоматически управляет соединениями (см. документацию библиотеки [Apache HttpClient](https://hc.apache.org/httpcomponents-client-4.5.x/quickstart.html)).
```
CloseableHttpClient httpclient = HttpClients.createDefault();
``` 
* Составляем  GET запрос в который передан адрес по которому нужно послать информацию за конкретную дату.
```
HttpGet request = new HttpGet(url + "&date=2024-09-05");
```
* Посылаем запрос методом execute с сохранением ответа в переменную и добавляем исключения в сигнатуру метода который предложила IDEA `throws IOException` (При возникновении ошибки ввода, вывода эта ошибка будет скинута на операционную систему).
```
CloseableHttpResponse response = httpclient.execute(request);
```
* Для того чтобы прочесть ответ добавляем сущность способную принимать поток входящих данных в программу.
```
Scanner sc = new Scanner(response.getEntity().getContent());
```
* Выводим на экран что лежит в этом сканере.
```
System.out.println(sc.nextLine());
```
**4. Запускаем программу, получаем строку ответа.**
* После получения строки ответа можно закоментировать часть кода, которая в дальнейшем будет мешать отрабатывать ему (Ответ сервера можно прочитать один раз либо Scanner либо ObjectMapper, который будет применяться в дальнейшем).
```
//        Scanner sc = new Scanner(response.getEntity().getContent());
//        System.out.println(sc.nextLine());
```
**5. Создаем отдельный json файл `answer.json` в который помещаем скопированную строку ответа из консоли (ключ-значение).**
```
{
  "copyright": "Eric Benson",
  "date": "2024-09-05",
  "explanation": "About 70,000 light-years across, NGC 247 is a spiral galaxy smaller than our Milky Way. Measured to be only 11 million light-years distant it is nearby though. Tilted nearly edge-on as seen from our perspective, it dominates this telescopic field of view toward the southern constellation Cetus. The pronounced void on one side of the galaxy's disk recalls for some its popular name, the Needle's Eye galaxy. Many background galaxies are visible in this sharp galaxy portrait, including the remarkable string of four galaxies just below and left of NGC 247 known as Burbidge's Chain. Burbidge's Chain galaxies are about 300 million light-years distant. NGC 247 itself is part of the Sculptor Group of galaxies along with shiny spiral NGC 253.",
  "hdurl": "https://apod.nasa.gov/apod/image/2409/NGC247-Hag-Ben2048.JPG",
  "media_type": "image",
  "service_version": "v1",
  "title": "NGC 247 and Friends",
  "url": "https://apod.nasa.gov/apod/image/2409/NGC247-Hag-Ben1024.JPG"
}
```
**6. Переводим формат json в формат который java будет понимать.**
* Cоздаем класс `NasaAnswer.java` в папке java, который будет полностью соответствовать формату `answer.json` файла.
* Заводим поля моего класса.
```
public class NasaAnswer {
    String date;
    String explanation;
    String hdurl;
    String media_type;
    String service_version;
    String title;
    String url;
    String copyright;
```
**7. В pom.xml добавляем библиотеку [Jackson Databind](https://mvnrepository.com/search?q=Jackson+Databind).**
 ```
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.17.0</version>
        </dependency>
```
**8. Для того, чтобы связывать класс `NasaAnswer` c json форматом, генерируем конструктор со специальными особенностями.**
```
 public NasaAnswer(@JsonProperty("date") String date,
                      @JsonProperty("explanation") String explanation,
                      @JsonProperty("hdurl") String hdurl,
                      @JsonProperty("media_type") String media_type,
                      @JsonProperty("service_version") String service_version,
                      @JsonProperty("title") String title,
                      @JsonProperty("url") String url,
                      @JsonProperty("copyright") String copyright) {
        this.copyright = copyright;
        this.date = date;
        this.explanation = explanation;
        this.hdurl = hdurl;
        this.media_type = media_type;
        this.service_version = service_version;
        this.title = title;
        this.url = url;
    }
```
**9. Для того чтобы преобразовывать ответ сервера в объект класса `NasaAnswer`.**
* В классе Main cоздаем объект ObjectMapper из библиотеки Jackson, который будет использоваться для (де)сериализации JSON – преобразования JSON-строк в Java-объекты и наоборот.
```
ObjectMapper mapper = new ObjectMapper();
```
* Создаем объект класса `NasaAnswer`, с помощью `mapper` у которого есть метод `readValue` указываем откуда мы должны прочитать данные `response.getEntity().getContent()` и обязательно указываем к какому типу данных этот ответ сервера преобразовать `NasaAnswer.class`.
```
NasaAnswer answer = mapper.readValue(response.getEntity().getContent(), NasaAnswer.class);
```
* Копируем ссылку в отдельную переменную.
```
String imageUrl = answer.url;
```
* Для скачивания картинки делаем новые GET запрос.
```
HttpGet imageRequest = new HttpGet(imageUrl);
```
* Создаем массив который методом `split` разбивает по слешу /.
```
String[] urlSplitted = imageUrl.split("/");
```
* Дальше нужно обратиться к последнему элементу массива берем длинну массива -1 `[urlSplitted.length - 1]`.
```
String fileName = urlSplitted[urlSplitted.length - 1];
```
* Посылаем запрос заставляя `httpclient` выполнить задание.
```
CloseableHttpResponse image = httpclient.execute(imageRequest);
```
* Чтобы записать картинку ввиде файла используем стандартые средства которые встроены в Java, например `FileOutputStream` специальный класс который позволяет записывать бинарные данные в файл с указанием где будет распологаться картинка(для этого в проекте создадим папку `Image`).
```
FileOutputStream fos = new FileOutputStream("Image/" + fileName);
```
* Обращаемся к `image` где хранится ответ от сервера, который включает в себя картинку, `.writeTo(fos)` записывайся сюда.
```
image.getEntity().writeTo(fos);
```
**10. Запускаем программу, получаем картинку за указанную дату.**