import argparse
from pathlib import Path
import random
import time
import re

# --- Вспомогательные функции ---

def format_size(size_bytes):
    """Преобразует байты в удобочитаемый формат (КБ, МБ и т.д.)."""
    if size_bytes < 1024:
        return f"{size_bytes} B"
    size_kb = size_bytes / 1024
    if size_kb < 1024:
        return f"{size_kb:.2f} KB"
    size_mb = size_kb / 1024
    return f"{size_mb:.2f} MB"

def read_gitignore_patterns(base_path: Path):
    """
    Читает .gitignore в базовой директории и компилирует паттерны в регулярные выражения.
    """
    gitignore_path = base_path / ".gitignore"
    patterns = []
    
    if gitignore_path.exists():
        with open(gitignore_path, 'r') as f:
            for line in f:
                line = line.strip()
                # Игнорируем комментарии и пустые строки
                if not line or line.startswith('#'):
                    continue
                # Преобразуем паттерн gitignore в регулярное выражение
                # Например, *.pyc -> .*?\.pyc$
                pattern = re.escape(line).replace(r'\*', '.*?')
                # Добавляем якорь в конец, чтобы избежать совпадений внутри имен
                if not pattern.endswith('/'):
                     pattern += '$' 
                patterns.append(re.compile(pattern))
        print(f"✅ Загружено {len(patterns)} правил из .gitignore.")
    return patterns

def is_ignored(path: Path, patterns: list):
    """Проверяет, должен ли путь быть проигнорирован."""
    path_str = str(path)
    
    # Сначала проверяем саму папку
    if any(p.search(path_str) for p in patterns):
        return True
    
    # Проверяем файлы
    if path.is_file() and path.name.lower() in ['.gitignore', 'license', 'readme.md']:
        return True
        
    return False

def analyze_directory(base_path: Path, max_depth: int = -1, ignore_patterns: list = None):
    """
    Сканирует директорию с поддержкой максимальной глубины и игнорирования.
    """
    file_summary = {}
    total_files = 0
    total_size_bytes = 0
    
    if ignore_patterns is None:
        ignore_patterns = []

    # Используем Path.rglob() для рекурсивного поиска
    for path in base_path.rglob('*'):
        
        # Проверка глубины
        try:
            relative_depth = len(path.relative_to(base_path).parts)
            if max_depth != -1 and relative_depth > max_depth:
                continue
        except ValueError:
            continue # Пропускаем, если путь не является дочерним

        # Проверка игнорирования
        if is_ignored(path, ignore_patterns):
            continue
            
        if path.is_file():
            try:
                size = path.stat().st_size
            except OSError:
                continue

            total_files += 1
            total_size_bytes += size
            
            # Получаем расширение
            ext = path.suffix.lower() if path.suffix else "NO_EXT"
            
            if ext not in file_summary:
                file_summary[ext] = {'count': 0, 'total_size': 0}
                
            file_summary[ext]['count'] += 1
            file_summary[ext]['total_size'] += size
            
    return file_summary, total_files, total_size_bytes

# --- Главная часть программы ---

if __name__ == "__main__":
    
    parser = argparse.ArgumentParser(
        description="Анализатор файловой системы с поддержкой .gitignore и Pathlib."
    )
    # 
    parser.add_argument(
        'path', 
        type=str, 
        default='.', 
        nargs='?', 
        help='Путь к директории для анализа (по умолчанию - текущая).'
    )
    parser.add_argument(
        '-d', '--depth', 
        type=int, 
        default=-1, 
        help='Максимальная глубина сканирования (-1 для неограниченной).'
    )
    parser.add_argument(
        '-i', '--ignore-git', 
        action='store_true', 
        help='Использовать правила из .gitignore.'
    )
    
    args = parser.parse_args()
    
    base_path = Path(args.path)
    if not base_path.is_dir():
        print(f"❌ ОШИБКА: Путь '{args.path}' не является директорией.")
        exit(1)

    ignore_patterns = []
    if args.ignore_git:
        ignore_patterns = read_gitignore_patterns(base_path)

    # --- Запуск анализа ---
    
    print("\n--- 💎 ГИПЕР-АНАЛИЗ ФАЙЛОВОЙ СИСТЕМЫ ---")
    print(f"Сканирование пути: {base_path.resolve()}")
    print("-" * 70)
    
    summary, total_files, total_size = analyze_directory(base_path, args.depth, ignore_patterns)

    # --- Генерация отчета ---
    
    print(f"Анализ завершен. Всего файлов: {total_files}")
    print(f"Общий объем: {format_size(total_size)}")
    print("-" * 70)

    # Сортировка по общему размеру
    sorted_summary = sorted(summary.items(), key=lambda item: item[1]['total_size'], reverse=True)

    print(f"| {'Расширение':<12} | {'Файлов':<6} | {'Объем':<12} |")
    print("-" * 35)

    for ext, data in sorted_summary:
        size_formatted = format_size(data['total_size'])
        print(f"| {ext:<12} | {data['count']:<6} | {size_formatted:<12} |")

    print("-" * 35)
    print("Отчет завершен.")
