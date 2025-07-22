### Инициализация
Для инициализации выполните команду `alembic init --t async migrations` в корне проекта: это создаст необходимую структуру для работы с миграциями. Последний параметр команд указывает название директории, в которой будут содержаться создаваемые миграции. Флаг `--t async` (или `--template async`) указывает, что необходимо использовать асинхронный шаблон.

```python

connectable = AsyncEngine(
    engine_from_config(
        config.get_section(config.config_ini_section),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
        future=True,
    )
)
```

Доработайте текущие настройки так, чтобы информация о нужной БД стягивалась из переменных окружения из файла `.env`. Измените файл `migrations/env.py` на следующий:
```python
from alembic import context from dotenv import load_dotenv

load_dotenv('.env') 
config = context.config 
import os 
config.set_main_option('sqlalchemy.url', os.environ['DATABASE_DSN'])
```

Сохраните изменения и проверьте, что Alembic будет подключаться именно к той БД, которая указана в `.env`. Выполните команду `alembic current`. Вы увидите похожий вывод:
```bash
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
```
### Подключение
На основе ORM подключения

### Создание миграции
Alembic предоставляет возможность создать сгенерированный файл миграции в автоматическом режиме. Для этого выполните команду `alembic revision --autogenerate -m 01_initial-db`. Флаг `--autogenerate` указывает на то, что текущая структура таблиц БД будет сравниваться с описанными моделями приложения, а на основании отличий сгенерируется скрипт изменений.

Чтобы точно унаследовать все объекты до передачи информации в Alembic, соберите все модели в одном месте. В файле `__init__.py` модуля `models` добавьте список доступных моделей:
```python
__all__ = [
    "Base",
    "Entity", 
]

from .base import Base
from .entity import Entity
```
Далее добавьте пару изменений в файл `migrations/env`:
```python

from src.models import Base

target_metadata = Base.metadata

```

### Применение миграций
Для выполнения миграций понадобится команда `alembic upgrade head`. Она выполнит и применит все неприменённые ранее миграции. Список всех ревизий можно посмотреть по команде `alembic history`.

Выполните команду `alembic upgrade head`. В консоли появится похожий вывод:
```bash
2022-09-30 11:23:23,077 - alembic.runtime.migration - INFO - Context impl PostgresqlImpl.
2022-09-30 11:23:23,077 - alembic.runtime.migration - INFO - Will assume transactional DDL.
2022-09-30 11:23:23,080 - alembic.runtime.migration - INFO - Running upgrade  -> 88dfa4ef9883, 01_initial-db
```

### Кастомная реализция миграций

```python
from alembic.runtime.migration import MigrationContext
from alembic.script import ScriptDirectory
from alembic. config import Config 
from alembic.runtime. environment import EnvironmentContext

async def run_alembic_upgrade() -> None:
	alembic_cfg = Config()
	alembic_cfg.set_main_option("script_location", "/migrations")
	script_directory = ScriptDirectory.from_config(alembic_cfg)
	async with engine.begin() as conn:
		def run_migrations(conn):
			context = MigrationContext. configure (conn)
			
			def process_revision_context (rev, context):
				return script directory upgrade_revs ("head", rev)
	
			with EnvironmentContext(
				config=alembic_cfg,
				script=script_directory,
				fn=process_revision_context,
			) as env_ctx:
				env_ctx.configure (
					connection=context.connection, include_schemas=True
				)
				env_ctx.run_migrations()
		await conn. run_sync(run_migrations)
	await engine.dispose()
```
