## PagePagination-Search-Filter-Ordering
(Task_Tracker - ToDo projesi üzerinden )

### Django proje kurulumu:

```py
- python -m venv env
- ./env/Scripts/activate 
- pip install djangorestframework
- pip install python-decouple
- pip freeze > requirements.txt
- django-admin startproject core .
- python manage.py runserver 
```



### Repodan çektiğimiz bir projeyi requirements deki indirilen paket/versiyonları ile birlikte install etmek için kullanılacak komut ->
```py
- python -m venv env
- ./env/Scripts/activate 
- pip install -r requirements.txt
```

- .gitignore ve .env dosyalarını oluştur.
- settings.py daki SECRET_KEY ' i config ile .env ye ekle

settings.py
```py
from decouple import config
SECRET_KEY = config('SECRET_KEY')
```

.env
```py
SECRET_KEY =django-insecure-&2_9wl^*c1v&z-x0g121-qceca2nm&tys+=a_!$9(6x28vech&
```



- superuser oluştur ->

```powershell
- python manage.py createsuperuser
- python manage.py runserver
```

### Faker

https://faker.readthedocs.io/en/master/

- Bu ders için bir veri yığınına ihtiyacımız var. 
  Bunun için faker kullanacağız.

### Faker

```powershell
- pip install faker
- pip freeze > requirements.txt
```

- create faker.py under the todo app
-faker.py
```py
'''
    # https://faker.readthedocs.io/en/master/
    $ pip install faker # install faker module
    python manage.py flush # delete all exists data from db. dont forget: createsuperuser
    python manage.py shell
    from student_api.faker import run
    run()
    exit()
'''

from .models import Todo
from faker import Faker

def run():
    fake = Faker() # Faker(["tr-TR"])
    for todo in range(200):
        Todo.objects.create(
            title = fake.sentence(),
            description = fake.text(),
            priority = fake.random_int(min=1, max=3),
            is_done = fake.boolean()
        )
    print('Finished')
```



```powershell
python manage.py shell
from todo.faker import run
run()
exit()
```



### Pagination - Filter - Search


### Pagination

- Pagination rest framework ile default olarak 
  geliyor. ekstra modül yüklemeye gerek yok.

- Şimdi biz dummy data ürettik ve bunların 
  üzerinde pagination işlemleri yapacağız.

https://www.django-rest-framework.org/api-guide/pagination/

- dokümanı incelediğimizde üç tip pagination olduğunu gördük.
  - PageNumberPagination (En çok kullanılan)
  - LimitOffsetPagination
  - CursorPagination


### PageNumberPagination 
- En çok kullanılan, en baştan 10 kayıt getir!

### LimitOfsetPagination 
- 51 den sonraki 10 kayıt getir

### CursorPagination
- Gizlilik esas (kaç kayıt olduğunun belli olmasını istemediğimiz durumlarda kullanılır.)



#### PageNumberPagination
- Bir sayfada şu kadar olsun diyoruz ve sayfa 
  sayfa api yi ayırıyor.


- Pagination ayarları yaparken iki tip ayar vardır;
  1- Global ayar, (genel ayar) 
  2- Local ayar, (bölgesel ayar)

##### 1- Global ayar ile PageNumberPagination :

- Genel olarak projenin tamamında aynı 
  pagination kuralları geçerli olsun istiyorsak bunu kullanabiliriz.
- Pagination kullanmak için settings.py a 
  aşağıdaki kodları yazacağız.
- İlk önce PageNumberPagination kullanacağımız 
  için pagination olarak onu yazdık.

settings.py ->

```py
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20
}
```

- sayfalandırma yapıldığını ve her sayfada 20 
  data nın gösterildiğini görüyoruz.

- api ye baktığımızda, artık datanın results 
  içerisine alındığımı görüyoruz,

- count olarak data sayısını verdi,
- next olarak bir sonraki sayfanın linkini 
  verdi,

- previous olarak bir önceki sayfanın linkini 
  verdi,


##### 2- Local ayar ile PageNumberPagination :

- View bazında pagination kurallarını 
  ayarlamak için kullandığımız yöntemdir. Pagination Customize yapacağız. 
- Önce bir pagination.py isminde bir file 
  create ediyoruz.
- Global de kullandığımız (settings.py da 
  yazdığımız) rest_framework.pagination dan PageNumberPagination ı import ediyoruz.
- PageNumberPagination ı custom yapacağız, 
  PageNumberPagination dan inherit ederek bir class tanımlıyoruz, CustomPageNumberPagination ismini veriyoruz.
- Şimdi biz burada local olarak bir pagination 
  yazıyoruz ama settings.py da da golabal olarak PageNumberPagination belirlemiştik. Sıkıntı değil, global olarak uygular ancak locale gelince farklı bir pagination var ise localde kini dikkate alır, aynı css (external, internal, inline) de olduğu gibi.
- view de custom pagination yaptığımızda artık 
  settings.py da belirlediğimiz global ayarları kullanmayacak.


- Şimdi gidip pagination.py daki CustomPageNumberPagination 
  class ımızın custumize ını tamamlayalım.
  page_size = 5
  page_query_param = 'sayfa'   yaptık.

pagination.py ->

```py
from rest_framework.pagination import (
    PageNumberPagination,
)

class CustomPageNumberPagination(PageNumberPagination):
    page_size = 5
    page_query_param = 'sayfa'
```



- TodoView view imizde dokümanda belirtildiği 
  gibi djangonun istediği şekilde view imize ekliyoruz; (Tabi kullanabilmek için import ediyoruz.)
  from .pagination import CustomPageNumberPagination
  pagination_class = CustomPageNumberPagination

views.py ->

```py
# pagination imports
class TodoView(ModelViewSet):
    
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer

    pagination_class = CustomPageNumberPagination
    
```

- Artık TodoView view imiz globalde tanımladığımız 
  PageNumberPagination yerine CustomPageNumberPagination ı kullanacak.



- Test edelim, evet çalıştı, her sayfada page_size olarak 
  belirttiğimiz 5 data gösteriyor. Artık global yerine, PageNumberPagination ı customize ettiğimiz CustomPageNumberPagination ile pagination yapıyoruz.


- CustomPageNumberPagination da başka ayarlar yapalım, 
  mesela;
  page_query_param = 'sayfa'   diyelim. url de sayfa=2 şeklinde gösterdik.


NOT: 
- Concrete Viewslerde veya ModelViewSetlerde otomatik 
  algılanıyor. Diğerlerinde Function viewslerde veya apiview lerden inherit edildiğinde pagination kullanacağınızın ayrıca belirtilmesi gerekiyor.
- Genelde Concrete veya MVS kullanıldığı için oradan 
  çalışıyoruz.



#### LimitOffsetPagination
- Bir aralık veriyoruz, mesela 20 ile 50 arasını 
  getir.

##### 1- Global ayar ile LimitOffsetPagination :

- Önceki pagination kodlarını yoruma alıyoruz, 
  (settings.py daki global ve views.py daki local kodları yoruma alıyoruz.)
- LimitOffsetPagination kullanmak için settings.py a 
  aşağıdaki kodları yazacağız.


settings.py ->

```py
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination',
    'PAGE_SIZE': 20
}
```

- sayfalandırma yapıldığını ve her sayfada 20 
  data nın gösterildiğini görüyoruz.

- api ye baktığımızda, artık datanın results 
  içerisine alındığını görüyoruz,

- count olarak data sayısını verdi,

- next olarak bir sonraki sayfanın linkini 
  verdi, limitoffset olduğunu da belirtti,

- previous olarak bir önceki sayfanın linkini 
  verdi, limitoffset olduğunu da belirtti,

- url kısmından limit ve offset değerlerini değiştirerek paginationda değişiklikler yapabiliyoruz.


##### 2- Local ayar ile LimitOffsetPagination :

- Daha önce create ettiğimiz pagination.py file a 
  gidip custom pagination umuzu yazıyoruz.
- Global de kullandığımız (settings.py da 
  yazdığımız) rest_framework.pagination dan LimitOffsetPagination ı import ediyoruz.
- LimitOffsetPagination ı custom yapacağız, 
  LimitOffsetPagination dan inherit ederek bir class tanımlıyoruz, CustomLimitOffsetPagination ismini veriyoruz.
- view de custom pagination yaptığımızda artık 
  settings.py da belirlediğimiz global ayarları kullanmayacak, yoruma alabiliriz ya da silebiliriz.
- Şimdi gidip pagination.py daki CustomLimitOffsetPagination 
  class ımızın customize ını tamamlayalım.
  default_limit = 10   yaptık.


pagination.py ->

```py
from rest_framework.pagination import (
    PageNumberPagination,
    LimitOffsetPagination,
)

class CustomPageNumberPagination(PageNumberPagination):
    page_size = 5
    page_query_param = 'sayfa'

class CustomLimitOffsetPagination(LimitOffsetPagination):
    default_limit = 10
    limit_query_param = 'adet'
    offset_query_param = 'baslangic'
```

- Biz bu oluşturma aşamasındaki 
  CustomLimitOfsetPagination ı TodoView view imizde local olarak kullanacağız. 
- TodoView view imizde dokümanda belirtildiği 
  gibi djangonun istediği şekilde view imize ekliyoruz; (Tabi kullanabilmek için import ediyoruz.)
  from .pagination import CustomLimitOffsetPagination
  pagination_class = CustomLimitOffsetPagination

views.py ->

```py
# pagination imports
from .pagination import (
    CustomPageNumberPagination,
    CustomLimitOffsetPagination,
)

class TodoView(ModelViewSet):
    
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer

    # pagination_class = CustomPageNumberPagination
    pagination_class = CustomLimitOffsetPagination
```

- Artık TodoView view imiz globalde tanımladığımız 
  LimitOffsetPagination yerine CustomLimitOffsetPagination ı kullanacak.


- Test edelim, evet çalıştı, her sayfada default_limit olarak 
  belirttiğimiz 10 data gösteriyor. Artık global yerine, LimitOffsetPagination ı customize ettiğimiz CustomLimitOffsetPagination ile pagination yapıyoruz.

- Limit değişmiyor, 10ar 10ar bir sonrakine geçiyor.
 

- CustomLimitOffsetPagination da başka ayarlar yapalım, 
  mesela;
    limit_query_param = 'adet'
    offset_query_param = 'baslangic'



#### CursorPagination
- Bulunduğu konumdan sonraki, kaç tane isteniyorsa 
  o kadar getiriyor.
- Kullanıcının diğer datalar hakkında kestirimde bulunmasını engelleyen bir güvenlik de sağlıyor.

##### 1- Global ayar ile CursorPagination :

- Önceki pagination kodlarını yoruma alıyoruz, 
  (settings.py daki global ve views.py daki local kodları yoruma alıyoruz.)
- CursorPagination kullanmak için settings.py a 
  aşağıdaki kodları yazacağız.

settings.py ->

```py
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.CursorPagination',
    'PAGE_SIZE': 20
}
```

- Bu şekilde çalıştırırsak bize hata verecektir. 
  Çalışması için bir referans noktasına ihtiyacı var. Bizden db de default olarak "created" isminde bir date field/sütun istiyor. Bu sütuna/field a göre bir sıralama yapıyor, imleci o sıralama üzerinde hareket ettiriyor.

- Bunun için bir "created" field ı oluşturmamız 
  lazım. modelimize gidiyoruz, "created" isminde field oluşuruyoruz.   
  created = models.DateTimeField(auto_now_add=True)

models.py ->

```py
from django.db import models

class Todo(models.Model):
    
    PRIORITY = (
        (1, 'High'),
        (2, 'Medium'),
        (3, 'Low')
    )

    title = models.CharField(max_length=64)
    description = models.TextField(null=True, blank=True)
    priority = models.PositiveSmallIntegerField(choices=PRIORITY, default=2)
    is_done = models.BooleanField(default=False)
    created_date = models.DateTimeField(auto_now_add=True)
    updated_date = models.DateTimeField(auto_now=True)
    created = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```


```powershell
- py manage.py makemigrations
Select an option: 1
[default: timezone.now] >>> timezone.now 
- py manage.py migrate
```


- sayfalandırma yapıldığını gördük ve "next" 
  kısmındaki linkte "cursor" karşısında karışık karakterden oluşan bir set var. 

- Biz tüm datalara otomatik aynı create date i verdiğimiz için sanki bir tane data varmış gibi davranıyor. Bunu aşmak için;

- Admin panelden yeni bir veri girişi yapınca çalışıyor.



##### 2- Local ayar ile CursorPagination :

- Daha önce create ettiğimiz pagination.py file a 
  gidip custom pagination umuzu yazıyoruz.
- Global de kullandığımız (settings.py da 
  yazdığımız) rest_framework.pagination dan CursorPagination ı import ediyoruz.
- CursorPagination ı custom yapacağız, 
  CursorPagination dan inherit ederek bir class tanımlıyoruz, CustomCursorPagination ismini veriyoruz.
- view de custom pagination yaptığımızda artık 
  settings.py da belirlediğimiz global ayarları kullanmayacak, yoruma alabiliriz ya da silebiliriz.

- pagination.py daki CustomCursorPagination 
  class ımızın custumize ını tamamlayalım.
  cursor_query_param = "imlec"
  page_size = 10
  ordering = 'id'   yaptık.


pagination.py ->

```py
from rest_framework.pagination import (
    PageNumberPagination,
    LimitOffsetPagination,
    CursorPagination,
)

class CustomPageNumberPagination(PageNumberPagination):
    page_size = 5
    page_query_param = 'sayfa'

class CustomLimitOffsetPagination(LimitOffsetPagination):
    default_limit = 10
    limit_query_param = 'adet'
    offset_query_param = 'baslangic'

class CustomCursorPagination(CursorPagination):
    cursor_query_param = "imlec"
    page_size = 10
    ordering = 'id'
    # ordering = '-created'


```


- CustomCursorPagination ı TodoView view imizde local olarak kullanacağız. 
- TodoView view imizde dokümanda belirtildiği 
  gibi djangonun istediği şekilde view imize ekliyoruz; (Tabi kullanabilmek için import ediyoruz.)
  from .pagination import CustomCursorPagination
  pagination_class = CustomCursorPagination

views.py ->

```py
# pagination imports
from .pagination import (
    CustomPageNumberPagination,
    CustomLimitOffsetPagination,
    CustomCursorPagination,
)

class TodoView(ModelViewSet):
    
    queryset = Student.objects.all()
    serializer_class = StudentSerializer

    # pagination_class = CustomPageNumberPagination
    # pagination_class = CustomLimitOffsetPagination
    pagination_class = CustomCursorPagination
```

- Artık TodoView view imiz globalde tanımladığımız 
  CursorPagination yerine CustomCursorPagination ı kullanacak.



- Test edelim, evet çalıştı, her sayfada page_size 
  olarak belirttiğimiz 10 data gösteriyor. Artık global yerine, CursorPagination ı customize ettiğimiz CustomCursorPagination ile pagination yapıyoruz.

- ordering kısmında belirttiğimiz field ı (id) referans 
  alarak ona göre sıralı gösteriyor.

- sayfa sayısı göstermiyor.
 

- Evet pagination konusu bu kadar. 
- Şimdi views.py da en çok kullanılan CustomPageNumberPagination ı active hale getiriyoruz. Bundan sonra böyle çalışacağız.



### FILTER

- Filter özelliği rest framework ile default olarak 
  gelmiyor. djangorestframework-filters paketini kurmamız gerekiyor.

```powershell
- pip install django-filter
- pip freeze > requirements.txt
```

- settings.py da INSTALLED_APPS e 'django_filters' olarak ekliyoruz.

settings.py ->

```py
INSTALLED_APPS = [
    'django.contrib.staticfiles',
    # Modules:
    'rest_framework',
    'django_filters',
    # Apps:
    'todo',
]

```

https://www.django-rest-framework.org/api-guide/filtering/


- Yine filter da da pagination da olduğu gibi global ve local olmak üzere iki tip filtreleme ayarlayabiliyoruz.

##### 1- Global ayar ile Filter :

- Genel olarak projenin tamamında aynı 
  filter kuralları geçerli olsun istiyorsak bunu kullanabiliriz.
- filter kullanmak için settings.py a 
  aşağıdaki kodları yazacağız.


- settings.py da, REST_FRAMEWORK kısmında 
  'DEFAULT_FILTER_BACKENDS': ['django_filters.rest_framework.DjangoFilterBackend'],
  olarak ekliyoruz.

settings.py ->

```py
REST_FRAMEWORK = {
    # for filtering
    'DEFAULT_FILTER_BACKENDS': ['django_filters.rest_framework.DjangoFilterBackend'],
}
```

- Sonraki aşama; nasıl ki kullandığımız view de 
  pagination ı tanıtıyoruz, filter için de 
    filterset_fields = ['title', 'description']
  yazarak filtreleme için kullanacağı fieldları belirtiyoruz.

views.py ->

```py
from .pagination import (
    CustomPageNumberPagination,
    CustomLimitOffsetPagination,
    CustomCursorPagination,
)

class TodoView(ModelViewSet):
    
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer

    pagination_class = CustomPageNumberPagination

    filterset_fields = ['title', 'description']
```

- runserver ile çalıştırıyoruz ve filter butonunun 
  geldiğini gördük. Filtreleme yaptık, çalışıyor. case sencitive dir. Büyük/küçük harf duyarlıdır.

- filter benzer karakterleri değil, eşit olan 
  karakterleri bulup getirir.



##### 2- Local ayar ile Filter :

- View bazında filter kurallarını 
  ayarlamak için kullandığımız yöntemdir. Filter Customize yapacağız. 
- settings.py da global olarak yaptığımız filter 
  ayarını yoruma alıyoruz,
- views.py daki view imizde filter işlemleri 
  yapacağız, ama önce settings.py da yoruma aldığımız DjangoFilterBackend i views.py da django_filters.rest_framework dan import etmeliyiz.
- Arkasından view imizde; 
    filter_backends = [DjangoFilterBackend]
  ekliyoruz.
- global filter ayarında olduğu gibi; 
    filterset_fields = ['title', 'description']
  filterset fieldlarımızı da yazıyoruz,

views.py ->

```py
from django_filters.rest_framework import DjangoFilterBackend

class TodoView(ModelViewSet):
    
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer

    pagination_class = CustomPageNumberPagination
    
    filter_backends = [DjangoFilterBackend]
    filterset_fields = ['title', 'description']
```

- Çalıştırıyoruz, evet çalışıyor, filter linki geliyor ve filterset_filter da belirttiğimiz fieldlar ile filtreleme yapabiiyoruz.




### SEARCH

- search, like gibi çalışıyor. filter ise 1'e 1 eşitini arıyor, 
  
- Search rest framework ile default olarak 
  geliyor. ekstra modül yüklemeye gerek yok.

- Yine filter ve pagination da olduğu gibi global ve local olmak üzere iki tip search ayarlayabiliyoruz.

##### 1- Global ayar ile Search :

- Genel olarak projenin tamamında aynı 
  search kuralları geçerli olsun istiyorsak bunu kullanabiliriz.

- search kullanmak için settings.py a 
  aşağıdaki kodları yazacağız.

- settings.py da, REST_FRAMEWORK kısmında 
    'DEFAULT_FILTER_BACKENDS': ['rest_framework.filters.SearchFilter'],
  olarak ekliyoruz.

settings.py -> 

```py
# Search Global
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': ['rest_framework.filters.SearchFilter']
}
```


views.py ->

```py
    search_fields = ['title', 'description']
```


##### 2- Local ayar ile Search :

- View bazında search kurallarını 
  ayarlamak için kullandığımız yöntemdir. Search Customize yapacağız. 
- settings.py da global olarak yaptığımız search 
  ayarını yoruma alıyoruz,

- views.py da rest_framework.filters dan 
  SearchFilter ı import ediyoruz.

- filter_backends kısmına import ettiğimiz  
  SearchFilter ı da ekliyoruz.
    filter_backends = [DjangoFilterBackend, SearchFilter]

- Ve hangi fieldlar içinde search yapsın istiyorsak o 
  fieldları belirtiyoruz;
    search_fields = ['title', 'description']

views.py ->
```py
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.filters import SearchFilter

class TodoView(ModelViewSet):
    
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer

    pagination_class = CustomPageNumberPagination
    
    filter_backends = [DjangoFilterBackend, SearchFilter]
    filterset_fields = ['title', 'description']
    
    search_fields = ['title', 'description']
```



### ORDERING

- views.py da rest_framework.filters 
  dan OrderingFilter import edilir.

- views.py da filter_backends kısmına 
  OrderingFilter eklenir.

views.py ->

```py
from rest_framework.filters import SearchFilter, OrderingFilter

    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
```

- Bu şekilde bırakılırsa tüm 
  fieldlarda ordering yapılması sağlanır.
  
- Ancak istenirse ordering_fields 
  verilerek sadece istenen fieldlarda ordering yapılması da sağlanabilir,
    ordering_fields = = ['title']
  sadece title fieldında ordering yapılmasına izin verir.

views.py ->

```py
from rest_framework.filters import SearchFilter, OrderingFilter

    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]

    ordering_fields = ['title']
```


- ordering_fields = ['title','description']  #* filter boxta hangi seçenekler çıksın istiyorsanız onu yazıyorsunuz
    
- ordering = ['title']  #* default olarak ilk açıldığında buraya yazdığımıza göre sıralıyor




## Perm. Auth.

# Permission - Authentication

Authentication => bir kişinin ya da bir şeyin iddia ettiği olduğunu doğrulamak için yapılan işlemdir. Bu genellikle bir kullanıcı adı ve parola gibi kimlik bilgileri ya da bir parmak izi veya yüz tanıma gibi biyometrik veriler kullanılarak yapılır.

Authorization =>  doğrulanmış kullanıcının izinlerine göre belirli kaynaklara veya eylemlere erişimi verme veya reddetme işlemidir. Örneğin, doğrulanmış bir kullanıcının belirli belgeleri okumaya izin verilmiş olabilir, ancak düzenleme veya silme yetkisi olmayabilir.


- Rest-framework de bazı ayarlar yapılırken global ve 
  view bazında olmak üzere iki şekilde yapılabiliyor. (pagination, filter, search te olduğu gibi.)

- Bu permission-authentication için de geçerli. Best 
  practice olarak;
   - authentication global,
   - permission view bazında ayarlanır,


## Permission

https://www.django-rest-framework.org/api-guide/permissions/

- 'DEFAULT_PERMISSION_CLASS' default olarak AllowAny 
  olarak ayarlanmıştır. (Herkese izin ver.)

- Permission ları view bazında ayarlamak best 
  practice. 
    - class_based viewlerde "permission_class" 
      attribute ünün içinde default permission ları veya custom permissionları tanımlıyoruz. (bir liste veya tupple içerisinde.)
         permission_classes = [IsAuthenticated]

    - function_based viewlerde decorator @ ile "permission_class" 
      attribute ünün içinde default permission ları veya custom permissionları tanımlıyoruz. (bir liste veya tupple içerisinde.)
          @permission_classes([IsAuthenticated])

- rest-framework default permission class lar;
  - AllwAny  -> Herkese izin ver.
  - IsAuthenticated  -> Login olmuş user.
  - IsAdminUser  -> staff statusü olan user.
  - IsAuthenticatedOrReadOnly  -> 
        Login olmuş usera herşeye izin ver, login olmamış usera sadece okuyabilsin.
  - DjangoModelPermissions  -> Grup tablosundaki userlara 
        tanımlanan permissionlar.
  - DjangoModelPermissionsOrAnonReadOnly  -> Grup tablosunda 
        tanımlanan userlardan bazılarına verilen permissionlar, gruptaki diğer userlar sadece read yapabilsin.
  - DjangoObjectPermissions  -> 
  - Custom Permissions  -> 
        - has_permission(seld, request, view)
        - has_object_permission(seld, request, view, obj)
  

## Authentication

https://www.django-rest-framework.org/api-guide/authentication/

- BasicAuthentication -> Kullanıcı adı ve şifre ile 
  yapılan, çok da güvenli olmayan bir yöntem. Çok önerilen bir authentication değil.

- TokenAuthentication -> Genel olarak kullanılan yapıdır. 
  Kullanıcı login olduğunda db de user için kullanıcı adı ve şifresinden bağımsız olarak unique bir token üretilir. Artık o token ile user ın statüsü belirlenip ona göre işlem yapılır. Logout olduğunda user ın token ı silinir. Tekrardan login olunca yeni bir token üretilir ve kullanıcıya tanımlanır, authentication ve permission işlemleri yapılır.

- SesionAuthentication -> Genelde template yapısı ile 
  kullanılır. Rest-frameworkün de kendi içerisinde kullanabileceği bir template yapısı var, ancak çok kullanılmıyor. 



## Authentication Ayarları

- Authentication ları global tanımlamak en mantıklı yöntem. 
  Tüm endpointleri bir authentication sistemi altına alıyoruz. 

https://www.django-rest-framework.org/api-guide/authentication/

### BasicAuthentication

#### BasicAuthentication Global 

- dokümanda belirtildiği şekliyle setting.py a 
  BasicAuthentication ı ekliyoruz. SessionAuthentication kullanmayacağız, silebiliriz.

settings.py ->

```py
    REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.BasicAuthentication',
    ]
}
```

- Şu an BasicAuthentication olarak globalimize username 
  ve password ile giriş yapılarak authenticate olunabilecek bir sistem tanımladık.

##### IsAuthentication

- Biz bir authentication tanımladık. Ancak authenticate 
  olmadan da endpointe istek attığımızda yine de verilere erişebiliyoruz. Burada permissionlar devreye giriyor. Mesela IsAuthenticated permissionu kullanarak eğer user authenticate değil ise user ı erişim engeli getirebiliriz. Deneyelim ;

- user authenticated ise yani IsAuthenticated True ise -> 
  şu endpointe ulaşabilsin!

- Bu işlemi views.py da yapıyoruz. Önce rest_framework.
  permissions dan IsAuthenticated permissionu import ediyoruz.(rest frameworkün permission larından IsAuthenticated permissionu import ediyoruz.) 

- views.py da StudentMVS calss based view ini 
  kullanıyoruz. permission_class olarak da IsAuthenticated kullan diyoruz.
    permission_classes = [IsAuthenticated]

views.py ->
```py
from rest_framework.permissions import IsAuthenticated

class TodoView(ModelViewSet):

    permission_classes = [IsAuthenticated]
```

- Artık view imizde import ederek yazdığımız bu kod 
  ile StudentMVS viewinin endpointi olan student/ 'a sadece authenticate olan user lar ulaşabilsin, herkes ulaşamasın! Default ayarımız neydi AllowAny yani herkes ulaşabilir. Şimdi yazdığımız kod ile bu class based viev ile Read, Create, Update, Delete, Get, Retrieve tüm methodlarda authenticate şartı getiriyoruz. Deniyoruz;

- api/student endpointine istek 
  attığımızda bir pop-up çıkartıp, bizden useraname ve password istiyor. 

- Bu yöntem kullanılabilir değil.
- Hali hazırda biz django admin 
  panelden kullanıcı oluşturarak işlem yapıyoruz. Ancak bu API mantığına ters. Bizim API da bir endpoint yazıp kullanıcıya frontend ten register, login, logout olabilmeleri için bilgilerini göndermelerini sağlayıp, backend de kullanıcının gönderdiği bilgilerle db de bir user create edeceğiz.

- Bundan sonraki get, post, put, delete isteklerini 
  postman dan yapacağız. BasicAuthentication ve IsAuthenticated ile postman den kullanıcı girişi yapmadan ve yaptıktan sonra olmak üzere istek atıyoruz. (Authorization -> Basic Auth -> username:, password:) Kullanıcı girişi yaptıktan sonra isteğimize yanıt veriyor, kullanıcı girişi yapmadan yaptığımız isteklere yanıt vermiyor.

##### IsAdminUser

- Bu işlemi views.py da yapıyoruz. Önce rest_framework.
  permissions dan IsAdminUser permissionu import ediyoruz.(rest frameworkün permission larından IsAdminUser permissionu import ediyoruz.) 

- views.py da StudentMVS view inde permission_class 
  olarak da IsAdminUser kullan diyoruz.
    permission_classes = [IsAdminUser]

views.py ->
```py
from rest_framework.permissions import (
    IsAuthenticated, 
    IsAdminUser,
)

class TodoView(ModelViewSet):

    permission_classes = [IsAdminUser]
```

- permissions ımızı IsAdminUser olarak ayarladık. Yani 
  kullanıcı admin ise işlemlere izin ver, değil ise izin verme!
- Hemen admin panelden staff statüsü olmayan bir user 
  create ediyoruz ve onun username ve passwordü ile işlem yapmaya çalışıyoruz.
- Admin olmayan user ile yani staff statüsü olmayan 
  user işlem yapmaya çalışınca izin vermiyor, admin user ile işlem yapınca izin veriyor.


##### IsAuthenticatedOrReadOnly

- Bu işlemi views.py da yapıyoruz. Önce rest_framework.
  permissions dan IsAuthenticatedOrReadOnly permissionu import ediyoruz.(rest frameworkün permission larından IsAuthenticatedOrReadOnly permissionu import ediyoruz.) 

- views.py da StudentMVS view inde permission_class 
  olarak da IsAdminUser kullan diyoruz.
    permission_classes = [IsAuthenticatedOrReadOnly]

views.py ->
```py
from rest_framework.permissions import (
    IsAuthenticated, 
    IsAdminUser,
    IsAuthenticatedOrReadOnly,
)

class TodoView(ModelViewSet):

    permission_classes = [IsAuthenticatedOrReadOnly]
```

- permissions ımızı IsAuthenticatedOrReadOnly olarak 
  ayarladık. Yani kullanıcı admin ise işlemlere izin ver, değil ise sadece read yapabilsin!
- staff statüsü olmayan bir user ile post ile create 
  etmeye çalışıyoruz izin vermiyor, anack get işlemine izin veriyor.
- Admin olan user, yani staff statüsü olan 
  user get ve post ile işlem yapmaya çalışınca izin veriyor.


### TokenAuthentication

#### TokenAuthentication Global 

- dokümanda belirtildiği şekliyle setting.py a 
  TokenAuthentication ı ekliyoruz. SessionAuthentication kullanmayacağız, silebiliriz.

- Ayrıca bir de INSTALLED_APPS e 'rest_framework.authtoken' 
  isminde bir app eklememiz gerekiyor. Bu app sayesinde kullanıcıyı ve o kullanıcıya ait key i tutan db de Token isminde bir tablo oluşturuluyor. Token tablosu, User tablosuyla OneToOne ilişkili.Bunu yazıp migrate yaptığımız zaman, authtoken ile ilgili bir tablo oluşacak, burada hangi user ın hangi token a sahip olduğunu tutan bir tablo oluşacak.

settings.py ->

```py
INSTALLED_APPS = [
    # 3rd party packages
    'rest_framework',
    'django_filters',
    'rest_framework.authtoken',
]


    REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ]
}
```

```powershell
- py manage.py migrate
```

- Admin panele gidip bakalım nasıl bir tablo eklenmiş. Auth 
  Token app inin altında Token tablosu eklenmiş. Burada tüm kullanıcılarımıza manuel olarak token oluşturabiliyoruz. Ancak kullanıcı register, login olduğunda otomatik oluşturmasını ve logout olduğunda da otomatik silinmesini sağlayacağız.

- Şimdilik tüm kullanıcılara manuel olarak token oluşturalım;


- Şu an TokenAuthentication olarak globalimize username 
  ve password ile giriş yapılarak authenticate olunabilecek bir sistem tanımladık.

- Şimdi bunu test etmek için permission ı IsAdminUser a 
  çevirelim, farkı daha net görelim.
    permission_classes = [IsAdminUser]
  Yani diyoruz ki user ın staff statüsü varsa, bu endpointten isteklere cevap ver!

- Postman e gidiyoruz, Authorization da No Auth seçip, Headers 
  kısmında en alta Key kısmına Authorization, Value kısmına Token yazıp, admin panelden admin userın token ını alıp bir boşluk bırakarak Token ın yanına yapıştırıyoruz. 
  Authorization -> Token ccfcd09df148d240c53a104195b9ca6a921029f2

- Test için Get methodu ile istek atıyoruz, admin olan user ın token ı ile 
  çalışıyor, admin olmayan user ın token ı ile çalışmıyor.

- Postman de code snippets kısmından da frontend de nasıl istek 
  atılacağı görülebilir.

## user app (for register, login, logout)

- Şimdi user register, login, logout işlemleri için yeni bir 
  app oluşturup tüm bu işlemleri bu app içerisinde yapacağız.

```powershell
- py manage.py startapp user
```

- user app imizi settings de INSTALLED_APPS e ekliyoruz.

settings.py ->
```py
INSTALLED_APPS = [
    
    # my_apps
    'student_api',
    'user',
]
```

### register

(default User modelinden faydalanarak bir register endpointi için serializers.py da serializer oluşturmaya çalışıyoruz.)

- user app içerisinde bir model oluşturmayacağız, djangonun 
  default olarak verdiği User modelini kullanacağız. 

- email zorunlu ve unique hale getirebiliriz. Bunu serializers 
  da yapacağız.

- Not: User modelini models.py da customize ettiğimiz zaman kendimize 
  ait bir User model oluşturmuş oluruz, onu specific olarak settings.py da 
    AUTH_USER_MODEL = 'JJHGHJ/NJKJK/USER'
  şeklinde yolunu belirtmemiz gerekir.
  Belirttiğimiz bu yol ile de Customize ettiğimiz User modelini kullanırken de buradan import etmemiz gerekir;
    from django.conf import settings
    settings.AUTH_USER_MODEL
  serializer da bu customize ettiğimiz modeli kullanmak için bu importları yapmamız  gerekir.


- Ancak burada customize yapmadığımız için, o yolu değil de default 
  User modelini kullanarak bir register endpointi oluşturacağız, bunun  için bir serializer yazmamız gerekiyor.
- Eğer User modelinde değişiklik yapmak istemiyorsak direkt class 
  Meta: diyip model = User ile devam edebiliriz.

- Ancak biz,  User modelimz de email zorunlu bir alan değil ama biz 
  zorunlu ve unique alan olsun diyoruz.

- Bunun için değişiklik yapacağımız fieldları belirtip ondan sonra 
  class Meta diyip devam ediyoruz.
    email = serializers.EmailField(required=True)  
  serializers da zorunlu alan için required=True diyoruz. blanck=True, null=True değil.
  
  Ayrıca unique olmasını istiyoruz, bunun için (şu linkten incelenebilir)
  https://www.django-rest-framework.org/api-guide/validators/
    from rest_framework.validators import UniqueValidator
    email = serializers.EmailField(required=True, validators=[UniqueValidator(queryset=User.objects.all())])

- Şu an modelimizdeki zorunlu bir alan olmayan email field ını, zorunlu ve 
  unique hale getirerek bu özelliğini değiştirdik. Bu işlemi db de değil de db ye kaydetmeden önce bu şekilde kontrol ederek db ye unique olarak kaydedilmesini sağladık. Ön taraftan abir kontrol sağlayarak, email gelecek, check edeceğiz, arkada bu email var mı ona bakacağız, varsa validation error, yoksa db ye kaydetmesi için göndereceğiz.

user/serializers.py ->

```py
from rest_framework import serializers
from django.contrib.auth.models import User
from rest_framework.validators import UniqueValidator

class RegisterSerializer(serializers.ModelSerializer):
    email = serializers.EmailField(required=True, validators=[UniqueValidator(queryset=User.objects.all())])
    
    class Meta:
        model = User
```

- Şimdi password field ına özellik ekleyelim; 
  password normalde ilk olarak CharField olarak giriliyor, django daha sonra sha256 ile criptoluyor, ama ilk başta girildiğinde string bir ifade. Şöyle bir ifade var write_only=True -> Bu fieldı sadece yazma işlemi yaparken post, put yaparken kullan, read yaparken yani get yaparken dönme demek. 
    Register create ettik, bize response olarak user ın bilgileri
  dönecek, o dönen bilgilerde yani get ile çağrıalan bilgilerde password ü göstermesini istemiyoruz. Sadece yazma işlemlerinde post, put işlemlerinde bu field ı kullanmasını belirtmek için write_only=True parametresini kullanıyoruz.
    (Bir de read_only=True var o da post put isteklerinde kullanma,
  sadece get isteklerinde bu field ı dön demektir.)

    Bir de password2 = serializers.CharField(write_only=True, required=True)
  diye User modelimizde olmayan bir field tanımlıyoruz, bunu da password confirmation için tanımlıyoruz. Bu fieldı sadece password ü doğru girip girmediğini kontrol için isteyeceğiz. Ayrıca bunu required=True zorunlu bir alan yapıyoruz. password için required=True demedik, çünkü zaten User modelimizde password zorunlu bir alan.

    Bu serializer a User modelimizde 2 field ın özelliğini değiştirdik,
  bir de password confirm için ekstradan password2 field ı ekledik.

    Sonra modelimizin altında hangi fieldları kullanacağımızı 
  yazıyoruz,  
    fields = (
      'id',
      'username',
      'email',
      'first_name',
      'last_name',
      'password',
      'password2',
    )

user/serializers.py ->

```py
from rest_framework import serializers
from django.contrib.auth.models import User
from rest_framework.validators import UniqueValidator

class RegisterSerializer(serializers.ModelSerializer):
    email = serializers.EmailField(required=True, validators=[UniqueValidator(queryset=User.objects.all())])
    password = serializers.CharField(write_only=True)
    password2 = serializers.CharField(write_only=True, required=True)
    # first_name = serializers.CharField(required=True)
    
    class Meta:
        model = User
        fields = (
            'id',
            'username',
            'email',
            'first_name',
            'last_name',
            'password',
            'password2',
        )
```

1- 
  - Şimdi biz normalde serializer yazarken ModelSerializer 
    kullandığımızda def creaet() methodunu override etmemize gerek yoktu ama biz burada bu modelde olmayan bir field (password2) kullandık.
2- 
  - Ayrıca password ü de direkt db ye save edeceği için, hash lenmemiş birşekilde db ye save edecektir. Bunu hash lenmiş bir şekilde göndermek için özel bir method kullanıyoruz. Onu da uygulayabilmemiz için def create() methodunu override etmemiz gerekiyor.  
3- 
  - password ve password2 için birbiri ile uyuşuyor mu diye validation yapmamız gerekiyor. 

- Şimdi bu yukarıdaki işlemleri yapacağız. İndentaions a 
  dikkat, class Meta ile aynı seviyede olacak.

- (3) Önce ismi sabit olan def validate() methodunu 
  kullanıyoruz. password leri kontrol ediyoruz eğer eşit değilse hata raise ediyoruz. Eğer bir hata yoksa data yı return et!
    def validate(self, data):
        if data['password'] != data['password2']:
            raise serializers.ValidationError(
              {"password" : "Password fields didn't match"
            )
        return data

```py
    def validate(self, data):
        if data['password'] != data['password2']:
            raise serializers.ValidationError(
                {"password": "Password fields didn't match."}
            )
        return data
```

- (1-2) validated_data, create ederken, db ye gönderirken 
  hangi fieldları göndereceğimizi, alacağımız dataya işaret ediyor. Buradaki işlemde password2 yi kullanmayacağız, create ederken kullanmayacağız. Bizim için olması gerekenler neler? username, email, first_name, last_name, password (Bu fieldlar var bizim db mizde, modelimizde, password2 yok). Bizim bu fields dictionary si içerisindeki bir elemanı çıkarmamız yeterli olur. Bunu nasıl çıkarıyorduk? Python dictionary methodlarından pop() methodu ile bir dictionary içerisinden password2 elemanını çıkarabiliriz. 
    validate_data.pop('password2')

  User create ederken password ü sonrasında ekleyeceğiz, yani user create ederken password ü de kullanmayacağız, password field ını create ederken göndermemize gerk yok, daha sonrasında bu password ü hash lenmiş bir şekilde kaydetmesini söyleyeceğiz. O yüzden password field ını da pop() ile çıkarıyoruz ama birazdan kullanacağımız password değişkenine tanımlıyoruz.
    password = validated_data.pop('password')
  
  Bu hali ile yani password field ı olmadan elimizde kalan fieldlarla User modelinde bir user create edip onu da user değişkenine tanımlıyoruz,
    user = User.objects.create(**validated_data) # ** şu demek -> username=validate_data['username], email=validate_data['email'], ....
  
  Password ü olmadan create ettiğimiz user a set_password() methodu ile bir üst satırda aldığımız string password datasını sha256 ile cryptolayıp db ye o şekilde kaydedilmesini sağlıyor.
    user.set_password(password)
  
  User ın password ünü de sha256 ile crytolanmış bir şekilde ekledikten sonra, save() methoduyla db ye kaydediyoruz.
    user.save()
  
  En son create ettikten sonra user ın datalarını dönmesini istiyoruz.
    return user

```py
    def create(self, validated_data):
        validated_data.pop('password2') # password2 kullanılmayacağı için dict den çıkardık.
        password = validated_data.pop('password') # password u daha sonra set etmek için değişkene atadık.
        user = User.objects.create(**validated_data) # ** şu demek -> username=validate_data['username], email=validate_data['email'], ....
        user.set_password(password) # password un encryte olarak db ye kaydedilmesini sağlıyor.
        user.save()
        return user
```


user/serializers.py ->

```py
from rest_framework import serializers
from django.contrib.auth.models import User
from rest_framework.validators import UniqueValidator

class RegisterSerializer(serializers.ModelSerializer):
    email = serializers.EmailField(required=True, validators=[UniqueValidator(queryset=User.objects.all())])
    password = serializers.CharField(write_only=True)
    password2 = serializers.CharField(write_only=True, required=True)
    # first_name = serializers.CharField(required=True)
    
    class Meta:
        model = User
        fields = (
            'id',
            'username',
            'email',
            'first_name',
            'last_name',
            'password',
            'password2',
        )
    
    def validate(self, data):
        if data['password'] != data['password2']:
            raise serializers.ValidationError(
                {"password": "Password fields didn't match."}
            )
        return data
    
    def create(self, validated_data):
        validated_data.pop('password2') # password2 kullanılmayacağı için dict den çıkardık.
        password = validated_data.pop('password') # password u daha sonra set etmek için değişkene atadık.
        user = User.objects.create(**validated_data) # ** şu demek -> username=validate_data['username], email=validate_data['email'], ....
        user.set_password(password) # password un encryte olarak db ye kaydedilmesini sağlıyor.
        user.save()
        return user

```

- RegisterSerializer ımızı yazdık. Artık user create etmek için 
  bir view oluşturmamız gerekiyor. Bu view de sadece create yapmak istiyoruz, bunun için bir class based view imiz vardı. Concrete View lerden CreateAPIView. Bunu kullanabiliriz.

- views.py da RegisterSerializer serializerımızı, 
  django.contrib.auth.models dan User modelimizi,
  ve rest_framework.generics den CreateAPIView i import edip, (Biz burada sadece CreateAPIView kullanacağız.)

- RegisterView imizi CreateAPIView den inherit ederek yazıyoruz.
  Bizden iki field istiyor; queryset, serializer_class
    queryset = User.objects.all()
    serializer_class = RegisterSerializer

user/views.py ->
```py
from .serializers import RegisterSerializer
from django.contrib.auth.models import User
from rest_framework.generics import CreateAPIView

class RegisterView(CreateAPIView):
    queryset = User.objects.all()
    serializer_class = RegisterSerializer

```

- view imizi yazdık, şimdi bu RegisterView isimli view imize url/
  endpoint tanımlamamız gerekiyor.

main/urls.py ->
```py
from django.contrib import admin
from django.urls import path, include
from .views import real_home

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', real_home, name='real_home'),
    path('api/', include('student_api.urls')),
    path('user/', include('user.urls')),
]
```


user/urls.py ->

```py
from django.urls import path
from .views import RegisterView

urlpatterns = [
    path('register/', RegisterView.as_view(), name='register'),
]
```

- user/register ' a bir istek geldiği zaman, RegisterView view 
  yönlendir -> Register view i de RegisterSerializer ı kullanarak bize bir endpoint dönüyor, burada gerekli bilgileri giriyoruz ve validate ve create ile db ye kaydediyor.

- Test ediyoruz, user/register/ ile istek atıyoruz bize 
  serializer da belirttiğimiz fieldlar ile sadece POST yapabileceğimiz bir ekran veriyor. Bu register ekranı ile user create edebileceğiz. O zaman da bize response olarak user dönecek.

- first_name modelimizde zorunlu alan olmamasına rağmen biz 
  serializerda onu zorunlu alan olarak belirtince, doldurmamızı istedi.

- password validation çalışıyor.

- password ve password2 için write_only=True demiştik. sadece post 
  yaparken kullanıyor, read ederken göstermiyor onları.


### login

- register ımızı yaptık, email, password oluşturdu. Artık u kullanıcıyı login yapacağız.

- admin panelden kontrol ediyoruz, kullanıcılar create edilmiş fakat token create edilmemiş.
- Bizim bu kullanıcıya otomatik olarak bir token create etmemiz gerekiyor. Authentication a gidiyoruz ve GeneratingToken başlığını inceliyoruz. Bunun birkaç yöntemi var;
- 
  1- signal -> Bir tabloda bir işlem gerçekleştiğinde, diğer 
     tabloda bir işlemi tetiklemek. (user tablosunda bir user create edildiğinde, token tablosunda bir token create et.) (user tablosunda bir user create edildiğinde, şu field ı generate et.)
     tabloda bir işlemi tetiklemek. (user tablosunda bir user create edildiğinde, token tablosunda bir token create et.) (bir tabloda birşey silindiğinde, diğer tabloda şu işlemi yap.)
     Bu yöntemde user create olduğunda token üretiliyor.

  2- for döngüsüyle tüm userlara token create etmek. 
     Çok kullanılan bir yöntem değil. Tercih edilen bir yöntem değil.

  3- rest_framework.authtoken içerisinde obtain_auth_token isimli bir 
     view var. obtain_auth_token ile token create etme yönteminde ise user login olduğunda, eğer token yoksa token create ediyor, varsa da o token ı getiriyor.

- Burada obtain_auth_token ile token olluşturacağız, sonra signal 
  yazarak token oluşturacağız, daha sonra da bir paket yardımıyla token create etme yöntemi gösterilecek.

- oiçin bir login btain_auth_token aslında bir login view i. Bizim 
  endpointi oluşturuyor. Login işlemini bizim için yapıyor, username, passwordü girdiğimizde, db ye bakıyor, bu user için bir token var mı?, varsa o token ı bize response olarak dönüyor, yoksa create edip yine response olarak dönüyor.

- user app imizin urls.py ına gidip, bizim için bir login 
  endpointi oluşturacak olan obtain_auth_token için rest_framework.authtoken dan views i import edip, daha sonra endpoint olarak dokümanda belirttiği
    path('api-token-auth/', views.obtain_auth_token)
  path ini urlspatterns e ekliyoruz. Tabi biz bu endpoint in ismini login/ olarak değiştirebiliriz. 
    path('login/', views.obtain_auth_token)

user/urls.py ->

```py
from django.urls import path
from .views import RegisterView
from rest_framework.authtoken import views

urlpatterns = [
    path('register/', RegisterView.as_view(), name='register'),
    path('login/', views.obtain_auth_token)
]
```

- Şimdi gidip bu endpointi postman de deneyelim.
- http://127.0.0.1:8000/user/login/ endpointine, body 
  kısmından, post methodu ile, username ve password ile istek attığımda kullanıcıyı login ediyor ve bir token create edip response olarak bize dönüyor.

- Frontend de user login olduktan sonra, token dönen 
  bu response alınıp header a konulacak ve loginden sonra bu kullanıcının yaptığı her istekte bu token kullanılarak istek atılacak. Mantık bu!

- Şimdi buraya kadar prosedür şöyle işliyor -> 
  kullanıcı register oluyor, daha sonra login sayfasına yönlendiriliyor ve login oluyor, login olduktan sonra token create edilip response olarak token dönülüyor. Kullanıcı bu token ı alarak bundan sonraki isteklerini bu tokenla yapabiliyor.

#### register -> login

- Ancak bazı siteler kullanıcı register olduktan sonra 
  onu login sayfasına yönlendirmeden otomatik olarak login ediyor. Böyle olduğu durumlarda kullanıcıya register olduktan sonra, onu login edip, token üretip, register bilgileri ile birlikte oluşturduğu token ı da response olarak dönmek gerekir ki frontend de kullanıcı register olduğunda dönen verilerin içinden token ı alsın ve bundan sonraki isteklerinde bu token ı kullanabilsin.

- register olduğunduktan sonra dönen verileri override edip 
  içeisine token verisini de ekleyecğiz. Biz register ı CreateAPIView ile yapıyorduk, source code una gidiyoruz, create() methodunu görüyoruz, onun da source code una gidiyoruz, bakıyoruz response olarak serializer.data dönüyor. Yani bizim serializer daki datayı dönüyor. 
  
  Yani o zaman biz parent modelimizdeki bir methodu (creaet() methodunu) override ederek, onun response olarak döndüğü serializer.data verisinin içerisine, user register olduktan sonra onun için bir token create edip, bu token verisini de ekleyerek o şekilde response olarak dönmesini sağlayacağız ki user register edildikten sonra dönen veri içerisinde token da bulunsun ve frontend de, dönen bu token verisi alınarak user login olmuş halde istek atabilecek duruma gelsin.

  Tamam o zaman biz bu create() methodunu override edeceğiz ve içerisine token datasını da koyacağız. Override ederken override edeceğimiz methodu alıp view imizin içine yazıyoruz;
    def create(self, request, *args, **kwargs):
  Şimdi bu methodu override edeceğiz. Bunun iki yöntemi var;

    - 1. ve en kolay olanı super() methodu ile override 
      edeceğimiz methoddaki return eden datayı miras alıyoruz.(super() methodu ile bir üst class ın methodunda dönen datayı inherit edip, sonra bu datanın içerisine ne koymak istiyorsanız koyarsınız.) Aslında burda bize return eden data, response objesi. Yani içerisinde serializer.data, status, headers olan bir response class ı return ediyor. Biz bu return edilen response objesini super() methoduyla miras alıp, tekrar bir response değişkenine tanımlıyoruz.
    def create(self, request, *args, **kwargs):
        response = super().cerate(self, request, *args, **kwargs)

  response değişkenine tanımladığımız veride neler vardı? 
      (serializer.data, status=status.HTTP_201_CREATED, headers=headers)
  Artık biz response değişkeni içerinde bulunan serializer.data ya, response.data ile ulaşabiliriz.
      print(response.data)
      print(response.status_code)
  
  Evet test ettik ve response.data ile dönen dictionary tipindeki veriye ulaştık. Artık biz bu dictionary olarak dönen response değişkeni içine yeni bir key-value veri yani token ekleyebiliriz.

  Tamam o zaman Token modelimizi import edip, bu user a ait bir token create edeceğiz.
    from rest_framework.authtoken.models import Token
  Şimdi bu usera ait bir token create edelim;
      token = Token.objects.create(user_id=response.data['id'])
  response.data içerisindeki bu id li user'a token create et dedik, 

  sonra response.data ya 'token' isminde key ekleyip value su olarak da; hemen yukarıda, bir üst satırda oluştrduğumuz token ın key ini tanımlıyoruz.
      response.data['token'] = token.key
  
  Tamamdır, artık create methodundaki response a, (yani register ile user create ettikten sonra return edilen dataya) oluşturduğumuz token ı ekledik. Şimdi en son bu response u return edeceğiz.
      return response

- Evet böylelikle pythonda parent class ın methodunu override 
  etmiş olduk.

user/views.py ->

```py
from .serializers import RegisterSerializer
from django.contrib.auth.models import User
from rest_framework.generics import (
    CreateAPIView,
    ListCreateAPIView, # detay gerektirmeyen
    RetrieveUpdateDestroyAPIView, # detay gerektiren
)
from rest_framework.authtoken.models import Token

class RegisterView(CreateAPIView):
    queryset = User.objects.all()
    serializer_class = RegisterSerializer
    
    def create(self, request, *args, **kwargs):
        response = super().create(request, *args, **kwargs)
        token = Token.objects.create(user_id=response.data['id'])
        response.data['token'] = token.key
        # print(response.data)
        return response
```

- Test ediyoruz, user register olunca dönen datada kendisi için 
  oluşturulmuş token da bulunuyor ve frontend kısmından bu token alınıp header kısmına eklenip login olmuş bir şekilde endpointlere istek atabilir.
- Artık register olduktan sonra bu user ı authenticate olarak 
  sistem içerisine dahil edebiliriz.



- Logout
- Şimdi logout yapacağız;
- logout da herhangi bir veri almayacağımız için serializer filan yazmaya gerek yok. 
- views.py dosyasına gidip logout_view imizi direct yazıyoruz.
- Basit bir function view imiz var, serializer a filan gerek yok çünkü bir veri gelmeyecek, sadece post methoduyla logout endpointine istek attığımızda isteğin head kısmında kimin token ı varsa, yani o istek kime ait ise onu logout edecek yani token tablosundan token ını silecek bu kadar. @api_view decorator ünü import etmemiz lazım.

<views.py> ->

```py
@api_view(['POST'])
def logout_view(request):
    if request.method == 'POST':
        request.user.auth_token.delete()
        data = {
            'message': 'logout'
        }
        return Response(data)

```

- Bu ne yapıyor? request.method post ise, request.user.auth_token.delete() ile o user ımızın token ını delete edecek token tablosundan, data response edecek, içinde de logout yazan bir mesaj olan bir data response edecek.

- Şimdi bu view imize bir url yazalım; 

user_api <urls.py>

```py
    path('logout/', logout_view, name='logout'),

```

- test ediyoruz; herhangi bir user ın token ile /user/logout/ endpoint inden post methoduyla istek atınca, token ı sildi ve logout etti.
