import os
import random
import time

def generate_mock_files(base_path="project_data", num_files=20):
    """
    Имитирует создание случайной структуры директории 
    и нескольких файлов с разными расширениями.
    """
    
    # Расширения, которые мы будем имитировать
    extensions = ['.py', '.js', '.html', '.css', '.txt', '.jpg', '.png', '.log', '']
    
    # 1. Создание базовой директории
    if not os.path.exists(base_path):
        os.makedirs(base_path)
    
    # 2. Создание вложенной директории
    sub_path = os.path.join(base_path, "sub_folder")
    if not os.path.exists(sub_path):
        os.makedirs(sub_path)

    print(f"✅ Создана имитация файловой системы в: {base_path}")
    
    # 3. Создание случайных файлов
    for i in range(num_files):
        # Случайное имя и расширение
        ext = random.choice(extensions)
        filename = f"file_{i+1}_{random.randint(100, 999)}{ext}"
        
        # Случайное размещение: в основной или вложенной папке
        target_dir = base_path if random.random() > 0.3 else sub_path
        file_path = os.path.join(target_dir, filename)
        
        # Записываем что-то в файл, чтобы он имел размер
        with open(file_path, 'w') as f:
            f.write(f"This is mock content for {filename}.")
            
        # Случайный размер (имитация)
        os.truncate(file_path, random.randint(100, 5000))
        
    return base_path

def analyze_directory(base_path):
    """
    Сканирует директорию (включая поддиректории) и группирует файлы по расширению.
    """
    file_summary = {}
    total_files = 0
    total_size_bytes = 0
    
    # os.walk рекурсивно обходит все поддиректории
    # 
    for root, dirs, files in os.walk(base_path):
        for file in files:
            file_path = os.path.join(root, file)
            
            # Получаем размер файла
            try:
                size = os.path.getsize(file_path)
            except OSError:
                continue # Пропускаем, если не можем получить размер
            
            # Получаем расширение (например, .txt или пустая строка для файлов без расширения)
            _, ext = os.path.splitext(file)
            ext = ext.lower() if ext else "NO_EXT" # Приводим к нижнему регистру
            
            # Обновляем сводку
            if ext not in file_summary:
                file_summary[ext] = {'count': 0, 'total_size': 0, 'files': []}
                
            file_summary[ext]['count'] += 1
            file_summary[ext]['total_size'] += size
            file_summary[ext]['files'].append(file_path)
            
            total_files += 1
            total_size_bytes += size
            
    return file_summary, total_files, total_size_bytes

def format_size(size_bytes):
    """Преобразует байты в удобочитаемый формат (КБ, МБ и т.д.)."""
    if size_bytes < 1024:
        return f"{size_bytes} B"
    size_kb = size_bytes / 1024
    if size_kb < 1024:
        return f"{size_kb:.2f} KB"
    size_mb = size_kb / 1024
    return f"{size_mb:.2f} MB"

# --- Главная часть программы ---

MOCK_DIR = "random_project_analyzer"

# 1. Генерация имитации файловой системы
base_dir = generate_mock_files(MOCK_DIR, 30)

# 2. Анализ директории
summary, total_files, total_size = analyze_directory(base_dir)

# 3. Генерация отчета
print("\n--- 📝 ОТЧЕТ ПО АНАЛИЗУ ДИРЕКТОРИИ ---")
print(f"Сканирование завершено. Всего файлов: {total_files}")
print(f"Общий объем: {format_size(total_size)}")
print("-" * 50)

# Сортируем по количеству файлов
sorted_summary = sorted(summary.items(), key=lambda item: item[1]['count'], reverse=True)

for ext, data in sorted_summary:
    size_formatted = format_size(data['total_size'])
    print(f"| {ext:<8} | Файлов: {data['count']:<4} | Объем: {size_formatted:<10} |")

print("-" * 50)

# Опционально: вывод файлов с ошибками или необычных файлов
if 'NO_EXT' in summary:
     print(f"⚠️ Найдено {summary['NO_EXT']['count']} файлов без расширения.")
     
# Очистка имитационной папки (для предотвращения захламления)
# os.system(f"rm -rf {MOCK_DIR}") # Используйте, если работаете в UNIX/Linux

print("Для очистки имитационной папки удалите директорию 'random_project_analyzer'.")
