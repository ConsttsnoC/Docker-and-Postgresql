<div><div class="article-formatted-body article-formatted-body article-formatted-body_version-2"><div xmlns="http://www.w3.org/1999/xhtml"><p>Приветствую всех!</p><p>Я начинающий разработчик и хочу поделиться своим решением проблемы, с которой столкнулся и не нашел готового решения в интернете. Мой подход может не быть оптимальным, но может быть полезен другим новичкам.  </p><p>Предполагаю, что если вы нашли эту статью, то уже знакомы с процессом установки Docker и использования Django, поэтому не буду расписывать их детально. Но у вас уже должен быть установлен Docker.<br><em>Я работаю на windows, поэтому если у вас другая операционная система, то команды могут отличаться.</em></p><p><strong>Шаг 1.</strong>  Самый сложный.<br>Создаем рабочую папку.</p><p><strong>Шаг 2. <br></strong>Создаем или сразу запускаем виртуальное окружение если оно у вас есть.<br><br><strong>Создаем:<br></strong>В терминале переходим в рабочую папку, которую только что создали. Переход выполняется посредством команды <code>cd </code>и путь к вашей папке.<br>Пример:<br><code>C:\Users\User&gt;cd C:\Test<br></code>Далее, находясь в рабочий папке, введите так же в терминале, команду для создания виртуального окружения: <code>python -m venv +---&gt; (название)<br></code>Пример: <code>C:\Test&gt;python -m venv myenv</code></p><p><strong>Активируем:<br></strong>В терминале пишем путь до папки <code>Scripts<br></code>Пример:<br><code>C:\Test&gt;cd C:\Test\myenv\Scripts<br></code>Затем прописываем команду <code>activate</code>, которая и активируем наше виртуальное окружение.<br>Пример:<br><code>C:\Test\myenv\Scripts&gt;activate<br></code>Если вы сделали правильно, то успешная активация виртуального окружения будет выглядеть следующим образом:<br><code>(myenv) C:\Test\myenv\Scripts&gt;<br></code>Как можете заметить, вначале строки появилось название виртуального окружения в скобках. </p><p><strong>Шаг 3.<br></strong>Возвращаемся в корневую папку проекта (Шаг 1)<br>Пример:<br><code>(myenv) C:\Test\myenv\Scripts&gt;cd C:\Test</code><br>Создаем файл <code>requirements.txt</code>, в  терминале пишите команду:<br><code>pip freeze &gt; requirements.txt</code>  <br>После чего в вашей директории появится файл <code>requirements.txt ,</code>в котором будут отображаться ваши установленные пакеты и зависимости. На данном этапе, если вы начинали проект с самого начала вместе со мной, то этот файл у вас будет пустой.</p><p><strong>Шаг 4.<br></strong>Устанавливаем пакеты, в терминале пишем следующие команды:<br><code>python -m pip install Django</code> - устанавливаем фреймворк Django с помощью менеджера пакетов pip в Python.<br>Пример:<br><code>(myenv) C:\Test&gt;python -m pip install Django</code> <br>Далее пишем другую команду в терминале:<br><code>pip install psycopg2-binary </code>- библиотека <code>psycopg2-binary</code> предоставляет Python-интерфейс для работы с СУБД PostgreSQL.  <br>Пример:<br><code>(myenv) C:\Test&gt;pip install psycopg2-binary<br></code>После установки, обновляем <code>requirements.txt</code>  посредством уже известной нам команды:<br><code>pip freeze &gt; requirements.txt<br></code>Теперь в файле <code>requirements.txt, </code>должны отображаться ваши установленные пакеты. </p><p><strong>Шаг 5.<br></strong>Создаем новый проект Django в корневой директории (Шаг 1) с заданным названием.<strong> <br></strong> <code>docker-compose run django django-admin startproject (название) .<br></code>Не забудь про точку в конце команды.<code><br></code>Пример:<br><code>(myenv) C:\Test&gt;docker-compose run django django-admin startproject testpostgre .<br></code></p><p>Далее:<br>Cоздаем новое Django приложение в корневой директории  с новым названием.<br><code>python manage.py startapp (название)</code><br>Пример:<br><code>(myenv) C:\Test\testpostgre&gt;python manage.py startapp app</code></p><p>Далее:<br>Переносим вручную файл manage.py в корневую директорию (папка из шага 1)</p><p><strong>Шаг 6.<br></strong>Открываем свой IDE, и в корневой директории (в папке из "Шаг 1") создаем два файла:</p><ol><li><p>Dockerfile (название файла)</p></li></ol><pre><code class="python hljs"><span class="hljs-comment"># Используем официальный образ Python в качестве базового образа</span>
FROM python
<span class="hljs-comment"># Устанавливаем рабочую директорию внутри контейнера</span>
WORKDIR /usr/src/app
<span class="hljs-comment"># Копируем файл requirements.txt внутрь контейнера</span>
COPY requirements.txt ./
<span class="hljs-comment"># Устанавливаем зависимости, описанные в файле requirements.txt</span>
RUN pip install -r requirements.txt</code></pre><ol start="2"><li><p>docker-compose.yml (название файла)</p></li></ol><pre><code># Определение версии Docker Compose и начало описания сервисов
version: '3'

services:
  django:
    # Сборка образа для сервиса django из текущей директории
    build: .
    # Задание имени контейнера для сервиса django
    container_name: django
    # Задание команды, которую нужно запустить при запуске контейнера для сервиса django
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/usr/src/app
    # Открытие порта на хостовой машине и перенаправление на порт в контейнере
    ports:
      - 8000:8000
    # Зависимость от другого сервиса
    depends_on:
      - pgdb

  pgdb:
    # Использование готового образа postgres
    image: postgres
    # Задание переменных окружения для контейнера с postgres
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
     # Задание имени контейнера для сервиса pgdb
    container_name: pgdb
     # Связывание тома с директорией в контейнере для сохранения данных postgres
    volumes:
      - pgdbdata:/var/lib/postgresql/data/

volumes:
  pgdbdata: null
</code></pre><p>Таким образом структура вашего проекта должна выглядеть примерно так:</p><figure class=""><img src="https://habrastorage.org/r/w1560/getpro/habr/upload_files/a31/fe9/656/a31fe96561137196650fda8f1f8a502d.png" width="489" height="420" data-src="https://habrastorage.org/getpro/habr/upload_files/a31/fe9/656/a31fe96561137196650fda8f1f8a502d.png"></figure><p><strong>Шаг 7.<br></strong>Открываем файл <code>settings.py</code>и меняем там следующие строки кода:</p><pre><code>Так было:
ALLOWED_HOSTS = []
Так стало:
ALLOWED_HOSTS = ['localhost', '127.0.0.1', '0.0.0.0']</code></pre><p>Далее спускаемся ниже по коду:</p><pre><code>Так было:
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
    
Так стало:
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app', # Добавляем наше приложение из "Шаг 5"
    'django.contrib.postgres', #это модуль Django, который предоставляет интеграцию с базой данных PostgreSQL 
]</code></pre><p>Далее меняем следующие строки:</p><pre><code>Так было:
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}


Так стало:
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',   # Используется PostgreSQL
        'NAME': 'postgres', # Имя базы данных
        'USER': 'postgres', # Имя пользователя
        'PASSWORD': 'postgres', # Пароль пользователя
        'HOST': 'pgdb', # Наименование контейнера для базы данных в Docker Compose
        'PORT': '5432',  # Порт базы данных
    }
}</code></pre><p><strong>Шаг 8.<br></strong>Переходим в файл models.py и создаем там модель для теста БД.</p><pre><code class="python hljs"><span class="hljs-comment"># Импорт модуля models из библиотеки Django</span>
<span class="hljs-keyword">from</span> django.db <span class="hljs-keyword">import</span> models
<span class="hljs-comment"># Определение класса MyModel, который наследует модель Django Model</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyModel</span>(<span class="hljs-params">models.Model</span>):</span>
    <span class="hljs-comment"># Определение поля id типа AutoField как первичного ключа</span>
    <span class="hljs-built_in">id</span> = models.AutoField(primary_key=<span class="hljs-literal">True</span>)
    <span class="hljs-comment"># Определение поля phone типа CharField с максимальной длиной в 20 символов</span>
    phone = models.CharField(max_length=<span class="hljs-number">20</span>)

<span class="hljs-comment"># Определение метода __str__, который будет использоваться для представления экземпляров модели в виде строки</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__str__</span>(<span class="hljs-params">self</span>):</span>
        <span class="hljs-keyword">return</span> self.phone
</code></pre><p>Далее открываем файл admin.py и регистрируем созданную нами модель в админ-панели Django:</p><pre><code class="python hljs"><span class="hljs-comment"># Импорт модуля admin из библиотеки Django.contrib</span>
<span class="hljs-keyword">from</span> django.contrib <span class="hljs-keyword">import</span> admin
<span class="hljs-comment"># Импорт модели MyModel из текущего каталога (".")</span>
<span class="hljs-keyword">from</span> .models <span class="hljs-keyword">import</span> MyModel
<span class="hljs-comment"># Регистрация модели MyModel для административного сайта</span>
admin.site.register(MyModel)</code></pre><p><strong>Шаг 9.</strong></p><p>Запускает все сервисы, определенные в файле <code>docker-compose.yml</code> в текущей директории, в режиме detached (фоновый режим) с помощью команды в терминале:<br><code>docker-compose up</code><br>Пример:<br><code>C:\Test&gt; docker-compose up</code></p><p>Далее:<br>Открываем браузер и переходим по адресу <a href="http://localhost:8000/" rel="noopener noreferrer nofollow">http://localhost:8000/</a><br>И если вы всё правильно запустили, то вас будет ждать там следующая картина:</p><figure class="full-width "><img src="https://habrastorage.org/r/w1560/getpro/habr/upload_files/6f1/ae3/032/6f1ae3032e826d77c2373717f239bf0d.png" width="1788" height="996" data-src="https://habrastorage.org/getpro/habr/upload_files/6f1/ae3/032/6f1ae3032e826d77c2373717f239bf0d.png"></figure><p>Далее:</p><p>Производите миграцию, выполнив команду в терминале:<br><code>docker-compose run django python manage.py migrate<br></code>Пример:<br><code>C:\Test&gt; docker-compose run django python manage.py migrate</code><br></p><p>Далее:</p><p>Создаем супер-пользователя для входа в админ-панель:<br>В терминале, находясь в папке (Шаг 1) пишем следующую команду:<br><code>docker-compose run django python manage.py createsuperuser<br></code>Пример:<br><code>C:\Test&gt; docker-compose run django python manage.py createsuperuser<br></code>Затем вам нужно ввести имя пользователя:<br><code>Username (leave blank to use 'root'): </code>- вводите любое имя латинице<br>После будет поле Email:<br><code>Email address: </code>- можете вводить или пропустить нажав Enter<br>Далее поле с паролем, сразу учтите, при вводе пароля, он отображаться не будет, не пугайтесь. Как ввели пароль, нажмите Enter, и потом повторите пароль ещё раз.<br><code>Password:<br>Password (again):<br></code>Если вы введете легкий пароль, то вас спросят:<br><code>Bypass password validation and create user anyway? [y/N]: </code>- введите "y" и нажмите Enter<br>Строка <code>Superuser created successfully. </code>означает что вы все сделали правильно.</p><p>Далее:</p><p>Поднимаете ваши контейнера, с помощью команды <code>docker-compose up<br></code>Пример: C:\Test&gt; docker-compose up<br>Открываете браузер и переходите по адресу: <a href="http://localhost:8000/admin" rel="noopener noreferrer nofollow">http://localhost:8000/admin</a><br>Должны увидеть следующее:</p><figure class="full-width "><img src="https://habrastorage.org/r/w1560/getpro/habr/upload_files/17b/8e5/5db/17b8e55dbaffdfb0756fe42bd3935aa7.png" width="1783" height="991" data-src="https://habrastorage.org/getpro/habr/upload_files/17b/8e5/5db/17b8e55dbaffdfb0756fe42bd3935aa7.png"></figure><p>Вводите имя пользователя и пароль, которые только что создали. </p><p>Нажимаем на My models (или как вы назвали свой класс в "Шаг 8")</p><figure class="full-width "><img src="https://habrastorage.org/r/w1560/getpro/habr/upload_files/57f/5bb/abe/57f5bbabe8a86040d026ef4c63683be0.png" width="1916" height="992" data-src="https://habrastorage.org/getpro/habr/upload_files/57f/5bb/abe/57f5bbabe8a86040d026ef4c63683be0.png"></figure><p>Добавляете несколько данных для проверки работоспособности.</p><p><strong>Шаг 10.</strong></p><p>Входим в интерактивный режим Postgres внутри контейнера Docker.<br>Для этого нужно прописать следующую команду в терминале:<br> <code>docker exec -it &lt;имя_контейнера&gt; psql -U &lt;имя_пользователя&gt; &lt;имя_базы_данных&gt;</code>  <br>Пример:<br><code>docker exec -it pgdb psql -U postgres postgres</code>  <br></p><p>Чтобы посмотреть таблицы в PostgreSQL из интерактивного режима <code>psql</code>, вы можете выполнить команду <code>\dt;</code>, которая выведет список всех таблиц в текущей базе данных.  <br>Пример:<br><code>postgres=# \dt;</code></p><p>Получаем список всех таблиц в нашей базе данных:</p><figure class=""><img src="https://habrastorage.org/r/w1560/getpro/habr/upload_files/42f/cc6/b02/42fcc6b029e90c1645dd09c215ebfe90.png" width="510" height="463" data-src="https://habrastorage.org/getpro/habr/upload_files/42f/cc6/b02/42fcc6b029e90c1645dd09c215ebfe90.png"></figure><p>Для того, чтобы посмотреть данные в таблице <code>app_mymodel</code>, необходимо выполнить следующую команду:  <br><code>SELECT * FROM app_mymodel;</code>  <br><br>Ну вот и всё, теперь у вас в контейнере Docker запущен Django и PostgreSQL.</p><p><strong>БОНУС:<br></strong>Если перейти по ссылке <a href="https://hub.docker.com/_/postgres" rel="noopener noreferrer nofollow">https://hub.docker.com/_/postgres</a><br>И скопировать текст команды <code>docker pull postgres</code></p><figure class="full-width "><img src="https://habrastorage.org/r/w1560/getpro/habr/upload_files/a71/9d8/014/a719d8014d79329bfff374190b46d938.png" width="1289" height="309" data-src="https://habrastorage.org/getpro/habr/upload_files/a71/9d8/014/a719d8014d79329bfff374190b46d938.png"></figure><p>То, эта команда запустит процесс загрузки (pull) образа Docker для СУБД PostgreSQL из официального репозитория Docker Hub.<br><br>Для этого вставьте скопированный текст в терминал.<br>Пример:<br><code>(dtenv)C:\Docker_Test&gt; docker pull postgres</code></p><p>После того, как процесс загрузки завершится, образ будет доступен локально на вашей машине и вы сможете использовать его для создания и запуска контейнеров PostgreSQL в вашей среде.<br><br>Затем меняем немного код в файле <code>docker-compose.yml</code></p><pre><code>version: '3'

services:
  django:
    build: .
    container_name: django
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/usr/src/app
    ports:
      - 8000:8000
    depends_on:
      - pgdb

  pgdb:
    image: postgres
    restart: always
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    container_name: pgdb
    volumes:
      - pgdbdata:/var/lib/postgresql/data/

  adminer:
    image: adminer
    restart: always   #c 26 по 30 строку(вставлен новый фрагмент)
    ports:
      - 8080:8080
volumes:
  pgdbdata: null</code></pre><p>Вновь выполняем команду сборки <code>docker-compose up<br></code>Пример:<br><code>(dtenv)C:\Docker_Test&gt; docker-compose up</code></p><p>После проделанных манипуляций, можем перейти по адресу <a href="http://127.0.0.1:8080/" rel="noopener noreferrer nofollow">http://127.0.0.1:8080/</a><br>И увидеть там вход в БД:</p><figure class="full-width "><img src="https://habrastorage.org/r/w1560/getpro/habr/upload_files/a21/73a/27c/a2173a27cf1b9b7938c3dfc24810c0ab.png" width="812" height="390" data-src="https://habrastorage.org/getpro/habr/upload_files/a21/73a/27c/a2173a27cf1b9b7938c3dfc24810c0ab.png"></figure><p>В разделе "Движок" меняете MySQL на PostgreSQL, в разделе "Сервер" пишите название Хоста указанного у вас в файле settings.py, если не можете найти, то посмотрите на картинку:</p><figure class="full-width "><img src="https://habrastorage.org/r/w1560/getpro/habr/upload_files/ae0/06f/6bc/ae006f6bc9f8b5f3dd6fe98f91177e02.png" width="753" height="223" data-src="https://habrastorage.org/getpro/habr/upload_files/ae0/06f/6bc/ae006f6bc9f8b5f3dd6fe98f91177e02.png"></figure><p>Далее вводите Имя пользователя и Пароль что вы указали, или если делали как я, то указывайте тоже самое. Название БД вводить не обязательно.<br>После входа, вы сможете просматривать таблицы вашей БД без труда и в нормальном виде!</p></div></div></div> 