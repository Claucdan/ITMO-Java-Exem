# Жизненный цикл запроса в рамках DispatcherServlet в Spring.

## Основные этапы обработки запроса
1. **Получение запроса (`HttpServletRequest`)**  
   - Запрос поступает в контейнер сервлетов (например, Tomcat) и передается в `DispatcherServlet`.
2. **Определение `HandlerMapping`**  
   - `DispatcherServlet` обращается к списку зарегистрированных **`HandlerMapping`** (например, `RequestMappingHandlerMapping`), чтобы определить, какой контроллер (или метод) должен обработать запрос.  
   - Сопоставление происходит на основе URL, HTTP-метода, заголовков и других параметров.
3. **Выполнение цепочки интерцепторов (`HandlerInterceptor`)**  
   - Если есть **preHandle**-интерцепторы, они выполняются до вызова метода контроллера.  
   - Если какой-то интерцептор вернет `false`, обработка прерывается.
4. **Вызов обработчика (`HandlerAdapter`)**  
   - `DispatcherServlet` использует **`HandlerAdapter`** (например, `RequestMappingHandlerAdapter`), чтобы выполнить метод контроллера.  
   - Параметры метода резолвятся с помощью **`ArgumentResolver`** (например, `@RequestParam`, `@RequestBody`).  
   - Метод контроллера возвращает результат (обычно имя view, DTO или `ResponseEntity`).
5. **Обработка результата (`HandlerReturnValueHandler`)**  
   - Результат передается в **`ViewResolver`** (если это имя view) или обрабатывается **`HttpMessageConverter`** (если это тело ответа, например, JSON).  
   - Если метод вернул `String`, `DispatcherServlet` ищет соответствующий шаблон (например, Thymeleaf, JSP).  
   - Если метод вернул `@ResponseBody`, результат сериализуется (например, в JSON).
6. **Пост-обработка интерцепторами (`postHandle`)**  
   - Если есть **postHandle**-интерцепторы, они выполняются после контроллера, но до рендеринга ответа.
7. **Рендеринг ответа (`View` или `HttpMessageConverter`)**  
   - Если используется **`ViewResolver`**, происходит рендеринг шаблона (например, JSP, Thymeleaf).  
   - Если используется **`HttpMessageConverter`**, тело ответа записывается в `HttpServletResponse`.
8. **Завершение обработки (`afterCompletion`)**  
   - Даже если возникло исключение, вызываются **afterCompletion**-интерцепторы (например, для очистки ресурсов).
9. **Отправка ответа клиенту**  
   - `DispatcherServlet` завершает обработку и отправляет ответ через `HttpServletResponse`.

## Обработка исключений
- Если на любом этапе возникает исключение, `DispatcherServlet` делегирует его обработку **`HandlerExceptionResolver`** (например, `@ExceptionHandler` или `DefaultHandlerExceptionResolver`).  
- Можно настроить глобальные обработчики через `@ControllerAdvice`.

## Схематично
```
1. Запрос → DispatcherServlet  
2. HandlerMapping → определяет Controller  
3. HandlerInterceptor.preHandle()  
4. HandlerAdapter → вызывает метод контроллера  
5. ReturnValueHandler → обрабатывает результат  
6. ViewResolver / HttpMessageConverter  
7. HandlerInterceptor.postHandle()  
8. Рендеринг ответа  
9. HandlerInterceptor.afterCompletion()  
10. Ответ клиенту
```

Этот цикл обеспечивает гибкость Spring MVC, позволяя настраивать обработку запросов на каждом этапе.
