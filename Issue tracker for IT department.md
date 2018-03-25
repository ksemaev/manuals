**Первичная настройка трекера заявок RT от SecureScout**

**(rt-4.4.1 на** **Ubuntu 16.04.1 LTS GNU/Linux 4.4.0-51-generic x86\_64)**

Вообще вот официальный readme для текущей версии (текущая по опыту стабильная и колдовать уже вроде бы не нужно): [https://docs.bestpractical.com/rt/4.4.1/README.html](https://docs.bestpractical.com/rt/4.4.1/README.html)

1) Для первичной настройки (да и вообще на будущее, так как обычно трекер находится в локалке, и доступ извне защищает отдельный firewall) выключаю UFW:

**sudo ufw disable**

2) Ставлю всю что необходимо для работы с БД (при установке нужно будет задать пароль пользователю root внутри mysql):

**sudo apt-get install mysql-server mysql-client libmysqlclient-dev**

3) Оптимизирую размер буфера под работу SQL а именно в файле / **etc/mysql/mysql.conf.d/mysqld.cnf** добавляю в раздел Fine Tuning строку (50-80% ОЗУ):

**innodb\_buffer\_pool\_size = 2048M**

4) Запускаю мастер настройки безопасности mysql и перезапускаю демон:

**sudo mysql\_secure\_installation**

**sudo service mysql restart**

5) Ставлю все что необходимо для работы веб-сервера:

**sudo apt-get install make apache2 libapache2-mod-fcgid libssl-dev libyaml-perl libgd2-xpm-dev libgd-gd2-perl libgraphviz-perl**

6) Создаю пользователя для трекера заявок и добавляю его в группу пользователей apache:

**sudo adduser --system --group rt**

**sudo usermod -aG rt www-data**

------------------------------------------------------------

7) Для отправки с сервера почтовых сообщений ставлю нужные пакеты (при установке укаываю режим работы Satellite, имя системы могу не менять, в качестве relay в моем случае выступает _smtp.yandex.ru_):

**sudo apt-get install postfix mailutils**

8) В основной конфигурационный файл postfix **/etc/postfix/main.cf** добавляю (для красоты после s_mtp\_tls\_session\_cache\_database_ ):

**smtp\_sasl\_auth\_enable = yes**

**smtp\_sasl\_password\_maps = hash:/etc/postfix/sasl\_passwd**

**smtp\_sasl\_security\_options = noanonymous**

**smtp\_sasl\_type = cyrus**

**smtp\_sasl\_mechanism\_filter = login**

**smtp\_sender\_dependent\_authentication = yes**

**sender\_dependent\_relayhost\_maps = hash:/etc/postfix/sender\_relay**

**smtp\_generic\_maps = hash:/etc/postfix/generic**

**smtp\_tls\_CAfile = /etc/postfix/cacert.pem**

**smtp\_use\_tls = yes**

9) Создаю файл **/etc/postfix/sasl\_passwd** и указываю логин и пароль для аутентификации на внешнем почтовом сервере (на том который указали при установке postfix в качестве релея) в формате:

**smtp.yandex.ru    alert@example.com:password**

Затем меняю режим доступа к файлу и заугружаю информацию в postfix:

**sudo chmod 400 /etc/postfix/sasl\_passwd**

**sudo postmap /etc/postfix/sasl\_passwd**

10) Создаю файл **/etc/postfix/generic** в котором указываю что для отправки писем с этого сервера используется ящик alert в формате:

**@полное\_имя\_сервера  ** [**alert@example.com**](mailto:alert@example.com)

После этого загружаю информацию в postfix: **sudo postmap /etc/postfix/generic**

11) Создаю файл **/etc/postfix/sender\_relay** в котором говорю что для домена example.com будет использоваться relay smtp.yandex.ru в формате:

**@example.com    smtp.yandex.ru**

И загружаю конфиг в postfix:   **sudo postmap /etc/postfix/sender\_relay**

12) Копирую доступный (в разных версиях ОС тут бывают разные сертификаты) корневой сертификат в файл _/etc/postfix/cacert.pem_и перезапускаю демон:

**sudo cat /etc/ssl/certs/thawte\_Primary\_Root\_CA.pem | sudo tee -a /etc/postfix/cacert.pem**

**sudo service postfix restart**

13) Создаю тестовое сообщение и отправляю его:

**echo &quot;Hello World&quot; | mail -s &quot;Test Message&quot; myemail@gmail.com -aFrom:alert@example.com**

14) Проверяю доставку письма к себе на [myemail@gmail.com](mailto:myemail@gmail.com) и если что-то пошло не так изучаю лог:

**tail /var/log/mail.log**

---------------------------------------------------------

15) Устанавливаю все необходимое для установки rt (устанавливать буду из скаченнного пакета):

**sudo apt-get install perl make**

16) Скачиваю установочный файл (выбрать версию можно вбив в браузер ссылку и стерев имя пакета):

**cd ~**

**wget** [https://download.bestpractical.com/pub/rt/release/rt-4.4.1.tar.gz](https://download.bestpractical.com/pub/rt/release/rt-4.4.1.tar.gz)

17) Распакую пакет во временную директорию и перейду в нее:

**sudo tar xzvf**  **rt-4.4.1.tar.gz**  **-C /tmp**

**cd /tmp/rt-4.4.1**

18) Для подготовки пакета к установке нужно запустить скрипт _./configure_ указав ему необходимые параметры (можно их посмотреть командой _./configure --help_)

**sudo ./configure --with-web-user=www-data --with-web-group=www-data --enable-graphviz --enable-gd**** --enable-externalauth**

19) Теперь нужно настроить для работы CPAN:

**sudo cpan**

**yes**

**o conf prerequisites**** \_ ****policy follow**  ** **

**o conf build**** \_ ****requires**** \_ ****install**** \_ ****policy**

**o conf commit**

**q**

20) Для проверки готовности системы к установке запускаю тестирование и так как точно не хватает многих модулей - запускаю автоматическую подготовку системы

P.S. Второй шаг может длиться до получаса, при этом может уйти в цикл на какой-нибудь поломанной зависимости и придется гуглить в чем там дело.

_У меня, к примеру, спросил готов ли я работать с моей версией perl, ведь в ней не поддерживается JSON:XL. Я выбрал yes_.

_Также спросил строить ли ему модуль XStash - я решил что пусть строит, yes._

_От проведения дополнительных тестов я отказался, no._

P.P.S Необходимо повторять эти команды по очереди, пока testdeps нее скажет что все в порядке.

**sudo make testdeps**

**sudo make fixdeps**

21) Завершающий этап установки: запуск сценарий установки и инициализация БД (нужен пароль от mysql root из шага №2)

**sudo make install**

**sudo make initialize-database**

22) Для корректной работы веб интерфейса необходимо указать в файле **/etc/apache2/sites-available/000-default.conf** имя сервера (или внешнее имя сайта, которое будете привязывать к RT) и настройки сайта (в том числе откоментировать нужное, в зависимости от версии Apache):

    **ServerName rt.example.com**

**     AddDefaultCharset UTF-8**

**     DocumentRoot /opt/rt4/share/html**

**     Alias /NoAuth/images/ /opt/rt4/share/html/NoAuth/images/**

**     ScriptAlias / /opt/rt4/sbin/rt-server.fcgi/**

**                &lt;Location /&gt;**

**                    ## Apache version &lt; 2.4 (e.g. Debian 7.2)**

**                    #Order allow,deny**

**                    #Allow from all**

**                    ## Apache 2.4**

**                    Require all granted**

**                &lt;/Location&gt;**

23) Необходимо перезапустить веб-сервер и проверить доступность сайта rt по выбранному вам адресу (или имени). Логин: _root_, пароль: _password_.

**sudo service apache2 restart**

-----------
24) Редактирую под себя файл конфигурации **/opt/rt4/etc/RT\_SiteConfig.pm** (язык, имя сайта, адрес и имя для обращения и т.д.), для применения нужно перезапустить apahe:

**Set( @LexiconLanguages, qw(en ru));**

**Set( $rtname, &#39;rt.example.com&#39;);**

**Set( $WebDomain, &#39;****rt.example.com****&#39;);**

**Set( $Organisation, &#39;rt.example.com&#39;);**

**Set( @ReferrerWhitelist, qw(rt.example.com:80 12.34.56.78:80));**

**Set( $Timezone , &#39;Europe/Moscow&#39;);**

**Set( $LogoLinkURL, &#39;** [**http://**](http://rt.example.com/) [**rt.example.com**](http://rt.example.com/) [**/**](http://rt.example.com/)**&#39;);**

25) Прочие настройки могу изменить через веб-интерфейс:

Root - настройки - персональные данные - пароль

Администратор – утилиты – оформление (меняем лого и можем сменить цвета).

Администратор – общие – обзор RT (меняем внешний вид главной страницы).

Администратор – общие – шаблоны (можно перевести на русский язык текст в шаблонах).

Администратор – общие – скриплеты (здесь редко нужно что-то менять: это действия выполняемые при каких-то изменениях по заявке).

27) Для настройки связки rt с доменом редактирую файл конфигурации (предварительно создаю в домене учентную запись для связки с rt с правами пользователя домена, в моем случае connect@mydomain.local) **/opt/rt4/etc/RT\_SiteConfig.pm**

_P.S. я импортирую пользователей из всего домена, а можно указать в_ _$LDAPBase конкретное подразделение._

_P.P.S мануал по плагину:_ [_https://metacpan.org/pod/RT::Authen::ExternalAuth_](https://metacpan.org/pod/RT::Authen::ExternalAuth)

**Set($LDAPHost,&#39;domaincontroller.mydomain.local&#39;);**

**Set($LDAPUser,&#39;mydomain\connect&#39;);**

**Set($LDAPPassword,&#39;password&#39;);**

**Set($LDAPBase,&#39;dc=****mydomain****,dc=local&#39;);**

**Set($LDAPFilter, &#39; (&amp;(objectCategory=person))&#39;);**

**Set($LDAPMapping, {Name         =&gt; &#39;sAMAccountName&#39;,**

**                  RealName        =&gt; &#39;cn&#39;,**

**                  EmailAddress =&gt; &#39;mail&#39;**

**});**

**Set($LDAPCreatePrivileged, 1);**

**Set($LDAPUpdateUsers, 1);**

**Set($AutoCreateNonExternalUsers, 1);**

28) Выполняю тестовый импорт пользователей (группы втягивать не хочу) и проверяю успешное выполнение команды в лог файле:

**sudo**  **/opt/rt4/sbin/rt-ldapimport**  **--debug &gt; ldapimport.debug 2&gt;&amp;1**

**cat ldapimport.debug**

29) В случае отсутствия ошибок делаю реальный импорт (разрабы рекомендуют добавить эту задачу в cron для регулярного импорта пользователей).

После импорта в &quot;Администратор - Пользователи - выбрать&quot; нужно отключить учетные записи которые не будут входиь в rt (тот же connect например).

**sudo  **** /opt/rt4/sbin/rt-ldapimport **** --import**

30) Для включения аутентификации через контроллер домена добавляю в конфигурационый файл   **/opt/rt4/etc/RT\_SiteConfig.pm** настройки. Для проверки корректной работы нужно перезапустить apache и войти в RT с любой импортированной доменной учетной записью.

**Set($ExternalAuthPriority,  [&#39;My\_AD&#39;] );**

**Set($ExternalInfoPriority, [&#39;My\_AD&#39;] );**

**Set( $UserAutocreateDefaultsOnLogin, { Privileged =&gt; 1 , Lang =&gt; &#39;ru&#39;} );**

**Set($ExternalSettings, {**

**    &#39;My\_AD&#39;       =&gt;  {**

**        &#39;type&#39;             =&gt;  &#39;ldap&#39;,**

**        &#39;server&#39;           =&gt;  &#39;domaincontroller.**** mydomain ****.local&#39;,**

**        &#39;user&#39;                      =&gt;  &#39;**** mydomain ****\connect&#39;,**

**         &#39;pass&#39;                      =&gt;  &#39;password&#39;,**

**        &#39;base&#39;             =&gt;  &#39;dc=**** mydomain ****,dc=local&#39;,**

**        &#39;filter&#39;           =&gt;  &#39;(objectCategory=person)&#39;,**

**        &#39;attr\_match\_list&#39; =&gt; [&#39;Name&#39;],**

**         &#39;attr\_map&#39;      =&gt; {**

**        &#39;Name&#39;          =&gt; &#39;sAMAccountName&#39;,**

**        &#39;EmailAddress&#39;  =&gt; &#39;mail&#39;,**

**        &#39;RealName&#39;      =&gt; &#39;cn&#39;,**

**         **** },**

**    },**

**} );**

----------

31) Если необходимо настроить авторизацию при помощи веб-сервера (в дополнение к предыдущим), то нужно отредактировать настройки сайта трекера, в моем случае это сайт по умолчанию **/etc/apache2/sites-available/000-default.conf**

**&lt;Location /&gt;**

**        Require valid-user**

**        AuthType Basic**

**        AuthName &quot;Трекер заявок&quot;**

**        AuthBasicProvider ldap**

**        AuthLDAPURL &quot;ldap://domaincontroller.**** mydomain****.local/dc=mydomain,dc=local?sAMAccountName?sub?(objectClass=\*)&quot;**

**        AuthLDAPBindDN &quot;cn=connect,dc=mydomain,dc=local&quot;**

**        AuthLDAPBindPassword &quot;password&quot;**

**&lt;/Location** _&gt;_

32) Для работы аутентификации в ldap через Apache нужно включить соответсвующий модуль:

**sudo a2enmod authnz\_ldap**

**sudo service apache2 restart**

33) В настройках RT (файл **/opt/rt4/etc/RT\_SiteConfig.pm** ) следует разрешить аутентификацию сервером Apache (и перезапустить демон apache):

**Set($WebRemoteUserAuth, 1);**