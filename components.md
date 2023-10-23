# Компонентная архитектура
<!-- Состав и взаимосвязи компонентов системы между собой и внешними системами с указанием протоколов, ключевые технологии, используемые для реализации компонентов.
Диаграмма контейнеров C4 и текстовое описание. 
-->
## Компонентная диаграмма

```plantuml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

AddElementTag("microService", $shape=EightSidedShape(), $bgColor="CornflowerBlue", $fontColor="white", $legendText="microservice")
AddElementTag("storage", $shape=RoundedBoxShape(), $bgColor="lightSkyBlue", $fontColor="white")

Person(admin, "Администратор", "Владелец маркетплейса")
Person(seller, "Продавец", "Реализует собственную продукцию на платформе")
Person(buyer, "Покупатель", "Покупает желаемый товар у любого из продавцов")

System_Ext(web_site, "Клиентский веб-сайт", "HTML, CSS, JavaScript, React", "Веб-интерфейс")

Rel(seller, web_site, "Создание страницы магазина, добавление карточки товара, изменение информации о товаре, удаление товара")
Rel(buyer, web_site, "Просмотр карточек товаров, поиск товаров по маске названия, добавление товара в корзину, оформление доставки товара")
Rel(admin, web_site, "Модерация товаров, рекомендательная система товаров для покупателей, предоставление аналитики продавцам, поиск продавца и покупателя по маске")


System_Boundary(conference_site, "Маркетплейс") {
  '  Container(web_site, "Клиентский веб-сайт", ")
   Container(auth_service, "Сервис авторизации", "C++", "Сервис управления пользователями", $tags = "microService")    
   Container(item_service, "Сервис создания карточки товара", "C++", "Создания карточки товара продавцом", $tags = "microService") 
   Container(suggestion_service, "Сервис ленты товаров", "C++", "Создание ленты товаров для пользователя", $tags = "microService")
   Container(recsys_service, "Рекомендательная система", "C++", "Определение релевантных товаров для пользователя", $tags = "microService")
   ContainerDb(db, "База данных", "MariaDB", "Хранение данных о товарах, продавцах и пользователях", $tags = "storage")
   
}

Rel(web_site, auth_service, "Авторизация покупателя/продавца", "localhost/auth")
Rel(auth_service, db, "INSERT/SELECT/UPDATE", "SQL")

Rel(web_site, item_service, "Работа с карточкой товара продавцом", "localhost/construct_item")
Rel(item_service, db, "INSERT/SELECT/UPDATE", "SQL")

Rel(web_site, recsys_service, "Работа рекомендательной системы", "localhost/recommend")
Rel(recsys_service, db, "INSERT/SELECT/UPDATE", "SQL")

Rel(web_site, suggestion_service, "Работа с блогами", "localhost/main")
Rel(suggestion_service, db, "INSERT/SELECT/UPDATE", "SQL")

@enduml
```
## Список компонентов  

### Сервис авторизации
**API**:
-	Создание нового пользователя
      - входные параметры: login, пароль, имя, фамилия, отчество, email, обращен (г-н/г-жа), дата рождения, пол, город проживания
      - выходные параметры: отсутствуют
-	Поиск пользователя по логину
     - входные параметры:  login
     - выходные параметры: имя, фамилия, email, обращение (г-н/г-жа)
-	Поиск пользователя по маске имени и фамилии
     - входные параметры: маска фамилии, маска имени
     - выходные параметры: login, имя, фамилия, email, обращение (г-н/г-жа)

### Сервис управления карточкой товара
**API**:
- Создание карточки товара
  - Входные параметры: название товара, категория, цена, производитель, описание, картинка товара, рейтинг, id продавца товара
  - Выходыне параметры: id товара
- Изменение карточки товара:
  - Входные параметры: id карточки товара, словарь актуальных параметров в виде json 
  - Выходные параметры: отсутствуют
- Удаление товара:
  - Входные параметры: id карточки товара
  - Выходные параметры: отсутствуют

### Сервис просмотра карточек товаров
**API**
- Получение списка товаров для стартовой страницы:
  - Входные данные: массив id релевантных товаров для пользователя, полученнные из рекоменлательной системы
  - Выходные данные: массив карточек релевантных товаров
- Поиск товара
  - Входные параметры: атрибуты карточки товара
  - Выходные параметры: массив id искомого товара и товаров из искомой категории (искомый товар на первом месте массива, далее релевантные)
- Просмотр категорий товаров:
  - Входные данные: категория товара
  - Выходные данные: массив карточек релевантных товаров 

### Сервис рекомендательная система
**API**:
- Рекомендация для клиента
  - Входные параметры: атрибуты клиента из сервиса авторизации и агрегаты по его активности 
  - Выходные параметры: массив id 10 наиболее релевантных карточек к покупке



### Модель данных
```puml
@startuml

class Company {
  id
  c_name
  c_first_name
  c_last_name
  c_sex
  c_email
  c_city
  c_adress
  c_rating
  c_registry_date
  c_bank_accaunt
  b_phone_number
}

class Item_category {
  id
  ic_category_name
}

class Item_card {
  id
  c_id
  i_title
  ic_id
  i_price
  i_rating
  i_description
  i_change_date
}

class Buyer {
  id
  b_login
  b_first_name
  b_last_name
  b_email
  b_date_of_birth
  b_sex
  b_city
  b_adress
  b_registry_date
  b_phone_number
}


Item_card <|-- Item_category
Item_card <- Company
Buyer <- Item_card

@enduml
```