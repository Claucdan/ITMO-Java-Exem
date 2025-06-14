# Инструменты и типовые решения для аутентификации и авторизации запросов.
## Spring Security
### Что это?
Spring Security это Java/JavaEE framework, предоставляющий механизмы построения систем аутентификации и авторизации, а также другие возможности обеспечения безопасности для корпоративных приложений, созданных с помощью Spring Framework.
### Как работает?
Данный проект в своей работе использует паттерн «Цепочки обязанностей». При выполнение http запроса происходит поэтапная обработка фильтрами, которые производят валидацию запроса.
![Pasted image 20250610155356.png](../photos/Pasted%20image%2020250610155356.png)
![Pasted image 20250610155430.png](../photos/Pasted%20image%2020250610155430.png)
### Фильтры
#### WebAsyncManagerIntegrationFilter 
Согласно документации он «интегрирует» SecurityContext с WebAsyncManager который ответственен за асинхронные запросы.

#### SecurityContextPersistenceFilter 
Ищет SecurityContext в сессии и заполняет SecurityContextHolder если находит. По умолчанию используется ThreadLocalSecurityContextHolderStrategy которая хранит SecurityContext в ThreadLocal переменной.

#### HeaderWriterFilter
Просто добавляет заголовки в response.

#### LogoutFilter
Проверяет совпадает ли url c паттерном ([pattern='/logout', POST] - по умолчанию) и запускает процедуру логаута.
1. Удаляется Csrf токен
2. Завершается сессия
3. Чистится SecurityContextHolder

#### BasicAuthenticationFilter
Фильтр проверяет, есть ли заголовок Authorization со значением начинающийся на Basic  
Если находит, извлекает логин\пароль и передает их в AuthenticationManager
#### RequestCacheAwareFilter
Оборачивает существущий запрос в SecurityContextHolderAwareRequestWrapper
1. Пользователь заходит на защищённый url.  
2. Его перекидывает на страницу логина.  
3. После успешной авторизации пользователя перекидывает на страницу которую он запрашивал в начале.  
Именно для восстановления оригинального запроса существует этот фильтр.  
Внутри проверяется есть ли сохраненный запрос, если есть им подменяется текущий запрос.  
Запрос сохраняется в сессии. BasicAuthenticationFilter - фильтр проверяет, есть ли заголовок Authorization со значением начинающийся на Basic  
Если находит, извлекает логин\пароль и передает их в AuthenticationManager.
#### AnonymousAuthenticationFilter
Если к моменту выполнения этого фильтра SecurityContextHolder пуст, т.е. не произошло аутентификации фильтр заполняет объект SecurityContextHolder анонимной аутентификацией — AnonymousAuthenticationToken с ролью «ROLE_ANONYMOUS».  

Это гарантирует что в SecurityContextHolder будет объект, это позволяет не бояться NullPointerException, а также более гибко подходить к настройке доступа для неавторизованных пользователей.

#### SessionManagementFilter
На этом этапе производятся действия связанные с сессией. Это может быть:
- смена идентификатора сессии
- ограничение количества одновременных сессий
- сохранение SecurityContext в securityContextRepository  

В обычном случае происходит следующе:  
SecurityContextRepository с дефолтной реализацией HttpSessionSecurityContextRepository сохраняет SecurityContext в сессию.  
Вызывается sessionAuthenticationStrategy.onAuthentication.

Происходят 2 вещи:  
1. По умолчанию включенна защита от session fixation attack, т.е. после аутенцификации меняется id сессии.  
2. Если был передан csrf токен, генерируется новый csrf токен
#### ExceptionTranslationFilter
К этому моменту SecurityContext должен содержать анонимную, либо нормальную аутентификацию. ExceptionTranslationFilter прокидывает запрос и ответ по filter chain и обрабатывает возможные ошибки авторизации.

#### FilterSecurityInterceptor
На последнем этапе происходит авторизация на основе url запроса.FilterSecurityInterceptor наследуется от AbstractSecurityInterceptor и решает, имеет ли текущий пользователь доступ до текущего url. Существует другая реализация MethodSecurityInterceptor который отвественнен за допуск до вызова метода, при использовании аннотаций @Secured\@PreAuthorize. Внутри вызывается AccessDecisionManager. Есть несколько стратегий принятия решения о том давать ли допуск или нет, по умолчанию используется: AffirmativeBased
### AuthenticationManager
AuthenticationManager представляет из себя интерфейс, который принимает Authentication и возвращает тоже Authentication.  
### AuthenticationProvider
Когда мы передаем объект Authentication в ProviderManager, он перебирает существующие AuthenticationProvider-ры и проверяет поддерживает ли AuthenticationProvider эту имплементацию Authentication.

В результате внутри AuthenticationProvider.authenticate мы уже можем привести переданный Authentication в нужную реализацию без исключений приведения.   
  
Далее из конкретной реализации вытаскиваем credentionals.  
  
Если аутентификация не удалась AuthenticationProvider должен бросить исключение, ProviderManager поймает его и попробует следующий AuthenticationProvider из списка, если ни один AuthenticationProvider не вернет успешную аутентификацию, то ProviderManager пробросит последний пойманное исключение.