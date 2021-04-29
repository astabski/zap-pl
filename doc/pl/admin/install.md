## Instalacja oprogramowania

Bardzo się staraliśmy, aby to oprogramowanie działało na popularnych platformach hostingowych - takicj jakie sie używa do hostowania blogów Wordpressa czy witryn opartych na Drupal. $Projectname będzie działać na większości systemów VPS opartych na Linux. Platformy Windows LAMP, takie jak XAMPP i WAMP nie są obecnie oficjalnie obsługiwane - jednak z zadowoleniem przyjmujemy łatki, jeśli uda ci się to uruchomić.

Należy pamiętać, że to oprogramowanie to coś więcej niż prosta aplikacja internetowa. Jest to złożony system komunikacji i zarządzania treścią, który bardziej przypomina serwer poczty internetowej niż serwer WWW. Aby zapewnić niezawodność i wydajność, komunikaty są dostarczane w tle i umieszczane w kolejce do późniejszego dostarczenia, gdy lokacje są niedostępne. Ten rodzaj funkcjonalności wymaga nieco więcej od systemu hosta niż typowy blog. Nie każdy dostawca hostingu PHP/MySQL będzie w stanie spełnić te wymagania. Wielu to zapewnia - ale lepiej jest zapoznać się z wymaganiami i potwierdzić je u dostawcy usług hostingowych jeszcze przed instalacją (a w szczególności przed zawarciem długoterminowej umowy).

Jeśli napotkasz problemy z instalacją, prosimy o informację o tym, za pośrednictwem [systemu śledzenia spraw projektu](https://github.com/isfera/social), skąd pobrałeś oprogramowanie. Podaj jak najwięcej informacji o swoim środowisku operacyjnym i jak najwięcej szczegółów na temat wszelkich komunikatów o błędach, które możesz zobaczyć, abyśmy mogli zapobiec temu w przyszłości. Ze względu na dużą różnorodność działania istniejących systemów i platform PHP, możemy mieć tylko ograniczone możliwości debugowania instalację PHP lub pozyskania brakujących modułów - ale zrobimy to, starając się rozwiązywać ogólne problemy z kodem.

### Zanim zaczniesz 

Wybierz nazwę domeny i ewentualnie poddomeny dla swojego serwera.

Oprogramowanie można zainstalować tylko w katalogu głównym domeny lub poddomeny i nie może ono działać na alternatywnych portach TCP.

Wymagane jest szyfrowanie SSL komunikacji z serwerem WWW a stosowany certyfikat SSL musi buć "prawidłowy dla przeglądarki". Nie można uzywać certyfikatów z podpisem własnym!

Przetestuj swój certyfikat przed instalacją. Narzędzie internetowe do testowania certyfikatu jest dostępne pod adresem "http://www.digicert.com/help/". Odwiedzając witrynę po raz pierwszy, użyj adresu URL SSL („https://”). Pozwoli to uniknąć późniejszych problemów. 

Bezpłatne certyfikaty zgodne z przeglądarkami są dostępne od dostawców, takich jak StartSSL i LetsEncrypt. 

Jeśli stosujesz LetsEncrypt do dostarczania certyfikatów i tworzenia pliku w ramach usługi "well-known" lub "acme-challenge", tak aby LetsEncrypt mógł zweryfikować własność domeny, usuń lub zmień nazwę katalogu `.well-known`, gdy tylko plik
certyfikatu zostanie wygenerowany. Oprogramowanie zapewnia własny program obsługi usługu "well-known", gdy jest instalowany a  istnienie tego katalogu podczas dalszego działania serwera może uniemożliwić poprawne działanie niektórych usług. To nie powinno być problemem w Apache, ale może być problemem z nginx lub innym serwerze WWW.

### Instalacja

#### Wymagania

- Apache z włączoną obsługą mod-rewrite i dyrektywą "AllowOverride All", więc możesz użyć lokalnego pliku .htaccess. Niektórzy z powodzeniem używali nginx i lighttpd. Przykładowe skrypty konfiguracyjne są dostępne dla tych platform w katalogu instalacyjnym. Największe wsparcia ma Apache i Nginx. 

- PHP 7.2 lub wersja późniejsza (z wersją 8.0 włącznie). 

- Dostęp do *wiersza poleceń* PHP z ustawionym na `true` argumentem `register_argc_argv` w pliku php.ini - oraz bez ograniczeń w stosowaniu funkcji exec() i proc_open(), które często nakładają operatorzy hostingu. 

- Rozszerzenia curl, gd (z obsługą co najmniej jpeg i png), mysqli, mbstring, xml, xmlreader (FreeBSD), zip i openssl. Zamiast biblioteki gd, można używać rozszerzenia imagick ale nie jest ono wymagane i MOŻE być również wyłączone za pomocą opcji konfiguracyjnej. 

- Jakaś forma serwera pocztowego lub bramy poczty elektronicznej na której działa funkcja mail() PHP.

- MySQL 5.5.3 lub w wersji nowszej lub serwer MariaDB lub Postgres. Wyszukiwanie bez rozróżniania wielkości liter nie jest obsługiwane w Postgres. Nie jest to szkodliwe, ale węzły z Postgres raczej nie powinny być używane jako serwery katalogów z powodu tego ograniczenia. 
    
- Możliwość planowania zadań przy użyciu crona.

- Wymagana jest instalacja w głównym katalogu domeny lub poddomeny (bez składnika katalog/ścieżka w adresie URL).

### Procedura instalacyjna

**1. Rozpakuj pliki projektu do katalogu głównego obszaru dokumentów serwera WWW.**
    
Jeśli kopiujesz drzewo katalogów na swój serwer WWW, upewnij się, że kopiujesz również `.htaccess` - ponieważ pliki "z kropką" są często ukryte i normalnie nie są kopiowane.

Jeśli możesz to zrobić, zalecamy użycie git do sklonowania repozytorium źródłowego, zamiast używania spakowanego pliku tar lub zip. To znacznie ułatwia aktualizację oprogramowania. Polecenie Linuksa do sklonowania repozytorium do katalogu `mywebsite` jest następujące 

        git clone https://github.com/isfera/social.git mywebsite
        cd mywebsite

a następnie w dowolnym momencie możesz pobrać najnowsze zmiany za pomocą polecenia

        git pull

Utwórz foldery `cache/smarty3` i `store`, jeśli nie istnieją i upewnij się, że są możliwe do zapisu przez serwer internetowy.

		mkdir -p store
       mkdir -p cache/smarty3
       		
       chmod -R 775 store cache

Tu zakładamy, że katalog instalacyjny $Projectname (lub wirtualnego hosta) jest skonfigurowany tak, że prawa własności ma administrator serwera WWW i grupa do której należy zarówno administrator serwera jak i właściciel procesu serwera WWW. Właściciel procesu serwera WWW (np. www-data) nie musi a nawet nie powinien mieć uprawnień zapisu do pozostałej części instalacji (tylko odczyt).
 
**2. Zainstaluj dodatki.**

Następnie należy sklonować repozytorium dodatków (osobno). Dla przykładu nadamy temu repozytorium nazwę `zaddons` (w Twoim przypadku może to być inna nazwa, ale uwzglednij to w poleceniach).

		util/add_addon_repo https://github.com/isfera/social-addons.git zaddons

Aby aktualizować drzewo dodatków trzeba znajdować się w głównym katalogu instalacji $Projectname i wydać polecenie aktualizacji dla tego repozytorium.

       cd mywebsite
       util/update_addon_repo zaddons

**3. Utwórz pustą bazę danych ustaw uprawnienia dostępu.**

Utwórz pustą bazę danych i zanotuj szczegóły dostępu (nazwa hosta, nazwa użytkownika, hasło, nazwa bazy danych). Biblioteki bazy danych PDO powrócą do komunikacji przez gniazdo, jeśli nazwa hosta to "localhost", ale mogą wystąpić z tym problemy. Użyj tego, jeśli Twoje wymagania na to pozwalają. W przeciwnym razie, jeśli baza danych jest udostępniana na serwerze lokalnym, jako nazwy hosta wpisz "127.0.0.1". Korzystając z MySQL lub MariaDB, ustaw kodowanie znaków bazy danych na utf8mb4, aby uniknąć problemów z kodowaniem za pomocą emoji. Wszystkie tabele wewnętrzne są tworzone z kodowaniem utf8mb4_general_ci, więc jeśli ustawisz kodowania na utf8 a nie na utf8mb4, mogą wystąpić problemy.   

Wewnętrznie używamy teraz biblioteki PDO do połączeń z bazami danych. Jeśli napotkasz konfigurację bazy danych, której nie można wyrazić w formularzu konfiguracyjnym (na przykład przy użyciu MySQL z nietypową lokalizacją gniazda), możesz dostarczyć
ciąg połączenia PDO jako nazwę hosta bazy danych. Na przykład:
	
	mysql:unix_socket=/my/special/socket_path

W razie potrzeby nadal należy wypełnić wszystkie inne obowiązujące wartości formularza.  

**4. Utwórz pusty plik konfuguracyjnego .htconfig.php**

Jeśli serwer WWW nie będzie mógł dokonywać zapisów do plików innych niż te podane w pkt. 2 (a nie powinien), trzeba ręcznie utworzyć plik o nazwie .htconfig.php (zlokalizowany w katalogu głównym instalacji) i uczynić go zapisywalnym przez serwer internetowy.

**5. Odwiedź swoją witrynę internetową** 

Odwiedź swoją witrynę za pomocą przeglądarki internetowej i postępuj zgodnie z instrukcjami. Zanotuj wszelkie komunikaty o błędach i popraw je przed kontynuowaniem. Jeśli korzystasz z protokołu SSL z uznanym certyfikatem, użyj schematu https:// w adresie URL swojej witryny internetowej.

**6. Jeżeli automatyczna instalacja nie powiedzie się.**

Jeżeli automatyczna instalacja nie powiedzie się z jakichś powodów, sprawdź co następuje:

- Czy istnieje ".htconfig.php"? If not, edit htconfig.php and change system settings. Rename to .htconfig.php
- Czy wypełniona została baza danych? Jeśli nie, zaimportuj zawartość pliku "install/schema_xxxxx.sql" w phpmyadmin lub w wierszu poleceń mysql (zamieniając 'xxxxx' na właściwy typ bazy danych).

**7. Zarejestruj swoje osobiste konto.**

At this point visit your website again, and register your personal account. 
Registration errors should all be recoverable automatically. 
If you get any *critical* failure at this point, it generally indicates the
database was not installed correctly. You might wish to move/rename 
.htconfig.php to another name and empty (called 'dropping') the database 
tables, so that you can start fresh.

In order for your account to be given administrator access, it should be the
first account created, and the email address provided during registration
must match the "administrator email" address you provided during 
installation. Otherwise to give an account administrator access,
add 4096 to the account_roles for that account in the database. 

For your site security there is no way to provide administrator access
using web forms.

**8. Ustawienie zadań crona**

Set up a cron job or scheduled task to run the Cron manager once every 10-15 
minutes to perform background processing and maintenance. Example:

	cd /base/directory; /path/to/php Zotlabs/Daemon/Run.php Cron

Change "/base/directory", and "/path/to/php" as appropriate for your situation.

If you are using a Linux server, run "crontab -e" and add a line like the 
one shown, substituting for your unique paths and settings:

	*/10 * * * *	cd /home/myname/mywebsite; /usr/bin/php Zotlabs/Daemon/Run.php Cron > /dev/null 2>&1

You can generally find the location of PHP by executing "which php". If you 
have troubles with this section please contact your hosting provider for 
assistance. The software will not work correctly if you cannot perform this
step.

You should also be sure that `App::$config['system']['php_path']` is set
correctly in your .htconfig.php file, it should look like (changing it to the
correct PHP location):

	App::$config['system']['php_path'] = '/usr/local/php72/bin/php';
  

### Jeżeli rzeczy nie działają tak jak powinny


##### If you get the message "System is currently unavailable. Please try again later"

	
Check your database settings. It usually means your database could not be 
opened or accessed. If the database resides on the same machine, check that
the database server name is "127.0.0.1" or the word "localhost". 

##### 500 Internal Error

This could be the result of one of our Apache directives not being 
supported by your version of Apache. Examine your apache server logs.
Also check your file permissions. Your website and all contents must generally 
be world-readable.

It is likely that your web server reported the source of the problem in
its error log files. Please review these system error logs to determine what 
caused the problem. Often this will need to be resolved with your hosting
provider or (if self-hosted) your web server configuration. 

##### 400 and 4xx "File not found" errors

First check your file permissions. Your website and all contents must 
generally be world-readable.

Ensure that mod-rewite is installed and working, and that your
.htaccess file is being used. To verify the latter, create a file test.out
containing the word "test" in the top web directory, make it world 
readable and point your web browser to

http://yoursitenamehere.com/test.out

This file should be blocked. You should get a permission denied message.

If you see the word "test" your Apache configuration is not allowing your 
.htaccess file to be used (there are rules in this file to block access
to any file with .out at the end, as these are typically used for system logs).

Make certain the .htaccess file exists and is readable by everybody, then 
look for the existence of "AllowOverride None" in the Apache server 
configuration for your site. This will need to be changed to 
"AllowOverride All".  

	If you do not see the word "test", your .htaccess is working, but it is 
likely that mod-rewrite is not installed in your web server or is not working.

	On most flavours of Linux,

% a2enmod rewrite
% service apache2 restart

Consult your hosting provider, experts on your particular Linux 
distribution or (if Windows) the provider of your Apache server software if 
you need to change either of these and can not figure out how. There is 
a lot of help available on the web. Google "mod-rewrite" along with the 
name of your operating system distribution or Apache package.

  
##### If you see an error during database setup that DNS lookup failed

This is a known issue on some versions of FreeBSD, because 
dns_get_record() fails for some lookups. Create a file in your top webserver 
folder called '.htpreconfig.php' and inside it put the following:

<?php
App::$config['system']['do_not_check_dns'] = 1;

This should allow installation to proceed. Once the database has been
installed, add the same config statement (but not the '<?php' line) to the 
.htconfig.php file which was created during installation. 

##### If you are unable to write the file .htconfig.php during installation due to permissions issues:

create an empty file with that name and give it world-write permission.
For Linux:

% touch .htconfig.php
% chmod 777 .htconfig.php

Retry the installation. As soon as the database has been created, 

******* this is important *********

% chmod 755 .htconfig.php

##### Apache processes hanging, using as much CPU as they can

This seems to happen sometimes if you use mpm_prefork and the PHP process
started by Apache cannot get database access.

Consider the following settings:

In /etc/apache2/mods-enabled/mpm_prefork.conf (Debian, path and file name
may vary depending on your OS and distribution), set

	GracefulShutdownTimeout 300

This makes sure that Apache processes that are running wild will not do so
forever, but will be killed if they didn't stop five minutes after a
shutdown command was sent to the process.

If you expect high load on your server (public servers, e.g.), also make
sure that Apache will not spawn more processes than MySQL will accept
connections.

In the default Debian configuration, in
/etc/apache2/mods-enabled/mpm_prefork.conf the maximum number of workers
is set to 150:

	MaxRequestWorkers 150

However, in /etc/mysql/my.cnf the maximum number of connection is set to
100:

	max_connections = 100

150 workers are a lot and probably too much for small servers. However you
set those values, make sure that the number of Apache workers is smaller
than the number of connections MySQL accepts, leaving some room for other
stuff on your server that might access MySQL, and the communication poller
whichneeds MySQL access, too. A good setting for a medium-sized hub might be
to keep MySQL's max_connections at 100 and set mpm_prefork's MaxRequestWorkers
to 70.

Here you can read more about Apache performance tuning:
https://httpd.apache.org/docs/2.4/misc/perf-tuning.html

There are tons of scripts to help you with fine-tuning your Apache
installation. Just search with your favorite search engine 
'apache fine-tuning script'.
