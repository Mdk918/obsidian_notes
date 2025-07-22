### Конфиги и флаги

 - Безголовый режим(работа на сервере) - по умолчанию включен. 
 ```python
	 browser = p.chromium.launch(headless=False)
 ```
- Загрузки(включение/выключение доступности загрузок) - по умолчанию выключен.
```python
	context = browser.new_context(accept_downloads=True)
```
- Игнорирование ошибок https - продолжение работы и пропуск ошибок https. По умолчанию выключен.
```python
	context = browser.new_context(ignore_https_errors=Fasle)
```
- Трейсинг работы запущенного браузера:
```python
	context.tracing.start(screenshots=True, snapshots=True) # Запуск
	context.tracing.stop(path="trace.zip") # Остановка
```

```bash
playwright show-trace trace.zip # Просмотр трейса через интерфейс
```
### Ошибки и проблемы
 - Пустой путь к сертификатам / ошибки сертификатов TLS - не может найти путь к корневым удостоверяющим сертификатам, необходимо указать вручную через переменную **SSL_CERT_FILE**. Узнать местоположение сертификатам можно через certifi
```python
	cert_path = certifi.where()

	os.environ["SSL_CERT_FILE"] = путь к сертификату
```
