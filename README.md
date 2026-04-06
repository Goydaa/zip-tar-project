#🗜️ ZIPFILE и TARFILE
 
ЧТО ТАКОЕ ZIPFILE И TARFILE? 
 
zipfile и tarfile - это встроенные модули Python для работы с ZIP и TAR архивами. Представьте, что у вас есть 100 файлов, которые нужно отправить по почте. Вместо того чтобы прикреплять 100 файлов по отдельности, вы упаковываете их в один ZIP-архив. Эти модули позволяют делать это программно из Python-кода. 
 
### Почему именно zipfile и tarfile? 
 
- Встроены в Python - не нужно ничего устанавливать 
- Работают на Windows, Linux и macOS одинаково 
- Можно читать файлы из архива без распаковки 
- Поддерживают сжатие для экономии места 
 
--- 
 
ДЛЯ ЧЕГО ЭТОТ ПРОЕКТ? 
 
Этот проект создан, чтобы на практике изучить работу с архивами в Python. 
 
### Что мы делаем: 
 
1. Создаем ZIP-архивы из любой папки 
2. Создаем TAR.GZ архивы (сжатые) 
3. Смотрим содержимое архива без распаковки 
4. Распаковываем архивы 
 
--- 
 
КАК ЭТО РАБОТАЕТ? 
 
ZIP - самый популярный формат архивов. Поддерживается везде: Windows, Mac, Linux, телефоны. 
 
TAR.GZ - формат, популярный в Linux. Сначала файлы упаковываются в TAR (без сжатия), потом сжимаются GZIP. Обычно используется для дистрибутивов программ. 
 
--- 
 
ЧТО НУЖНО ДЛЯ РАБОТЫ? 
 
### Программы: 
 
- Python 3.8 или новее 
- Командная строка (cmd) 
 
### Знания: 
 
- Базовый Python (функции, циклы, условия) 
- Как работают файлы и папки 
 
--- 
 
ПОШАГОВАЯ ИНСТРУКЦИЯ ЗАПУСКА 
 
### Шаг 1: Создаем тестовую папку с файлами 
 
mkdir test_data 
echo Hello > test_data\file1.txt 
echo World > test_data\file2.txt 
 
### Шаг 2: Запускаем скрипт 
 
cd src 
python archive_manager.py --source ../test_data 
 
### Шаг 3: Смотрим результат 
 
Скрипт создаст: 
- archive.zip - ZIP-архив 
- archive.tar.gz - TAR.GZ архив 
- Выведет содержимое обоих архивов 
 
--- 
 
ЧТО МЫ ВИДИМ НА ЭКРАНЕ? 
 
После запуска скрипт показывает: 
 
- "ZIP created: archive.zip" - архив создан 
- "TAR.GZ created: archive.tar.gz" - архив создан 
- "Contents: archive.zip" - начинается вывод содержимого 
- "test_data/file1.txt: 8 bytes" - файл и его размер 
- "test_data/file2.txt: 8 bytes" - файл и его размер 
 
--- 
 
РАЗБОР КОДА 
 
Посмотрим на файл src/archive_manager.py и разберем каждую часть: 
 
### Импорт модулей 
 
import os           # для работы с файлами и папками 
import zipfile      # для ZIP-архивов 
import tarfile      # для TAR-архивов 
import argparse     # для аргументов командной строки 
 
### Создание ZIP-архива 
 
def create_zip(source_dir, output_path): 
    with zipfile.ZipFile(output_path, 'w', zipfile.ZIP_DEFLATED) as zipf: 
        for root, dirs, files in os.walk(source_dir): 
            for file_name in files: 
                file_path = os.path.join(root, file_name) 
                arcname = os.path.relpath(file_path, os.path.dirname(source_dir)) 
                zipf.write(file_path, arcname) 
 
**Объяснение:** 
 
- ZipFile(output_path, 'w', ZIP_DEFLATED) - открываем архив на запись со сжатием 
- os.walk(source_dir) - рекурсивно обходим все папки и файлы 
- zipf.write(file_path, arcname) - добавляем файл в архив 
- arcname - сохраняет структуру папок внутри архива 
 
### Создание TAR.GZ архива 
 
def create_targz(source_dir, output_path): 
    with tarfile.open(output_path, 'w:gz') as tarf: 
        tarf.add(source_dir, arcname=os.path.basename(source_dir)) 
 
**Объяснение:** 
 
- tarfile.open(output_path, 'w:gz') - открываем архив с сжатием GZIP 
- tarf.add(source_dir, arcname) - добавляем всю папку в архив 
- arcname - задает имя папки внутри архива 
 
### Просмотр содержимого архива 
 
def list_archive(archive_path): 
    if zipfile.is_zipfile(archive_path): 
        with zipfile.ZipFile(archive_path, 'r') as zipf: 
            for info in zipf.infolist(): 
                if not info.filename.endswith('/'): 
                    print(f"{info.filename}: {info.file_size} bytes") 
    elif tarfile.is_tarfile(archive_path): 
        with tarfile.open(archive_path, 'r:gz') as tarf: 
            for member in tarf.getmembers(): 
                if member.isfile(): 
                    print(f"{member.name}: {member.size} bytes") 
 
**Объяснение:** 
 
- is_zipfile() - проверяет, является ли файл ZIP-архивом 
- is_tarfile() - проверяет, является ли файл TAR-архивом 
- infolist() - возвращает список всех файлов в ZIP с метаданными 
- getmembers() - возвращает список всех файлов в TAR с метаданными 
 
### Распаковка архива 
 
def extract_archive(archive_path, extract_dir): 
    os.makedirs(extract_dir, exist_ok=True) 
    if zipfile.is_zipfile(archive_path): 
        with zipfile.ZipFile(archive_path, 'r') as zipf: 
            zipf.extractall(extract_dir) 
    elif tarfile.is_tarfile(archive_path): 
        with tarfile.open(archive_path, 'r:gz') as tarf: 
            tarf.extractall(extract_dir) 
 
**Объяснение:** 
 
- os.makedirs(extract_dir, exist_ok=True) - создает папку, если её нет 
- extractall(extract_dir) - распаковывает все файлы в указанную папку 
- Структура папок внутри архива сохраняется 
 
--- 
 
КАКИЕ БЫВАЮТ РЕЖИМЫ? 
 
 
--- 
 
ПРИМЕРЫ ИСПОЛЬЗОВАНИЯ 
 
### Пример 1: Посмотреть содержимое архива 
 
python archive_manager.py --list archive.zip 
 
### Пример 2: Распаковать архив 
 
python archive_manager.py --extract archive.zip 
 
### Пример 3: Создать архив другой папки 
 
python archive_manager.py --source "C:\Мои документы" --zip-output backup.zip 
 
 
--- 
 
ПРИМЕР ТЕСТА 
 
Запускаем с тестовыми файлами: 
 
Создаем папку test_data с двумя файлами: 
- file1.txt (6 байт: "Hello") 
- file2.txt (6 байт: "World") 
 
**Результат:** 
 
- archive.zip создан, размер около 200 байт 
- archive.tar.gz создан, размер около 200 байт 
- В обоих архивах лежат file1.txt и file2.txt 
- При распаковке структура папок сохраняется 
 
--- 
 
ЗАКЛЮЧЕНИЕ 
 
zipfile и tarfile - это мощные, но простые модули. Они встроены в Python, поэтому не требуют установки. С их помощью можно автоматизировать создание бэкапов, упаковку файлов для отправки и распаковку скачанных дистрибутивов. Главное преимущество - мы пишем код, который работает одинаково на всех операционных системах. 
