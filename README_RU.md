# РАЗБОР КОДА 
 
Посмотрим на файл src/archive_manager.py и разберем каждую часть: 
 
import os 
import zipfile 
import tarfile 
import argparse 
 
**import os** - модуль для работы с файловой системой (проверка существования папок, создание путей) 
 
**import zipfile** - модуль для создания и чтения ZIP-архивов 
 
**import tarfile** - модуль для создания и чтения TAR-архивов (с поддержкой GZIP сжатия) 
 
**import argparse** - модуль для обработки аргументов командной строки 
 
--- 
 
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
 
**def create_zip(source_dir, output_path):** - создаем функцию, которая принимает два параметра: исходную папку и имя выходного архива 
 
**if not os.path.exists(source_dir):** - проверяем, существует ли указанная папка 
 
**with zipfile.ZipFile(output_path, 'w', zipfile.ZIP_DEFLATED) as zipf:** - открываем архив на запись со сжатием 
 
**for root, dirs, files in os.walk(source_dir):** - рекурсивно обходим все папки и файлы 
 
**arcname = os.path.relpath(file_path, os.path.dirname(source_dir))** - вычисляем относительный путь для сохранения структуры 
 
**zipf.write(file_path, arcname)** - добавляем файл в архив 
 
--- 
 
def create_targz(source_dir, output_path): 
    if not os.path.exists(source_dir): 
        print(f"Folder not found: {source_dir}") 
        return 
    with tarfile.open(output_path, 'w:gz') as tarf: 
        tarf.add(source_dir, arcname=os.path.basename(source_dir)) 
    print(f"TAR.GZ created: {output_path}") 
 
**def create_targz(source_dir, output_path):** - создаем функцию для TAR.GZ архива 
 
**with tarfile.open(output_path, 'w:gz') as tarf:** - открываем TAR архив со сжатием GZIP 
 
**tarf.add(source_dir, arcname=os.path.basename(source_dir))** - добавляем всю папку в архив 
 
--- 
 
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
 
**def list_archive(archive_path):** - функция для просмотра содержимого архива 
 
**if zipfile.is_zipfile(archive_path):** - проверяем, является ли файл ZIP-архивом 
 
**zipf.infolist()** - возвращает список всех файлов в архиве с метаданными 
 
**elif tarfile.is_tarfile(archive_path):** - проверяем, является ли файл TAR-архивом 
 
**tarf.getmembers()** - возвращает список всех элементов в TAR архиве 
 
--- 
 
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
 
**def extract_archive(archive_path, extract_dir):** - функция для распаковки архива 
 
**os.makedirs(extract_dir, exist_ok=True)** - создаем папку для распаковки, если её нет 
 
**zipf.extractall(extract_dir)** - распаковывает все файлы из ZIP архива 
 
**tarf.extractall(extract_dir)** - распаковывает все файлы из TAR архива 
 
--- 
 
def main(): 
    parser = argparse.ArgumentParser(description='Archive Manager') 
    parser.add_argument('--source', help='Source directory') 
    parser.add_argument('--extract', help='Extract archive') 
    parser.add_argument('--list', help='List archive contents') 
    parser.add_argument('--zip-output', default='archive.zip') 
    parser.add_argument('--tar-output', default='archive.tar.gz') 
    args = parser.parse_args() 
 
**def main():** - главная функция, которая запускается при выполнении скрипта 
 
**parser = argparse.ArgumentParser()** - создаем парсер аргументов командной строки 
 
**parser.add_argument('--source')** - аргумент для указания исходной папки 
 
**parser.add_argument('--extract')** - аргумент для распаковки архива 
 
**parser.add_argument('--list')** - аргумент для просмотра содержимого 
 
--- 
 
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
 
**try:** - блок обработки ошибок 
 
**if args.list:** - если пользователь ввел --list, показываем содержимое 
 
**elif args.extract:** - если ввел --extract, распаковываем 
 
**elif args.source:** - если ввел --source, создаем архивы 
 
--- 
 
if __name__ == "__main__": 
    main() 
 
**if __name__ == "__main__":** - проверяем, что скрипт запущен напрямую 
 
**main()** - вызываем главную функцию 
