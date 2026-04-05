# РАЗБОР КОДА 
 
Посмотрим на файл `src/archive_manager.py` и разберем каждую строчку: 
 
```python 
import os 
import zipfile 
import tarfile 
import argparse 
``` 
 
**import os** - модуль для работы с файловой системой 
 
**import zipfile** - модуль для создания и чтения ZIP-архивов 
 
**import tarfile** - модуль для создания и чтения TAR-архивов 
 
**import argparse** - модуль для обработки аргументов командной строки 
 
--- 
 
```python 
def create_zip(source_dir, output_path): 
    if not os.path.exists(source_dir): 
        print(f"Folder not found: {source_dir}") 
        return 
    with zipfile.ZipFile(output_path, 'w', zipfile.ZIP_DEFLATED) as zipf: 
        for root, dirs, files in os.walk(source_dir): 
            for file_name in files: 
                file_path = os.path.join(root, file_name) 
                arcname = os.path.relpath(file_path, os.path.dirname(source_dir)) 
                zipf.write(file_path, arcname) 
    print(f"ZIP created: {output_path}") 
``` 
 
**def create_zip(source_dir, output_path):** - создаем функцию, которая принимает два параметра: исходную папку и имя выходного архива 
 
**if not os.path.exists(source_dir):** - проверяем, существует ли указанная папка. Если нет - выводим ошибку и выходим 
 
**with zipfile.ZipFile(output_path, 'w', zipfile.ZIP_DEFLATED) as zipf:** - открываем архив на запись ('w') со сжатием (ZIP_DEFLATED). Контекстный менеджер with автоматически закроет архив после завершения блока 
 
**for root, dirs, files in os.walk(source_dir):** - рекурсивно обходим все папки и файлы внутри source_dir. root - текущая папка, dirs - список подпапок, files - список файлов 
 
**arcname = os.path.relpath(file_path, os.path.dirname(source_dir))** - вычисляем относительный путь, чтобы сохранить структуру папок внутри архива 
 
**zipf.write(file_path, arcname)** - добавляем файл в архив с указанным именем 
 
--- 
 
```python 
def create_targz(source_dir, output_path): 
    if not os.path.exists(source_dir): 
        print(f"Folder not found: {source_dir}") 
        return 
    with tarfile.open(output_path, 'w:gz') as tarf: 
        tarf.add(source_dir, arcname=os.path.basename(source_dir)) 
    print(f"TAR.GZ created: {output_path}") 
``` 
 
**def create_targz(source_dir, output_path):** - создаем функцию для TAR.GZ архива 
 
**with tarfile.open(output_path, 'w:gz') as tarf:** - открываем TAR архив со сжатием GZIP ('w:gz') 
 
**tarf.add(source_dir, arcname=os.path.basename(source_dir))** - добавляем всю папку в архив. arcname задает имя корневой папки внутри архива 
 
--- 
 
```python 
def list_archive(archive_path): 
    if not os.path.exists(archive_path): 
        print(f"Archive not found: {archive_path}") 
        return 
    print(f"\nContents: {archive_path}") 
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
    else: 
        print("Unknown archive format") 
``` 
 
**def list_archive(archive_path):** - функция для просмотра содержимого архива 
 
**if zipfile.is_zipfile(archive_path):** - проверяем, является ли файл ZIP-архивом 
 
**with zipfile.ZipFile(archive_path, 'r') as zipf:** - открываем ZIP архив для чтения ('r') 
 
**for info in zipf.infolist():** - infolist() возвращает список всех файлов в архиве с их метаданными (размер, дата, права доступа) 
 
**elif tarfile.is_tarfile(archive_path):** - если не ZIP, проверяем, является ли TAR архивом 
 
**for member in tarf.getmembers():** - getmembers() возвращает список всех элементов в TAR архиве 
 
**if member.isfile():** - проверяем, что это файл, а не папка 
 
--- 
 
```python 
def extract_archive(archive_path, extract_dir): 
    if not os.path.exists(archive_path): 
        print(f"Archive not found: {archive_path}") 
        return 
    os.makedirs(extract_dir, exist_ok=True) 
    if zipfile.is_zipfile(archive_path): 
        with zipfile.ZipFile(archive_path, 'r') as zipf: 
            zipf.extractall(extract_dir) 
    elif tarfile.is_tarfile(archive_path): 
        with tarfile.open(archive_path, 'r:gz') as tarf: 
            tarf.extractall(extract_dir) 
    print(f"Archive extracted to: {extract_dir}") 
``` 
 
**def extract_archive(archive_path, extract_dir):** - функция для распаковки архива 
 
**os.makedirs(extract_dir, exist_ok=True)** - создаем папку для распаковки, если её нет. exist_ok=True - не выдавать ошибку, если папка уже существует 
 
**zipf.extractall(extract_dir)** - распаковывает все файлы из ZIP архива в указанную папку, сохраняя структуру 
 
**tarf.extractall(extract_dir)** - распаковывает все файлы из TAR архива 
 
--- 
 
```python 
def main(): 
    parser = argparse.ArgumentParser(description='Archive Manager') 
    parser.add_argument('--source', help='Source directory') 
    parser.add_argument('--extract', help='Extract archive') 
    parser.add_argument('--list', help='List archive contents') 
    parser.add_argument('--zip-output', default='archive.zip') 
    parser.add_argument('--tar-output', default='archive.tar.gz') 
    args = parser.parse_args() 
``` 
 
**def main():** - главная функция, которая запускается при выполнении скрипта 
 
**parser = argparse.ArgumentParser(description='Archive Manager')** - создаем парсер аргументов командной строки 
 
**parser.add_argument('--source', help='Source directory')** - добавляем аргумент --source, который принимает путь к исходной папке 
 
**parser.add_argument('--extract', help='Extract archive')** - аргумент --extract для распаковки архива 
 
**parser.add_argument('--list', help='List archive contents')** - аргумент --list для просмотра содержимого 
 
**parser.add_argument('--zip-output', default='archive.zip')** - аргумент для имени ZIP архива (по умолчанию archive.zip) 
 
**parser.add_argument('--tar-output', default='archive.tar.gz')** - аргумент для имени TAR.GZ архива 
 
**args = parser.parse_args()** - разбираем аргументы, которые ввел пользователь 
 
--- 
 
```python 
    try: 
        if args.list: 
            list_archive(args.list) 
        elif args.extract: 
            extract_archive(args.extract, args.extract + '_extracted') 
        elif args.source: 
            create_zip(args.source, args.zip_output) 
            create_targz(args.source, args.tar_output) 
            list_archive(args.zip_output) 
            list_archive(args.tar_output) 
        else: 
            print("Usage: python archive_manager.py --source ./test_data") 
    except Exception as e: 
        print(f"Error: {e}") 
``` 
 
**try:** - начинаем блок обработки ошибок 
 
**if args.list:** - если пользователь ввел --list, показываем содержимое архива 
 
**elif args.extract:** - если ввел --extract, распаковываем архив 
 
**elif args.source:** - если ввел --source, создаем оба архива и показываем их содержимое 
 
**create_zip(args.source, args.zip_output)** - вызываем функцию создания ZIP 
 
**create_targz(args.source, args.tar_output)** - вызываем функцию создания TAR.GZ 
 
**else:** - если не ввел никаких аргументов, показываем подсказку 
 
**except Exception as e:** - если произошла любая ошибка, выводим её 
 
--- 
 
```python 
if __name__ == "__main__": 
    main() 
``` 
 
**if __name__ == "__main__":** - проверяем, что скрипт запущен напрямую, а не импортирован как модуль 
 
**main()** - вызываем главную функцию 
